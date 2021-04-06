<template>
  <v-layout column justify-center align-center>
    <v-col cols="12" md="6">
      <h2>Enter words separated by a space</h2>
      <v-textarea
        v-model="wordInput"
        solo
        label="Input your list of words, space separated. (apple orange mango)"
      ></v-textarea>
      <div v-if="wordOutpt.length > 0" class="display-block pa-6 display-2">
        <p>{{ wordOutpt }}</p>
      </div>
      <v-btn :disabled="calculating" color="primary" @click="generate()">
        Mash
      </v-btn>
      <v-btn plain @click="numSyllable = (numSyllable % 5) + 1">
        Syllables {{ numSyllable }}
      </v-btn>
      <div>
        <v-chip
          v-for="(word, index) in wordsTried"
          :key="index"
          class="ma-2"
          color="default"
        >
          {{ word }}
        </v-chip>
      </div>
    </v-col>
  </v-layout>
</template>

<script>
import syllable from 'syllable'
export default {
  data() {
    return {
      calculating: false,
      numSyllable: 2,
      wordsTried: [],
      faveWords: [],
      wordOutpt: '',
      wordInput:
        'apple arugula asparagus artichoke avocado banana blueberry broccoli carrot celery cherry eggplant figs grape grapefruit guava garlic ginger kale kiwi lemon mushroom melon mango nectarine nuts orange onion pepper pineapple pear peach raisin raspberry radish lettuce spinach squash sprouts strawberry tangerine watermelon yam zucchini'
    }
  },
  computed: {
    wordList() {
      return this.wordInput
        .toLowerCase()
        .replace(/[^a-zA-Z ]+/gi, '')
        .split(' ')
    },
    startingLetterList() {
      // [a,a,b,b,a,b]
      return this.wordList
        .filter((item) => item.length >= 3)
        .map((item) => {
          return item.charAt(0)
        })
    },
    startingLetterProbability() {
      // {sum: 10, a: 5, b: 5}
      return this.startingLetterList.reduce(
        (a, v) => {
          a.sum++
          if (a[v]) a[v]++
          else a[v] = 1
          return a
        },
        { sum: 0 }
      )
    },
    endingLetterList() {
      return this.wordList
        .filter((item) => item.length >= 3)
        .map((item) => {
          return item.slice(-1)
        })
    },
    endingLetterProbability() {
      // {sum: 10, a: 5, b: 5}
      return this.endingLetterList.reduce(
        (a, v) => {
          a.sum++
          if (a[v]) a[v]++
          else a[v] = 1
          return a
        },
        { sum: 0 }
      )
    },
    letterMap() {
      const network = Object.create(null)
      this.wordList
        .map((item) => '@#' + item + '$%')
        .forEach((item) => {
          let i = 1
          while (i < item.length - 1) {
            const l = item.charAt(i - 1)
            const m = item.charAt(i)
            const r = item.charAt(i + 1)

            // buid network
            if (!network[l + m]) network[l + m] = {}
            if (!network[l + m][r]) network[l + m][r] = 0
            network[l + m][r]++

            // inc
            i++
          }
        })
      return network
    }
  },
  methods: {
    selectThirdLetter(a, b) {
      if (!this.letterMap[a + b]) return false
      const arr = []
      let maxWeight = 0
      for (const [key, value] of Object.entries(this.letterMap[a + b])) {
        maxWeight += value
        arr.push([key, maxWeight])
      }
      const selectLetter = Math.random() * maxWeight
      for (let i = 0; i < arr.length; i++) {
        if (arr[i][1] >= selectLetter) return arr[i][0]
      }
      return false
    },
    buildWord(stack = 1000) {
      // max word rebuilds
      if (stack === 0) return '!'
      let word = '@#'
      let limit = 20
      let acceptable = true
      while (limit > 0) {
        const a = word.charAt(word.length - 2)
        const b = word.charAt(word.length - 1)
        const newLetter = this.selectThirdLetter(a, b)
        if (newLetter === false) break
        if (newLetter === '$') break
        word += newLetter
        limit--
      }
      word = word.replace('@#', '')
      word = word.charAt(0).toUpperCase() + word.slice(1)
      if (
        this.wordList.includes(word.toLowerCase()) ||
        this.wordsTried.includes(word) ||
        syllable(word) !== this.numSyllable ||
        word.length < 4
      ) {
        acceptable = false
      }
      if (acceptable) return word
      return this.buildWord(stack - 1)
    },
    generate() {
      this.calculating = true
      setTimeout(() => {
        this.wordOutpt = this.buildWord()
        this.wordsTried.push(this.wordOutpt)
        this.calculating = false
      }, 200)
    },
    countSyllable(word) {
      return syllable(word)
    }
  }
}
</script>
