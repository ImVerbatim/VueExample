<template>
 <div class="chat-app" style="border-right: 1px solid #424242">
    <div class="col">     
        <div class="row-content center z-depth-3" style="border-bottom: 1px solid #424242">
             <span class="flow-text white-text">{{ title }}</span>
        </div>

        <div class="row" style="position: relative; margin-bottom: 0px;">
            <div id="chat-messages" class="card-content left" style="margin: 10px;" v-html="chatContent">
            </div>
        </div>
    </div>

    <!-- if a session is joined -->
    <div class="col blue-grey darken-4 z-depth-1">
        <div class="row chat-box" style="margin-bottom:0px;margin-left:5px" v-if="joined">
            <div class="input-field col s4 left box">
                <input type="text" v-model="newMsg" placeholder="Say something here" @keyup.enter="send">
            </div>
            <a class="btn-floating lime lighten-2" style="margin: 10px; margin-top:20px" @click="send">
                <i class="material-icons" style="line-height: 40px; color: black;">send</i>
            </a>
            <a class="btn-floating lime lighten-2" style="margin: 10px; margin-top:20px" @click="close">
                <i class="material-icons" style="line-height: 40px; color: black;">close</i>
            </a>
        </div>

        <!-- if a session is not joined -->
        <div class="row chat-box" style="margin-bottom:0px; margin-left:5px" v-if="!joined">
            <div class="input-field col s4 box">
                <input type="email" v-model.trim="email" placeholder="Email">
            </div>
            <div class="input-field col s4 box" style="margin-left:5px">
                <input type="text" v-model.trim="username" placeholder="Username">
            </div>
          <a class="btn-floating lime lighten-2" style="margin: 10px; margin-top:20px" @click="join">
                <i class="material-icons" style="line-height: 40px; color: black;">done</i>
            </a>
        </div>
    </div>
</div>

</template>

<script>
import * as emojione from "emojione";
import $ from "jquery";
export default {
  name: 'Chat',

  props: {
  title: String
  },

  data: function() {
    return {
        ws: null, // Our websocket
        newMsg: '', // Holds new messages to be sent to the server
        chatContent: '', // A running list of chat messages displayed on the screen
        email: null, // Email address used for grabbing an avatar
        username: null, // Our username
        joined: false, // True if email and username have been filled in
        autoReconnectInterval: 100,
        debug: true
    }
  },

    methods: {
        send: function () {
            if (this.newMsg != '') {
                this.ws.send(
                    JSON.stringify({
                        email: this.email,
                        username: this.username,
                        message: $('<p>').html(this.newMsg).text() // Strip out html
                    }
                ));
                this.newMsg = ''; // Reset newMsg
            }
        },

        join: function () {
            if (!this.email) {
                Materialize.toast('You must enter an email', 2000);
                return
            }
            if (!this.username) {
                Materialize.toast('You must choose a username', 2000);
                return
            }
            this.email = $('<p>').html(this.email).text();
            this.username = $('<p>').html(this.username).text();
            this.joined = true;
            this.open();
        },

        profileURL: function() {
            return '../assets/logo.png';
        },

        maybeReconnectToWebsocket: function(event) {
            switch (event.code){
                case 1000:	// CLOSE_NORMAL
                    console.log("WebSocket: closed");
                    break;
                default:	// Abnormal closure
                    this.reconnect(event);
                    break;
                }
                //this.onclose(e);
        },

        reconnect: function(event) {
            console.log(`WebSocketClient: retry in ${this.autoReconnectInterval}ms`,event);
            this.ws.addEventListener('message',null);
            this.ws.addEventListener('close',null);
            this.ws = null;
            var that = this;
            setTimeout(function(){
                console.log("WebSocketClient: reconnecting...");
                that.open();
            },this.autoReconnectInterval);
        },

        open: function() {
            var self = this;
            if(!this.debug)
                this.ws = new WebSocket(((window.location.protocol === "https:") ? "wss://" : "ws://") + "calculator-example.herokuapp.com" + "/ws");
            else
                this.ws = new WebSocket(((window.location.protocol === "https:") ? "wss://" : "ws://") + "localhost:8000" + "/ws");

            this.ws.addEventListener('message', function(event) {
                var msg = JSON.parse(event.data);
                self.chatContent += '<div class="chip lime lighten-2">'
                        + '<i class="material-icons left black-text">account_circle</i>'
                            + msg.username
                        + '</div>'
                    + emojione.toImage(msg.message) + '<br/>'; // Parse emojis
                var element = document.getElementById('chat-messages');
                element.style.color = "#FFFFFF";
                element.scrollTop = element.scrollHeight; // Auto scroll to the bottom
            });

            this.ws.addEventListener('close', (event) => {
                this.maybeReconnectToWebsocket(event);
            });
            },

        close: function() {
            this.ws.close(1000);
            this.joined = false;
        },
        
    }
}
</script>

<style >


#chat-messages {
    min-height: 10vh;
    height: 60vh;
    width: 100%;
    overflow-y: scroll;
    color:white;
}

.box {
    padding:5px;
    border: 1px solid #616161;
     border-radius: 4px;
     background-color:#37474f;
}

input, select, textarea{
    color: #FFFFFF;
}

.material-icons.left {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    vertical-align: middle;
    line-height: inherit;
}

.row.chat-box {
    margin-right: 0px;
    margin-left: 0px;
}


.chat-app {
    flex: 0 1 auto;
}
</style>
