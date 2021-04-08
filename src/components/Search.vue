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
        <li v-for="item in podData" :key="item.transcript._id">
            <div class="bg-block" v-if="item.pod_data">
                <h3> {{item.pod_data.hits.hits[0]._source.podcast_name}} </h3>
                <h4> {{item.episode_data.episode_name}} </h4>
                <p> {{item.transcript._source.transcript}} </p>
            </div>
        </li>
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
                type: match
            }
        })
        .then(res => {
            console.log(res.data)
            this.podData = res.data.results
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
