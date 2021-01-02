<template>
  <a-card :title="beatmap.title" style="width: 300px">
    <a
      slot="extra"
      :href="'https://cadenzavr.com/?beatmap=' + beatmap.fileUrl"
      title="Play in Cadenza"
    >
      <a-icon type="caret-right" />
    </a>
    <a slot="extra" :href="beatmap.fileUrl">
      <a-icon type="download" />
    </a>
    <a-icon
      class="play-icon"
      type="play-circle"
      theme="filled"
      :style="{
        'background-image': 'url(' + beatmap.imageUrl + ')',
      }"
      @click="playAudioPreview"
    />
    <a-descriptions class="card" :column="1">
      <a-descriptions-item label="Artist">{{
        beatmap.artist
      }}</a-descriptions-item>
      <a-descriptions-item label="Creator">{{
        beatmap.creator
      }}</a-descriptions-item>
      <a-descriptions-item label="Difficulties">
        <div
          v-for="(mode, index) in Object.keys(beatmap.difficulties)"
          :key="index"
        >
          {{ modes[mode] }}:
          <a-tag
            v-for="(difficulty, difficultyIndex) in beatmap.difficulties[mode]"
            :key="difficultyIndex"
            :color="getDifficultyColor(mode, difficulty[1])"
            >{{ difficulty[0] }}</a-tag
          >
        </div>
      </a-descriptions-item>
      <a-descriptions-item label="Tags">
        <small>
          {{ beatmap.tags.join(' ') }}
        </small>
      </a-descriptions-item>
    </a-descriptions>
  </a-card>
</template>

<script>
export default {
  props: {
    beatmap: {
      type: Object,
      default: null,
    },
    audio: {
      type: HTMLAudioElement,
      default: null,
    },
  },

  data() {
    const MODES = Object.freeze({
      7: 'cadenza',
      1: 'osu!taiko',
      3: 'osu!mania',
    })
    return {
      modes: MODES,
    }
  },

  methods: {
    playAudioPreview() {
      this.audio.src = this.beatmap.audioPreviewUrl
      this.audio.play()
    },

    getDifficultyColor(mode, difficulty) {
      if (mode === '1') {
        if (difficulty >= 7.5) {
          return 'purple'
        } else if (difficulty >= 5) {
          return 'red'
        } else if (difficulty >= 2.5) {
          return 'orange'
        } else {
          return 'green'
        }
      }
    },
  },
}
</script>

<style>
.ant-descriptions-row > td {
  padding-bottom: 0;
}
.play-icon {
  font-size: 64px;
  width: 128px;
  height: 128px;
  background-size: auto 100%;
  background-position: center;
  cursor: pointer;
}
.play-icon > svg {
  position: relative;
  top: 32px;
  left: 0;
}
</style>
