<template>
    <div>
        <v-container>
            <v-progress-linear v-if="loading" :indeterminate="loading"></v-progress-linear>
            <div v-else>
            <v-row align-content="center" justify="center">
                <v-col>
                    <v-card>
                        <v-card-title>Settings</v-card-title>
                        <v-card-text>
                            <p>Modify what things you want to receive in your text. Also add a personal touch by including a mantra or reminder that is important to you.</p>
                            <v-checkbox v-model="disciplineJar" label=" A Cookie from the Discipline Jar (You'll always get this)" disabled></v-checkbox>
                            <v-checkbox v-model="form.Receive_MotivtnlVid_Bool" label="Motivational Video (Coming Soon)" disabled></v-checkbox>
                            <v-checkbox v-model="form.Receive_Workout_Bool" label="Workout Plan (Coming Soon)" disabled></v-checkbox>
                            <v-textarea v-model="form.Personal_Message_Tx" label="Enter a personalized mantra..." outlined></v-textarea>
                        </v-card-text>
                        <v-container>
                            <v-alert v-if="settingsSuccessPost" type="success">Settings successfully saved!</v-alert>
                            <v-alert v-if="settingsErrorPost" type="error">Uh oh, there was an issue</v-alert>
                            <v-progress-linear v-if="settingsPosting" :indeterminate="settingsPosting"></v-progress-linear>
                            <v-card-actions>
                                <v-btn outlined @click="saveSettings()">Save</v-btn>
                                <v-spacer></v-spacer>
                                <v-btn v-if="form.Active" outlined color="error" @click="showUnsub = true">Unsubscribe from Texts</v-btn>
                                <v-btn v-else outlined color="green" @click="updateUserStatus(true)">Subscribe to Texts</v-btn>
                            </v-card-actions>
                        </v-container>
                    </v-card>
                </v-col>
            </v-row>
            </div>
        </v-container>

        <v-overlay v-if="showUnsub" opacity="0.75">
            <p>Are you sure you want to unsubscribe?</p>
            <v-row align-content="center" justify="center">
                <v-btn @click="updateUserStatus(false)" class="unsubOverlayBtns" outlined color="error">Yes</v-btn>
                <v-btn @click="showUnsub = false" class="unsubOverlayBtns" outlined>No</v-btn>
            </v-row>
        </v-overlay>
    </div>
</template>

<script>

export default {
    data() {
        return {
            form: {
                Personal_Message_Tx: null,
                Receive_MotivtnlVid_Bool: false,
                Receive_Workout_Bool: false,
                Active: true
            },
            disciplineJar: true,
            authedUser: {
                user: null,
                userId: null, 
                jwtToken: null
            },
            showUnsub: false,
            loading: true,

            settingsPosting: false,
            settingsSuccessPost: false,
            settingsErrorPost: false,

            statusSuccessPost: false,
            statusErrorPost: false,
        }
    },
    mounted() {
        this.getSettings();
    },
    methods: {
        saveSettings() {
            this.settingsPosting = true

            const apiUrl = this.$store.state.baseURL
            const env = this.$store.state.env
            const phoneNum = this.$ls.get('signedInUserPhoneNum')
            
            const userSettings = {'PhoneNum': phoneNum,
                                    'Settings': {
                                        'ReceiveMotivationalVideo': this.form.Receive_MotivtnlVid_Bool,
                                        'ReceiveWorkoutPlan': this.form.Receive_Workout_Bool,
                                        'PersonalizedMessage': this.form.Personal_Message_Tx
                                    }}

            axios.post(apiUrl + env + 'user/settings', JSON.stringify(userSettings))
                .then(response => (console.log(response.data), 
                                    this.settingsPosting = false,
                                    this.settingsSuccessPost = true,
                                    setTimeout(() => (this.settingsSuccessPost = false), 1500)))
                .catch(error => (console.log(error), this.settingsErrorPost = true))
        },
        getSettings() {
            const apiUrl = this.$store.state.baseURL
            const env = this.$store.state.env
            const phoneNum = this.$ls.get('signedInUserPhoneNum')

            axios.get(apiUrl + env + 'user/settings?PhoneNum='+phoneNum)
                .then(response => (console.log(response.data), this.form = response.data, this.loading = false))
                .catch(error => (console.log(error)))
        },
        updateUserStatus(status) {
            this.settingsPosting = true

            const apiUrl = this.$store.state.baseURL
            const env = this.$store.state.env
            const phoneNum = this.$ls.get('signedInUserPhoneNum')
            
            const userSettings = {'PhoneNum': phoneNum,
                                    'Active': status}

            axios.post(apiUrl + env + 'user/status', JSON.stringify(userSettings))
                .then(response => (console.log(response.data), 
                                    this.settingsPosting = false,
                                    this.settingsSuccessPost = true,
                                    this.showUnsub = false,
                                    setTimeout(() => (this.settingsSuccessPost = false, this.$router.go(0)), 1500)))
                .catch(error => (console.log(error), this.settingsErrorPost = true))
        }
    }
}
</script>

<style scoped>
.unsubOverlayBtns {
    margin: 0 10px 10px 10px;
}
</style>