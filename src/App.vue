<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <div class="decibel-loudness-container">
    <p id="decibel-text">Pressione Começar para medir o volume.</p>
    <button @click="getLocalStream">Começar</button>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {

  },
  methods: {
    async getLocalStream() {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
      const audioContext = new AudioContext();
      const mediaStreamAudioSourceNode = audioContext.createMediaStreamSource(stream);
      const analyserNode = audioContext.createAnalyser();
      mediaStreamAudioSourceNode.connect(analyserNode);

      const pcmData = new Float32Array(analyserNode.fftSize);
      const onFrame = () => {
          analyserNode.getFloatTimeDomainData(pcmData);
          let sumSquares = 0.0;
          for (const amplitude of pcmData) { sumSquares += amplitude*amplitude; }
          document.getElementById("decibel-text").innerText = Math.round(20 * Math.log(Math.sqrt(sumSquares / pcmData.length)) + 136) + 'dB';
          setTimeout(() => {
            window.requestAnimationFrame(onFrame);  
          }, 100);
      };
      window.requestAnimationFrame(onFrame);
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.decibel-loudness-container {
  border: 1px solid black;

  position: absolute;
  top: calc(50vh - 5rem);
  left: calc(50vw - 10rem);

  height: 10rem;
  width: 20rem;
}
</style>
