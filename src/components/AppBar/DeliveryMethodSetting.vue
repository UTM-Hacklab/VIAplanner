<template>
    <v-menu
      v-model="menu"
      :close-on-content-click="false"
      offset-y
    >
    <template v-slot:activator="{ on, attrs }">
        <v-btn
            dark
            v-bind="attrs"
            v-on="on"
            icon
            class="mr-1"
        >
            <v-icon>mdi-cog</v-icon>
        </v-btn>
    </template>
    <v-card style="background-color: #f0f0f0" min-width="260px">
        <v-card-subtitle style="text-align: center; font-size: 18px">
            Delivery Methods
        </v-card-subtitle>
        <v-divider/>
            <v-container>
                <v-checkbox
                    v-model="deliveryMethod"
                    value="Online"
                    label="prefer online"
                    append-icon="mdi-laptop"
                    @change="setDeliveryMethod('Online')"
                ></v-checkbox>
                <v-checkbox
                    v-model="deliveryMethod"
                    value="InPerson"
                    label="prefer in-person"
                    append-icon="mdi-teach"
                    @change="setDeliveryMethod('InPerson')"
                ></v-checkbox>
                <v-checkbox
                    v-model="deliveryMethod"
                    value="Mixed"
                    label="mixed"
                    append-icon="mdi-account-convert"
                    @change="setDeliveryMethod('Mixed')"
                ></v-checkbox>
            </v-container>
        <v-card-actions style="justify-content: center">
            <v-btn 
            text 
            dark 
            style="background-color: #2196F3"
            @click="regen">
                Re-generate Timetables
            </v-btn>
        </v-card-actions>
    </v-card>
    </v-menu>
</template>
<script>
import { mapMutations, mapActions } from "vuex";
export default {
    data() {
        return {
            deliveryMethod: "Mixed",
            menu: false
        }
    },
    computed: {

    },
    methods: {
        ...mapMutations(["setPreferredDeliveryMethod"]),
        ...mapActions(["resetTimetable"]),
        setDeliveryMethod (method) {
            this.setPreferredDeliveryMethod(method)
        },
        regen() {
            this.menu = false
            this.resetTimetable(true)
        }
    }
}
</script>
<style scoped>
    .container {
        padding-left: 20px !important;
        padding-right: 20px !important;
        padding-top: 0px !important;
        padding-bottom: 0px !important;
    }
</style>