# ARSLAN-VIDEOS
<!doctype html>
<html lang="ar">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>بوت صناعة الفيديو 🎬</title>
<style>
  body { font-family: Tahoma, Arial; direction: rtl; text-align: center; background: #fff9cc; color:#222; padding:20px; }
  h1 { color:#444; }
  .box { background: #fff066; border-radius: 12px; padding: 16px; margin: 12px auto; max-width:600px; box-shadow:0 2px 6px rgba(0,0,0,0.2); }
  input, button, textarea, select {
    font-size:15px; padding:8px; margin:6px 0; border:1px solid #aaa; border-radius:6px;
  }
  button {
    background:#ffd000; color:#222; font-weight:bold; cursor:pointer;
  }
  button:hover { background:#ffb300; }
  .hidden { display:none; }
  video { max-width:100%; margin-top:10px; border:2px solid #444; border-radius:8px; }
</style>
</head>
<body>
  <h1>✨ بوت صناعة الفيديو (بالعربية)</h1>

  <div class="box">
    <h2>🎤 صنع فيديو من النصوص</h2>
    <textarea id="textInput" rows="5" style="width:100%" placeholder="اكتب النصوص هنا (كل سطر = شريحة)"></textarea><br>
    <button onclick="makeTextVideo()">⬇️ توليد فيديو من النصوص</button>
    <div id="textResult"></div>
  </div>

  <div class="box">
    <h2>🖼️ صنع فيديو من الصور</h2>
    <input id="imgInput" type="file" accept="image/*" multiple><br>
    <button onclick="makeImageVideo()">⬇️ توليد فيديو من الصور</button>
    <div id="imgResult"></div>
  </div>

  <div class="box">
    <h2>🎶 صنع فيديو مع موسيقى خلفية</h2>
    <input id="musicInput" type="file" accept="audio/*"><br>
    <input id="imgInput2" type="file" accept="image/*" multiple><br>
    <button onclick="makeMusicVideo()">⬇️ توليد فيديو بالصور + موسيقى</button>
    <div id="musicResult"></div>
  </div>

  <canvas id="canvas" class="hidden"></canvas>

<script>
function makeTextVideo(){
  const lines = document.getElementById('textInput').value.split("\n").filter(t=>t.trim());
  if(!lines.length){ alert("اكتب نصوص أولاً"); return; }
  createVideo(lines.map(t=>({type:'text',content:t})), document.getElementById("textResult"));
}

function makeImageVideo(){
  const files = document.getElementById('imgInput').files;
  if(!files.length){ alert("اختر صور أولاً"); return; }
  Promise.all([...files].map(loadImage)).then(imgs=>{
    createVideo(imgs.map(img=>({type:'image',content:img})), document.getElementById("imgResult"));
  });
}

function makeMusicVideo(){
  const files = document.getElementById('imgInput2').files;
  const music = document.getElementById('musicInput').files[0];
  if(!files.length){ alert("اختر صور أولاً"); return; }
  Promise.all([...files].map(loadImage)).then(imgs=>{
    createVideo(imgs.map(img=>({type:'image',content:img})), document.getElementById("musicResult"), music);
  });
}

function loadImage(file){
  return new Promise((res,rej)=>{
    const img=new Image();
    img.onload=()=>res(img);
    img.onerror=rej;
    img.src=URL.createObjectURL(file);
  });
}

async function createVideo(slides, container, musicFile){
  container.innerHTML="⏳ جاري توليد الفيديو...";
  const canvas=document.getElementById("canvas");
  canvas.width=640; canvas.height=360;
  const ctx=canvas.getContext("2d");
  const stream=canvas.captureStream(25);
  let audioEl=null;
  if(musicFile){
    audioEl=new Audio(URL.createObjectURL(musicFile));
    const dest=new AudioContext().createMediaStreamDestination();
    const src=new AudioContext().createMediaElementSource(audioEl);
    src.connect(dest); src.connect(new AudioContext().destination);
    stream.addTrack(dest.stream.getAudioTracks()[0]);
  }
  const rec=new MediaRecorder(stream,{mimeType:"video/webm"});
  const chunks=[];
  rec.ondataavailable=e=>chunks.push(e.data);
  rec.onstop=()=>{
    const blob=new Blob(chunks,{type:"video/webm"});
    const url=URL.createObjectURL(blob);
    container.innerHTML=`<a href="${url}" download="video.webm">⬇ تحميل الفيديو</a><br><video src="${url}" controls></video>`;
  };
  rec.start();

  if(audioEl){ audioEl.play(); }

  for(let slide of slides){
    ctx.fillStyle="#ffd000"; ctx.fillRect(0,0,640,360);
    if(slide.type==="text"){
      ctx.fillStyle="#000"; ctx.font="24px Tahoma"; ctx.textAlign="center";
      ctx.fillText(slide.content,320,180);
    } else if(slide.type==="image"){
      const scale=Math.min(640/slide.content.width,360/slide.content.height);
      const w=slide.content.width*scale, h=slide.content.height*scale;
      ctx.drawImage(slide.content,(640-w)/2,(360-h)/2,w,h);
    }
    await wait(2000);
  }

  rec.stop();
  if(audioEl){ audioEl.pause(); }
}

function wait(ms){ return new Promise(r=>setTimeout(r,ms)); }
</script>
</body>
</html>
