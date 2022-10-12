<template>
  <v-app>
    <v-main>
      <v-container>
        <v-row>
          <v-col cols="2" align="center">
            <v-btn
              color="green darken-1"
              @click="
                loadingClipsDialog = true;
                loadClips();
              "
              :disabled="settings.oauthToken === 'none'"
              >Load Clips</v-btn
            >
          </v-col>
          <v-col cols="2" align="center">
            <v-btn color="blue darken-1" @click="emitDateDialog = true" :disabled="settings.oauthToken === 'none'">Clip Range</v-btn>
          </v-col>
          <v-col cols="2" align="center">
            <v-btn color="#9146ff" @click="connectTwitch()">Connect Twitch</v-btn>
          </v-col>
          <v-spacer></v-spacer>
          <v-col cols="3" v-if="allClips.length > 1" style="color: #43a047"
            ><h3>{{ allClips.length }} clips are currently loaded!</h3></v-col
          >
          <v-col cols="3" v-if="allClips.length == 1" style="color: #43a047"
            ><h3>{{ allClips.length }} clip is currently loaded!</h3></v-col
          >
          <v-col cols="3" v-if="allClips.length == 0" style="color: #e53935"><h3>No clips loaded!</h3></v-col>
        </v-row>
        <v-row>
          <v-col align="center">
            <h3 style="color: #9146ff;">Clips from {{ currentStartDate }} till {{ currentEndDate }}</h3>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-simple-table>
              <template v-slot:default>
                <thead>
                  <tr>
                    <th class="text-left">Title</th>
                    <th class="text-left">Clip Autor</th>
                    <th class="text-left">Views</th>
                    <th class="text-left">Game</th>
                    <th class="text-left">Date</th>
                    <th class="text-left">Voters</th>
                    <th class="text-left">Rating</th>
                    <th class="text-left">Voted</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(item, a) in allClips" :key="a" style="cursor: pointer" @click="loadUrl(allClips.indexOf(item))">
                    <td>{{ item.name }}</td>
                    <td>{{ item.author }}</td>
                    <td>{{ item.views }}</td>
                    <td>{{ item.game }}</td>
                    <td>{{ item.date }}</td>
                    <td>{{ item.voters }}</td>
                    <td>{{ item.averageVote }}</td>
                    <td>{{ item.voted }}</td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
          </v-col>
        </v-row>
      </v-container>
      <v-row justify="center">
        <v-dialog v-model="emitDateDialog" persistent max-width="290">
          <v-card>
            <v-card-title v-if="!startOrEndDate" class="headline" style="color: #1A237E"
              >Please choose the <br />
              start date</v-card-title
            >
            <v-card-title v-if="startOrEndDate" class="headline" style="color: #1A237E"
              >Please choose the <br />
              end date</v-card-title
            >
            <v-divider></v-divider>
            <v-date-picker color="#1A237E" v-if="!startOrEndDate" v-model="currentStartDate"></v-date-picker>
            <v-date-picker color="#1A237E" v-if="startOrEndDate" v-model="currentEndDate"></v-date-picker>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="red darken-1"
                text
                @click="
                  emitDateDialog = false;
                  startOrEndDate = false;
                  buttonValue = 'Next';
                "
                >Cancel</v-btn
              >
              <v-btn color="green darken-1" @click="nextDate()" text>{{ buttonValue }}</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!--<v-dialog v-model="loadingClipsDialog" max-width="290">
                    <v-card align="center" min-height="200">
                       <v-card-title class="headline" style="color: purple">Loading clips...</v-card-title>
                       <br>
                      <v-progress-circular :size="70" :width="7" color="purple" indeterminate></v-progress-circular>
                    </v-card>
                  </v-dialog>-->
      </v-row>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data: () => ({
    timeOptions: ["Day", "Week", "Month"],
    currentTimeOptions: "Month",
    emitDateDialog: false,
    startOrEndDate: false,
    currentStartDate: new Date().toISOString().substr(0, 10),
    currentEndDate: new Date().toISOString().substr(0, 10),
    buttonValue: "Next",
    loadingClipsDialog: false,
  }),

  methods: {
    connectTwitch: function() {
      if (this.settings.oauthToken === "none") {
        window.open(
          "https://id.twitch.tv/oauth2/authorize?response_type=token&client_id=19e3dcu3gcraaro165ldrodkroql1e&redirect_uri=https://clipofthemonth.github.io/&scope=user_read"
        );
      }
    },
    loadClips: function() {
      this.allClips.splice(0, this.allClips.length);
      var streamerIDRequest = new XMLHttpRequest();
      streamerIDRequest.open("GET", `https://api.twitch.tv/helix/users?login=${this.settings.streamername}`, false);
      streamerIDRequest.setRequestHeader("Accept", "application/vnd.twitchtv.v5+json");
      streamerIDRequest.setRequestHeader("Client-ID", "19e3dcu3gcraaro165ldrodkroql1e");
      streamerIDRequest.setRequestHeader("Authorization", `Bearer ${this.settings.oauthToken}`);
      streamerIDRequest.send(null);
      var parsedStreamerIDResponse = JSON.parse(streamerIDRequest.responseText);
      var streamerID = parsedStreamerIDResponse.data[0].id;

      var clipRequest = new XMLHttpRequest();
      clipRequest.open(
        "GET",
        `https://api.twitch.tv/helix/clips?broadcaster_id=${streamerID}&started_at=${this.currentStartDate}T00:00:00Z&ended_at=${this.currentEndDate}T00:00:00Z&first=100`,
        false
      );
      clipRequest.setRequestHeader("Accept", "application/vnd.twitchtv.v5+json");
      clipRequest.setRequestHeader("Client-ID", "19e3dcu3gcraaro165ldrodkroql1e");
      clipRequest.setRequestHeader("Authorization", `Bearer ${this.settings.oauthToken}`);
      clipRequest.send(null);

      var response = clipRequest.responseText;
      var parsedResponse = JSON.parse(response);
      var clipArray = parsedResponse.data;
      for (var i = 0; i < clipArray.length; i++) {
        var currentClip = clipArray[i];
        var gameRequest = new XMLHttpRequest();
        gameRequest.open("GET", `https://api.twitch.tv/helix/games?id=${currentClip.game_id}`, false);
        gameRequest.setRequestHeader("Accept", "application/vnd.twitchtv.v5+json");
        gameRequest.setRequestHeader("Client-ID", "19e3dcu3gcraaro165ldrodkroql1e");
        gameRequest.setRequestHeader("Authorization", `Bearer ${this.settings.oauthToken}`);
        gameRequest.send(null);
        console.log(gameRequest.responseText);

        this.allClips.push({
          name: currentClip.title,
          slug: currentClip.id,
          url: currentClip.url,
          author: currentClip.creator_name,
          views: currentClip.view_count,
          game: JSON.parse(gameRequest.responseText).data[0].name,
          date: currentClip.created_at.split("T")[0],
          votes: 0,
          voted: false,
          voters: 0,
          averageVote: 0.0,
        });
      }
    },
    loadUrl: function(index) {
      window.open(this.allClips[index].url, "_blank");
    },
    nextDate: function() {
      if (this.startOrEndDate) {
        this.emitDateDialog = false;
        this.startOrEndDate = false;
        this.buttonValue = "Next";
      } else {
        this.startOrEndDate = true;
        this.buttonValue = "Save";
      }
    },
  },
  props: ["allClips", "settings"],
};
</script>
