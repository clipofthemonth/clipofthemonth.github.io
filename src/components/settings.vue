<template>
   <v-app>
      <v-main>
        <v-container>
            <v-sheet rounded="lg" min-height="70vh">
            <v-row>
               <v-col align="center"> <h1>Settings</h1></v-col>
            </v-row>

            <v-row>
               <v-col cols=1></v-col>
               <v-col cols=3> <h3>Your twitch name:</h3></v-col>
               <v-col cols=4>
                  <v-text-field label="Streamername" outlined dense v-model="settings.streamername"></v-text-field>
               </v-col>
            </v-row>
            <v-row>
               <v-col cols=1></v-col>
               <v-col cols=3> <h3>Voting started message:</h3></v-col>
               <v-col cols=4>
                  <v-text-field label="Voting started message" outlined dense v-model="settings.votingStartedMessage"></v-text-field>
               </v-col>
            </v-row>
            <v-row>
               <v-col cols=1></v-col>
               <v-col cols=3> <h3>Voting over message:</h3></v-col>
               <v-col cols=4>
                  <v-text-field label="Voting over message" outlined dense v-model="settings.votingEndedMessage"></v-text-field>
               </v-col>
            </v-row>
            <v-row>
               <v-col cols=1></v-col>
               <v-col cols=3> <h3>Let authors vote own clip</h3></v-col>
               <v-col cols=4>
                  <v-select v-model="settings.voteOwnClip" dense :items="selectOnOf" label="Vote own clip" outlined></v-select>
               </v-col>
            </v-row>
            <v-row align="center" justify="left">
               <v-col cols=1></v-col>
               <v-col cols=3> <h3>Darkmode</h3></v-col>
               <v-col cols=4>
                  <v-switch v-model="this.$vuetify.theme.dark" dense outlined @click="darkMode()"></v-switch>
               </v-col>
            </v-row>            
            <v-divider horizontal></v-divider>
            <br>
            <v-row>
               <v-col cols=1></v-col>
               <v-col cols=10>
                  <v-card elevation="2">
                     <v-container>
                        <v-row>
                           <v-col cols=1></v-col>
                           <v-col>
                              <h3>About</h3>
                              The Clip of the month tool is developed by S3vro (<a href="https://twitch.tv/s3vro">Twitch</a>,<a href="https://twitter.com/Sevr0o">Twitter</a>).
                              You are free to use it in your streams, but if so, please credit the creator.
                              <br>
                              If there are any bugs or feature request you can send me a message on Discord: <a>S3vro#5133</a>
                              <br> <br>
                              I hope you enjoy the tool and if you do please let others know!
                              <br>
                              <br>
                              <h3>New Features</h3>
                              <br>
                              <li>Added DarkMode</li>
                              <br>
                              Current site version: 0.9.2                              
                           </v-col>
                           <v-col cols=1></v-col>
                        </v-row>
                     </v-container>
                  </v-card>
               </v-col>
            </v-row>
            </v-sheet>
        </v-container>
      </v-main>
   </v-app>
</template>

<script>
   export default {
     data: () => ({
        selectOnOf: ["On","Off"]
     }),
     methods: {
        darkMode: function (){
            this.$vuetify.theme.dark = !this.$vuetify.theme.dark;
        }
      },  
     props: [
        'settings'
     ]
   }
</script>