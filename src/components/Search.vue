<template>
  <div class="content">
    <h1>Search for podcast content: </h1>
    <div>
        <form>
        <input class="clear-right margin-1 input-field" v-model="searchTerm" />
        <h4>Search settings:</h4>
        <div>
        <label for="checkbox">Phrase search </label>
        <input type="checkbox" id="checkbox" v-model="matchType">
        </div>
        <div>
        <label for="numResults">Maximum number of results </label>
        <input type="number" id="numResults" v-model="numResults">
        </div>
        <div class="margin-1">
            <button class="search-button" @click.prevent="search" v-bind:disabled="searchTerm.length === 0 || this.searching" >Search</button>
        </div>
        </form>
    </div>
    <div class="center" v-if="this.searching">
        <img v-bind:src="'/spinner.gif'"/>
    </div>
    <h2 v-if="!this.searching && this.took !== undefined">Found {{podData.length}} episode(s) in {{this.took / 1000}} seconds</h2>
    <ul class="result-list" v-if="!this.searching">
        <div v-for="(podcasts, index) in podData" :key="index" class="bg-block">
            <h3 v-if="podcasts[0].pod_data"> {{podcasts[0].pod_data.podcast_name}} </h3>
            <h4> episode index: {{index}} </h4>
            <div v-for="(episode, index) in podcasts" :key="index">
                <a v-bind:href="'https://open.spotify.com/episode/' + episode.episode_data.episode_uri.split(':')[2]" target="_blank" rel="noopener noreferrer"> {{episode.episode_data.episode_name}} </a>
                <p> <b>Length:</b> <span v-if="Math.floor((episode.episode_data.duration) / 60) !== 0">{{Math.floor((episode.episode_data.duration) / 60)}}h</span> {{Math.round(episode.episode_data.duration % 60)}}m {{Math.round((episode.episode_data.duration % 1)*60)}}s </p>
                <div class="bg-block" v-for="segment in episode.transcript.transcripts" :key="segment.index">
                    <p>
                        <span v-bind:title="segment.startTime"><b>Starts at:</b> {{ secondsToHMS(segment.startTime) }} </span>
                        <span v-bind:title="segment.endTime"><b>Ends at:</b> {{ secondsToHMS(segment.endTime) }}</span>
                    </p>
                    <p>segment index: {{segment.index}}</p>
                    <Transcript v-bind:transcript="segment.transcript" v-bind:searchTerm="searchTerm" v-bind:phrase="matchType"/>
                </div>
            </div>
        </div>
    </ul>
  </div>
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
        axios.get("http://localhost:5000/search", {
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
h3 {
  margin: 40px 0 0;
}
.content {
    margin: 1rem;
}
.clear-right {
    display: block;
}
.margin-1 {
    margin: 1rem 0 1rem 0;
}

.bg-block {
    background-color: #F0EFEF;
}

.bg-block p {
    margin: 0
}

.input-field {
    padding: 0.2rem;
    border-radius: 4px;
    border: 1px solid lightgray;
    width: 50%;
}

.search-button {
    padding: 0.5rem;
    border-radius: 4px;
    background-color: lightblue;
    border-style: none;
}

ul {
    list-style-type: none;
}

.center {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 50vh;
}

.result-list {
    padding: 0;
}
</style>
