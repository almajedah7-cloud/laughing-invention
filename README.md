<!doctype html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>قراءة الأعداد بالصوت</title>
  <style>
    body{font-family:system-ui,Segoe UI,Tahoma,Arial; margin:20px; background:#f7f9ff;}
    h1{margin:0 0 8px;}
    .panel{background:#fff; padding:14px; border-radius:12px; box-shadow:0 2px 10px rgba(0,0,0,.06); margin-bottom:14px;}
    label{display:block; margin:10px 0 6px; font-weight:600;}
    input, select{width:100%; padding:10px; border-radius:10px; border:1px solid #d7dbe7; font-size:16px;}
    .grid{display:grid; grid-template-columns:repeat(5, 1fr); gap:10px;}
    button.num{
      padding:14px 8px; border:0; border-radius:12px; background:#2b6cff; color:#fff;
      font-size:20px; font-weight:700; cursor:pointer;
    }
    button.num:active{transform:scale(.98);}
    .row{display:flex; gap:10px; align-items:center; flex-wrap:wrap;}
    .row button{
      padding:10px 14px; border-radius:12px; border:0; cursor:pointer; font-weight:700;
    }
    .speak{background:#19a974; color:#fff;}
    .stop{background:#ff5a5f; color:#fff;}
    .current{font-size:44px; font-weight:800; text-align:center; padding:10px 0;}
    .hint{color:#4b5563; font-size:14px; margin-top:6px;}
  </style>
</head>
<body>

  <div class="panel">
    <h1>🔊 مبادرة قراءة الأعداد</h1>
    <div class="hint">اضغط على العدد ليتم نطقه. إذا لم يعمل الصوت على جهاز الطالب، غيّر الصوت من القائمة.</div>
  </div>

  <div class="panel">
    <div class="current" id="current">0</div>

    <div class="row">
      <button class="speak" id="speakBtn">اقرأ 🔊</button>
      <button class="stop" id="stopBtn">إيقاف ⛔</button>
    </div>

    <label>اختر اللغة/الصوت</label>
    <select id="voiceSelect"></select>

    <label>نطاق الأعداد</label>
    <input id="maxNum" type="number" value="20" min="10" max="1000" />
    <div class="hint">غيريها إلى 100 مثلًا، ثم اضغطي خارج الحقل لتحديث الأزرار.</div>
  </div>

  <div class="panel">
    <div class="grid" id="grid"></div>
  </div>

<script>
  const currentEl = document.getElementById('current');
  const gridEl = document.getElementById('grid');
  const voiceSelect = document.getElementById('voiceSelect');
  const maxNumInput = document.getElementById('maxNum');
  const speakBtn = document.getElementById('speakBtn');
  const stopBtn = document.getElementById('stopBtn');

  let voices = [];
  let selectedVoice = null;
  let currentNumber = 0;

  function speak(text){
    window.speechSynthesis.cancel();
    const u = new SpeechSynthesisUtterance(text);
    if (selectedVoice) u.voice = selectedVoice;
    u.lang = (selectedVoice && selectedVoice.lang) ? selectedVoice.lang : "ar-SA";
    u.rate = 0.95;
    u.pitch = 1.0;
    window.speechSynthesis.speak(u);
  }

  function setCurrent(n){
    currentNumber = n;
    currentEl.textContent = n;
  }

  function buildGrid(max){
    gridEl.innerHTML = "";
    for(let i=0;i<=max;i++){
      const b = document.createElement('button');
      b.className = "num";
      b.textContent = i;
      b.onclick = () => {
        setCurrent(i);
        speak(String(i));
      };
      gridEl.appendChild(b);
    }
  }

  function loadVoices(){
    voices = window.speechSynthesis.getVoices();
    voiceSelect.innerHTML = "";

    const sorted = [...voices].sort((a,b)=>{
      const aAr = a.lang && a.lang.toLowerCase().startsWith("ar");
      const bAr = b.lang && b.lang.toLowerCase().startsWith("ar");
      return (bAr - aAr) || a.name.localeCompare(b.name);
    });

    sorted.forEach((v, idx) => {
      const opt = document.createElement('option');
      opt.value = idx;
      opt.textContent = `${v.name} (${v.lang})`;
      voiceSelect.appendChild(opt);
    });

    selectedVoice = sorted.find(v => v.lang && v.lang.toLowerCase().startsWith("ar")) || sorted[0] || null;
    const selectedIndex = sorted.findIndex(v => v === selectedVoice);
    if (selectedIndex >= 0) voiceSelect.value = String(selectedIndex);
  }

  voiceSelect.addEventListener('change', () => {
    const sorted = [...voices].sort((a,b)=>{
      const aAr = a.lang && a.lang.toLowerCase().startsWith("ar");
      const bAr = b.lang && b.lang.toLowerCase().startsWith("ar");
      return (bAr - aAr) || a.name.localeCompare(b.name);
    });
    selectedVoice = sorted[Number(voiceSelect.value)] || null;
    speak(String(currentNumber));
  });

  speakBtn.onclick = () => speak(String(currentNumber));
  stopBtn.onclick = () => window.speechSynthesis.cancel();

  maxNumInput.addEventListener('change', () => {
    const max = Math.max(0, Math.min(1000, Number(maxNumInput.value || 20)));
    buildGrid(max);
  });

  window.speechSynthesis.onvoiceschanged = loadVoices;
  loadVoices();

  buildGrid(Number(maxNumInput.value || 20));
  setCurrent(0);
</script>

</body>
</html>
