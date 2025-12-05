<!doctype html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>absensi - webcode friendly</title>

<!-- libs -->
<script src="https://unpkg.com/html5-qrcode@2.3.8/minified/html5-qrcode.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<style>
  :root{font-family:system-ui,Segoe UI,Roboto,Arial;--card:#071220;--muted:#98a3b3}
  body{margin:12px;background:#071426;color:#e6eef8}
  .wrap{max-width:900px;margin:0 auto}
  header{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px}
  h1{font-size:18px;margin:0}
  nav{display:flex;gap:8px}
  button{background:#1f6feb;border:0;padding:8px 10px;border-radius:8px;color:#fff;cursor:pointer}
  .card{background:#0b1220;padding:12px;border-radius:10px;margin-bottom:12px}
  input,select{width:100%;padding:8px;border-radius:8px;border:1px solid #24303f;background:transparent;color:#e6eef8}
  table{width:100%;border-collapse:collapse;margin-top:8px}
  th,td{padding:6px;border-bottom:1px solid #18202b;font-size:13px;text-align:left}
  .small{font-size:12px;color:var(--muted)}
  .ok{color:#67f1a1}.bad{color:#ff9b9b}
  .mask{letter-spacing:2px}
  .center{text-align:center}
  .hidden{display:none}
  .controls{display:flex;gap:8px;flex-wrap:wrap}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>absensi sekolah (webcode)</h1>
    <nav>
      <button data-navigate="#login">login admin</button>
      <button data-navigate="#scan">scan camera</button>
      <button data-navigate="#history">riwayat</button>
    </nav>
  </header>

  <!-- halaman: login admin -->
  <section id="page-login" class="card">
    <h2 style="margin:0 0 8px">login admin</h2>
    <div class="small">masukin password admin untuk lihat id full & export</div>
    <div style="margin-top:8px">
      <input id="admin-pass" type="password" placeholder="password admin" />
      <div style="margin-top:8px;display:flex;gap:8px">
        <button id="btn-admin-login">login</button>
        <button id="btn-show-pass">tampilkan contoh password</button>
      </div>
      <div id="login-msg" class="small" style="margin-top:8px"></div>
    </div>
  </section>

  <!-- halaman: scan -->
  <section id="page-scan" class="card hidden">
    <h2 style="margin:0 0 8px">scan qr / barcode</h2>
    <div class="small">mode + sumber</div>
    <div style="display:flex;gap:8px;margin-top:8px">
      <select id="mode">
        <option value="masuk">masuk (pagi)</option>
        <option value="pulang">pulang (siang)</option>
      </select>
      <select id="source">
        <option value="camera">kamera</option>
        <option value="file">upload gambar</option>
      </select>
      <div style="flex:1" class="controls">
        <button id="start-scan">mulai scan</button>
        <button id="stop-scan" disabled>stop</button>
      </div>
    </div>

    <div id="reader" style="margin-top:12px;max-width:480px"></div>

    <div id="file-area" class="hidden" style="margin-top:8px">
      <input id="file-input" type="file" accept="image/*" />
    </div>

    <div id="scan-result" class="small" style="margin-top:8px"></div>

    <div style="margin-top:10px" class="small">
      catatan: format qr harus json: {"id":"AXQ14","name":"Nama Lengkap"}
    </div>
  </section>

  <!-- halaman: history -->
  <section id="page-history" class="card hidden">
    <h2 style="margin:0 0 8px">riwayat absensi</h2>
    <div class="small">data tersimpan di localStorage. klik eksport untuk simpan ke WPS (download json) atau csv/pdf.</div>
    <div style="margin-top:8px;display:flex;gap:8px;flex-wrap:wrap">
      <button id="export-json">simpan ke wps (download json)</button>
      <button id="export-csv">export csv</button>
      <button id="export-pdf">export pdf</button>
      <button id="clear-logs">hapus semua (local)</button>
    </div>

    <table id="log-table" style="margin-top:10px">
      <thead><tr><th>waktu</th><th>nama</th><th>status</th><th>mode</th><th>id</th></tr></thead>
      <tbody></tbody>
    </table>
  </section>
</div>

<script>
/*
  versi webcode multi-halaman.
  admin password: Yayasan (Hidayatul) Muslimin528@#$&
  DB disematkan di file (offline). format QR: {"id":"AXQ14","name":"Adi Faeda Mubarok"}
  save: localStorage + download json untuk "simpan ke wps"
*/

// ====== konfigurasi ======
const ADMIN_PASS = "Yayasan (Hidayatul) Muslimin528@#$&";
const STORAGE_KEY = "absensi_webcode_v1";
const EXPORT_ID_KEY = "export_id_pref_v1";

const DB = [
  {id: "AXQ14", name: "Adi Faeda Mubarok"},
  {id: "BRL729", name: "Agung Wijaya Pratama"},
  {id: "CTM483", name: "Alif Rizky Saputra"},
  {id: "DZF192", name: "Aisy Istiqomah Malik"},
  {id: "ELK857", name: "Amanda Putri"},
  {id: "FRQ630", name: "Amanda Juliyanti"},
  {id: "GNV245", name: "Dafa Dendika Kurniawan"},
  {id: "HJP906", name: "Kurnia Mega Saputra"},
  {id: "IKS371", name: "Madung saputra"},
  {id: "JWD528", name: "Marsel"},
  {id: "KQL804", name: "Muhammad Iqbal Atwa Ramadhan"},
  {id: "LMR662", name: "Melda Celviana"},
  {id: "MTP439", name: "Raditya Prayoga"},
  {id: "NVC510", name: "Reza Riski Saputra"},
  {id: "OPX783", name: "Ria Nopita Sari"},
  {id: "PQH234", name: "Rika Ramadhani"},
  {id: "QZN951", name: "Riani Azzahra"},
  {id: "RKB608", name: "Sandi Nur Wagesti"},
  {id: "SJF327", name: "Septi"},
  {id: "TLM590", name: "Shavix Nugraha"},
  {id: "UMC441", name: "Soni Jaya Dinata"},
  {id: "VXR726", name: "Tomi Febriansyah"},
];

const masuk_deadline = {hour:7, minute:25};   // <= diterima
const pulang_deadline = {hour:12, minute:30}; // >= diterima

// ====== util ======
function norm(s){ return String(s||"").trim().replace(/\s+/g,' ').toLowerCase(); }
function maskId(id){ if(!id) return ""; if(id.length<=2) return id[0]+'*'; return id[0] + '*'.repeat(Math.max(1,id.length-2)) + id[id.length-1]; }
function nowString(){ return new Date().toLocaleString('id-ID',{hour12:false}); }
function isBeforeOrEqual(date, hr, min){ const d=new Date(date); if(d.getHours()<hr) return true; if(d.getHours()>hr) return false; return d.getMinutes() <= min; }
function isAfterOrEqual(date, hr, min){ const d=new Date(date); if(d.getHours()>hr) return true; if(d.getHours()<hr) return false; return d.getMinutes() >= min; }

let logs = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
let isAdmin = false;
let html5QrCode = null;
let scanning = false;

// ====== routing (hash) ======
const pages = {
  "#login": document.getElementById('page-login'),
  "#scan": document.getElementById('page-scan'),
  "#history": document.getElementById('page-history')
};
function showPage(hash){
  Object.values(pages).forEach(p=> p.classList.add('hidden'));
  const p = pages[hash] || pages['#scan'];
  p.classList.remove('hidden');
  // extra: render history when showing history
  if(hash === '#history') renderLogs();
}
window.addEventListener('hashchange', ()=> showPage(location.hash));
document.querySelectorAll('[data-navigate]').forEach(btn=> btn.addEventListener('click', ()=> { location.hash = btn.dataset.navigate; }));

// default
if(!location.hash) location.hash = '#scan';
showPage(location.hash);

// ====== login admin ======
document.getElementById('btn-show-pass').addEventListener('click', ()=>{
  alert('password admin: ' + ADMIN_PASS);
});
document.getElementById('btn-admin-login').addEventListener('click', ()=>{
  const v = document.getElementById('admin-pass').value;
  const msg = document.getElementById('login-msg');
  if(v === ADMIN_PASS){
    isAdmin = true;
    msg.textContent = 'login berhasil. sekarang lo bisa lihat id full di riwayat & export full id.';
    // redirect to history
    location.hash = '#history';
  } else {
    msg.textContent = 'password salah.';
  }
});

// ====== scan logic ======
const startBtn = document.getElementById('start-scan');
const stopBtn = document.getElementById('stop-scan');
const sourceSel = document.getElementById('source');
const fileInput = document.getElementById('file-input');
const readerElem = document.getElementById('reader');
const scanResult = document.getElementById('scan-result');
const modeSel = document.getElementById('mode');

sourceSel.addEventListener('change', ()=>{
  if(sourceSel.value === 'file'){
    document.getElementById('file-area').classList.remove('hidden');
  } else {
    document.getElementById('file-area').classList.add('hidden');
  }
});

startBtn.addEventListener('click', async ()=>{
  scanResult.textContent = '';
  if(sourceSel.value === 'camera'){
    if(!html5QrCode) html5QrCode = new Html5Qrcode("reader");
    try{
      const devices = await Html5Qrcode.getCameras();
      if(!devices || devices.length===0){ scanResult.textContent = 'tidak ada kamera tersedia'; return; }
      const cameraId = devices[0].id;
      await html5QrCode.start(cameraId, {fps:10, qrbox:{width:300,height:200}},
        (decodedText)=> handleScan(decodedText),
        (err)=>{ /* ignore */ }
      );
      scanning = true;
      startBtn.disabled = true; stopBtn.disabled = false;
      scanResult.textContent = 'scanner aktif, arahkan kamera ke QR';
    }catch(e){
      scanResult.textContent = 'gagal akses kamera: ' + String(e);
    }
  } else {
    fileInput.value = null;
    fileInput.click();
  }
});

stopBtn.addEventListener('click', async ()=>{
  if(html5QrCode && scanning){ try{ await html5QrCode.stop(); }catch(e){} scanning=false; startBtn.disabled=false; stopBtn.disabled=true; readerElem.innerHTML = ''; scanResult.textContent = 'scanner dihentikan'; }
});

// file fallback
fileInput.addEventListener('change', async (ev)=>{
  const f = ev.target.files[0];
  if(!f) return;
  // if camera running, stop
  if(html5QrCode && scanning){ try{ await html5QrCode.stop(); }catch(e){} scanning=false; startBtn.disabled=false; stopBtn.disabled=true; }
  if(!html5QrCode) html5QrCode = new Html5Qrcode("reader");
  scanResult.textContent = 'memproses gambar...';
  try{
    const res = await html5QrCode.scanFileV2(f, true);
    const decodedText = (res && res.decodedText) ? res.decodedText : (Array.isArray(res) && res[0] && res[0].decodedText ? res[0].decodedText : null);
    if(!decodedText){ scanResult.textContent = 'gagal membaca QR pada gambar'; return; }
    handleScan(decodedText);
  }catch(err){
    scanResult.textContent = 'gagal scan file: ' + String(err);
  }
});

// main validation
function handleScan(text){
  let payload;
  try { payload = JSON.parse(text); } catch(e){ scanResult.textContent = 'format QR invalid: harus JSON {"id":"...","name":"..."}'; return; }
  if(!payload.id || !payload.name){ scanResult.textContent = 'QR butuh id dan name'; return; }

  const found = DB.find(s=> s.id === payload.id);
  if(!found){
    scanResult.textContent = 'id tidak terdaftar: ' + maskId(payload.id);
    addLog(payload.id, payload.name, modeSel.value, 'ditolak', 'id tidak terdaftar');
    return;
  }
  if(norm(found.name) !== norm(payload.name)){
    scanResult.textContent = 'nama tidak cocok: terdeteksi "' + payload.name + '"';
    addLog(payload.id, payload.name, modeSel.value, 'ditolak', 'nama tidak cocok');
    return;
  }

  const now = new Date();
  const mode = modeSel.value;
  if(mode === 'masuk'){
    // boleh kepagian, tolak kalau lewat deadline
    if(!isBeforeOrEqual(now, masuk_deadline.hour, masuk_deadline.minute)){
      scanResult.textContent = 'absensi masuk ditolak (lewat jam ' + String(masuk_deadline.hour).padStart(2,'0') + ':' + String(masuk_deadline.minute).padStart(2,'0') + ')';
      addLog(payload.id, payload.name, mode, 'ditolak', 'lewat waktu masuk');
      return;
    } else {
      addLog(payload.id, payload.name, mode, 'diterima', 'tepat waktu');
      scanResult.textContent = 'absensi masuk diterima: ' + payload.name + ' (tepat waktu)';
      return;
    }
  } else if(mode === 'pulang'){
    // tolak kalau sebelum deadline pulang
    if(!isAfterOrEqual(now, pulang_deadline.hour, pulang_deadline.minute)){
      scanResult.textContent = 'absensi pulang ditolak (belum mencapai jam ' + String(pulang_deadline.hour).padStart(2,'0') + ':' + String(pulang_deadline.minute).padStart(2,'0') + ')';
      addLog(payload.id, payload.name, mode, 'ditolak', 'pulang terlalu awal');
      return;
    } else {
      addLog(payload.id, payload.name, mode, 'diterima', 'tepat waktu');
      scanResult.textContent = 'absensi pulang diterima: ' + payload.name + ' (tepat waktu)';
      return;
    }
  } else {
    scanResult.textContent = 'mode tidak dikenali';
  }
}

function addLog(id,name,mode,status,note){
  const rec = {id,name,mode,status,note,time: nowString()};
  logs.push(rec);
  localStorage.setItem(STORAGE_KEY, JSON.stringify(logs));
  // auto refresh if history open
  if(location.hash === '#history') renderLogs();
}

// ====== history / export ======
const logTableBody = document.querySelector('#log-table tbody');
function renderLogs(){
  logs = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
  logTableBody.innerHTML = '';
  const showFull = isAdmin; // full id only for admin
  logs.slice().reverse().forEach(l=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${l.time}</td>
      <td>${l.name}</td>
      <td class="${l.status==='diterima' ? 'ok' : 'bad'}">${l.note || l.status}</td>
      <td>${l.mode}</td>
      <td>${ showFull ? l.id : maskId(l.id) }</td>`;
    logTableBody.appendChild(tr);
  });
}

document.getElementById('export-json').addEventListener('click', ()=>{
  const dataStr = JSON.stringify(logs, null, 2);
  const blob = new Blob([dataStr], {type:'application/json'});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'absensi_backup_wps.json';
  a.click();
  URL.revokeObjectURL(a.href);
});

document.getElementById('export-csv').addEventListener('click', ()=>{
  const rows = [['waktu','nama','status','mode','id']];
  const showFull = isAdmin;
  logs.forEach(l=> rows.push([l.time, l.name, l.status, l.mode, showFull ? l.id : maskId(l.id)]));
  const csv = rows.map(r=> r.map(c=> `"${String(c).replace(/"/g,'""')}"`).join(',')).join('\n');
  const blob = new Blob([csv], {type:'text/csv'});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob); a.download = 'absensi.csv'; a.click(); URL.revokeObjectURL(a.href);
});

document.getElementById('export-pdf').addEventListener('click', async ()=>{
  try{
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF('p','pt','a4');
    doc.setFontSize(14);
    doc.text('laporan absensi', 40, 40);
    doc.setFontSize(10);
    let y = 64;
    const showFull = isAdmin;
    doc.text('waktu - nama - status - mode - id', 40, y); y += 18;
    logs.forEach(l=>{
      if(y > 760){ doc.addPage(); y = 40; }
      const line = `${l.time} | ${l.name} | ${l.note || l.status} | ${l.mode} | ${ showFull ? l.id : maskId(l.id) }`;
      doc.text(line.slice(0,120), 40, y);
      y += 14;
    });
    doc.save('laporan_absensi.pdf');
  }catch(e){
    alert('pdf export gagal: ' + e);
  }
});

document.getElementById('clear-logs').addEventListener('click', ()=>{
  if(confirm('hapus semua log lokal?')){
    logs = []; localStorage.removeItem(STORAGE_KEY); renderLogs();
    alert('dihapus');
  }
});

// initial render
renderLogs();
  / server sekolah yang mau pake.

// selesai
</script>
</body>
</html>
