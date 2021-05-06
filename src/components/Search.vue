<template>
  <v-main>
    <v-container>
        <v-form>
            <v-row>
                <v-col cols="12" sm="8" md="8">
                    <v-text-field
                        placeholder="Search podcast content.."
                        outlined
                        v-model="searchTerm"
                    ></v-text-field>
                </v-col>
                <v-col cols="12" sm="4" md="4">
                    <v-btn large color="primary" @click.prevent="search" v-bind:disabled="searchTerm.length === 0 || this.searching" >Search</v-btn>
                </v-col>
            </v-row>
            <v-row>
                <v-col cols="12" sm="4" md="4">
                    <v-checkbox label="Phrase search" type="checkbox" id="checkbox" v-model="matchType"></v-checkbox>
                </v-col>
                <v-col cols="12" sm="4" md="4">
                    <v-text-field label="Maximum number of results" type="number" id="numResults" v-model="numResults"></v-text-field>
                </v-col>
            </v-row>
        </v-form>
    </v-container>
    <v-container>
        <div v-if="this.searching">
            <img v-bind:src="'/spinner.gif'"/>
        </div>
        <h2 v-if="!this.searching && this.took !== undefined">Found {{podData.length}} episode{{podData.length > 1 ? 's' : ''}} in {{this.took / 1000}} seconds</h2>
        <v-expansion-panels v-if="!this.searching">
            <v-expansion-panel v-for="(podcasts, index) in podData" :key="index">
                <v-expansion-panel-header>
                    <p v-if="podcasts[0].pod_data"> <b>{{podcasts[0].pod_data.podcast_name}} </b> ({{podcasts.length}} matching episode{{podcasts.length > 1 ? 's' : ''}})</p>
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                    <v-expansion-panels>
                        <v-expansion-panel v-for="(episode, index) in podcasts" :key="index">
                            <v-expansion-panel-header>
                                <p> {{episode.episode_data.episode_name}} 
                                    <span><a v-bind:href="'https://open.spotify.com/episode/' + episode.episode_data.episode_uri.split(':')[2]" target="_blank" rel="noopener noreferrer"> to episode </a></span>
                                </p>
                            </v-expansion-panel-header>
                            <v-expansion-panel-content>
                                <p> <b>Episode length:</b> <span v-if="Math.floor((episode.episode_data.duration) / 60) !== 0">{{Math.floor((episode.episode_data.duration) / 60)}}h</span> {{Math.round(episode.episode_data.duration % 60)}}m {{Math.round((episode.episode_data.duration % 1)*60)}}s </p>
                                <div v-for="segment in episode.transcript.transcripts" :key="segment.index">
                                    <p>
                                        <span>Relevant segment </span>
                                        <span v-bind:title="segment.startTime"><b>starts at:</b> {{ secondsToHMS(segment.startTime) }} </span>
                                        <span v-bind:title="segment.endTime"><b>and ends at:</b> {{ secondsToHMS(segment.endTime) }}</span>
                                    </p>
                                    <Transcript v-bind:transcript="segment.transcript" v-bind:searchTerm="searchTerm" v-bind:phrase="matchType"/>
                                </div>
                            </v-expansion-panel-content>
                        </v-expansion-panel>
                    </v-expansion-panels>
                </v-expansion-panel-content>
            </v-expansion-panel>
        </v-expansion-panels>
    </v-container>
  </v-main>
</template>

<script>
import axios from "axios"
import Transcript from './Transcript.vue'

export default {
  name: 'Search',
  data: () => ({
      searchTerm: "",
      matchType: false,
      numResults: 50,
      podData: [],
      took: undefined,
      searching: false,
  }),
  components: {
      Transcript
  },
  methods: {
      secondsToHMS(timeAsString = "") {
        let duration = timeAsString.replace("s", "")
        let result = ""
        let hours = Math.floor((duration) / 3600)
        let minutes = Math.floor((duration / 60) % 60)
        let seconds = Math.floor((duration % 3600) % 60)
        
        if (hours > 0) result += hours + "h "
        if (minutes > 0) result += minutes + "m "
        result += seconds + "s"

        return result
      },
      search() {
        let match = "match"
        if (this.matchType) {
            match = "match_phrase"
        }
        this.searching = true
        axios.get("http://localhost:5000/api/search", {
            params: {
                search: this.searchTerm,
                type: match,
                size: this.numResults
            }
        })
        .then(res => {
            console.log(res.data)
            this.podData = res.data.results
            this.took = res.data.took
            this.searching = false
        })
        .catch(err => {
            console.log(err)
            this.searching = false
        });
      }
  }
}
</script>

<style scoped>
div {
    text-align: left;
}

</style>
