<script setup lang="ts">
import { computed, onMounted, onBeforeUnmount, ref } from 'vue'

type Track = {
  title: string
  artist: string
  src: string
  cover: string
  color: string
}

const tracks: Track[] = [
  {
    title: '夜航星',
    artist: '不才',
    src: 'https://music.163.com/song/media/outer/url?id=1810269641.mp3',
    cover: 'https://p2.music.126.net/DEZ3_yJ5Q7eO7s0D9A6azg==/109951165632303043.jpg?param=600y600',
    color: '#9e8cfc',
  },
  {
    title: 'Cornfield Chase',
    artist: 'Hans Zimmer',
    src: 'https://music.163.com/song/media/outer/url?id=29750099.mp3',
    cover: 'https://p1.music.126.net/s6RcGoE7gQJcSGSdmYvJAA==/109951165626331175.jpg?param=600y600',
    color: '#7dd3fc',
  },
  {
    title: 'Intro',
    artist: 'The xx',
    src: 'https://music.163.com/song/media/outer/url?id=31598603.mp3',
    cover: 'https://p2.music.126.net/xuAoPHolxOqOM-2mqZNUHw==/109951163296734227.jpg?param=600y600',
    color: '#f0abfc',
  },
]

const audio = ref<HTMLAudioElement | null>(null)
const currentIndex = ref(0)
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)
const volume = ref(0.72)
const isReady = ref(false)

const currentTrack = computed(() => tracks[currentIndex.value])
const progress = computed(() => (duration.value ? (currentTime.value / duration.value) * 100 : 0))
const appStyle = computed(() => ({ '--accent': currentTrack.value.color }))

const formatTime = (seconds: number) => {
  if (!Number.isFinite(seconds)) return '00:00'
  const min = Math.floor(seconds / 60)
  const sec = Math.floor(seconds % 60)
  return `${String(min).padStart(2, '0')}:${String(sec).padStart(2, '0')}`
}

const syncAudioSource = () => {
  if (!audio.value) return
  audio.value.src = currentTrack.value.src
  audio.value.volume = volume.value
  audio.value.load()
}

const play = async () => {
  if (!audio.value) return
  try {
    await audio.value.play()
    isPlaying.value = true
  } catch {
    isPlaying.value = false
  }
}

const pause = () => {
  audio.value?.pause()
  isPlaying.value = false
}

const togglePlay = () => {
  if (isPlaying.value) pause()
  else play()
}

const selectTrack = async (index: number) => {
  currentIndex.value = index
  syncAudioSource()
  currentTime.value = 0
  if (isPlaying.value) await play()
}

const previous = () => {
  const nextIndex = currentIndex.value === 0 ? tracks.length - 1 : currentIndex.value - 1
  selectTrack(nextIndex)
}

const next = () => {
  const nextIndex = (currentIndex.value + 1) % tracks.length
  selectTrack(nextIndex)
}

const seek = (event: MouseEvent) => {
  if (!audio.value || !duration.value) return
  const bar = event.currentTarget as HTMLElement
  const rect = bar.getBoundingClientRect()
  const ratio = Math.min(1, Math.max(0, (event.clientX - rect.left) / rect.width))
  audio.value.currentTime = ratio * duration.value
}

const setVolume = (event: Event) => {
  const target = event.target as HTMLInputElement
  volume.value = Number(target.value)
  if (audio.value) audio.value.volume = volume.value
}

const onKeydown = (event: KeyboardEvent) => {
  if (event.code === 'Space') {
    event.preventDefault()
    togglePlay()
  }
  if (event.code === 'ArrowLeft') previous()
  if (event.code === 'ArrowRight') next()
}

onMounted(() => {
  syncAudioSource()
  window.addEventListener('keydown', onKeydown)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKeydown)
})
</script>

<template>
  <main class="music-shell" :style="appStyle">
    <div class="ambient ambient-a"></div>
    <div class="ambient ambient-b"></div>
    <div class="ambient ambient-c"></div>

    <section class="player-card">
      <header class="top-bar">
        <div class="brand">
          <span class="status-dot"></span>
          <div>
            <p>Yaoxi Music</p>
            <span>Glass Player</span>
          </div>
        </div>
        <div class="kbd">Space 播放 / 暂停</div>
      </header>

      <section class="hero">
        <div class="disc-wrap" :class="{ playing: isPlaying }">
          <div class="disc-glow"></div>
          <img :src="currentTrack.cover" :alt="currentTrack.title" class="cover" />
        </div>

        <div class="track-panel">
          <p class="eyebrow">Now Playing</p>
          <h1>{{ currentTrack.title }}</h1>
          <p class="artist">{{ currentTrack.artist }}</p>

          <div class="progress-meta">
            <span>{{ formatTime(currentTime) }}</span>
            <span>{{ formatTime(duration) }}</span>
          </div>
          <button class="progress" @click="seek" aria-label="调整播放进度">
            <span :style="{ width: `${progress}%` }"></span>
          </button>

          <div class="controls">
            <button @click="previous" aria-label="上一首">‹</button>
            <button class="play" @click="togglePlay" aria-label="播放或暂停">
              {{ isPlaying ? 'Ⅱ' : '▶' }}
            </button>
            <button @click="next" aria-label="下一首">›</button>
          </div>
        </div>
      </section>

      <section class="queue-card">
        <div class="queue-head">
          <div>
            <p>Playlist</p>
            <span>{{ tracks.length }} tracks</span>
          </div>
          <label class="volume">
            Vol
            <input type="range" min="0" max="1" step="0.01" :value="volume" @input="setVolume" />
          </label>
        </div>

        <button
          v-for="(track, index) in tracks"
          :key="track.title"
          class="track-row"
          :class="{ active: index === currentIndex }"
          @click="selectTrack(index)"
        >
          <img :src="track.cover" :alt="track.title" />
          <span class="track-text">
            <strong>{{ track.title }}</strong>
            <small>{{ track.artist }}</small>
          </span>
          <span class="track-state">{{ index === currentIndex && isPlaying ? 'ON' : 'PLAY' }}</span>
        </button>
      </section>
    </section>

    <audio
      ref="audio"
      preload="metadata"
      @loadedmetadata="duration = audio?.duration || 0; isReady = true"
      @timeupdate="currentTime = audio?.currentTime || 0"
      @ended="next"
      @play="isPlaying = true"
      @pause="isPlaying = false"
    ></audio>
  </main>
</template>
