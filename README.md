<!doctype html>
<html lang="bn">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Wi-Fi QR Suite • Matrix UI</title>

  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap" rel="stylesheet">

  <style>
    :root{
      --bg:#020405;
      --neon:#37ff7a;
      --cyan:#2bd8ff;
      --text:#d7ffe4;
      --muted:rgba(215,255,228,.72);
      --stroke:rgba(55,255,122,.18);
      --shadow:0 0 18px rgba(55,255,122,.22), 0 0 55px rgba(55,255,122,.10);
      --danger:#ff5f5f;
      --warn:#ffd36e;
      --cardA:rgba(6,18,12,.55);
      --cardB:rgba(2,10,7,.30);
    }

    /* Horror / Red mode */
    body.horror{
      --neon:#ff2d2d;
      --cyan:#ff7b7b;
      --stroke:rgba(255,45,45,.20);
      --shadow:0 0 18px rgba(255,45,45,.22), 0 0 55px rgba(255,45,45,.10);
    }

    *{ box-sizing:border-box; }
    html,body{ height:100%; }
    body{
      margin:0;
      background:
        radial-gradient(1200px 900px at 20% 10%, rgba(55,255,122,.10), transparent 55%),
        radial-gradient(900px 700px at 80% 20%, rgba(43,216,255,.10), transparent 55%),
        var(--bg);
      color:var(--text);
      font-family:"Share Tech Mono", ui-monospace, Menlo, Consolas, monospace;
      overflow-x:hidden;
    }

    canvas#matrix{
      position:fixed; inset:0; width:100%; height:100%;
      z-index:-2; opacity:.88;
      filter:saturate(1.1) contrast(1.1);
    }
    .veil{
      position:fixed; inset:0; z-index:-1;
      background:linear-gradient(180deg, rgba(0,0,0,.35), rgba(0,0,0,.72));
      pointer-events:none;
    }

    .wrap{
      min-height:100%;
      display:flex; align-items:center; justify-content:center;
      padding:18px;
    }

    .card{
      width:min(980px, 100%);
      border-radius:22px;
      background:linear-gradient(180deg, var(--cardA), var(--cardB));
      border:1px solid var(--stroke);
      box-shadow: 0 0 0 1px rgba(255,255,255,.03) inset,
                  0 18px 55px rgba(0,0,0,.62),
                  var(--shadow);
      backdrop-filter: blur(10px);
      padding:14px;
    }

    /* ===== Header / Profile ===== */
    .header{
      display:flex;
      gap:12px;
      align-items:center;
      justify-content:space-between;
      padding:6px 8px 10px;
    }
    .profile{
      display:flex; gap:12px; align-items:center; min-width:0;
    }
    .avatar{
      width:54px; height:54px;
      border-radius:999px;
      border:1px solid rgba(255,255,255,.10);
      box-shadow: 0 0 0 3px rgba(0,0,0,.25) inset, 0 0 18px rgba(55,255,122,.18);
      object-fit:cover;
      background:rgba(255,255,255,.06);
      flex:0 0 auto;
    }
    .pinfo{ min-width:0; display:grid; gap:4px; }
    .name{
      font-size:14px;
      letter-spacing:.4px;
      white-space:nowrap;
      overflow:hidden;
      text-overflow:ellipsis;
    }
    .role{
      font-size:12px;
      color:var(--muted);
      line-height:1.35;
      max-width:65ch;
    }
    .badge{
      flex:0 0 auto;
      padding:10px 12px;
      border-radius:14px;
      border:1px solid rgba(43,216,255,.25);
      background:rgba(0,0,0,.28);
      color:rgba(43,216,255,.95);
      box-shadow:0 0 18px rgba(43,216,255,.15);
      font-size:12px;
      letter-spacing:.5px;
      text-transform:uppercase;
    }

    .glowline{
      height:1px; margin:8px 8px 10px;
      background: linear-gradient(90deg, transparent, rgba(55,255,122,.55), rgba(43,216,255,.45), transparent);
      opacity:.85;
    }

    /* ===== Tabs ===== */
    .tabs{
      display:flex;
      gap:10px;
      padding:0 8px 8px;
      flex-wrap:wrap;
    }
    .tabbtn{
      border:1px solid rgba(55,255,122,.26);
      background:rgba(0,0,0,.30);
      color:var(--text);
      padding:10px 12px;
      border-radius:14px;
      cursor:pointer;
      transition: transform .15s ease, box-shadow .25s ease, border-color .25s ease;
      display:inline-flex;
      align-items:center;
      gap:8px;
      font-size:12px;
      user-select:none;
    }
    .tabbtn:hover{
      transform: translateY(-1px);
      box-shadow: 0 0 18px rgba(55,255,122,.16);
      border-color: rgba(55,255,122,.55);
    }
    .tabbtn.active{
      background: linear-gradient(180deg, rgba(55,255,122,.20), rgba(0,0,0,.35));
      box-shadow: 0 0 22px rgba(55,255,122,.14);
      border-color: rgba(55,255,122,.60);
    }
    .rightBtns{ margin-left:auto; display:flex; gap:10px; flex-wrap:wrap; }
    .miniBtn{
      border:1px solid rgba(55,255,122,.22);
      background:rgba(0,0,0,.25);
      color:var(--text);
      padding:10px 12px;
      border-radius:14px;
      cursor:pointer;
      font-size:12px;
      transition:.2s;
    }
    .miniBtn:hover{ border-color:rgba(55,255,122,.55); box-shadow:0 0 18px rgba(55,255,122,.14); transform:translateY(-1px); }

    /* ===== Panels Layout ===== */
    .grid{
      display:grid;
      grid-template-columns: 1.1fr .9fr;
      gap:12px;
      padding:8px;
    }
    @media (max-width: 860px){
      .grid{ grid-template-columns: 1fr; }
    }

    .panel{
      border-radius:18px;
      border:1px solid rgba(55,255,122,.16);
      background:rgba(0,0,0,.20);
      padding:12px;
      box-shadow:0 0 0 1px rgba(255,255,255,.02) inset;
      transition: transform .35s ease, box-shadow .35s ease;
    }
    .panel:hover{
      transform: translateY(-2px);
      box-shadow: 0 0 0 1px rgba(255,255,255,.02) inset, 0 0 28px rgba(55,255,122,.09);
    }

    label, .label{
      display:block;
      font-size:12px;
      color:rgba(215,255,228,.82);
      margin:10px 0 6px;
    }

    input, select{
      width:100%;
      padding:12px 12px;
      border-radius:14px;
      border:1px solid rgba(55,255,122,.22);
      background:rgba(0,0,0,.30);
      color:var(--text);
      outline:none;
      transition: box-shadow .25s ease, border-color .25s ease;
      font-family:inherit;
    }
    input::placeholder{ color:rgba(215,255,228,.45); }
    input:focus, select:focus{
      border-color: rgba(55,255,122,.55);
      box-shadow: 0 0 0 4px rgba(55,255,122,.12), 0 0 26px rgba(55,255,122,.12);
    }

    .row2{ display:grid; grid-template-columns: 1fr 1fr; gap:10px; }
    .btns{ display:flex; flex-wrap:wrap; gap:10px; margin-top:12px; }
    button{
      border:1px solid rgba(55,255,122,.26);
      background:rgba(0,0,0,.35);
      color:var(--text);
      padding:12px 14px;
      border-radius:14px;
      cursor:pointer;
      transition: transform .15s ease, box-shadow .25s ease, border-color .25s ease, opacity .2s ease;
      font-family:inherit;
      font-size:12px;
    }
    button:hover{
      transform: translateY(-1px);
      box-shadow: 0 0 20px rgba(55,255,122,.18);
      border-color: rgba(55,255,122,.55);
    }
    .primary{
      background: linear-gradient(180deg, rgba(55,255,122,.22), rgba(0,0,0,.35));
      box-shadow: 0 0 22px rgba(55,255,122,.14);
    }
    .danger{ border-color: rgba(255,80,80,.28); }
    .danger:hover{ border-color: rgba(255,80,80,.6); box-shadow: 0 0 18px rgba(255,80,80,.12); }
    .warn{ border-color: rgba(255,211,110,.30); }
    .warn:hover{ border-color: rgba(255,211,110,.75); box-shadow: 0 0 18px rgba(255,211,110,.14); }

    .small{ font-size:12px; color:rgba(215,255,228,.72); line-height:1.55; }

    /* ===== QR Preview boxes ===== */
    .qrwrap{
      border-radius:18px;
      background:#fff;
      padding:18px;
      min-height:260px;
      display:flex; align-items:center; justify-content:center;
      box-shadow: 0 10px 35px rgba(0,0,0,.45);
      position:relative; overflow:hidden;
    }
    .qrwrap::after{
      content:"";
      position:absolute; inset:-40%;
      background: radial-gradient(circle at 30% 30%, rgba(55,255,122,.22), transparent 55%),
                  radial-gradient(circle at 80% 20%, rgba(43,216,255,.18), transparent 60%);
      mix-blend-mode: multiply;
      transform: rotate(18deg);
      pointer-events:none;
      opacity:.85;
    }
    .qrwrap img, .qrwrap video{
      width:100%;
      max-width:340px;
      height:auto;
      position:relative; z-index:1;
      background:#fff;
      padding:14px;
      border-radius:14px;
    }
    .qrwrap img:not([src]){ display:none; }
    .qrwrap video{ display:none; }

    .mono{
      font-size:12px;
      color:rgba(215,255,228,.92);
      border-radius:14px;
      border:1px solid rgba(55,255,122,.18);
      background:rgba(0,0,0,.22);
      padding:10px;
      word-break: break-all;
      min-height:44px;
      white-space:pre-wrap;
    }

    .status{
      margin-top:10px;
      padding:10px 12px;
      border-radius:14px;
      border:1px solid rgba(43,216,255,.22);
      background:rgba(0,0,0,.22);
      color:rgba(43,216,255,.92);
      font-size:12px;
      line-height:1.6;
      display:none;
      white-space:pre-line;
    }
    .status.bad{
      border-color: rgba(255,95,95,.26);
      color: rgba(255,215,215,.95);
    }

    .kv{
      display:grid;
      grid-template-columns: 140px 1fr;
      gap:10px;
      margin-top:10px;
      align-items:center;
    }
    .k{ color:rgba(215,255,228,.70); font-size:12px; }
    .v{
      border:1px solid rgba(55,255,122,.18);
      background:rgba(0,0,0,.18);
      border-radius:12px;
      padding:9px 10px;
      font-size:13px;
      color:rgba(215,255,228,.95);
      word-break:break-word;
    }

    /* History list (compact) */
    .hist{ display:grid; gap:10px; margin-top:10px; }
    .histItem{
      border-radius:16px;
      border:1px solid rgba(55,255,122,.16);
      background:rgba(0,0,0,.18);
      padding:10px;
      display:grid;
      gap:8px;
    }
    .histTop{
      display:flex; justify-content:space-between; gap:10px; align-items:flex-start;
      font-size:12px; color:rgba(215,255,228,.9);
    }
    .histTop .time{ color:rgba(215,255,228,.65); }
    .histBtns{ display:flex; flex-wrap:wrap; gap:8px; }
    .mini{
      padding:8px 10px;
      border-radius:12px;
      font-size:12px;
      border:1px solid rgba(55,255,122,.18);
      background:rgba(0,0,0,.28);
      cursor:pointer;
      color:rgba(215,255,228,.95);
    }

    /* Modal */
    .modal{
      position:fixed; inset:0;
      display:none;
      align-items:center; justify-content:center;
      padding:18px;
      background:rgba(0,0,0,.72);
      z-index:50;
    }
    .modal.show{ display:flex; }
    .modalCard{
      width:min(520px,100%);
      border-radius:20px;
      background:rgba(0,0,0,.55);
      border:1px solid rgba(55,255,122,.22);
      box-shadow: var(--shadow);
      padding:14px;
      backdrop-filter: blur(10px);
    }
    .modalHead{
      display:flex; justify-content:space-between; align-items:center; gap:10px;
      margin-bottom:10px;
    }
    .close{
      border:1px solid rgba(255,255,255,.18);
      background:rgba(255,255,255,.06);
      padding:10px 12px;
      border-radius:12px;
      cursor:pointer;
      color:var(--text);
      font-family:inherit;
    }
    .qrBigWrap{
      background:#fff;
      border-radius:18px;
      padding:18px;
      display:flex; justify-content:center; align-items:center;
      box-shadow: 0 10px 30px rgba(0,0,0,.45);
    }
    .qrBigWrap img{
      width:100%;
      max-width:380px;
      height:auto;
      background:#fff;
      padding:14px;
      border-radius:14px;
    }

    /* Hide work canvas */
    canvas#work{ display:none; }

    /* Print */
    @media print{
      body{ background:#fff !important; color:#000 !important; }
      canvas#matrix, .veil, .modal{ display:none !important; }
      .wrap{ padding:0 !important; }
      button{ display:none !important; }
      .card{
        width:100% !important;
        box-shadow:none !important;
        border:1px solid #ddd !important;
        background:#fff !important;
      }
      .panel{ background:#fff !important; border:1px solid #ddd !important; }
      .mono,.v{ border:1px solid #ddd !important; background:#fff !important; color:#000 !important; }
      .badge,.status{ border:1px solid #ddd !important; background:#fff !important; color:#000 !important; box-shadow:none !important; }
      .glowline{ display:none !important; }
      .role,.small,label,.label,.k{ color:#000 !important; }
    }
  </style>
</head>

<body>
  <canvas id="matrix"></canvas>
  <div class="veil"></div>

  <div class="wrap">
    <div class="card" id="reportCard">
      <!-- ===== Profile Header ===== -->
      <div class="header">
        <div class="profile">
          <!-- 🔴 এখানে তোমার ছবি url বা local file path দাও -->
          <img class="avatar" id="ownerImg" alt="Owner" src="https://i.imgur.com/4M34hi2.png">
          <div class="pinfo">
            <div class="name" id="ownerName">Md Rifat Islam • Developer</div>
            <div class="role" id="ownerRole">Web Developer • Matrix UI • QR Tools • HTML/CSS/JS • Experience: 2+ years</div>
          </div>
        </div>
        <div class="badge">Wi-Fi QR Suite</div>
      </div>

      <div class="glowline"></div>

      <!-- ===== Tabs ===== -->
      <div class="tabs">
        <button class="tabbtn active" id="tabGen">🟢 Text → QR</button>
        <button class="tabbtn" id="tabDec">🔵 QR → All Info</button>

        <div class="rightBtns">
          <button class="miniBtn" id="toggleHorror">🔴 Horror Mode</button>
          <button class="miniBtn" id="printAll">🖨️ Print / PDF</button>
        </div>
      </div>

      <!-- ====== GENERATOR VIEW ====== -->
      <section id="viewGen">
        <div class="grid">
          <div class="panel">
            <label>Wi-Fi নাম (SSID)</label>
            <input id="g_ssid" placeholder="যেমন: MyWifi" autocomplete="off" />

            <div class="row2">
              <div>
                <label>Security</label>
                <select id="g_auth">
                  <option value="WPA">WPA/WPA2/WPA3</option>
                  <option value="WEP">WEP</option>
                  <option value="nopass">Open (No Password)</option>
                </select>
              </div>
              <div>
                <label>Hidden?</label>
                <select id="g_hidden">
                  <option value="false">না</option>
                  <option value="true">হ্যাঁ (Hidden SSID)</option>
                </select>
              </div>
            </div>

            <label>Password</label>
            <input id="g_pass" type="password" placeholder="পাসওয়ার্ড" autocomplete="off" />

            <div class="btns">
              <button class="primary" id="g_gen">Generate QR</button>
              <button id="g_download">Download PNG</button>
              <button id="g_copy">Copy Wi-Fi Text</button>
              <button id="g_clear" class="danger">Clear</button>
            </div>

            <div class="small" style="margin-top:10px;">
              PDF করতে: উপরে <b>Print / PDF</b> চাপুন → Destination: <b>Save as PDF</b>
            </div>

            <div id="g_status" class="status" style="display:none"></div>
          </div>

          <div class="panel">
            <div class="label">QR Preview</div>
            <div class="qrwrap">
              <img id="g_qr" alt="QR will appear here" />
            </div>
            <div class="small" style="margin-top:10px;">QR-এর ভিতরের Wi-Fi টেক্সট:</div>
            <div class="mono" id="g_wifitext">WIFI:...</div>
          </div>
        </div>
      </section>

      <!-- ====== DECODER VIEW ====== -->
      <section id="viewDec" style="display:none">
        <div class="grid">
          <div class="panel">
            <div class="small">
              Upload বা Camera Scan করে Wi-Fi QR ডিকোড করুন। ডাটা localStorage এ সেভ থাকবে।<br>
              <span style="color:rgba(255,215,215,.9)">শুধু নিজের/অনুমতি থাকা নেটওয়ার্কে ব্যবহার করুন।</span>
            </div>

            <div class="label" style="margin-top:12px;">Option A — Upload Wi-Fi QR Image</div>
            <input id="d_file" type="file" accept="image/*" />

            <div class="btns">
              <button class="primary" id="d_decode">Decode Upload</button>
              <button class="warn" id="d_camStart">Open Camera</button>
              <button id="d_camStop" style="display:none">Stop Camera</button>
              <button class="primary" id="d_autoConnect">Auto Connect</button>
              <button id="d_clear" class="danger">Clear</button>
            </div>

            <div id="d_status" class="status"></div>

            <div class="label" style="margin-top:14px;">Decoded Raw Text</div>
            <div id="d_raw" class="mono">—</div>

            <div class="label" style="margin-top:14px;">Wi-Fi Details</div>
            <div class="kv"><div class="k">SSID</div><div class="v" id="d_ssid">—</div></div>
            <div class="kv"><div class="k">Password</div><div class="v" id="d_pass">—</div></div>
            <div class="kv"><div class="k">Security</div><div class="v" id="d_sec">—</div></div>
            <div class="kv"><div class="k">Hidden</div><div class="v" id="d_hidden">—</div></div>

            <div class="btns" style="margin-top:12px;">
              <button id="d_copyRaw">Copy Raw</button>
              <button id="d_copyWifi">Copy Wi-Fi String</button>
              <button id="d_txt">Download TXT</button>
              <button id="d_png">Download PNG Report</button>
              <button id="d_exportHistory">Export History TXT</button>
              <button id="d_clearHistory" class="danger">Clear History</button>
            </div>

            <div class="label" style="margin-top:14px;">History</div>
            <div id="d_history" class="hist"></div>
          </div>

          <div class="panel">
            <div class="label">Preview / Camera</div>
            <div class="qrwrap">
              <img id="d_previewImg" alt="preview" />
              <video id="d_video" playsinline></video>
            </div>

            <div class="small" style="margin-top:10px;">
              Auto Connect: “Modal QR” দেখাবে → ফোনের ক্যামেরা/সিস্টেম স্ক্যান দিয়ে কানেক্ট করতে পারবেন।
            </div>

            <div class="label" style="margin-top:14px;">Parsed JSON</div>
            <div id="d_json" class="mono">{}</div>

            <div class="label" style="margin-top:14px;">Manual Connect Guide</div>
            <div class="mono">
              Android: Settings → Wi-Fi → QR/Scan → Scan the QR<br>
              iPhone: Camera → Scan QR → Join
            </div>
          </div>
        </div>
      </section>
    </div>
  </div>

  <!-- Work canvas -->
  <canvas id="work"></canvas>

  <!-- Auto-connect modal -->
  <div class="modal" id="modal">
    <div class="modalCard">
      <div class="modalHead">
        <div style="display:grid; gap:4px;">
          <div style="font-size:13px; color:rgba(215,255,228,.95)">AUTO CONNECT MODE</div>
          <div style="font-size:12px; color:rgba(215,255,228,.70)">এই QR স্ক্যান করলেই Wi-Fi connect prompt আসবে</div>
        </div>
        <button class="close" id="closeModal">Close</button>
      </div>
      <div class="qrBigWrap">
        <img id="connectQr" alt="connect qr" />
      </div>
      <div class="btns" style="margin-top:10px;">
        <button id="copyFromModal">Copy Wi-Fi String</button>
      </div>
      <div class="small" style="margin-top:10px;">
        যদি আপনার ব্রাউজার থেকে Wi-Fi settings খোলা না যায়, manual route ব্যবহার করুন।
      </div>
    </div>
  </div>

  <!-- libs -->
  <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>

  <script>
    /********** Matrix Rain **********/
    const matrix = document.getElementById("matrix");
    const mctx = matrix.getContext("2d");
    const chars = "アァカサタナハマヤャラワン0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ$+-*/=%\"'<>[]{}()#@";
    const fontSize = 16;
    let drops = [];

    function recalcColumns(){
      const columns = Math.max(1, Math.floor(innerWidth / fontSize));
      drops = Array(columns).fill(1);
    }
    function resizeMatrix(){
      const dpr = Math.max(1, window.devicePixelRatio || 1);
      matrix.width = Math.floor(innerWidth * dpr);
      matrix.height = Math.floor(innerHeight * dpr);
      matrix.style.width = innerWidth + "px";
      matrix.style.height = innerHeight + "px";
      mctx.setTransform(dpr,0,0,dpr,0,0);
      recalcColumns();
    }
    window.addEventListener("resize", resizeMatrix);

    function tick(){
      mctx.fillStyle = "rgba(0,0,0,0.08)";
      mctx.fillRect(0,0,innerWidth,innerHeight);
      mctx.font = fontSize + "px Share Tech Mono";

      const isHorror = document.body.classList.contains("horror");
      for(let i=0;i<drops.length;i++){
        const t = chars[Math.floor(Math.random()*chars.length)];
        mctx.fillStyle = (Math.random() > 0.965)
          ? (isHorror ? "rgba(255,90,90,0.95)" : "rgba(43,216,255,0.95)")
          : (isHorror ? "rgba(255,45,45,0.95)" : "rgba(55,255,122,0.95)");
        mctx.fillText(t, i*fontSize, drops[i]*fontSize);

        if(drops[i]*fontSize > innerHeight && Math.random() > 0.975) drops[i]=0;
        drops[i]++;
      }
      requestAnimationFrame(tick);
    }
    resizeMatrix();
    requestAnimationFrame(tick);

    /********** Helpers **********/
    const $ = (id) => document.getElementById(id);

    function showStatus(id, msg, bad=false){
      const el = $(id);
      el.style.display = "block";
      el.classList.toggle("bad", !!bad);
      el.textContent = msg;
    }
    function hideStatus(id){ $(id).style.display = "none"; }

    async function copyText(msg){
      try{
        await navigator.clipboard.writeText(msg);
      }catch{
        const ta=document.createElement("textarea");
        ta.value=msg; document.body.appendChild(ta);
        ta.select(); document.execCommand("copy"); ta.remove();
      }
    }

    function downloadText(filename, content){
      const blob = new Blob([content], {type:"text/plain;charset=utf-8"});
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url; a.download = filename;
      document.body.appendChild(a); a.click(); a.remove();
      URL.revokeObjectURL(url);
    }

    function nowStamp(){
      const d=new Date();
      return d.toISOString().replace("T"," ").slice(0,19);
    }

    function escapeHtml(s){
      return String(s).replace(/[&<>"']/g, m => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#039;'}[m]));
    }

    /********** Tabs **********/
    const tabGen = $("tabGen");
    const tabDec = $("tabDec");
    const viewGen = $("viewGen");
    const viewDec = $("viewDec");

    function setTab(which){
      const gen = which === "gen";
      tabGen.classList.toggle("active", gen);
      tabDec.classList.toggle("active", !gen);
      viewGen.style.display = gen ? "" : "none";
      viewDec.style.display = gen ? "none" : "";
      // stop camera if switching away
      if(gen) stopDecoderCamera();
    }
    tabGen.addEventListener("click", ()=>setTab("gen"));
    tabDec.addEventListener("click", ()=>setTab("dec"));

    /********** Horror Mode + Print **********/
    $("toggleHorror").addEventListener("click", ()=>{
      document.body.classList.toggle("horror");
    });
    $("printAll").addEventListener("click", ()=>window.print());

    /********** Generator (Text -> QR) **********/
    function escapeWifiValue(v){
      return String(v).replace(/([\\;,:"])/g, "\\$1");
    }

    function buildWifiString(){
      const ssid = $("g_ssid").value.trim();
      const auth = $("g_auth").value;
      const hidden = $("g_hidden").value === "true";
      if(!ssid) throw new Error("SSID দিতে হবে!");

      let pass = $("g_pass").value;

      if(auth !== "nopass"){
        if(!pass) throw new Error("Password দিতে হবে (অথবা Security Open দিন)!");
        pass = escapeWifiValue(pass);
      }else{
        pass = "";
      }

      // better compatibility: include H only when true
      const Hpart = hidden ? "H:true;" : "";
      return `WIFI:T:${auth};S:${escapeWifiValue(ssid)};P:${pass};${Hpart};`;
    }

    function setQRFromText(text){
      const url =
        "https://api.qrserver.com/v1/create-qr-code/?" +
        "size=520x520&margin=18&data=" + encodeURIComponent(text);

      $("g_qr").src = url;
      $("g_wifitext").textContent = text;
    }

    function gen(){
      try{
        hideStatus("g_status");
        const wifi = buildWifiString();
        setQRFromText(wifi);
      }catch(e){
        showStatus("g_status", e.message || "কিছু ভুল হয়েছে", true);
      }
    }

    $("g_gen").addEventListener("click", gen);

    $("g_auth").addEventListener("change", ()=>{
      const isOpen = $("g_auth").value === "nopass";
      $("g_pass").disabled = isOpen;
      if(isOpen) $("g_pass").value = "";
    });

    $("g_clear").addEventListener("click", ()=>{
      $("g_ssid").value="";
      $("g_pass").value="";
      $("g_auth").value="WPA";
      $("g_hidden").value="false";
      $("g_pass").disabled=false;
      $("g_qr").removeAttribute("src");
      $("g_wifitext").textContent="WIFI:...";
      hideStatus("g_status");
    });

    $("g_copy").addEventListener("click", async ()=>{
      try{
        const wifi = buildWifiString();
        await copyText(wifi);
        showStatus("g_status","কপি হয়ে গেছে ✅", false);
        setTimeout(()=>hideStatus("g_status"), 900);
      }catch(e){
        showStatus("g_status", e.message || "কপি করা যায়নি", true);
      }
    });

    // reliable download (fetch->blob)
    $("g_download").addEventListener("click", async ()=>{
      const src = $("g_qr").src;
      if(!src){ showStatus("g_status","আগে QR Generate করুন", true); return; }
      try{
        const res = await fetch(src);
        const blob = await res.blob();
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.download = "wifi-qr.png";
        document.body.appendChild(a);
        a.click(); a.remove();
        URL.revokeObjectURL(url);
      }catch{
        showStatus("g_status","Download blocked. Screenshot/Print ব্যবহার করুন।", true);
      }
    });

    ["g_ssid","g_pass"].forEach(id=>{
      $(id).addEventListener("keydown",(e)=>{ if(e.key==="Enter") gen(); });
    });

    /********** Decoder (QR -> Info) **********/
    let lastDecoded = null;
    let cameraStream = null;
    let cameraLoopRAF = null;

    const LS_KEY = "wifi_qr_history_v1";

    function loadHistory(){
      try{ return JSON.parse(localStorage.getItem(LS_KEY) || "[]") || []; }
      catch{ return []; }
    }
    function saveHistory(list){
      localStorage.setItem(LS_KEY, JSON.stringify(list));
    }

    function addToHistory(entry){
      const list = loadHistory();
      const exists = list.some(x => x.raw === entry.raw);
      const newList = exists ? list : [entry, ...list].slice(0, 40);
      saveHistory(newList);
      renderHistory();
    }

    function renderHistory(){
      const list = loadHistory();
      const wrap = $("d_history");
      wrap.innerHTML = "";
      if(!list.length){
        wrap.innerHTML = `<div class="mono">No history yet. Decode করলে এখানে সেভ হবে।</div>`;
        return;
      }

      list.forEach((it, idx)=>{
        const div = document.createElement("div");
        div.className = "histItem";
        div.innerHTML = `
          <div class="histTop">
            <div>
              <div><b>${escapeHtml(it.ssid || "—")}</b> • ${escapeHtml(it.security || "—")}</div>
              <div class="time">${escapeHtml(it.time || "")}</div>
            </div>
            <div style="text-align:right; color:rgba(215,255,228,.7)">${it.hidden ? "Hidden" : ""}</div>
          </div>
          <div class="mono">${escapeHtml(it.raw || "")}</div>
          <div class="histBtns">
            <button class="mini" data-act="load" data-idx="${idx}">Load</button>
            <button class="mini" data-act="copy" data-idx="${idx}">Copy</button>
            <button class="mini" data-act="del" data-idx="${idx}">Delete</button>
          </div>
        `;
        wrap.appendChild(div);
      });

      wrap.querySelectorAll("button.mini").forEach(b=>{
        b.addEventListener("click", async ()=>{
          const act = b.getAttribute("data-act");
          const idx = Number(b.getAttribute("data-idx"));
          const list = loadHistory();
          const it = list[idx];
          if(!it) return;

          if(act === "load"){
            renderDecoded(it.raw);
            showStatus("d_status","Loaded from history ✅", false);
          }
          if(act === "copy"){
            await copyText(it.raw || "");
            showStatus("d_status","Copied ✅", false);
            setTimeout(()=>hideStatus("d_status"), 800);
          }
          if(act === "del"){
            list.splice(idx,1);
            saveHistory(list);
            renderHistory();
            showStatus("d_status","Deleted ✅", false);
          }
        });
      });
    }

    function unescapeWifiValue(v){
      return String(v || "").replace(/\\([\\;,:"])/g, "$1");
    }

    function parseWifiString(raw){
      const out = { format:"unknown", ssid:"", password:"", security:"", hidden:false, raw:String(raw||"") };
      const s = String(raw||"").trim();
      if(!s) return out;
      if(!/^WIFI:/i.test(s)) return out;

      out.format = "wifi";
      const body = s.replace(/^WIFI:/i, "");

      const tokens = [];
      let cur="", esc=false;
      for(let i=0;i<body.length;i++){
        const ch = body[i];
        if(esc){ cur += ch; esc=false; continue; }
        if(ch === "\\"){ cur += ch; esc=true; continue; }
        if(ch === ";"){ tokens.push(cur); cur=""; }
        else cur += ch;
      }
      if(cur) tokens.push(cur);

      const map = {};
      for(const t of tokens){
        const idx = t.indexOf(":");
        if(idx === -1) continue;
        const k = t.slice(0, idx).trim().toUpperCase();
        const v = t.slice(idx+1);
        if(k) map[k] = v;
      }

      const T = (map["T"] || "").toUpperCase();
      out.security = T || "—";
      out.ssid = unescapeWifiValue(map["S"] || "");
      out.password = unescapeWifiValue(map["P"] || "");
      out.hidden = String(map["H"] || "").toLowerCase() === "true";

      if(out.security === "WPA" || out.security === "WPA2" || out.security === "WPA3") out.security = "WPA/WPA2/WPA3";
      if(out.security === "WEP") out.security = "WEP";
      if(T === "NOPASS" || out.security === "NOPASS") out.security = "Open (No Password)";
      if(out.security === "Open (No Password)") out.password = "";

      return out;
    }

    async function decodeFromImageElement(img){
      const canvas = $("work");
      const ctx = canvas.getContext("2d", {willReadFrequently:true});
      const maxW = 1400;
      const scale = Math.min(1, maxW / img.naturalWidth);
      const w = Math.floor(img.naturalWidth * scale);
      const h = Math.floor(img.naturalHeight * scale);

      canvas.width = w; canvas.height = h;
      ctx.drawImage(img, 0, 0, w, h);

      const imageData = ctx.getImageData(0,0,w,h);
      const code = jsQR(imageData.data, imageData.width, imageData.height, { inversionAttempts:"attemptBoth" });
      if(!code || !code.data) throw new Error("QR কোড পাওয়া যায়নি (ছবি ব্লার/ক্রপ/লো কনট্রাস্ট হতে পারে)");
      return code.data;
    }

    async function decodeFromFile(file){
      const img = new Image();
      const url = URL.createObjectURL(file);
      try{
        await new Promise((res, rej)=>{
          img.onload = ()=>res();
          img.onerror = ()=>rej(new Error("ইমেজ লোড হয়নি"));
          img.src = url;
        });
        $("d_previewImg").src = img.src;
        $("d_video").style.display = "none";
        return await decodeFromImageElement(img);
      } finally {
        URL.revokeObjectURL(url);
      }
    }

    function decodeFromVideoFrame(){
      const video = $("d_video");
      if(video.readyState < 2) return null;

      const canvas = $("work");
      const ctx = canvas.getContext("2d", {willReadFrequently:true});

      const w = Math.min(1280, video.videoWidth || 0);
      const h = Math.floor((w / (video.videoWidth || 1)) * (video.videoHeight || 1));
      if(!w || !h) return null;

      canvas.width = w;
      canvas.height = h;
      ctx.drawImage(video, 0, 0, w, h);

      const imageData = ctx.getImageData(0,0,w,h);
      const code = jsQR(imageData.data, imageData.width, imageData.height, { inversionAttempts:"attemptBoth" });
      return code && code.data ? code.data : null;
    }

    function renderDecoded(rawText){
      $("d_raw").textContent = rawText || "—";
      const parsed = parseWifiString(rawText || "");

      $("d_ssid").textContent   = parsed.format==="wifi" ? (parsed.ssid || "—") : "—";
      $("d_pass").textContent   = parsed.format==="wifi" ? (parsed.password || "—") : "—";
      $("d_sec").textContent    = parsed.format==="wifi" ? (parsed.security || "—") : "—";
      $("d_hidden").textContent = parsed.format==="wifi" ? (parsed.hidden ? "Yes (Hidden)" : "No") : "—";

      const jsonObj = {
        format: parsed.format,
        ssid: parsed.ssid,
        password: parsed.password,
        security: parsed.security,
        hidden: parsed.hidden,
        raw: parsed.raw
      };
      $("d_json").textContent = JSON.stringify(jsonObj, null, 2);

      lastDecoded = { ...jsonObj, time: nowStamp() };

      if(parsed.format === "wifi"){
        addToHistory(lastDecoded);
      }
    }

    function stopDecoderCamera(){
      if(cameraLoopRAF) cancelAnimationFrame(cameraLoopRAF);
      cameraLoopRAF = null;

      const video = $("d_video");
      video.pause();
      video.srcObject = null;
      video.style.display = "none";

      if(cameraStream){
        cameraStream.getTracks().forEach(t=>t.stop());
        cameraStream = null;
      }

      $("d_camStart").style.display = "inline-flex";
      $("d_camStop").style.display = "none";
    }

    async function startDecoderCamera(){
      try{
        hideStatus("d_status");
        const video = $("d_video");

        cameraStream = await navigator.mediaDevices.getUserMedia({
          video: { facingMode: { ideal: "environment" } },
          audio: false
        });

        video.srcObject = cameraStream;
        await video.play();

        $("d_previewImg").removeAttribute("src");
        video.style.display = "block";

        $("d_camStart").style.display = "none";
        $("d_camStop").style.display = "inline-flex";

        showStatus("d_status","Camera ON ✅ এখন QR সামনে ধরুন (Auto-scan চলছে)…", false);

        const loop = ()=>{
          const data = decodeFromVideoFrame();
          if(data){
            renderDecoded(data);
            showStatus("d_status","Live Scan ✅ Decoded & Saved", false);
            stopDecoderCamera(); // stop after first decode (clean UX)
            return;
          }
          cameraLoopRAF = requestAnimationFrame(loop);
        };
        cameraLoopRAF = requestAnimationFrame(loop);

      }catch(e){
        showStatus("d_status", (e && e.message) ? e.message : "Camera open করা যায়নি (permission?)", true);
      }
    }

    function getWifiStringForConnect(){
      if(!lastDecoded || lastDecoded.format !== "wifi") return null;
      return lastDecoded.raw;
    }

    function openConnectModal(){
      const wifi = getWifiStringForConnect();
      if(!wifi){
        showStatus("d_status","Wi-Fi QR পাওয়া যায়নি। আগে একটি Wi-Fi QR Decode করুন।", true);
        return;
      }
      const url =
        "https://api.qrserver.com/v1/create-qr-code/?" +
        "size=700x700&margin=18&data=" + encodeURIComponent(wifi);

      $("connectQr").src = url;
      $("modal").classList.add("show");
    }

    function buildReportText(){
      const t = lastDecoded;
      if(!t) return "No decoded data.";
      return [
        "WIFI QR DECODE REPORT",
        "=====================",
        `Time: ${t.time || ""}`,
        `Format: ${t.format || ""}`,
        "",
        `SSID: ${t.ssid || ""}`,
        `Password: ${t.password || ""}`,
        `Security: ${t.security || ""}`,
        `Hidden: ${t.hidden ? "true" : "false"}`,
        "",
        "Raw:",
        t.raw || ""
      ].join("\\n");
    }

    async function downloadReportPng(){
      try{
        const node = $("reportCard");
        const canvas = await html2canvas(node, {scale: 2, backgroundColor: null});
        canvas.toBlob((blob)=>{
          if(!blob){ showStatus("d_status","PNG তৈরি হয়নি", true); return; }
          const url = URL.createObjectURL(blob);
          const a = document.createElement("a");
          a.href = url;
          a.download = "wifi-report.png";
          document.body.appendChild(a);
          a.click();
          a.remove();
          URL.revokeObjectURL(url);
        }, "image/png");
      }catch{
        showStatus("d_status","PNG export failed (browser restriction?)", true);
      }
    }

    // decoder events
    $("d_decode").addEventListener("click", async ()=>{
      const file = $("d_file").files && $("d_file").files[0];
      if(!file){ showStatus("d_status","একটা QR ইমেজ সিলেক্ট করুন।", true); return; }
      try{
        showStatus("d_status","Decoding… (SYSTEM SCAN IN PROGRESS)", false);
        const raw = await decodeFromFile(file);
        renderDecoded(raw);
        if(!/^WIFI:/i.test(String(raw).trim())){
          showStatus("d_status","QR ডিকোড হয়েছে, কিন্তু এটা Wi-Fi QR ফরম্যাট না। Raw text দেখুন।", true);
        }else{
          showStatus("d_status","Decoded ✅ Saved to history", false);
        }
      }catch(e){
        showStatus("d_status", e.message || "Decode failed", true);
        renderDecoded("—");
      }
    });

    $("d_file").addEventListener("change", ()=>{
      hideStatus("d_status");
      const file = $("d_file").files && $("d_file").files[0];
      if(!file) return;
      const url = URL.createObjectURL(file);
      $("d_previewImg").src = url;
      $("d_video").style.display = "none";
      setTimeout(()=>URL.revokeObjectURL(url), 1500);
    });

    $("d_camStart").addEventListener("click", startDecoderCamera);
    $("d_camStop").addEventListener("click", ()=>{
      stopDecoderCamera();
      showStatus("d_status","Camera OFF", false);
      setTimeout(()=>hideStatus("d_status"), 900);
    });

    $("d_autoConnect").addEventListener("click", openConnectModal);

    $("d_copyRaw").addEventListener("click", async ()=>{
      const t = $("d_raw").textContent.trim();
      if(!t || t==="—"){ showStatus("d_status","আগে Decode করুন।", true); return; }
      await copyText(t);
      showStatus("d_status","Copied ✅", false);
      setTimeout(()=>hideStatus("d_status"), 800);
    });

    $("d_copyWifi").addEventListener("click", async ()=>{
      const wifi = getWifiStringForConnect();
      if(!wifi){ showStatus("d_status","Wi-Fi ডাটা নেই। আগে Wi-Fi QR decode করুন।", true); return; }
      await copyText(wifi);
      showStatus("d_status","Copied ✅", false);
      setTimeout(()=>hideStatus("d_status"), 800);
    });

    $("d_txt").addEventListener("click", ()=>{
      if(!lastDecoded || lastDecoded.format!=="wifi"){
        showStatus("d_status","Wi-Fi রিপোর্ট তৈরি করতে আগে Wi-Fi QR decode করুন।", true);
        return;
      }
      downloadText("wifi-report.txt", buildReportText());
      showStatus("d_status","TXT Downloaded ✅", false);
      setTimeout(()=>hideStatus("d_status"), 900);
    });

    $("d_png").addEventListener("click", async ()=>{
      await downloadReportPng();
      showStatus("d_status","PNG Downloaded ✅", false);
      setTimeout(()=>hideStatus("d_status"), 900);
    });

    $("d_exportHistory").addEventListener("click", ()=>{
      const list = loadHistory();
      if(!list.length){ showStatus("d_status","History খালি।", true); return; }
      const text = [
        "WIFI QR HISTORY EXPORT",
        "======================",
        "",
        ...list.map((it, i)=>[
          `#${i+1}`,
          `Time: ${it.time||""}`,
          `SSID: ${it.ssid||""}`,
          `Password: ${it.password||""}`,
          `Security: ${it.security||""}`,
          `Hidden: ${it.hidden ? "true":"false"}`,
          `Raw: ${it.raw||""}`,
          "----------------------",
          ""
        ].join("\\n"))
      ].join("\\n");
      downloadText("wifi-history.txt", text);
      showStatus("d_status","History TXT Downloaded ✅", false);
      setTimeout(()=>hideStatus("d_status"), 900);
    });

    $("d_clearHistory").addEventListener("click", ()=>{
      localStorage.removeItem(LS_KEY);
      renderHistory();
      showStatus("d_status","History cleared ✅", false);
      setTimeout(()=>hideStatus("d_status"), 900);
    });

    $("d_clear").addEventListener("click", ()=>{
      stopDecoderCamera();
      $("d_file").value = "";
      $("d_previewImg").removeAttribute("src");
      renderDecoded("—");
      lastDecoded = null;
      hideStatus("d_status");
    });

    // modal events
    $("closeModal").addEventListener("click", ()=> $("modal").classList.remove("show"));
    $("modal").addEventListener("click", (e)=>{
      if(e.target.id === "modal") $("modal").classList.remove("show");
    });
    $("copyFromModal").addEventListener("click", async ()=>{
      const wifi = getWifiStringForConnect();
      if(!wifi){ showStatus("d_status","Wi-Fi ডাটা নেই।", true); return; }
      await copyText(wifi);
      $("modal").classList.remove("show");
      showStatus("d_status","Copied ✅", false);
      setTimeout(()=>hideStatus("d_status"), 900);
    });

    // init decoder history
    renderHistory();
  </script>
</body>
</html>
