index.html
<!DOCTYPE html>
<html>

<head>

<meta name="viewport" content="width=device-width, initial-scale=1">

<title>Rock AI Studio</title>

<style>

body{
background:#0b0b0b;
font-family:system-ui;
color:white;
margin:0;
text-align:center;
}

header{
padding:25px;
font-size:32px;
font-weight:700;
background:#121212;
}

.container{
padding:30px;
}

button{
padding:18px 28px;
font-size:18px;
margin:10px;
border:none;
border-radius:12px;
background:#ff2b2b;
color:white;
}

.slider{
width:280px;
}

.card{

background:#151515;
padding:20px;
border-radius:16px;
margin:20px auto;
max-width:400px;

}

</style>

</head>

<body>

<header>
🎸 Rock AI Studio
</header>

<div class="container">

<div class="card">

<button onclick="play()">PLAY</button>
<button onclick="stop()">STOP</button>
<button onclick="sing()">VOCAL</button>

</div>

<div class="card">

<p>Tempo</p>
<input id="tempo" class="slider" type="range" min="90" max="180" value="120">

<p>Distortion</p>
<input id="dist" class="slider" type="range" min="0" max="600" value="300">

<p>Vocal Energy</p>
<input id="voice" class="slider" type="range" min="0.5" max="1.5" step="0.1" value="1">

</div>

</div>

<script>

let ctx
let timer
let step=0

const riff=[82,110,98,147,110,82,98,73]

function distortionCurve(amount){

let n=44100
let curve=new Float32Array(n)

for(let i=0;i<n;i++){

let x=i*2/n-1
curve[i]=(3+amount)*x*20/(Math.PI+amount*Math.abs(x))

}

return curve

}

function guitar(freq){

let osc=ctx.createOscillator()
let gain=ctx.createGain()
let dist=ctx.createWaveShaper()

dist.curve=distortionCurve(
document.getElementById("dist").value
)

osc.type="sawtooth"
osc.frequency.value=freq

gain.gain.value=.4

osc.connect(dist)
dist.connect(gain)
gain.connect(ctx.destination)

osc.start()
osc.stop(ctx.currentTime+.4)

}

function bass(freq){

let osc=ctx.createOscillator()
let gain=ctx.createGain()

osc.type="square"
osc.frequency.value=freq/2

gain.gain.value=.7

osc.connect(gain)
gain.connect(ctx.destination)

osc.start()
osc.stop(ctx.currentTime+.5)

}

function kick(){

let osc=ctx.createOscillator()
let gain=ctx.createGain()

osc.frequency.setValueAtTime(120,ctx.currentTime)
osc.frequency.exponentialRampToValueAtTime(.01,ctx.currentTime+.3)

gain.gain.setValueAtTime(1,ctx.currentTime)
gain.gain.exponentialRampToValueAtTime(.001,ctx.currentTime+.3)

osc.connect(gain)
gain.connect(ctx.destination)

osc.start()
osc.stop(ctx.currentTime+.3)

}

function snare(){

let buffer=ctx.createBuffer(1,44100,44100)
let data=buffer.getChannelData(0)

for(let i=0;i<44100;i++){
data[i]=Math.random()*2-1
}

let noise=ctx.createBufferSource()
noise.buffer=buffer

let gain=ctx.createGain()
gain.gain.value=.35

noise.connect(gain)
gain.connect(ctx.destination)

noise.start()
noise.stop(ctx.currentTime+.2)

}

function play(){

ctx=new(window.AudioContext||window.webkitAudioContext)()

loop()

}

function loop(){

let tempo=document.getElementById("tempo").value
let interval=60000/tempo/2

timer=setTimeout(()=>{

guitar(riff[step])
guitar(riff[step]*1.5)

bass(riff[step])

kick()

if(step%2==1){
snare()
}

step++

if(step>=riff.length) step=0

loop()

},interval)

}

function stop(){

clearTimeout(timer)

if(ctx){
ctx.close()
}

}

function sing(){

let words1=["Running","Breaking","Burning","Falling"]
let words2=["through","inside","beyond","under"]
let words3=["the fire","the night","the sound","the sky"]

let lyric=
words1[Math.floor(Math.random()*words1.length)]+" "+
words2[Math.floor(Math.random()*words2.length)]+" "+
words3[Math.floor(Math.random()*words3.length)]

let msg=new SpeechSynthesisUtterance(lyric)

msg.rate=1
msg.pitch=document.getElementById("voice").value

speechSynthesis.speak(msg)

}

</script>

</body>
</html>