<template>
  <div class="content">
    <h1>Search for podcast content: </h1>
    <div>
        <input class="clear-right margin-1 input-field" v-model="searchTerm" />
        <label for="checkbox">Check for phrase search</label>
        <input type="checkbox" id="checkbox" v-model="matchType">
        <div class="margin-1">
            <button class="search-button" @click.prevent="search" >Search</button>
        </div>
    </div>
    <h2>Top 10 Results: </h2>
    <ul>
        <div v-for="(podcasts, index) in podData" :key="index" class="bg-block">
            <h3 v-if="podcasts[0].pod_data"> {{podcasts[0].pod_data.hits.hits[0]._source.podcast_name}} </h3>
            <h4> episode index: {{index}} </h4>
            <div v-for="(episode, index) in podcasts" :key="index">
                <a v-bind:href="'https://open.spotify.com/episode/' + episode.episode_data.episode_uri.split(':')[2]" target="_blank" rel="noopener noreferrer"> {{episode.episode_data.episode_name}} </a>
                <p> episode length: {{episode.episode_data.duration }} </p>
                <div class="bg-block" v-for="segment in episode.transcript.transcripts" :key="segment.index">
                    <p>
                        <span> startTime: {{segment.startTime }} </span>
                        <span> endTime: {{segment.endTime }} </span>
                        <span> segment index: {{segment.index}} </span>
                    </p>
                    <p> {{segment.transcript}} </p>
                </div>
            </div>
        </div>
    </ul>
  </div>
</template>

<script>
import axios from "axios"

export default {
  name: 'Search',
  data: () => ({
      searchTerm: "",
      matchType: false,
      podData: []
  }),
  methods: {
      search() {
        let match = "match"
        if (this.matchType) {
            match = "match_phrase"
        }
        axios.get("http://localhost:5000/", {
            params: {
                search: this.searchTerm,
                type: match,
                size: 50
            }
        })
        .then(res => {
            console.log(res.data)
            this.podData = res.data
        })
        .catch(err => {
            console.log(err)
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
</style>
