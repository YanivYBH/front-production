<template>
  <b-form-group :label="label" :label-for="name" :description="description" class="mb-3" label-class="small">
    <!-- <b-input-group :prepend="prepend" :append="append">
      <b-form-input :id="name" v-model="inputVal" :type="type" v-mask="mask" :state="errors[name] ? false : null"
        :placeholder="placeholder" :disabled="disabled" />

    </b-input-group> -->
    <div class="d-flex ">
      <b-form-checkbox id="checkbox-1" v-model="recMode" name="checkbox-1" value="press" unchecked-value="hold">
        <div>Recording mode: <strong>{{ recMode.toUpperCase() }}</strong></div>
      </b-form-checkbox>

      
    </div>
    <div class="d-flex py-3 align-items-center" style="">

      <VueRecordAudio :mode="recMode" @stream="onStream" @result="onResult" />
      <div class="px-3">

        <b-link @click.prevent="startPause">
          <i v-if="isAudioLoaded&&audioProps.isPlaying" class="bi-pause-fill" style="font-size: 3rem" />
          <i v-else-if="isAudioLoaded&&!audioProps.isPlaying" class="bi-play-fill" style="font-size: 3rem" />
        </b-link>
      </div>
      <div class="" style="flex: 1;background: white; box-shadow: 0px 5px 14px 1px #f0f0f0;">
        <vue-wave-surfer :key="key" :src="audioFile" :options="audioOptions" ref="surf"
          @hook:mounted="audioComponentMounted">
        </vue-wave-surfer>
      </div>
    </div>

    <b-form-invalid-feedback :id="name + '-errors'" class="d-block" v-if="errors[name] && errors[name].length > 0">
      <div v-for="error in errors[name]" :key="error">
        {{ error }}
      </div>
    </b-form-invalid-feedback>
  </b-form-group>
</template>
<style lang="scss">
.vue-audio-recorder {
  background-color: #00AFF0 !important;
}

.vue-audio-recorder:hover {
  background-color: #00AFF0 !important;

}

.vue-audio-recorder.active {
  background-color: #ef5350 !important;

}
</style>
<script>
import Cursor from 'wavesurfer.js/dist/plugin/wavesurfer.cursor';
export default {
  props: [
    "label",
    "name",
    "value",
    "errors",
    "type",
    "placeholder",
    "prepend",
    "append",
    "disabled",
    "mask",
    "description",
  ],
  data: function () {
    return {
      recMode: "press",
      audioOptions: {
        plugins: [
          Cursor.create()
        ],

        waveColor: '#C6C7C8',
        progressColor: "#00AFF0",
        barWidth: 4,
        height: 100,
        responsive: true,
        hideScrollbar: true,
        barRadius: 4
      },
      audioFile: "img/sample.mp3",
      audioProps: {
        duration: 0,
        currentTime: 0,
        isPlaying: false
      },
      isAudioLoaded: false,
      getCurrentTimeInterval: 0,
      key: 0

    }
  },
  computed: {
    inputVal: {
      get() {
        return this.value;
      },
      set(val) {
        this.$emit("input", val);
      },
    },
  },
  mounted() {

  },
  methods: {
    onResult(data) {
      console.log('The blob data:', data);
      console.log('Downloadable audio', window.URL.createObjectURL(data));
      this.audioFile = window.URL.createObjectURL(data)
      this.key = this.key + 1
    },
    onStream(data) {
      console.log('STREAM', data);

    },

    startPause() {
      if (this.$refs.surf.waveSurfer) {
        this.$refs.surf.waveSurfer.playPause()
        this.audioProps.isPlaying = this.$refs.surf.waveSurfer.isPlaying()
        if (this.audioProps.isPlaying) {
          this.getCurrentTimeInterval = setInterval(() => {
            this.audioProps.currentTime = this.$refs.surf.waveSurfer.getCurrentTime()
            console.log(this.audioProps.currentTime)
          }, 1000);
        }
        else {
          clearInterval(this.getCurrentTimeInterval)
          this.getCurrentTimeInterval = 0
        }
      }
    },
    audioComponentMounted() {
      this.isAudioLoaded = true;
      this.getAudioProps();
      setTimeout(() => {
        this.audioProps.duration = this.$refs.surf.waveSurfer.getDuration()
        this.$refs.surf.waveSurfer.on('finish', () => {
          this.$refs.surf.waveSurfer.stop()
          clearInterval(this.getCurrentTimeInterval)
          this.getCurrentTimeInterval = 0
          this.getAudioProps();
        })
      }, 1000);
    },
    secondToMin(second) {
      let parsedSec = Math.round(second)
      var minutes = Math.floor(parsedSec / 60);
      var seconds = parsedSec - minutes * 60;
      return `${minutes}:${seconds}`
    },

    getAudioProps() {
      this.audioProps.duration = this.$refs.surf.waveSurfer.getDuration()
      this.audioProps.currentTime = this.$refs.surf.waveSurfer.getCurrentTime()
      this.audioProps.isPlaying = this.$refs.surf.waveSurfer.isPlaying()
    }

  }
};
</script>
