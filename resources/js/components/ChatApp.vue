<template>
    <div class="chat-app">
       <Conversation :contact="selectedContact" :messages="messages" @new="saveNewMessage"/>
       <ContactLists :contacts="contacts" @selected="startConversationWith"/>
    </div>
</template>

<script>

Vue.component('Conversation', require('./Conversation.vue').default);
Vue.component('ContactLists', require('./ContactLists.vue').default);
    export default {
        props: {
            user: {
                type: Object,
                required: true
            }
        },
        data(){
            return {
                selectedContact: null,
                messages: [],
                contacts: []
            };
        },
        mounted() {

            Echo.private('messages.'+this.user.id)
                .listen('NewMessage', (e) => {
                    this.handleIncoming(e.message);
                });


            axios.get('/contacts')
                .then((response) => {
                    this.contacts = response.data;
                });

            // Echo.private('message')
            //     .listen('NewMessage', (e) => {
            //         this.handleIncoming(e.message);
            //     });
        },
        methods: {
            startConversationWith(contact){
                this.updateUnreadCount(contact, true);
                axios.get(`/conversation/${contact.id}`)
                    .then((response) => {
                        this.messages = response.data;
                        this.selectedContact = contact;
                    });
            },
            saveNewMessage(text){
                this.messages.push(text);
            },
            handleIncoming(message){
                if(this.selectedContact && message.from == this.selectedContact.id){
                    this.saveNewMessage(message);
                    return;
                }
                this.updateUnreadCount(message.from_contact, false);
            },
            updateUnreadCount(contact, reset) {
                this.contacts = this.contacts.map((single) => {
                    if(single.id != contact.id)
                    {
                        return single;
                    }

                    if(reset)
                    {
                        single.unread = 0;
                    }
                    else
                    {
                        single.unread += 1;
                    }

                    return single;
                })
            }
        }
    }
</script>

<style lang="scss" scoped>
    .chat-app{
        display: flex;
    }
</style>
