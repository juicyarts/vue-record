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
export default {
  data() {
    return {
      isRecording: false,
      startedRecording: null,
      mediaRecorder: null,
    };
  },
  mounted() {
    this.grantPermissions();
  },
  methods: {
    grantPermissions() {
      let t = this;
      navigator.mediaDevices
        .getUserMedia({ audio: true })
        .then(function(stream) {})
        .catch(function(err) {});
    },
    record() {
      if (!this.isRecording) {
        this.isRecording = true;
        navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
          var options = {
            audioBitsPerSecond: 128000,
            mimeType: "audio/webm;codecs=opus",
          };
          this.mediaRecorder = new MediaRecorder(stream, options);
          this.mediaRecorder.ondataavailable = this.handleDataAvailable;
          this.mediaRecorder.start();
          this.startedRecording = performance.now();
        });
      } else {
        this.isRecording = false;
        this.mediaRecorder.stop();
        this.mediaRecorder.stream.getTracks().forEach((i) => i.stop());
      }
    },
    handleDataAvailable(e) {
      this.$refs.audio.src = URL.createObjectURL(e.data);
    },
  },
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
