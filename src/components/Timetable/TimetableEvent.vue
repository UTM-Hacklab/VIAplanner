<template>
    <div>
        <div v-if="event.start > 0">
            <v-dialog v-model="dialog" scrollable width="825px" @input="atInput">
                <template v-slot:activator="{ on }">
                    <div
                        @mouseover="hovered = true"
                        @mouseleave="hovered = false"
                        v-on="on"
                        class="event"
                        :style="{
                            background: getCourseColor(event.code),
                            height: getHeight,
                        }"
                    >
                        <h4>{{ event.code }}</h4>

                        <div class="lock-button">
                            <v-btn
                                small
                                dark
                                @click.stop="lockToggle"
                                v-if="locked"
                                icon
                            >
                                <v-icon>mdi-lock</v-icon>
                            </v-btn>
                            <v-btn
                                small
                                dark
                                @click.stop="lockToggle"
                                v-if="!locked && hovered"
                                icon
                            >
                                <v-icon>mdi-lock-open</v-icon>
                            </v-btn>
                        </div>

                        <v-row class="px-3">
                            <div>{{ event.sectionCode }} {{ deliveryMethod }}</div>
                            <v-spacer />
                            <div v-if="locations.length === 2 && duration === 1">
                                {{ locations[0] }}
                            </div>
                        </v-row>

                        <v-row class="px-3">
                            <div>{{ getFormattedTime(event.start, event.end) }}</div>
                            <v-spacer />
                            <div v-if="locations.length === 1 || duration != 1">
                                {{ locations[0] }}
                            </div>
                            <div
                                v-else-if="locations.length === 2 && duration === 1"
                            >
                                {{ locations[1] }}
                            </div>
                        </v-row>
                        <v-row
                            class="px-3"
                            v-if="locations.length === 2 && duration != 1"
                        >
                            <v-spacer />
                            <div>{{ locations[1] }}</div>
                        </v-row>
                    </div>
                </template>
                <course-section-picker
                    v-on:done="dialog = false"
                    :code="event.code"
                    ref="popUp"
                />,
            </v-dialog>
        </div>
        <div
            v-else-if="checkHalfHour(event.currStart, event.currEnd)"
            class="event empty-event"
            :style="{ height: getHeight }"
        />
        <div
            v-else
            v-ripple
            class="event empty-event"
            :style="{ height: getHeight, background: dynamicColor }"
            @mouseover="hovered = true"
            @mouseleave="hovered = false"
            @click.stop="lockedSectionToggle"
        >
            <div v-if="hovered">
                <v-row>
                    <v-col>
                        <p class="center unselectable">{{ dynamicText }}</p>
                    </v-col>
                </v-row>
            </div>
        </div>
    </div>
</template>

<script>
import CourseSectionPicker from "../Popup/CourseSectionPicker";
import { mapGetters, mapActions, mapMutations } from "vuex";
const convertSecondsToHours = (seconds) => {
    return seconds / 3600;
};
export default {
    name: "timetable-event",
    props: {
        event: {
            type: Object,
            default: () => {},
        },
        currDay: {
            type: String,
            default: "",
        },
        semester: String,
    },
    components: {
        CourseSectionPicker,
    },
    data() {
        return {
            hovered: false,
            dialog: false,
            height: window.innerHeight,
        };
    },
    created() {
        window.addEventListener("resize", this.handleResize);
    },
    computed: {
        ...mapGetters(["getCourseColor", "fallLockedSections", "winterLockedSections"]),
        //Duration of the event in hours
        duration() {
            //Real course
            if (this.event.start > 0) {
                return convertSecondsToHours(this.event.end - this.event.start);
            }
            //Empty or blocked hour
            else {
                return convertSecondsToHours(
                    this.event.currEnd - this.event.currStart
                );
            }
        },
        getHeight() {
            return `${this.duration * this.oneHourHeight}px`;
        },
        oneHourHeight() {
            // the height of a timetable event will be at least 65 px
            return (this.height - 168) / 9 > 65 ? (this.height - 168) / 9 : 65;
        },
        deliveryMethod() {
            if (this.event.method === "INPER") {
                return "(In Person)";
            } else if (this.event.method === "ROTATE") {
                return "(Rotate)";
            } else {
                return "(Sync)";
            }
        },
        locations() {
            return this.event.location.split("; ");
        },
        dynamicText() {
            return !this.locked ? "Block This Time" : "Unblock This Time";
        },
        // change the color in the event so it correct based on hovering or locked
        dynamicColor() {
            if (this.locked) {
                return "#d9d9d9";
            } else {
                return this.hovered ? "#d9d9d9" : "white";
            }
        },
        // stores the info of the current section
        currSecData() {
            return {
                name: `Locked Section`,
                courseCode: `Lock${this.semester}${this.currDay}${this.event.currStart}`,
                meeting_sections: [
                    {
                        sectionCode: "L0001",
                        instructors: ["NA"],
                        times: [
                            {
                                day: this.currDay,
                                start: this.event.currStart,
                                end: this.event.currStart + 3600,
                                location: "NA",
                            },
                        ],
                    },
                ],
            };
        },
        // lock the status of the current section
        locked() {
            let lockedSections = this.semester === "F" ? this.fallLockedSections : this.winterLockedSections
            for (var section of lockedSections) {
                if (
                    section === `${this.event.code}${this.event.sectionCode}` ||
                    section ===
                        `${this.currSecData.courseCode}${this.currSecData.meeting_sections[0].sectionCode}`
                ) {
                    return true;
                }
            }
            return false;
        },
    },
    methods: {
        ...mapActions(["deleteCourse"]),
        ...mapMutations(["lockSection", "unlockSection", "addCourse"]),
        handleResize() {
            this.height = window.innerHeight;
        },
        atInput() {
            var courseSectionPicker = this.$refs.popUp;
            if (typeof courseSectionPicker != "undefined") {
                courseSectionPicker.resetSelectedMeetingSections();
            }
        },
        checkHalfHour(currStart, end) {
            // if one hour
            if (Number.isInteger((end - currStart) / 3600)) {
                return false;
            }
            // half hour
            return true;
        },
        //Toggle locked status of this TimetableEvent when it is not empty (lock/unlock this section)
        lockToggle() {
            // modifies vuex based on the current section's lock status
            !this.locked
                ? this.lockSection(`${this.event.code}${this.event.sectionCode}`)
                : this.unlockSection(`${this.event.code}${this.event.sectionCode}`);
        },
        getFormattedTime(start, end) {
            var s = (start / 3600) % 12;
            if (s == 0) {
                s = 12;
            }
            var e = (end / 3600) % 12;
            if (e == 0) {
                e = 12;
            }
            let startHalf = Number.isInteger(s) ? "00" : "30";
            let endHalf = Number.isInteger(e) ? "00" : "30";
            return `${s - startHalf / 6 / 10}:${startHalf} - ${e -
                endHalf / 6 / 10}:${endHalf}`;
        },
        //Toggle locked status of this TimetableEvent when it is empty (block/unblock this hour)
        lockedSectionToggle() {
            if (!this.locked) {
                // if the user clicks on an empty timeslot, it will be added as a course in vuex
                this.addCourse({ course: this.currSecData });
                this.lockSection(
                    `${this.currSecData.courseCode}${this.currSecData.meeting_sections[0].sectionCode}`
                );
            } else {
                // if the user clicks on a lock timeslot, it will be removed
                this.deleteCourse({ code: this.currSecData.courseCode });
            }
        },
    },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css?family=Montserrat&display=swap");
* {
    font-family: "Montserrat", sans-serif;
}
.unselectable {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
.center {
    color: black;
    text-align: center;
    padding-top: 8px !important;
}
.event {
    border: 1px solid gray;
    color: white;
    padding: 8px;
    padding-top: 3px;
    position: relative;
    cursor: pointer;
    font-size: small;
}
.empty-event {
    background: white;
    border: 0.2px solid gray;
    cursor: pointer;
}
.course-code {
    margin-left: 3px;
}
.align-left {
    position: absolute;
    left: 0px;
}
.align-right {
    position: absolute;
    right: 0px;
}
.lock-button {
    position: absolute;
    right: 3px;
    top: 3px;
    z-index: 1;
}
</style>
