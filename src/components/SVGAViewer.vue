<template>
  <div class="viewer-container" @dragenter.stop.prevent @dragover.stop.prevent @drop.stop.prevent="dropFile">
    <div class="toolbar">
      <mu-text-field v-model="url"></mu-text-field>
      <mu-button color="primary" @click="urlChange">Fetch</mu-button>
      <mu-button color="primary" @click="clickFileInput">Upload</mu-button>
      <mu-button color="primary" @click="enterPlayerMode">Player</mu-button>
      <mu-button color="primary" @click="leavePlayerMode">Source</mu-button>
      <mu-switch v-model="HDPIMode" label="HDPI Mode"></mu-switch>
      <mu-text-field v-model="backgroundColor" style="margin-left: 8px;width: 80px"></mu-text-field>
    </div>
    <template v-if="playerMode">
      <mu-paper class="svga-info list" :z-depth="2">
        <mu-list v-if="videoItem">
          <mu-list-item>
            <mu-list-item-title>version</mu-list-item-title>
            <mu-list-item-action>{{videoItem.version}}</mu-list-item-action>
          </mu-list-item>
          <mu-list-item>
            <mu-list-item-title>FPS</mu-list-item-title>
            <mu-list-item-action>{{videoItem.FPS}}</mu-list-item-action>
          </mu-list-item>
          <mu-list-item>
            <mu-list-item-title>frames</mu-list-item-title>
            <mu-list-item-action>{{videoItem.frames}}</mu-list-item-action>
          </mu-list-item>
          <mu-list-item>
            <mu-list-item-title>width</mu-list-item-title>
            <mu-list-item-action>{{videoItem.videoSize.width}}</mu-list-item-action>
          </mu-list-item>
          <mu-list-item>
            <mu-list-item-title>height</mu-list-item-title>
            <mu-list-item-action>{{videoItem.videoSize.height}}</mu-list-item-action>
          </mu-list-item>
        </mu-list>
        <div class="player-control" v-if="videoItem">
          <mu-button icon color="primary" @click="playSVGA">
            <mu-icon value="play_arrow"></mu-icon>
          </mu-button>
          <mu-button icon color="primary" @click="pauseSVGA">
            <mu-icon value="pause"></mu-icon>
          </mu-button>
        </div>
      </mu-paper>
      <mu-paper class="svga-player info" :z-depth="2" :style="infoBackground">
        <div class="player" ref="player"></div>
      </mu-paper>
    </template>
    <template v-else>
      <mu-paper class="image-list list" :z-depth="2">
        <mu-list v-if="videoItem">
          <mu-list-item v-for="(item, key) in videoItem.images" :key="key" @click.native="selectSource(key)">
            <mu-list-item-title>{{ key }}</mu-list-item-title>
          </mu-list-item>
        </mu-list>
      </mu-paper>
      <mu-paper class="image-preview info" :z-depth="2" :style="infoBackground">
        <p v-if="sourceWidth || sourceHeight">width: {{sourceWidth}} height: {{sourceHeight}}</p>
        <img ref="sourceImage" :src="sourceImageSrc" @load="updateSourceImageSize">
      </mu-paper>
    </template>
  </div>
</template>

<script>
import SVGA from 'svgaplayerweb'

export default {
  data () {
    return {
      backgroundColor: '#fff',
      fileSelector: null,
      player: null,
      url: '',
      source: '',
      videoItem: null,
      HDPIMode: false,
      playerMode: true,
      selectedSourceId: '',
      sourceWidth: 0,
      sourceHeight: 0
    }
  },
  computed: {
    infoBackground () {
      return {
        backgroundColor: this.backgroundColor
      }
    },
    sourceImageSrc () {
      return this.videoItem && this.videoItem.images[this.selectedSourceId] ? `data:image/png;base64,${this.videoItem.images[this.selectedSourceId]}` : ''
    }
  },
  watch: {
    playerMode (val, old) {
      if (val !== old) {
        if (val) {
          this.initPlayer()
        }
      }
    },
    HDPIMode () {
      if (this.playerMode) {
        this.initPlayer()
      }
    },
    source () {
      if (this.playerMode) {
        this.initPlayer()
      }
    }
  },
  mounted () {
    let input = document.createElement('input')
    input.type = 'file'
    input.accept = '.svga'
    input.onchange = () => {
      this.selectFile()
    }
    this.fileSelector = input
  },
  methods: {
    clickFileInput () {
      this.fileSelector.click()
    },
    enterPlayerMode () {
      this.playerMode = true
    },
    leavePlayerMode () {
      this.playerMode = false
    },
    togglePlayerMode () {
      this.playerMode = !this.playerMode
    },
    toggleHDPIMode () {
      this.HDPIMode = !this.HDPIMode
      this.initPlayer()
    },
    urlChange () {
      if (this.url) {
        this.source = this.url
      }
    },
    dropFile (e) {
      let data = e.dataTransfer
      this.handleFileList(data.files)
    },
    selectFile () {
      let input = this.fileSelector
      this.handleFileList(input.files)
    },
    handleFileList (fileList) {
      let file = null
      for (let i = 0; i < fileList.length; i++) {
        if (fileList[i].name.includes('.svga')) {
          file = fileList[i]
          break
        }
      }
      if (file) {
        let reader = new FileReader()
        reader.readAsDataURL(file)
        reader.onload = () => {
          this.source = reader.result
        }
      }
    },
    setPlayerSize () {
      let dpr = this.HDPIMode ? (window.devicePixelRatio || 1) : 1
      this.$refs.player.style.width = `${this.videoItem.videoSize.width / dpr}px`
      this.$refs.player.style.height = `${this.videoItem.videoSize.height / dpr}px`
    },
    initPlayer () {
      let parser = new SVGA.Parser()
      parser.load(this.source, (videoItem) => {
        this.videoItem = videoItem
        this.player = new SVGA.Player(this.$refs.player)
        this.setPlayerSize()
        this.$nextTick(() => {
          this.player.setVideoItem(videoItem)
          this.playSVGA()
        })
      })
    },
    playSVGA () {
      this.player.startAnimation()
    },
    pauseSVGA () {
      this.player.pauseAnimation()
    },
    selectSource (id) {
      this.selectedSourceId = id
    },
    updateSourceImageSize () {
      let image = this.$refs.sourceImage
      this.sourceWidth = image.width
      this.sourceHeight = image.height
    }
  }
}
</script>

<style lang="stylus">
.viewer-container
  height 100vh
  padding 10px
  display grid
  grid-gap 10px
  grid-template-areas "toolbar toolbar" "list info"
  grid-template-rows 64px 1fr
  grid-template-columns 180px 1fr
  overflow hidden
.list
  grid-area list
.info
  grid-area info
.toolbar
  grid-area toolbar
  .mu-input, .mu-button
    margin-right 8px
.mu-item
  &:hover
    background-color #e0e0e0
.svga-player
  overflow auto
  .player
    margin 0 auto
.image-list, .image-preview
  overflow auto
</style>
