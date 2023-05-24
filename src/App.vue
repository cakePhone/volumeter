<template>
  <div class="main-container">

    <h1 class="text">Poluição Sonora e as Suas Consequências</h1>

    <div class="content-card info-card">
      <h2 class="card-name text">Poluição Sonora</h2>
      <p class="content-text text">(Trabalho pendente)Lorem ipsum, dolor sit amet consectetur adipisicing elit. At numquam facere nesciunt iure voluptatem ipsam animi assumenda modi, tenetur fugiat. Incidunt est repellendus, eligendi recusandae obcaecati alias commodi aliquid vel at, quasi nobis tempore officia molestias? Veniam, debitis sint? Esse omnis nam minima laboriosam facilis nesciunt exercitationem laudantium deleniti! Fugit quia porro facere iste, labore, dolores officia harum et eveniet voluptas odit inventore voluptates sit rem magnam? Ipsum minima impedit officiis repudiandae qui optio quasi sunt aliquid maiores tenetur fugit at inventore, eum aperiam accusantium. Aperiam provident architecto ducimus? Reiciendis aliquid dolores eveniet minima omnis nulla voluptas architecto alias eum!</p>
    </div>
    
    <div class="content-card">
      <h2 class="card-name text">Saúde Auditiva</h2>
      <p class="content-text text">Meça a sua saúde auditiva a partir deste teste rápido. Quando estiver pronto, clique no vídeo abaixo para começar. Quando começar a ouvir a frequência sonora, pause e veja o seu resultado.<br>
                                   (É recomendado o uso de fones de ouvido e o volume no máximo)
      </p>
      <iframe src="https://www.youtube-nocookie.com/embed/-Mm3avgcXDw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"></iframe>
      <p class="info-text text">Créditos ao autor do vídeo.</p>
    </div>

    <div id="volume-measure-card" class="content-card">
      <h2 class="card-name text">Barulhómetro</h2>
      <p class="content-text text">Meça a intensidade do barulho captado pelo dispositivo.<br>
                                   Muito barulho ao ser redor por extensos periodos de tempo podem prejudicar a sua audição.
      </p>
      <div class="meter-container">
        <meter id="volume-bar" max="0.7" :value="volumeBarValue"></meter>
        <p class="info-text text">Pressione Começar para medir o volume.</p>
      </div>
      <button @click="toggleRecord()">{{ isRecording ? 'Parar' : 'Começar' }}</button>
    </div>

    <!-- <div class="content-card">
      <h2 class="card-name text">Créditos</h2>
      <p class="content-text text">Lorem ipsum dolor sit, amet consectetur adipisicing elit. Magni, ab!</p>
    </div> -->

  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      isRecording: false,
      volumeBarValue: null,
    }
  },
  methods: {
    toggleRecord() {
      if(this.isRecording) {
        this.isRecording = false;
      } else {
        this.getLocalStream();
      }
    },

    async getLocalStream() {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false })
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();
      const mediaStreamAudioSourceNode = audioContext.createMediaStreamSource(stream);
      const analyserNode = audioContext.createAnalyser();
      mediaStreamAudioSourceNode.connect(analyserNode);

      this.isRecording = true;

      const pcmData = new Float32Array(analyserNode.fftSize);
      const onFrame = () => {
        if(this.isRecording) {
          analyserNode.getFloatTimeDomainData(pcmData);
          let sumSquares = 0.0;
          for (const amplitude of pcmData) { sumSquares += amplitude*amplitude; }
          this.volumeBarValue = Math.sqrt(sumSquares / pcmData.length);
          
          window.requestAnimationFrame(onFrame);
        } else {
          this.volumeBarValue = null;
        }
      };
      window.requestAnimationFrame(onFrame);
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
  width: fit-content;
  background-color: var(--background);
}

#app {
  display: flex;
  justify-content: center;

  height: fit-content;
  width: 100%;
}

h1 {
  margin-top: 2.5rem;
  
  font-weight: 400;
  font-size: 2.5rem;
  text-align: left;
  color: var(--on-background);
  
  width: 100%;
}

.main-container {
  height: fit-content;
  width: calc(100% - 4rem);

  display: flex;
  flex-wrap: wrap;
  justify-content: left;
}

.content-card {
  position: relative;
  
  height: fit-content;
  width: min(25rem, calc(100% - 2rem));
  
  display: flex;
  align-items: center;
  flex-direction: column;
  
  padding: 1.25rem;
  
  border-radius: 1rem;
  background: var(--secondary-container);
  
  box-shadow: 0px 5px 5px #00000076;

  margin-bottom: 2rem;
}

.info-card {
  width: 100%;
}

@media screen and (min-width: 600px) {
  .main-container {
    width: calc(100% - 2rem);
    margin-left: 2rem;
  }
  
  .content-card {
    margin-right: 2rem;
  }
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
  width: 100%;

  font-size: 1.1rem;
  line-height: 1.6rem;

  margin: 0;
}

.text {
  text-shadow: 0px 2px 1px #00000036;
}

#volume-measure-card {
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

.meter-container {
  display: flex;
  flex-direction: column;
  align-items: center;

  margin: 1rem 0;

  width: 100%;
}

iframe {
  height: 15rem;
  width: 100%;

  margin-top: 1rem;

  border-radius: 1rem;

  box-shadow: 0px 5px 5px #00000036
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