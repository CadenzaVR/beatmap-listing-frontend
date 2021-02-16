<template>
  <div id="content">
    <div id="menu-bar">
      <a-form
        :label-col="{ lg: { span: 2 } }"
        :wrapper-col="{ lg: { span: 12 } }"
      >
        <a-form-item label="Genre">
          <a-select id="genre-select" mode="multiple" @change="genreChanged">
            <a-select-option v-for="(value, name) in genres" :key="value">{{
              name
            }}</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="Language">
          <a-select
            id="language-select"
            mode="multiple"
            @change="languageChanged"
          >
            <a-select-option v-for="(value, name) in languages" :key="value">{{
              name
            }}</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="Mode">
          <a-select id="mode-select" mode="multiple" @change="modeChanged">
            <a-select-option v-for="(value, name) in modes" :key="value">{{
              name
            }}</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="Difficulty">
          <a-select
            id="difficulty-select"
            mode="multiple"
            @change="difficultyChanged"
          >
            <a-select-option
              v-for="(value, name) in difficulties"
              :key="value"
              >{{ name }}</a-select-option
            >
          </a-select>
        </a-form-item>
      </a-form>
    </div>
    <a-pagination
      v-model="pageNumber"
      :total="beatmaps.length"
      :page-size="itemsPerPage"
      @change="pageChanged"
    />
    <div id="card-container">
      <beatmap-card
        v-for="beatmap in beatmaps.slice(
          (pageNumber - 1) * itemsPerPage,
          pageNumber * itemsPerPage
        )"
        :key="beatmap.id"
        :beatmap="beatmap"
        :audio="audio"
      ></beatmap-card>
    </div>
    <a-pagination
      v-model="pageNumber"
      :total="beatmaps.length"
      :page-size="itemsPerPage"
      @change="pageChanged"
    />
    <div id="footer"></div>
  </div>
</template>

<script>
import BeatmapCard from '~/components/BeatmapCard.vue'
export default {
  components: { BeatmapCard },
  async asyncData({ $axios, $config }) {
    const allBeatmaps = (await $axios.get('/beatmaps.json')).data
    const beatmaps = allBeatmaps
    return { allBeatmaps, beatmaps }
  },
  data() {
    const GENRES = Object.freeze({
      unspecified: 0,
      'video game': 1,
      anime: 2,
      rock: 3,
      pop: 4,
      novelty: 5,
      'hip hop': 6,
      electronic: 7,
      metal: 8,
      classical: 9,
      folk: 10,
      jazz: 11,
    })

    const LANGUAGES = Object.freeze({
      unspecified: 0,
      english: 1,
      chinese: 2,
      french: 3,
      german: 4,
      italian: 5,
      japanese: 6,
      korean: 7,
      spanish: 8,
      swedish: 9,
      russian: 10,
      polish: 11,
      instrumental: 12,
    })

    const MODES = Object.freeze({
      cadenza: 7,
      'osu!taiko': 1,
      'osu!mania': 3,
    })

    const DIFFICULTIES = Object.freeze({
      Easy: 0,
      Medium: 1,
      Hard: 2,
      Insane: 3,
    })

    return {
      audio: new Audio(),
      itemsPerPage: 20,
      pageNumber: 1,
      formLayout: 'horizontal',
      allBeatmaps: [],
      beatmaps: [],
      difficulties: DIFFICULTIES,
      genres: GENRES,
      languages: LANGUAGES,
      modes: MODES,
      filters: {
        genre: Object.values(GENRES).map(() => false),
        language: Object.values(LANGUAGES).map(() => false),
        mode: Object.values(MODES).map(() => false),
        difficulty: Object.values(DIFFICULTIES).map(() => false),
      },
    }
  },
  methods: {
    genreChanged(values) {
      this.filters.genre.fill(false)
      for (const value of values) {
        this.filters.genre[value] = true
      }
      this.pageNumber = 1
      this.updateResults()
    },

    languageChanged(values) {
      this.filters.language.fill(false)
      for (const value of values) {
        this.filters.language[value] = true
      }
      this.pageNumber = 1
      this.updateResults()
    },

    modeChanged(values) {
      this.filters.mode.fill(false)
      for (const value of values) {
        this.filters.mode[value] = true
      }
      this.pageNumber = 1
      this.updateResults()
    },

    difficultyChanged(values) {
      this.filters.difficulty.fill(false)
      for (const value of values) {
        this.filters.difficulty[value] = true
      }
      this.pageNumber = 1
      this.updateResults()
    },

    pageChanged(value) {
      this.pageNumber = value
    },

    getDifficulty(mode, value) {
      switch (mode) {
        case '1':
          if (value >= 7.5) {
            return 3
          } else if (value >= 5) {
            return 2
          } else if (value >= 2.5) {
            return 1
          } else {
            return 0
          }
      }
    },

    updateResults() {
      const isGenreFiltered = this.filters.genre.includes(true)
      const isLanguageFiltered = this.filters.language.includes(true)
      const isModeFiltered = this.filters.mode.includes(true)
      const isDifficultyFiltered = this.filters.difficulty.includes(true)

      this.beatmaps = this.allBeatmaps.filter((beatmap) => {
        if (isGenreFiltered && !this.filters.genre[beatmap.genre]) {
          return false
        }
        if (isLanguageFiltered && !this.filters.language[beatmap.language]) {
          return false
        }
        for (const mode in beatmap.difficulties) {
          if (!isModeFiltered || this.filters.mode[parseInt(mode)]) {
            const difficulties = beatmap.difficulties[mode]
            if (!isDifficultyFiltered) {
              return true
            }
            for (const difficulty of difficulties) {
              if (
                this.filters.difficulty[this.getDifficulty(mode, difficulty[1])]
              ) {
                return true
              }
            }
          }
        }
        return false
      })
    },
  },
}
</script>

<style>
#content {
  padding: 100px;
}

#footer {
  padding: 50px 0 50px 0;
}

#card-container {
  margin: 10px 0 10px 0;
  display: flex;
  flex-wrap: wrap;
}
</style>
