```html
<div class="bar-chart">
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
</div>
```


```js
const setAudioSource = (audioNode, src) => { audioNode.src = src }
const play = audioNode => audioNode.play()
const pause = audioNode => audioNode.pause()
const stop = audioNode => {
  audioNode.pause()
  setAudioSource(audioNode, '')
  audioNode.removeAttribute("src");
}

const getState = (audioNode) => {
  return audioNode.paused
}

const audioElement = new Audio()
const context = new AudioContext();
const source = context.createMediaElementSource(audioElement);
const analyser = context.createAnalyser();
source.connect(analyser);
analyser.connect(context.destination);

let frequencyData = new Uint8Array(analyser.frequencyBinCount);

const { forEach } = Array.prototype


const bars = document.querySelector('#bar-chart')

function renderLoop() {
  requestAnimationFrame(renderLoop);
  analyser.getByteFrequencyData(frequencyData);

  // Let's update the bars!

  bars
  :: forEach((bar, index) => {
    bar.style.width = frequencyData[index] + 'px';
  })
};  

//Ready ? Set, Go!
renderLoop()
```
