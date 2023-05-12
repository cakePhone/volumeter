<template>
  <h1 class="text">Volumeter</h1>
  <div class="main-container">
    <div id="volume-measure-card" class="content-card">
      <h2 class="card-name text">Barulhómetro</h2>
      <p class="content-text text">Meça a intensidade do barulho captado pelo dispositivo</p>
      <div id="meter-container">
        <meter id="volume-bar" max="0.7" value="0"></meter>
        <p id="volume-text" class="info-text text">Pressione Começar para medir o volume.</p>
      </div>
      <button @click="getLocalStream">Começar</button>
    </div>

    <div class="content-card">
      <h2 class="card-name text">Saúde Auditiva</h2>
      <p class="content-text text">Meça a sua saúde auditiva a partir deste teste rápido. Quando estiver pronto, clique começar. Quando parar de ouvir a frequência sonora, clique parar. (É recomendado o uso de fones de ouvido e o volume no máximo)</p>
      <div id="test-container"></div>
      <button @click="!playingAudioTest; playNote()">Começar</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      count: 3,
      frequency: 200,
      playingAudioTest: false,
    }
  },
  methods: {
    async getLocalStream() {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false })
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
      
      let counter = setInterval(() => {
        if(this.count < 1) { document.getElementById("volume-text").innerText = "Pronto!"; clearInterval(counter); }
        else { document.getElementById("volume-text").innerHTML = 'Espere pelo buffer: ' + this.count; }
        this.count = this.count - 1;
      }, 1000);
    },

    playNote() {
      let audioCtx = new(window.AudioContext || window.webkitAudioContext)();

      let oscillator = audioCtx.createOscillator()
      let frequencyTest = setInterval(() => {
        oscillator.type = 'sine';
        oscillator.frequency.value = this.frequency; // value in hertz
        oscillator.connect(audioCtx.destination);
        if(!this.playingAudioTest) { clearInterval(frequencyTest); return } else { oscillator.start(); }
        oscillator.stop(1)
        
        this.frequency++;
      }, 1001);
    }
  }
}
</script>

<style>
:root {
  --primary: #20e19c;
  --on-primary: #003824;
  --primary-container: #005235;
  --on-primary-container: #51feb7;

  --secondary: #b4ccbc;
  --on-secondary: #20352a;
  --secondary-container: #364b3f;
  --on-secondary-container: #d0e8d8;

  --tertiary: #a4cdde;
  --on-tertiary: #063543;
  --tertiary-container: #234c5a;
  --on-tertiary-container: #c0e9fa;

  --background: #191c1a;
  --on-background: #e1e3df;
  
  --outline: #8a938c;
  --surface-variant: #404943;
  --on-surface-variant: #c0c9c1;

  font-family: 'Roboto';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

html, body {
  margin: 0;
  background-color: var(--background);
}

#app {
  display: flex;
  flex-direction: column;
  align-items: left;

  padding: 2rem;

  height: calc(100vh - 4rem);
  width: calc(100vw - 4rem);
}

h1 {
  font-weight: 400;
  font-size: 2.5rem;
  text-align: left;
  color: var(--on-background);
  
  width: 100%;
}

.main-container {
  height: fit-content;
  width: fit-content;

  display: flex;
  flex-wrap: wrap;
  justify-content: left;
}

.content-card {
  position: relative;

  height: fit-content;
  min-width: min(310px, 90vw);
  max-width: min(310px, 90vw);
  
  display: flex;
  align-items: center;
  flex-direction: column;

  padding: 1.25rem;
  
  border-radius: 1rem;
  background: var(--secondary-container);

  box-shadow: 0px 5px 5px #00000076;

  margin: 1rem 1rem 1rem 0;
}

.card-name {
  color: var(--on-secondary-container);

  font-weight: 400;
  font-size: 1.5rem;

  width: 100%;

  margin-top: 0.2rem;
}

.content-text {
  color: var(--on-secondary-container);
  height: fit-content;
  width: fit-content;

  margin: 0;
}

.text {
  text-shadow: 0px 2px 1px #00000036;
}

#volume-measure-card {
  width: 310px;
  margin-left: 0;
}

button {
  height: 40px;

  position: relative;
  bottom: 0;
  
  font-family: 'Roboto';
  font-weight: 450;
  font-size: 15px;
  color: var(--on-primary);

  margin-left: calc(100% - 90px);

  border: none;
  border-radius: 2rem;

  background-color: var(--primary);

  padding: 0 20px;

  box-shadow: 0px 2px 2px #00000036;

  transition: all 200ms ease;
}

button:active {
  box-shadow: none;
}

button:hover {
  box-shadow: 0px 2px 5px #00000076;
  bottom: 0.1rem;
}

#meter-container {
  display: flex;
  flex-direction: column;
  align-items: center;

  margin: 1rem 0;

  width: 100%;
}

meter {
  width: 100%;
}

meter::-webkit-meter-bar {
  background: var(--surface-variant);
}

meter::-webkit-meter-optimum-value {
  background: var(--primary);
}

.info-text {
  margin: 0.1rem;
  font-size: 0.8rem;
  color: var(--on-surface-variant);
}
</style>