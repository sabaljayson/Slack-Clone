<template>
 <div class="channels__container">
        <h2 class="ui inverted center aligned header">Channels <i class="add square icon add_channel" @click="openChannelModal"></i></h2>
        <div class="ui raised channels__list">
            <ul>
                <li class="channels__item" 
                    v-for="channel in channels" 
                    :key="channel.id" 
                    :class="{'is_active': setChannelActive(channel)}"
                    @click="changeChannel(channel)">
                    # {{ channel.name }}
               </li>
            </ul>
        </div>


        <!-- Modal Pour ajouter un Channel -->

        <div class="ui basic modal" id="channelModal">
            <div class="ui icon header">
                Add Channel
            </div>

            <div class="content">
                <div class="ui inverted form" :class="{'error': hasErrors }">
                    <div class="field">
                        <label for="new_channel">Name for Channel</label>
                        <input type="text" name="new_channel" v-model="new_channel" id="new_channel">
                    </div>

                    <div class="ui error message" v-if="hasErrors">
                        <p v-for="error in errors">{{ error }}</p>
                    </div>
                </div>
            </div>

            <div class="actions">
                <div class="ui red basic cancel inverted button">
                    <i class="remove icon"></i>Not add
                </div>
                <div class="ui green inverted button" @click="addChannel">
                    <i class="checkmark icon"></i>Add
                </div>
            </div>


        </div>

    </div>
</template>
<script>
     import {mapGetters} from 'Vuex'
       export default {
        name: 'channels', 
        data () {
            return {
                channels: [],
                new_channel: '',
                channelsRef: firebase.database().ref('channels'),
                errors: [],
                firstLoad: true     
            }
        },
        computed : {
            ...mapGetters(['currentChannel']),
            hasErrors () {
                return this.errors.length > 0
            }
        },
        mounted () {
            this.addListeners()
        },
        methods: {
            addListeners () {
                this.channelsRef.on('child_added', snap => {
                    this.channels.push(snap.val())
                    
                    if(this.firstLoad && this.channels.length > 0){
                        this.$store.dispatch("setCurrentChannel", this.channels[0])
                    }
                    this.firsLoad = false
                })
            },
            openChannelModal () {
                $("#channelModal").modal('show')
            },
            addChannel () {
                this.errors = []
                let key = this.channelsRef.push().key
                let newChannel = { id: key, name: this.new_channel }
                this.channelsRef.child(key).update(newChannel).then( () => {
                    this.new_channel = ''
                    $("#channelModal").modal('hide')
                }).catch( error => {
                    this.errors.push(error.message)
                })
            },
            changeChannel(channel){
                this.$store.dispatch('setPrivate', false)
                this.$store.dispatch('setCurrentChannel', channel)
            },
            detachListeners () {
                this.channelsRef.off()
            },
            setChannelActive(channel){
                return channel.id === this.currentChannel.id
            }
        },
        beforeDestroy () {
            this.detachListeners()
        }
    }
</script>
<style scoped>
    .channels__list {
        min-height: 100px;
        max-height: 300px!important;
        overflow-y: auto!important;
    }
    .channels__container ul{
        margin: 0;
        padding: 0;
        background-color: #4d394b;
    }
    .raised{
      background-color: #4d394b;  
    }
    .channels__item{
        height: 30px;
        margin: 8px;
        list-style: none;
        background-color: #4d394b;
        cursor: pointer;
        line-height: 30px;       
        border-radius: 2px;
        padding-left: 12px;
        font-weight: bold;
        font-size: 1.1em;
        color: #ab9ba9;
    }
    .channel__count{
        float:right;
    }
    .is_active{
        color: #fff;
        background-color: rgba(0,158,195,1);
    }
    .channels__item:hover{
        background-color: #3e313c;
    }
    .add_channel{
        cursor: pointer;
        color: #2ab27b;
    }
    .add_channel:hover{
        color: #689F38;
    } 
</style>
