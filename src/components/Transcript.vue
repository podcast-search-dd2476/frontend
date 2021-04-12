<template>
  <div>
    <p v-html="this.bolded"></p>
  </div>
</template>

<script>

export default {
  name: 'Transcript',
  props: {
      transcript: String,
      searchTerm: String,
      phrase: Boolean
  },
  created() {
    // Init array with false
    let words = []
    for (let i = 0; i < this.transcript.trim().split(" ").length; i++) {
        words.push(false)
    }

    if (this.phrase === true) {

        let searchTerms = this.searchTerm.trim().toLowerCase()
        // Split the transcript into an array, each slot is a word
        let transcriptWords = this.transcript.trim().split(" ")
        let phraseLength = searchTerms.split(" ").length

        let matchTerm = searchTerms
        for (let w = 0; w < transcriptWords.length; w++) {
            let word = ""
            // Build word
            for (let i = 0; i < phraseLength; i++) {
                if (w + i >= transcriptWords.length) break
                word += transcriptWords[w + i].toLowerCase()
                if (i < phraseLength - 1) word += " "
            }
            // Check if build matches search term
            if (word.match(`^${matchTerm}[.,!?:-_]?$`)) {
                for (let i = 0; i < phraseLength; i++) {
                    words[w + i] = true
                }
            }
        }

        let result = ""
        for (let i = 0; i < transcriptWords.length; i++) {
            if (words[i] === true) result += ` <b>${transcriptWords[i]}</b>`
            else result += " " + transcriptWords[i]
        }

        this.bolded = result

    } else {

        let searchTerms = this.searchTerm.trim().toLowerCase().split(" ")
        let transcriptWords = this.transcript.trim().split(" ")


        for (let st = 0; st < searchTerms.length; st++) {
            let matchTerm = searchTerms[st]

            for (let w = 0; w < transcriptWords.length; w++) {
                let word = transcriptWords[w].toLowerCase()
                if (word.match(`^${matchTerm}[.,!?:-_]?$`)) words[w] = true
            }
        }

        let result = ""
        for (let i = 0; i < transcriptWords.length; i++) {
            if (words[i] === true) result += ` <b>${transcriptWords[i]}</b>`
            else result += " " + transcriptWords[i]
        }

        this.bolded = result
    }

  },
  data: () => ({
      bolded: ""
  }),
}
</script>

<style scoped>
* {
    font-style: italic;
    font-size: 14px;
    color: rgb(114, 114, 114);
}
p {
    padding: 15px;
}

p::after {
    content: " ...";
}
</style>