<template>
  <v-app>
    <v-app-bar>

      <v-container>
        <v-row>
          <v-col cols="1">
            <v-avatar>
              <img src="./assets/logo.png">
            </v-avatar>
          </v-col>
          <v-col>
            <h1>Clip of the Month &copy; S3vro</h1>
          </v-col>
        </v-row>
      </v-container>

      <v-spacer></v-spacer>
          <v-tabs right class="ml-n9" color="amber lighten-1" v-model="current_link">
            <v-tabs-slider color="amber lighten-1"></v-tabs-slider>
              <v-tab v-for="link in links" :key="link">
                  {{ link }}
              </v-tab>
          </v-tabs>

    </v-app-bar>

    <v-main>
      <Dashboard :allClips="allClips" :settings="settings" v-show="current_link==0"/>
      <AllClips  :allClips="allClips" :settings="settings" v-show="current_link==1"/>
      <Settings  :settings="settings" v-show="current_link==2"/>
    </v-main>
  </v-app>
</template>

<script>
  import Dashboard from './components/dashboard.vue'
  import AllClips from './components/allclips.vue'
  import Settings from './components/settings.vue'
  export default {
  components: {
      Dashboard,
      AllClips,
      Settings
  },
  created(){
    if(this.settings.oauthToken === "none"){
            const queryString = location.hash;
            if(queryString !== ""){
              this.settings.oauthToken = queryString.split("&")[0].split("=")[1];
              window.history.replaceState({}, document.title, "/");
              console.log("Token!" + this.settings.oauthToken);
            }
         }
  },
    data: () => ({
      current_link: 0,
      links: ['Dashboard','All Clips','Settings',],
      allClips: [],
      settings: {streamername: "s3vro",votingStartedMessage: "/me Voting was started! Type !voteclip <number 0-10> to vote!",votingEndedMessage: "/me Voting is now over!",voteOwnClip: "Off",oauthToken: "none"}
    })
  }
</script>