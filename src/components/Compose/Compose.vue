<template>
    <div class="page-content">
        <RecipientBar :onContactListChanged="onContactListChanged"/>
        <Sendbar :onSend="sendMessage" :loading="sending" />
    </div>
</template>

<script>
import Vue from 'vue'

import { Api, Crypto, Util } from "@/utils/"
import Sendbar from '../Thread/Sendbar.vue'
import RecipientBar from './RecipientBar.vue'

export default {
    name: 'compose',

    mounted () {
        this.$store.commit('colors_default', this.$store.state.theme_global_default)
        this.$store.commit('colors_dark', this.$store.state.theme_global_dark)
        this.$store.commit('colors_accent', this.$store.state.theme_global_accent)

        this.$store.commit("loading", false);
        this.$store.commit('title', this.title);
    },

    data () {
        return {
            title: 'Compose',
            sending: false,
            selectedContacts: [],
        }
    },

    methods: {
        onContactListChanged(list) {
            this.selectedContacts = list;
        },
        sendMessage(message) {
            let to = "";

            if (this.selectedContacts.length <= 0)
                return Util.Snackbar("No recipient");

            this.selectedContacts.map((value) => { // Concat selected contacts
                to += value.phone + ",";
            });

            to = to.slice(0, to.length - 1); // Remove trailing comma
            this.sending = true;

            // send image, if one is attached
            if (this.$store.state.loaded_media) {
                Api.sendFile(this.$store.state.loaded_media, (file, messageId) => {
                    Api.createThreadWithImage(to, messageId, file.type);
                });
            }

            if (message.length > 0) {
                Api.createThread(to, message);
            }

            setTimeout(() => {
                this.$router.push('/');
            }, 1500);
        },
    },

    components: {
        RecipientBar,
        Sendbar,
    }

}
</script>

<style lang="scss" scoped>

    @import "../../assets/scss/_vars.scss";

    .page-content {
        height: calc(100vh - 145px);
    }

</style>
