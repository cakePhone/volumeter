<template>
  <div class="decibel-loudness-container">
    <p id="decibel-text">Pressione Começar para medir o volume.</p>
    <progress id="volume-bar" max="0.5" value="0"></progress>
    <button @click="getLocalStream">Começar</button>
  </div>
</template>

<script>
export default {
  name: 'App',
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
          document.getElementById("volume-bar").value = Math.sqrt(sumSquares / pcmData.length);
          
          window.requestAnimationFrame(onFrame);
      };
      window.requestAnimationFrame(onFrame);
      
      let count = 5;
      let counter = setInterval(() => {
        document.getElementById("decibel-text").innerHTML = 'Espere pelo buffer: ' + count;
        if(count === 0) { document.getElementById("decibel-text").innerText = "Pronto!"; clearInterval(counter); }
        console.log(count)
        count = count - 1;
      }, 1000);
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

  display: flex;
  justify-content: center;

  height: 100vh;
}

.decibel-loudness-container {
  border: 1px solid black;

  /* position: absolute;
  top: calc(50vh - 5rem);
  left: calc(50vw - 10rem); */

  height: 10rem;
  width: 20rem;
}
</style>