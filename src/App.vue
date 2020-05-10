<template>
  <div>
    <button @click="record" :class="['record', isRecording && 'active']">
      <span v-if="!isRecording" class="icon">Record</span>
      <span v-if="isRecording" class="icon">Stop</span>
    </button>
    <audio ref="audio" controls></audio>
  </div>
</template>

<script>
import RecordRTC from "recordrtc";

const recordOption = {
  type: "audio",
  bufferSize: 16384,
  mimeType: "audio/webm",
  recorderType: RecordRTC.StereoAudioRecorder,
  audioBitsPerSecond: 128000,
  numberOfAudioChannels: 2,
  // disble webRTC logs
  disableLogs: false
};

export default {
  data() {
    return {
      isRecording: false,
      hasMediaDeviceCaps: false,
      startedRecording: null,
      mediaRecorder: null,
      isEdge:
        !!navigator.userAgent.match(/Edge/i) &&
        (!!navigator.msSaveOrOpenBlob || !!navigator.msSaveBlob),
      isSafari: !!navigator.userAgent.match(/Safari/i),
      isIos: !!navigator.userAgent.match(/iPhone|iPad|iPod/i)
    };
  },
  mounted() {
    this.grantPermissions();
  },
  methods: {
    async grantPermissions() {
      try {
        await navigator.mediaDevices.getUserMedia({ audio: true });
        this.hasMediaDeviceCaps = true;
      } catch (err) {
        // could be used to show a hint that the device/browser does not support WebRTC.
        // should potentially never be the case but better be safe
        this.hasMediaDeviceCaps = false;
      }
    },
    async record() {
      if (!this.isRecording) {
        try {
          const stream = await navigator.mediaDevices.getUserMedia({
            audio: {
              echoCancellation: true
            }
          });

          if (this.isSafari) {
            recordOption.sampleRate = 44100;
            recordOption.bufferSize = 4096;
          }

          this.mediaRecorder = new RecordRTC(stream, recordOption);
          this.mediaRecorder.startRecording();
          this.startedRecording = performance.now();
          this.isRecording = true;
        } catch (error) {
          console.debug(error);
        }
      } else {
        if (this.mediaRecorder) {
          this.mediaRecorder.stopRecording(this.handleDataAvailable);
          this.isRecording = false;
        }
      }
    },
    async handleDataAvailable() {
      if (this.mediaRecorder) {
        const blob = await this.mediaRecorder.getBlob();
        this.$refs.audio.src = URL.createObjectURL(blob);
      }
    }
  }
};
</script>

<style lang="postcss">
:root {
  --red: #df1032;
  --blue: #256cff;
  --teal: #22e5a0;
  --grey: #161618;
  --border-radius: 0.5em;
}
.record {
  width: 90px;
  height: 90px;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 90px;
  border: 2px solid var(--blue);
  transition: background 120ms ease-in-out, transform 120ms ease-in-out;

  &:hover {
    background: var(--blue);
  }
  &:focus {
    outline: none;
  }
  &.active {
    background: var(--red);
    transform: scale(1.05);
    animation: none;
  }
}
</style>
