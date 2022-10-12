<template>
   <v-app>
      <v-main>
         <v-container>
            <v-row>
               <v-col cols="12" sm="8" align="center" justify="center">
                  <v-sheet min-height="392">
                        <iframe  v-if="allClips.length > 0" style="width:100%" :src="getLink(allClips[currentClip])" height="394" frameborder="0" scrolling="no"></iframe>
                        <v-alert v-if="allClips.length == 0" type="error">You have to load a set of clips first!</v-alert>
                  </v-sheet>
               </v-col>
               <v-col cols="12" sm="4">
                  <v-sheet rounded="lg" min-height="394">
                     <div  v-if="allClips[currentClip] != undefined">
                     <v-simple-table>
                        <tbody>
                           <tr>
                              <td class="font-weight-bold">Name</td>
                              <td>{{allClips[currentClip].name}}</td>
                           </tr>
                           <tr>
                              <td class="font-weight-bold">Author</td>
                              <td>{{allClips[currentClip].author}}</td>
                           </tr>
                           <tr>
                              <td class="font-weight-bold">Game</td>
                              <td>{{allClips[currentClip].game}}</td>
                           </tr>
                           <tr>
                              <td class="font-weight-bold">Views</td>
                              <td>{{allClips[currentClip].views}}</td>
                           </tr>     
                           <tr>
                              <td class="font-weight-bold">Date</td>
                              <td>{{allClips[currentClip].date}}</td>
                           </tr> 
                           <tr>
                              <td class="font-weight-bold">Votes</td>
                              <td>{{allClips[currentClip].votes}}</td>
                           </tr>       
                           <tr>
                              <td class="font-weight-bold">Voted</td>
                              <td>{{allClips[currentClip].voted}}</td>
                           </tr>     
                           <tr>
                              <td class="font-weight-bold">Clip Id</td>
                              <td>{{currentClip}}</td>
                           </tr>                                                                                                                
                         </tbody>
                     </v-simple-table>
                     </div>
                  </v-sheet>
               </v-col>
            </v-row>
            <v-divider></v-divider>
            <v-row>
               <v-col>
                  <v-sheet min-height="300" rounded="lg">
                     <v-container>
                        <v-row>
                           <v-col cols="6">
                              <v-container>
                                 <v-row>
                                    <v-col cols="4" align="center">
                                    <v-btn color="red darken-1" @click="previousClip()">Previous Clip</v-btn>
                                    </v-col>
                                    <v-col cols="4" align="center">
                                    <v-btn color="green darken-1" @click="nextClip()">Next Clip</v-btn>
                                    </v-col>
                                    <v-col cols="4" align="center">
                                    <v-btn color="blue darken-1" @click="openClip(currentClip)">Open Clip</v-btn>
                                    </v-col>
                                 </v-row>
                                 <v-row>
                                    <v-col cols="4" align="center">
                                       <v-btn color="blue darken-1" @click="openBestClip()">Open Best Clip</v-btn>
                                    </v-col>
                                    <v-col cols="4" align="center">
                                       <v-btn v-if="voting" color="red darken-1" @click="toggleVoting()">Stop Voting</v-btn>
                                       <v-btn v-if="!voting" color="green darken-1" @click="toggleVoting()">Start Voting</v-btn>
                                    </v-col>
                                    <v-col cols="4" align="center">
                                    <v-btn color="red darken-1" @click="deleteClip()">Remove Clip</v-btn>
                                    </v-col>
                                 </v-row>
 
                              </v-container>
                           </v-col>
                           <v-divider vertical></v-divider>
                           <v-col cols="5">
                              <h2 style="color: #9146ff;">Twitch Chat</h2>
                              <h3 v-if="!voting">The chat will only be shown during voting!</h3>
                              <div v-if="voting" id="chat_table" style="height:250px; overflow-y: auto; border-style:solid; border-color: #9146ff;">
                              <v-simple-table>
                                 <template v-slot:default>
                                    <tbody>
                                    <tr v-for="(item,i) in messages" :key="i">
                                       <td>{{item.time}}</td>
                                       <td v-bind:style="{ color: item.color}">{{item.username}}</td>
                                       <td >{{item.message}}</td>
                                    </tr>
                                    </tbody>
                                 </template>
                              </v-simple-table>
                              </div>
                           </v-col>
                        </v-row>
                     </v-container>
                  </v-sheet>
               </v-col>
            </v-row>
         <v-snackbar timeout="2000" v-model="emitErrorSnackbar" bottom color="error" outlined>{{errorText}}</v-snackbar>
          <v-snackbar timeout="2000" v-model="emitSuccessSnackbar" bottom color="success" outlined>{{successText}}</v-snackbar>
          <v-row justify="center">
                 <v-dialog v-model="emitVotedDialog" persistent max-width="290">
                    <v-card>
                       <v-card-title class="headline" style="color: #E53935;">
                          There was already <br> voted for this clip!
                       </v-card-title>
                       <v-card-text>Would you like to overwrite the previous votes by voting for this clip again?</v-card-text>
                       <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="green darken-1" text @click="votedAlertFunc(0)">No</v-btn>
                        <v-btn color="green darken-1" text @click="votedAlertFunc(1)">Yes</v-btn>
                       </v-card-actions>
                    </v-card>
                  </v-dialog>
          </v-row>
         </v-container>
      </v-main>
   </v-app>
</template>

<script>
const tmi = require("tmi.js")

   export default {
     data() { 
        return {
            voting: false,
            client: null,
            currentClip: 0,
            messages: [],
            emitErrorSnackbar: false,
            errorText: "Error",
            usersVoted: [],
            successText: "Success",
            emitSuccessSnackbar: false,
            emitVotedDialog: false,
     }
     },
      methods: {
         toggleVoting: function() {
            if(this.allClips.length > 0) {
               if(!this.allClips[this.currentClip].voted) {
                  this.voting=!this.voting;
                  if(this.voting) {
                     this.connectTwitchChat();
                  }else {
                     this.disconnectTwitchChat();
                     if(this.allClips[this.currentClip].votes > 0)
                     {
                        var votedClip = this.allClips[this.currentClip];

                        votedClip.voted=true;
                        votedClip.averageVote = votedClip.votes/votedClip.voters;
                     } 
                     this.usersVoted = [];
                     this.messages = [];
                  } 
               }else this.emitVotedDialog=true;
            }else this.emitError("You'll have to load your clips before doing this!");
         },
         connectTwitchChat: function() 
         {
            const opts = {
            identity: {
               username: 'ClipOfTheMonth',
               password: 'oauth:go86usbee015cydjidz4tc190ufomr'
            },
            channels: [
               this.settings.streamername
            ],
            connection: {
               secure: true,
               reconncet: true
            }
            };
            this.client = new tmi.client(opts);
            this.client.on('message', this.onMessageHandler);
            this.client.on('connected', this.onConnectedHandler);
            this.client.connect();
            console.log("Starting Connection...");
         },
         disconnectTwitchChat: function () {
            this.client.say(this.settings.streamername,this.settings.votingEndedMessage);
            this.client.disconnect();
            this.client = null;
            console.log("* Disconnected")
         }
         ,
         onConnectedHandler: function(addr,port) 
         {
            console.log(`* Connected to ${addr}:${port}`);
            this.client.say(this.settings.streamername,this.settings.votingStartedMessage);
         },
         onMessageHandler: function (target,context,msg,self) 
         {
            if(self) return;
            
            var userinfo =  context;
            var username = userinfo.username;
            var currentdate = new Date(); 
            this.messages.push({username: username,color: userinfo.color,message: msg,time: currentdate.getHours() + ":" + currentdate.getMinutes()});

            var msgSplited = msg.split(' ');
            if(msgSplited.length == 2)
            {
               if(msgSplited[0] === '!voteclip')
               {
                  const votingScore = msgSplited[1];
                  if(this.settings.voteOwnClip === "Off"){
                     if(username === this.allClips[this.currentClip].author) return;
                  }
                  
                  if(!isNaN(votingScore)) {
                     const votingScoreInt = parseInt(votingScore);
                     if( votingScoreInt<=10 && votingScoreInt>=0){
                        if(this.usersVoted.indexOf(username) == -1){
                           this.client.say(target,`${username} voted ${votingScoreInt}/10!`);
                           this.allClips[this.currentClip].votes+=votingScoreInt;
                           this.usersVoted.push(username);
                           this.allClips[this.currentClip].voters+=1;
                        }
                     }
                  }
               }
            }
            this.fixScroll();
         },
         fixScroll: function () {
            var table = this.$el.querySelector("#chat_table");
            //var height = table.scrollHeight;
            var height = table.scrollHeight + 48;
            if(table != null) table.scrollTop = height;
         },
         getLink: function (clip) {
            return `https://clips.twitch.tv/embed?clip=${clip.slug}&parent=clipofthemonth.github.io`;
         },
         previousClip: function () {
            if(this.voting){
               this.emitError("You can't do this during vote time! Disable voting to use this button!");
               return;
            }
            if(this.allClips.length > 0){
                  if((this.currentClip-1) >= 0) this.currentClip-=1;
            }else this.emitError("You'll have to load your clips before doing this!");            

         },
         nextClip: function () {
            if(this.voting){
               this.emitError("You can't do this during vote time! Disable voting to use this button!");
               return;
            }
            if(this.allClips.length > 0){
               if((this.currentClip+1) < this.allClips.length) this.currentClip+=1;
            }else this.emitError("You'll have to load your clips before doing this!");                        
         },
         openClip: function (clipindex) {
            if(this.voting){
               this.emitError("You can't do this during vote time! Disable voting to use this button!");
               return;
            }
            if(this.allClips.length > 0){
               window.open(this.allClips[clipindex].url,'_blank');
            }else this.emitError("You'll have to load your clips before doing this!");               
         },
         emitError: function (text) 
         {
            this.errorText = text;
            this.emitErrorSnackbar = true;
         },
         emitSucces: function(text)
         {
            this.successText = text;
            this.emitSuccessSnackbar = true;
         }
         ,
         deleteClip: function(){
            if(this.voting){
               this.emitError("You can't do this during vote time! Disable voting to use this button!");
               return;
            }
            if(this.allClips.length > 0){
               this.allClips.splice(this.currentClip,1)
               this.emitSucces("Clip deleted successfully!");
            }else this.emitError("You'll have to load your clips before doing this!");                  
         },
         openBestClip: function(){
            if(this.voting){
               this.emitError("You can't do this during vote time! Disable voting to use this button!");
               return;
            }
            if(this.allClips.length > 0){
            var allClipsCopy = this.allClips.slice();
            allClipsCopy.sort((a,b) => (a.averageVote > b.averageVote) ? -1:1);
            this.openClip(this.allClips.indexOf(allClipsCopy[0]));
            }else this.emitError("You'll have to load your clips before doing this!");
         },
         votedAlertFunc: function(type){
            this.emitVotedDialog = false;
            this.voting = false;
            if(type){
               this.allClips[this.currentClip].voted = false;
               this.allClips[this.currentClip].votes = 0;
               this.toggleVoting();
               this.emitSuccess("The clip was reset successfully!");
            }
         }
      },
      props: [
         'allClips',
         'settings'
      ]
   }
</script>