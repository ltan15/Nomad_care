<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NomadCare — Chiếc phao sinh tồn kỹ thuật số</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
<style>
  :root{
    --bg:#171b21;
    --surface:#1f252d;
    --surface-raised:#262e38;
    --border:#333c47;
    --text:#edeae3;
    --text-muted:#8992a0;
    --signal:#ff5a2b;
    --safe:#2bb3a3;
    --danger:#e0503a;
    --radius:12px;
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;}
  body{
    background:var(--bg); color:var(--text);
    font-family:'IBM Plex Sans', system-ui, -apple-system, sans-serif;
    -webkit-font-smoothing:antialiased;
  }
  h1,h2,h3,.label-tag, nav .title{
    font-family:'Barlow Condensed','Arial Narrow',sans-serif;
    text-transform:uppercase; letter-spacing:0.05em;
  }
  .mono{ font-family:'IBM Plex Mono','Courier New',monospace; }
  svg.icon{ width:18px; height:18px; stroke:currentColor; stroke-width:1.8; fill:none; stroke-linecap:round; stroke-linejoin:round; flex-shrink:0; }

  .app{ min-height:100vh; display:flex; flex-direction:column; }

  /* ---------- TOPBAR ---------- */
  .topbar{
    display:flex; align-items:center; justify-content:space-between; gap:16px;
    padding:16px 28px; border-bottom:1px solid var(--border);
    background:linear-gradient(180deg,#1c222a,#171b21); flex-wrap:wrap;
  }
  .brand{ display:flex; align-items:center; gap:12px; }
  .brand-mark{
    width:38px; height:38px; border-radius:9px; background:var(--signal);
    display:flex; align-items:center; justify-content:center;
    font-family:'Barlow Condensed',sans-serif; font-weight:700; font-size:19px; color:#171b21;
  }
  .brand-name{ font-size:21px; font-weight:700; letter-spacing:0.07em; }
  .brand-sub{ color:var(--text-muted); font-size:12px; }

  .status-cluster{ display:flex; align-items:center; gap:10px; flex-wrap:wrap; }
  .pill{
    display:flex; align-items:center; gap:8px; background:var(--surface);
    border:1px solid var(--border); padding:7px 13px; border-radius:999px;
    font-size:12.5px; cursor:default; white-space:nowrap;
  }
  .pill.clickable{ cursor:pointer; }
  .pill.clickable:hover{ border-color:var(--signal); }
  .led{ width:8px; height:8px; border-radius:50%; background:var(--danger); box-shadow:0 0 7px var(--danger); flex-shrink:0; }
  .led.on{ background:var(--safe); box-shadow:0 0 7px var(--safe); }

  /* ---------- BODY / NAV ---------- */
  .body{ display:flex; flex:1; min-height:0; }
  .panel-nav{ width:230px; flex-shrink:0; background:var(--surface); border-right:1px solid var(--border); padding:16px 10px; display:flex; flex-direction:column; gap:5px; }
  .nav-btn{
    display:flex; align-items:center; gap:10px; padding:11px 13px; border-radius:9px;
    border:1px solid transparent; background:transparent; color:var(--text-muted);
    cursor:pointer; text-align:left; font-family:inherit; width:100%;
  }
  .nav-btn .txt{ display:flex; flex-direction:column; gap:1px; }
  .nav-btn .tag{ font-size:10.5px; color:var(--signal); letter-spacing:0.1em; }
  .nav-btn .title{ font-size:13.5px; color:var(--text); font-weight:600; }
  .nav-btn:hover{ background:var(--surface-raised); }
  .nav-btn.active{ background:var(--surface-raised); border-color:var(--signal); }
  .nav-btn.active .tag{ color:var(--signal); }
  .nav-btn.active svg.icon{ stroke:var(--signal); }

  .content{ flex:1; padding:28px 34px; overflow-y:auto; }
  .module{ display:none; max-width:940px; }
  .module.active{ display:block; animation:fadein .25s ease; }
  @keyframes fadein{ from{opacity:0; transform:translateY(4px);} to{opacity:1; transform:translateY(0);} }
  .module h2{ font-size:27px; margin:0 0 6px; }
  .module .desc{ color:var(--text-muted); font-size:14px; margin:0 0 22px; max-width:660px; line-height:1.55; }

  .card{ background:var(--surface); border:1px solid var(--border); border-radius:var(--radius); padding:19px 21px; margin-bottom:16px; }
  .card h3{ font-size:13px; color:var(--text-muted); margin:0 0 13px; letter-spacing:0.08em; }

  .chip-row{ display:flex; flex-wrap:wrap; gap:8px; }
  .chip{ background:var(--surface-raised); border:1px solid var(--border); color:var(--text); padding:9px 14px; border-radius:8px; cursor:pointer; font-size:14px; }
  .chip:hover{ border-color:var(--signal); }
  .chip.picked{ background:var(--signal); color:#171b21; border-color:var(--signal); font-weight:600; }

  select, input[type=text], input[type=number]{ background:var(--surface-raised); border:1px solid var(--border); color:var(--text); padding:9px 12px; border-radius:8px; font-size:14px; font-family:inherit; }
  label{ font-size:12px; color:var(--text-muted); display:block; margin-bottom:6px; }
  .field{ margin-bottom:14px; }
  .row{ display:flex; gap:16px; flex-wrap:wrap; }
  .row > .field{ flex:1; min-width:180px; }

  button.action{ background:var(--signal); color:#171b21; border:none; padding:10px 20px; border-radius:8px; font-weight:700; cursor:pointer; font-size:13.5px; font-family:'Barlow Condensed',sans-serif; letter-spacing:0.05em; text-transform:uppercase; }
  button.action:hover{ filter:brightness(1.08); }
  button.action:disabled{ opacity:.4; cursor:not-allowed; }
  button.ghost{ background:transparent; color:var(--text-muted); border:1px solid var(--border); padding:9px 16px; border-radius:8px; cursor:pointer; font-size:13px; }
  button.ghost:hover{ border-color:var(--signal); color:var(--text); }

  .output-card{ background:#171b21; border:1px dashed var(--safe); border-radius:8px; padding:16px; margin-top:6px; }
  .output-card .phrase{ font-size:17px; color:var(--safe); }
  .output-card .meta{ color:var(--text-muted); font-size:12px; margin-top:8px; }
  .bin{ color:var(--text-muted); font-size:11px; margin-top:6px; word-break:break-all; }

  .facility-item{ display:flex; justify-content:space-between; align-items:center; padding:10px 0; border-bottom:1px solid var(--border); font-size:14px; }
  .facility-item:last-child{ border-bottom:none; }
  .facility-item .dist{ color:var(--signal); font-weight:700; }
  .facility-item .type{ color:var(--text-muted); font-size:12px; }

  svg.mapviz{ width:100%; height:220px; background:#14181e; border-radius:8px; border:1px solid var(--border); }
  .dot-user{ fill:var(--signal); } .dot-fac{ fill:var(--safe); }

  .route-seg{ display:flex; align-items:center; gap:10px; padding:8px 0; border-bottom:1px solid var(--border); font-size:14px; }
  .route-seg:last-child{ border-bottom:none; }
  .seg-badge{ font-size:11px; padding:3px 8px; border-radius:6px; font-weight:700; }
  .seg-badge.ok{ background:rgba(43,179,163,.15); color:var(--safe); }
  .seg-badge.dead{ background:rgba(224,80,58,.15); color:var(--danger); }
  .warn-banner{ background:rgba(255,90,43,.1); border:1px solid var(--signal); color:var(--signal); padding:12px 14px; border-radius:8px; font-size:13px; margin-top:10px; }

  #qrbox{ background:#fff; padding:12px; border-radius:8px; width:fit-content; }
  .log{ font-size:13px; }
  .log-entry{ padding:8px 0; border-bottom:1px solid var(--border); color:var(--text-muted); }
  .log-entry b{ color:var(--text); }
  .log-entry.warn b{ color:var(--signal); }
  .log-entry.ok b{ color:var(--safe); }

  .toggle-switch{ display:inline-flex; align-items:center; gap:10px; cursor:pointer; user-select:none; background:var(--surface-raised); border:1px solid var(--border); padding:10px 16px; border-radius:8px; }
  .switch-track{ width:38px; height:20px; border-radius:999px; background:#3a4351; position:relative; transition:.2s; }
  .switch-track::after{ content:''; position:absolute; top:2px; left:2px; width:16px; height:16px; border-radius:50%; background:#cfd6df; transition:.2s; }
  .switch-track.on{ background:var(--danger); }
  .switch-track.on::after{ transform:translateX(18px); background:#fff; }

  .sos-btn{
    width:100%; padding:22px; border:none; border-radius:12px; cursor:pointer;
    background:var(--danger); color:#fff; font-size:20px; font-weight:800;
    font-family:'Barlow Condensed',sans-serif; letter-spacing:0.06em;
    box-shadow:0 0 0 rgba(224,80,58,0.5);
  }
  .sos-btn:hover{ filter:brightness(1.08); }
  .sos-btn:disabled{ opacity:.5; cursor:not-allowed; }
  .sos-btn.urgent{ animation:sosPulse 1.1s ease-in-out infinite; }
  @keyframes sosPulse{
    0%{ box-shadow:0 0 0 0 rgba(224,80,58,0.55); }
    70%{ box-shadow:0 0 0 16px rgba(224,80,58,0); }
    100%{ box-shadow:0 0 0 0 rgba(224,80,58,0); }
  }
  #sosCard.urgent{ border-color:var(--danger); }

  .cpr-wrap{ display:flex; gap:24px; flex-wrap:wrap; align-items:center; }
  .cpr-stats{ display:flex; gap:20px; }
  .cpr-stat{ text-align:center; }
  .cpr-stat b{ font-size:26px; color:var(--signal); display:block; }
  .cpr-stat span{ color:var(--text-muted); font-size:11px; }
  #cprHands{ animation:compress 0.545s ease-in-out infinite; animation-play-state:paused; transform-origin:center; }
  @keyframes compress{ 0%,100%{ transform:translateY(0);} 50%{ transform:translateY(13px);} }

  .flip-frame{ min-height:70px; }
  .fa-illust{ width:220px; height:110px; background:#14181e; border:1px solid var(--border); border-radius:8px; display:block; }
  .step-dots{ display:flex; gap:6px; margin-top:14px; }
  .step-dot{ width:8px; height:8px; border-radius:50%; background:var(--border); }
  .step-dot.on{ background:var(--signal); }

  footer.note{ color:var(--text-muted); font-size:12px; padding:16px 34px; border-top:1px solid var(--border); }

  @media (max-width:780px){
    .body{ flex-direction:column; }
    .panel-nav{ width:100%; flex-direction:row; overflow-x:auto; }
    .nav-btn{ flex-shrink:0; }
  }
</style>
</head>
<body>
<svg width="0" height="0" style="position:absolute">
  <symbol id="i-chat" viewBox="0 0 24 24"><path d="M4 4h16v12H8l-4 4V4z"/></symbol>
  <symbol id="i-pin" viewBox="0 0 24 24"><path d="M12 21s7-7.2 7-12a7 7 0 1 0-14 0c0 4.8 7 12 7 12z"/><circle cx="12" cy="9" r="2.4"/></symbol>
  <symbol id="i-qr" viewBox="0 0 24 24"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="15" y="15" width="2.5" height="2.5"/><rect x="19" y="15" width="2" height="2"/><rect x="15" y="19" width="2" height="2"/><rect x="19" y="19" width="2" height="2"/></symbol>
  <symbol id="i-route" viewBox="0 0 24 24"><path d="M4 19c4-9 6 5 10-4s4 4 6-4"/><circle cx="4" cy="19" r="1.4" fill="currentColor" stroke="none"/><circle cx="20" cy="11" r="1.4" fill="currentColor" stroke="none"/></symbol>
  <symbol id="i-sms" viewBox="0 0 24 24"><path d="M4 4h16v12H8l-4 4V4z"/><line x1="12" y1="8" x2="12" y2="12"/><circle cx="12" cy="15" r="0.6" fill="currentColor" stroke="none"/></symbol>
  <symbol id="i-heart" viewBox="0 0 24 24"><path d="M3 12h4l2-6 3 11 2-8 2 3h5"/></symbol>
</svg>

<div class="app">

  <div class="topbar">
    <div class="brand">
      <div class="brand-mark">N</div>
      <div>
        <div class="brand-name">NOMADCARE</div>
        <div class="brand-sub">Chiếc phao sinh tồn kỹ thuật số — cho phượt thủ ở bất kỳ nơi hẻo lánh nào trên thế giới</div>
      </div>
    </div>
    <div class="status-cluster">
      <div class="pill clickable" id="connPill" onclick="toggleConnection()">
        <span class="led" id="connLed"></span>
        <span id="connLabel">MẤT SÓNG — OFFLINE MODE</span>
      </div>
      <div class="pill" id="cloudPill">☁ <span id="cloudLabel">Chờ kết nối để đồng bộ</span></div>
    </div>
  </div>

  

  <div class="body">
    <nav class="panel-nav" id="nav">
      <button class="nav-btn active" data-mod="mod1" onclick="showModule('mod1', this)">
        <svg class="icon"><use href="#i-chat"/></svg>
        <span class="txt"><span class="tag">MOD.01</span><span class="title">Dịch y tế 1 chạm</span></span>
      </button>
      <button class="nav-btn" data-mod="mod2" onclick="showModule('mod2', this)">
        <svg class="icon"><use href="#i-pin"/></svg>
        <span class="txt"><span class="tag">MOD.02</span><span class="title">Bản đồ cứu hộ</span></span>
      </button>
      <button class="nav-btn" data-mod="mod3" onclick="showModule('mod3', this)">
        <svg class="icon"><use href="#i-qr"/></svg>
        <span class="txt"><span class="tag">MOD.03</span><span class="title">QR sinh tồn</span></span>
      </button>
      <button class="nav-btn" data-mod="mod4" onclick="showModule('mod4', this)">
        <svg class="icon"><use href="#i-route"/></svg>
        <span class="txt"><span class="tag">MOD.04</span><span class="title">Dự báo mất sóng</span></span>
      </button>
      <button class="nav-btn" data-mod="mod5" onclick="showModule('mod5', this)">
        <svg class="icon"><use href="#i-sms"/></svg>
        <span class="txt"><span class="tag">MOD.05</span><span class="title">Buddy Check-in</span></span>
      </button>
      <button class="nav-btn" data-mod="mod6" onclick="showModule('mod6', this)">
        <svg class="icon"><use href="#i-heart"/></svg>
        <span class="txt"><span class="tag">MOD.06</span><span class="title">Sơ cứu ngoại tuyến</span></span>
      </button>
    </nav>

    <main class="content">

      <!-- MOD 1 -->
      <section class="module active" id="mod1">
        <h2>Giao tiếp y tế 1 chạm</h2>
        <p class="desc">Cây quyết định tĩnh: chạm chọn bộ phận → triệu chứng → mức độ. Mỗi đường đi trỏ tới 1 mã cố định, được mã hóa nhị phân và tra ra câu dịch sẵn — không sinh câu mới, không cần mạng. Danh sách ngôn ngữ dưới đây chỉ là ví dụ minh họa; kiến trúc cho phép nạp thêm gói ngôn ngữ cho bất kỳ khu vực hẻo lánh nào trên thế giới.</p>

        <div class="card">
          <h3>NGÔN NGỮ ĐÍCH (BẢN ĐỊA)</h3>
          <label>Gõ tên ngôn ngữ (vd: "Tiếng Thái", "English", "日本語"...) hoặc chọn từ gợi ý</label>
          <input type="text" id="targetLangInput" list="langOptions" value="Tiếng Thái" oninput="renderPhrase()" placeholder="Nhập ngôn ngữ bản địa...">
          <datalist id="langOptions">
            <option value="Tiếng Thái">
            <option value="Español (Mỹ Latinh / Tây Ban Nha)">
            <option value="Français (Pháp / Bắc Phi)">
            <option value="中文 (Trung Quốc)">
            <option value="日本語 (Nhật Bản)">
            <option value="한국어 (Hàn Quốc)">
            <option value="Tiếng Lào">
            <option value="Tiếng Khmer">
            <option value="English">
          </datalist>
        </div>

        <div class="card"><h3>BƯỚC 1 — BỘ PHẬN CƠ THỂ</h3><div class="chip-row" id="step1"></div></div>
        <div class="card" id="step2card" style="display:none;"><h3>BƯỚC 2 — LOẠI TRIỆU CHỨNG</h3><div class="chip-row" id="step2"></div></div>
        <div class="card" id="step3card" style="display:none;"><h3>BƯỚC 3 — MỨC ĐỘ</h3><div class="chip-row" id="step3"></div></div>

        <div class="card" id="resultCard" style="display:none;">
          <h3>KẾT QUẢ — HIỂN THỊ CHO NHÂN VIÊN Y TẾ BẢN ĐỊA</h3>
          <div class="output-card">
            <div class="phrase mono" id="phraseOut">—</div>
            <div class="meta" id="phraseMeta">—</div>
            <div class="bin mono" id="phraseBin">—</div>
          </div>
          <p style="margin-top:10px;"><button class="ghost" onclick="resetTree()">↺ Chọn lại từ đầu</button></p>
        </div>
      </section>

      <!-- MOD 2 -->
      <section class="module" id="mod2">
        <h2>Bản đồ vector cứu hộ</h2>
        <p class="desc">Vị trí của bạn được xác định <b>tự động</b> qua GPS của thiết bị (Geolocation API) — không cần nhập gì cả. Hệ thống lập tức tìm cơ sở y tế gần nhất từ dữ liệu offline đã đóng gói theo khu vực (regional data pack), rồi vẽ đường đi + đọc chỉ đường bằng giọng nói.</p>

        <div class="card">
          <h3>VỊ TRÍ CỦA BẠN</h3>
          <p class="desc" id="geoStatus" style="margin:0 0 12px;">📍 Đang tự động xác định vị trí của bạn...</p>
          <button class="action" onclick="detectUserLocation()">🔄 Xác định lại vị trí GPS</button>

          <details style="margin-top:16px;">
            <summary style="cursor:pointer; color:var(--text-muted); font-size:12.5px;">Nâng cao: mô phỏng vị trí khác (dùng khi test hoặc trình duyệt chặn quyền định vị)</summary>
            <div class="row" style="margin-top:12px;">
              <div class="field"><label>Vị trí giả lập (toàn cầu)</label>
                <select id="userPreset" onchange="applyPreset()">
                  <option value="custom">— Tùy chỉnh —</option>
                  <optgroup label="Vùng hẻo lánh, ít/không có sóng">
                    <option value="0">Cung trekking Everest, Nepal</option>
                    <option value="1">Cao nguyên Andes, gần Cusco, Peru</option>
                    <option value="2">Rìa sa mạc Sahara, Morocco</option>
                    <option value="3">Cao nguyên Scotland, Vương quốc Anh</option>
                  </optgroup>
                  <optgroup label="Đô thị lớn, phủ sóng đầy đủ">
                    <option value="4">Trung tâm Singapore</option>
                    <option value="5">Trung tâm Tokyo, Nhật Bản</option>
                    <option value="6">Trung tâm London, Anh</option>
                    <option value="7">Trung tâm Bogotá, Colombia</option>
                  </optgroup>
                </select>
              </div>
              <div class="field"><label>Vĩ độ (lat)</label><input type="number" id="userLat" step="0.0001" value="27.8000"></div>
              <div class="field"><label>Kinh độ (lng)</label><input type="number" id="userLng" step="0.0001" value="86.7000"></div>
            </div>
            <button class="ghost" onclick="findNearest()">Dùng tọa độ này thay vì GPS thật</button>
          </details>
        </div>

        <div class="card"><h3>SƠ ĐỒ VECTOR — ĐƯỜNG ĐI TỚI CƠ SỞ GẦN NHẤT</h3><svg class="mapviz" id="mapSvg" viewBox="0 0 600 220"></svg></div>
        <div class="card"><h3>TOP 3 GẦN NHẤT</h3><div id="facilityList"><p class="desc">Đang chờ xác định vị trí...</p></div></div>

        <div class="card" id="directionsCard" style="display:none;">
          <h3>CHỈ ĐƯỜNG BẰNG HÌNH ẢNH &amp; GIỌNG NÓI</h3>
          <div id="directionSteps"></div>
          <p style="margin-top:12px;">
            <button class="action" onclick="speakDirections()">🔊 Đọc chỉ đường (loa / tai nghe)</button>
            <button class="ghost" onclick="stopSpeaking()" style="margin-left:8px;">⏹ Dừng đọc</button>
          </p>
          <p class="desc" style="margin-top:8px;">Dùng Web Speech API của trình duyệt — phát qua loa điện thoại hoặc tai nghe đang kết nối, không cần tải giọng đọc qua mạng.</p>
        </div>
      </section>

      <!-- MOD 3 -->
      <section class="module" id="mod3">
        <h2>Mã QR sinh tồn — màn hình khóa</h2>
        <p class="desc">Đóng gói hồ sơ y tế cá nhân thành mã QR. Thuật toán tự nhận diện quốc gia hiện tại (theo GPS gần nhất) và chọn đúng ngôn ngữ hiển thị — khác Apple Medical ID vốn chỉ hiện ngôn ngữ cố định của máy. Danh sách bên dưới chỉ là ví dụ; hệ thống được thiết kế để nạp thêm gói ngôn ngữ cho bất kỳ quốc gia nào.</p>

        <div class="card">
          <h3>HỒ SƠ Y TẾ CÁ NHÂN</h3>
          <div class="row">
            <div class="field"><label>Nhóm máu</label><select id="bloodType"><option>O+</option><option>O-</option><option>A+</option><option>B+</option><option>AB+</option></select></div>
            <div class="field"><label>Dị ứng thuốc</label><input type="text" id="allergyInput" value="Penicillin"></div>
          </div>
          <div class="row">
            <div class="field"><label>Tiền sử bệnh</label><input type="text" id="historyInput" value="Hen suyễn"></div>
            <div class="field"><label>Liên hệ khẩn cấp</label><input type="text" id="contactInput" value="+84 909 123 456"></div>
          </div>
          <div class="field"><label>Quốc gia hiện tại (mô phỏng GPS)</label>
            <select id="qrCountry">
              <optgroup label="Châu Á">
                <option value="vn">Việt Nam</option>
                <option value="la">Lào</option>
                <option value="th">Thái Lan</option>
                <option value="kh">Campuchia</option>
                <option value="ph">Philippines</option>
                <option value="sg">Singapore</option>
                <option value="my">Malaysia</option>
                <option value="cn">Trung Quốc</option>
                <option value="jp">Nhật Bản</option>
                <option value="kr">Hàn Quốc</option>
                <option value="np">Nepal</option>
              </optgroup>
              <optgroup label="Châu Đại Dương">
                <option value="au">Australia</option>
              </optgroup>
              <optgroup label="Châu Âu">
                <option value="uk">Vương quốc Anh</option>
                <option value="es">Tây Ban Nha</option>
                <option value="is">Iceland</option>
              </optgroup>
              <optgroup label="Châu Mỹ">
                <option value="us">Hoa Kỳ</option>
                <option value="mx">Mexico</option>
                <option value="co">Colombia</option>
                <option value="pe">Peru</option>
              </optgroup>
              <optgroup label="Châu Phi">
                <option value="ma">Morocco</option>
                <option value="ke">Kenya</option>
              </optgroup>
            </select>
          </div>
          <button class="action" onclick="generateQR()">Tạo mã QR sinh tồn</button>
        </div>

        <div class="card" id="qrResultCard" style="display:none;">
          <h3>KẾT QUẢ</h3>
          <div class="row">
            <div>
              <div id="qrbox"></div>
              <div class="meta mono" id="qrPayload" style="margin-top:8px;color:var(--text-muted);font-size:11px;max-width:220px;word-break:break-all;"></div>
            </div>
            <div style="flex:2;min-width:260px;">
              <div class="output-card">
                <div class="meta" style="margin-bottom:8px;">Nhân viên y tế bản địa quét mã, thấy:</div>
                <div class="phrase" id="qrTranslated" style="font-size:15px;line-height:1.8;"></div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- MOD 4 -->
      <section class="module" id="mod4">
        <h2>Safe Route Predictor</h2>
        <p class="desc">Đồ thị các chặng đường (nhãn "có sóng"/"mất sóng" thu thập từ cộng đồng) áp dụng được cho bất kỳ cung đường hẻo lánh nào trên thế giới. Ví dụ bên dưới minh họa một cung trekking vùng núi cao. Dijkstra tìm lộ trình ngắn nhất, sau đó quét từng chặng để cảnh báo tải trước dữ liệu vùng mất sóng.</p>
        <div class="card">
          <div class="row">
            <div class="field"><label>Điểm đi (gõ tên địa điểm)</label><input type="text" id="routeFrom" list="routeNodesList" placeholder="vd: Lukla"></div>
            <div class="field"><label>Điểm đến (gõ tên địa điểm)</label><input type="text" id="routeTo" list="routeNodesList" placeholder="vd: Everest Base Camp"></div>
            <datalist id="routeNodesList"></datalist>
          </div>
          <button class="action" onclick="computeRoute()">Tính lộ trình &amp; quét vùng sóng</button>
        </div>
        <div class="card" id="routeResult" style="display:none;">
          <h3>LỘ TRÌNH NGẮN NHẤT</h3>
          <div id="routeSegs"></div>
          <div id="routeWarn"></div>
        </div>
      </section>

      <!-- MOD 5 -->
      <section class="module" id="mod5">
        <h2>Buddy Check-in</h2>
        <p class="desc">Khi mất tín hiệu data (nhưng sóng 2G/SMS còn), ứng dụng tự soạn và gửi SMS chứa tọa độ GPS cuối cùng cho người thân/hostel. Khi có mạng lại, toàn bộ lịch sử tự đồng bộ lên máy chủ bảo mật (Hybrid Cloud).</p>
        <div class="card">
          <label class="toggle-switch" onclick="toggleSignalLoss()">
            <span class="switch-track" id="sigTrack"></span>
            <span id="sigLabel">Giả lập: đang có tín hiệu data</span>
          </label>
        </div>

        <div class="card" id="sosCard">
          <h3>TÍN HIỆU SOS KHẨN CẤP</h3>
          <p class="desc" id="sosHint" style="margin:0 0 14px;">Nếu bạn gặp nguy hiểm, chỉ cần nhấn nút bên dưới — tín hiệu SOS kèm tọa độ GPS sẽ được gửi ngay tới máy chủ hệ thống NomadCare và liên hệ khẩn cấp (người thân/hostel), kể cả khi đang mất sóng data (qua SMS).</p>
          <button class="sos-btn" id="sosBtn" onclick="sendSOS()">🆘 GỬI TÍN HIỆU SOS</button>
        </div>

        <div class="card"><h3>NHẬT KÝ SỰ KIỆN</h3><div class="log" id="buddyLog"><div class="log-entry">Chưa có sự kiện nào.</div></div></div>
      </section>

      <!-- MOD 6: FIRST AID -->
      <section class="module" id="mod6">
        <h2>Offline First Aid — Sơ cứu ngoại tuyến</h2>
        <p class="desc">Gõ hoặc nói tên chấn thương bạn đang gặp phải — hệ thống tra cứu offline (không cần mạng) và hiển thị hướng dẫn sơ cứu dạng từng bước, dễ nhìn và làm theo ngay.</p>

        <div class="card">
          <h3>TÌM CHẤN THƯƠNG</h3>
          <div class="row">
            <div class="field" style="flex:3;">
              <label>Nhập hoặc nói triệu chứng/chấn thương (vd: "gãy tay", "bỏng nước sôi"...)</label>
              <input type="text" id="faSearch" placeholder="Nhập từ khóa..." oninput="renderSearchResults()">
            </div>
            <div class="field" style="flex:0 0 auto; align-self:flex-end;">
              <button class="action" id="micBtn" onclick="startVoiceSearch()">🎤 Nói</button>
            </div>
          </div>
          <div id="voiceStatus" class="desc" style="margin:-6px 0 10px;"></div>
          <div id="searchResults" class="chip-row"></div>
        </div>

        <div class="card">
          <h3>CHẤN THƯƠNG THƯỜNG GẶP (GỢI Ý NHANH)</h3>
          <div class="chip-row" id="quickChips"></div>
        </div>

        <!-- CPR (đặc biệt: có hoạt ảnh + đếm giờ) -->
        <div class="card fa-panel" id="fa-cpr" style="display:none;">
          <div style="display:flex; align-items:center; gap:10px; margin-bottom:2px;">
            <span style="font-size:28px;">❤️‍🩹</span>
            <h3 style="margin:0;">NHỊP ÉP TIM MÔ PHỎNG (~110 LẦN/PHÚT)</h3>
          </div>
          <div class="cpr-wrap">
            <svg width="160" height="150" viewBox="0 0 160 150">
              <ellipse cx="80" cy="110" rx="55" ry="26" fill="#2a323d" stroke="#3a4351"/>
              <g id="cprHands">
                <rect x="55" y="40" width="50" height="22" rx="8" fill="#ff5a2b"/>
                <rect x="62" y="20" width="36" height="24" rx="8" fill="#ff5a2b" opacity="0.75"/>
              </g>
            </svg>
            <div class="cpr-stats">
              <div class="cpr-stat"><b id="cprTimer">30s</b><span>Thời gian còn lại</span></div>
              <div class="cpr-stat"><b id="cprCount">0</b><span>Số lần ép</span></div>
            </div>
          </div>
          <p style="margin-top:14px;"><button class="action" id="cprBtn" onclick="startCPR()">▶ Bắt đầu mô phỏng 30 giây</button>
          <button class="ghost" onclick="stopCPR(false)" style="margin-left:8px;">⏸ Dừng</button></p>
          <p class="desc" style="margin-top:10px;">Ép giữa ngực, sâu ~5cm, để lồng ngực nảy lại hoàn toàn giữa các lần ép. Gọi cấp cứu trước khi bắt đầu nếu có thể.</p>
          <p style="margin-top:10px;"><a id="cprVideoLink" href="#" target="_blank" rel="noopener" class="ghost" style="text-decoration:none; display:inline-block;">🎥 Xem video hướng dẫn CPR (cần mạng)</a></p>
        </div>

        <!-- GENERIC STEP VIEWER (dùng chung cho mọi chấn thương khác) -->
        <div class="card fa-panel" id="fa-generic" style="display:none;">
          <div class="row" style="align-items:flex-end; margin-bottom:4px;">
            <div class="field" style="flex:2; margin-bottom:0;">
              <div style="display:flex; align-items:center; gap:10px;">
                <span id="faGenericIcon" style="font-size:28px;">🩹</span>
                <h3 id="faGenericTitle" style="margin:0;">—</h3>
              </div>
            </div>
            <div class="field" style="flex:1; margin-bottom:0;">
              <label>Ngôn ngữ hiển thị</label>
              <select id="faLang" onchange="renderGenericStep()">
                <option value="vi">Tiếng Việt</option>
                <option value="en">English</option>
                <option value="es">Español</option>
              </select>
            </div>
          </div>
          <svg id="faIllustration" class="fa-illust" viewBox="0 0 220 110"></svg>
          <div class="flip-frame">
            <p id="faGenericText" class="desc" style="margin:8px 0 0;"></p>
          </div>
          <div class="step-dots" id="faGenericDots"></div>
          <p style="margin-top:12px;"><button class="ghost" onclick="prevGenericStep()">← Trước</button>
          <button class="action" onclick="nextGenericStep()" style="margin-left:8px;">Bước tiếp theo →</button></p>
          <p style="margin-top:10px;"><a id="faVideoLink" href="#" target="_blank" rel="noopener" class="ghost" style="text-decoration:none; display:inline-block;">🎥 Xem video hướng dẫn (cần mạng)</a></p>
          <p class="desc" style="margin-top:8px;">Ngôn ngữ hiển thị hiện có: Tiếng Việt, English, Español. Các gói ngôn ngữ khác sẽ được bổ sung dần, cần chuyên gia y tế bản địa xác thực trước khi phát hành.</p>
        </div>

        <div class="card" id="faEmpty">
          <p class="desc" style="margin:0;">Chưa chọn chấn thương nào. Hãy tìm kiếm, nói, hoặc chọn 1 gợi ý nhanh ở trên để xem hướng dẫn sơ cứu.</p>
        </div>
      </section>

    </main>
  </div>

  <footer class="note">Dữ liệu, tọa độ, bản dịch và hướng dẫn sơ cứu trong bản demo này mang tính minh họa thuật toán — cần đội ngũ y tế, phiên dịch và bản đồ chuyên môn xác thực trước khi triển khai thực tế.</footer>
</div>

<script>
/* ============ HYBRID CLOUD + CONNECTION TOGGLE ============ */
let connected = false;
let pendingSync = 0;

function toggleConnection(){
  connected = !connected;
  document.getElementById('connLed').classList.toggle('on', connected);
  document.getElementById('connLabel').textContent = connected ? 'CÓ MẠNG — ONLINE' : 'MẤT SÓNG — OFFLINE MODE';
  updateCloudBadge();
  if(connected && pendingSync > 0){
    const n = pendingSync;
    pendingSync = 0;
    updateCloudBadge(`Đã đồng bộ ${n} bản ghi lên máy chủ bảo mật`);
  }
}
function updateCloudBadge(customMsg){
  const label = document.getElementById('cloudLabel');
  if(customMsg){ label.textContent = customMsg; return; }
  label.textContent = connected
    ? 'Đã đồng bộ'
    : (pendingSync > 0 ? `${pendingSync} bản ghi chờ đồng bộ` : 'Chờ kết nối để đồng bộ');
}
function queueSync(){
  if(connected){ updateCloudBadge('Đã đồng bộ ngay lập tức'); }
  else { pendingSync++; updateCloudBadge(); }
}

/* ============ NAV ============ */
function showModule(id, btn){
  document.querySelectorAll('.module').forEach(m=>m.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  document.querySelectorAll('.nav-btn').forEach(b=>b.classList.remove('active'));
  btn.classList.add('active');
}

/* ============ MOD 1: DECISION TREE TRANSLATE ============ */
const tree = {
  "Bụng": { "Đau": { "Nhẹ":"S01", "Dữ dội":"S02" }, "Buồn nôn / Nôn": { "Nhẹ":"S03", "Dữ dội":"S04" } },
  "Ngực": { "Khó thở": { "Nhẹ":"S05", "Dữ dội":"S06" }, "Đau tức": { "Nhẹ":"S07", "Dữ dội":"S08" } },
  "Da": { "Nổi mẩn / Dị ứng": { "Nhẹ":"S09", "Dữ dội":"S10" }, "Vết thương hở": { "Nhẹ":"S11", "Dữ dội":"S12" } },
  "Đầu": { "Chóng mặt": { "Nhẹ":"S13", "Dữ dội":"S14" }, "Đau đầu": { "Nhẹ":"S15", "Dữ dội":"S16" } }
};
const phrases = {
  S01:{vi:"Tôi bị đau bụng nhẹ.", en:"I have mild stomach pain.", es:"Tengo dolor de estómago leve.", fr:"J'ai un léger mal de ventre.", zh:"我肚子有点疼。", ja:"お腹が少し痛いです。", ko:"배가 조금 아파요.", th:"ปวดท้องเล็กน้อย", lo:"ເຈັບທ້ອງເລັກນ້ອຍ", km:"ឈឺពោះបន្តិច"},
  S02:{vi:"Tôi bị đau bụng dữ dội, cần cấp cứu.", en:"I have severe stomach pain, need urgent help.", es:"Tengo dolor de estómago severo, necesito ayuda urgente.", fr:"J'ai un mal de ventre sévère, j'ai besoin d'aide urgente.", zh:"我肚子剧烈疼痛,需要紧急帮助。", ja:"お腹がひどく痛いです。緊急の助けが必要です。", ko:"배가 심하게 아파요. 긴급 도움이 필요해요.", th:"ปวดท้องรุนแรง ต้องการความช่วยเหลือด่วน", lo:"ເຈັບທ້ອງແຮງ ຕ້ອງການຄວາມຊ່ວຍເຫຼືອດ່ວນ", km:"ឈឺពោះខ្លាំង ត្រូវការជំនួយបន្ទាន់"},
  S03:{vi:"Tôi buồn nôn nhẹ.", en:"I feel mildly nauseous.", es:"Tengo náuseas leves.", fr:"J'ai de légères nausées.", zh:"我有点恶心。", ja:"少し吐き気がします。", ko:"약간 메스꺼워요.", th:"คลื่นไส้เล็กน้อย", lo:"ຮູ້ສຶກປວດຮາກເລັກນ້ອຍ", km:"ចង់ក្អួតបន្តិច"},
  S04:{vi:"Tôi nôn liên tục, cần giúp đỡ.", en:"I'm vomiting repeatedly, I need help.", es:"Estoy vomitando repetidamente, necesito ayuda.", fr:"Je vomis sans arrêt, j'ai besoin d'aide.", zh:"我一直在呕吐,需要帮助。", ja:"何度も吐いています。助けが必要です。", ko:"계속 토하고 있어요. 도움이 필요해요.", th:"อาเจียนบ่อยครั้ง ต้องการความช่วยเหลือ", lo:"ຮາກເລື້ອຍໆ ຕ້ອງການຄວາມຊ່ວຍເຫຼືອ", km:"ក្អួតញឹកញាប់ ត្រូវការជំនួយ"},
  S05:{vi:"Tôi hơi khó thở.", en:"I'm slightly short of breath.", es:"Me falta un poco el aire.", fr:"J'ai un peu de mal à respirer.", zh:"我有点呼吸困难。", ja:"少し息苦しいです。", ko:"숨쉬기가 조금 힘들어요.", th:"หายใจลำบากเล็กน้อย", lo:"ຫາຍໃຈຍາກເລັກນ້ອຍ", km:"ដកដង្ហើមពិបាកបន្តិច"},
  S06:{vi:"Tôi rất khó thở, cần cấp cứu ngay.", en:"I can barely breathe, need emergency help now.", es:"Apenas puedo respirar, necesito ayuda de emergencia ahora.", fr:"J'ai beaucoup de mal à respirer, j'ai besoin d'aide d'urgence maintenant.", zh:"我几乎无法呼吸,现在需要紧急帮助。", ja:"ほとんど息ができません。今すぐ緊急の助けが必要です。", ko:"숨을 거의 쉴 수 없어요. 지금 긴급 도움이 필요해요.", th:"หายใจลำบากมาก ต้องการความช่วยเหลือฉุกเฉินทันที", lo:"ຫາຍໃຈຍາກຫຼາຍ ຕ້ອງການການຊ່ວຍເຫຼືອດ່ວນທັນທີ", km:"ដកដង្ហើមពិបាកខ្លាំង ត្រូវការជំនួយបន្ទាន់ឥឡូវនេះ"},
  S07:{vi:"Tôi bị tức ngực nhẹ.", en:"I have mild chest tightness.", es:"Tengo opresión leve en el pecho.", fr:"J'ai une légère oppression dans la poitrine.", zh:"我胸口有点闷。", ja:"胸が少し苦しいです。", ko:"가슴이 약간 답답해요.", th:"แน่นหน้าอกเล็กน้อย", lo:"ແໜ້ນເອິກເລັກນ້ອຍ", km:"តឹងទ្រូងបន្តិច"},
  S08:{vi:"Tôi đau tức ngực dữ dội.", en:"I have severe chest pain.", es:"Tengo dolor de pecho severo.", fr:"J'ai une douleur thoracique sévère.", zh:"我胸口剧烈疼痛。", ja:"胸がひどく痛いです。", ko:"가슴이 심하게 아파요.", th:"เจ็บหน้าอกอย่างรุนแรง", lo:"ເຈັບເອິກຢ່າງແຮງ", km:"ឈឺទ្រូងខ្លាំង"},
  S09:{vi:"Da tôi nổi mẩn nhẹ, có thể dị ứng.", en:"I have a mild rash, possibly allergic.", es:"Tengo una erupción leve, posiblemente alérgica.", fr:"J'ai une légère éruption cutanée, peut-être allergique.", zh:"我皮肤有点起疹子,可能是过敏。", ja:"軽い発疹があります。アレルギーかもしれません。", ko:"가벼운 발진이 있어요. 알레르기일 수 있어요.", th:"มีผื่นเล็กน้อย อาจแพ้", lo:"ມີຜື່ນເລັກນ້ອຍ ອາດແພ້", km:"មានកន្ទួលបន្តិច ប្រហែលអាឡែស៊ី"},
  S10:{vi:"Tôi bị dị ứng nặng, khó thở kèm nổi mẩn.", en:"I'm having a severe allergic reaction with breathing trouble.", es:"Tengo una reacción alérgica severa con dificultad para respirar.", fr:"J'ai une réaction allergique sévère avec des difficultés respiratoires.", zh:"我有严重过敏反应,并伴有呼吸困难。", ja:"重いアレルギー反応が出ていて、呼吸が苦しいです。", ko:"심한 알레르기 반응과 호흡 곤란이 있어요.", th:"แพ้อย่างรุนแรง หายใจลำบากร่วมกับผื่น", lo:"ແພ້ຢ່າງແຮງ ຫາຍໃຈຍາກຮ່ວມກັບຜື່ນ", km:"អាឡែស៊ីធ្ងន់ធ្ងរ ដកដង្ហើមពិបាកជាមួយកន្ទួល"},
  S11:{vi:"Tôi có vết thương hở nhỏ.", en:"I have a small open wound.", es:"Tengo una pequeña herida abierta.", fr:"J'ai une petite plaie ouverte.", zh:"我有一个小的开放性伤口。", ja:"小さな開いた傷があります。", ko:"작은 열린 상처가 있어요.", th:"มีแผลเปิดเล็กน้อย", lo:"ມີບາດແຜເປີດເລັກນ້ອຍ", km:"មានរបួសបើកតូច"},
  S12:{vi:"Tôi bị chảy máu nhiều từ vết thương.", en:"I'm bleeding heavily from a wound.", es:"Estoy sangrando mucho por una herida.", fr:"Je saigne abondamment d'une blessure.", zh:"我的伤口大量出血。", ja:"傷から大量に出血しています。", ko:"상처에서 피가 많이 나요.", th:"เลือดออกมากจากบาดแผล", lo:"ເລືອດອອກຫຼາຍຈາກບາດແຜ", km:"ហូរឈាមច្រើនពីរបួស"},
  S13:{vi:"Tôi hơi chóng mặt.", en:"I feel slightly dizzy.", es:"Me siento un poco mareado.", fr:"Je me sens un peu étourdi.", zh:"我有点头晕。", ja:"少しめまいがします。", ko:"약간 어지러워요.", th:"รู้สึกวิงเวียนเล็กน้อย", lo:"ຮູ້ສຶກວິນຫົວເລັກນ້ອຍ", km:"វិលមុខបន្តិច"},
  S14:{vi:"Tôi chóng mặt dữ dội, sắp ngất.", en:"I'm severely dizzy, about to faint.", es:"Estoy muy mareado, a punto de desmayarme.", fr:"Je suis très étourdi, je vais m'évanouir.", zh:"我头晕得很严重,快要晕倒了。", ja:"めまいがひどく、倒れそうです。", ko:"심하게 어지럽고 기절할 것 같아요.", th:"วิงเวียนอย่างรุนแรง ใกล้จะเป็นลม", lo:"ວິນຫົວແຮງ ໃກ້ຊິເປັນລົມ", km:"វិលមុខខ្លាំង ជិតដួល"},
  S15:{vi:"Tôi đau đầu nhẹ.", en:"I have a mild headache.", es:"Tengo un dolor de cabeza leve.", fr:"J'ai un léger mal de tête.", zh:"我有点头痛。", ja:"少し頭痛がします。", ko:"두통이 약간 있어요.", th:"ปวดหัวเล็กน้อย", lo:"ເຈັບຫົວເລັກນ້ອຍ", km:"ឈឺក្បាលបន្តិច"},
  S16:{vi:"Tôi đau đầu dữ dội, chưa từng đau như vậy.", en:"I have a severe headache, worst I've ever had.", es:"Tengo un dolor de cabeza severo, el peor que he tenido.", fr:"J'ai un mal de tête sévère, le pire que j'aie jamais eu.", zh:"我头痛得很严重,是我经历过最严重的一次。", ja:"ひどい頭痛です。今まで経験した中で一番ひどいです。", ko:"두통이 심해요. 지금까지 겪은 것 중 가장 심해요.", th:"ปวดหัวอย่างรุนแรง ไม่เคยเป็นแบบนี้มาก่อน", lo:"ເຈັບຫົວແຮງ ບໍ່ເຄີຍເປັນແບບນີ້ມາກ່ອນ", km:"ឈឺក្បាលខ្លាំង មិនធ្លាប់ឈឺបែបនេះពីមុនមក"}
};
let treeState = { part:null, symptom:null, severity:null };
function toBinary(str){ return str.split('').map(c=>c.charCodeAt(0).toString(2).padStart(8,'0')).join(' '); }
function renderStep1(){
  const wrap = document.getElementById('step1'); wrap.innerHTML = '';
  Object.keys(tree).forEach(part=>{
    const b = document.createElement('button');
    b.className = 'chip' + (treeState.part===part ? ' picked':'');
    b.textContent = part;
    b.onclick = ()=>{ treeState.part=part; treeState.symptom=null; treeState.severity=null; renderAll(); };
    wrap.appendChild(b);
  });
}
function renderStep2(){
  const card = document.getElementById('step2card'); const wrap = document.getElementById('step2');
  if(!treeState.part){ card.style.display='none'; return; }
  card.style.display='block'; wrap.innerHTML='';
  Object.keys(tree[treeState.part]).forEach(sym=>{
    const b = document.createElement('button');
    b.className = 'chip' + (treeState.symptom===sym ? ' picked':'');
    b.textContent = sym;
    b.onclick = ()=>{ treeState.symptom=sym; treeState.severity=null; renderAll(); };
    wrap.appendChild(b);
  });
}
function renderStep3(){
  const card = document.getElementById('step3card'); const wrap = document.getElementById('step3');
  if(!treeState.symptom){ card.style.display='none'; return; }
  card.style.display='block'; wrap.innerHTML='';
  Object.keys(tree[treeState.part][treeState.symptom]).forEach(sev=>{
    const b = document.createElement('button');
    b.className = 'chip' + (treeState.severity===sev ? ' picked':'');
    b.textContent = sev;
    b.onclick = ()=>{ treeState.severity=sev; renderAll(); queueSync(); };
    wrap.appendChild(b);
  });
}
// Bảng ánh xạ: tên ngôn ngữ người dùng gõ (hoặc mã) -> mã ngôn ngữ dùng tra bảng "phrases"
const langMap = {
  "tiếng thái": "th", "thai": "th", "th": "th",
  "español (mỹ latinh / tây ban nha)": "es", "espanol": "es", "spanish": "es", "es": "es",
  "français (pháp / bắc phi)": "fr", "francais": "fr", "french": "fr", "fr": "fr",
  "中文 (trung quốc)": "zh", "中文": "zh", "chinese": "zh", "zh": "zh",
  "日本語 (nhật bản)": "ja", "日本語": "ja", "japanese": "ja", "ja": "ja",
  "한국어 (hàn quốc)": "ko", "한국어": "ko", "korean": "ko", "ko": "ko",
  "tiếng lào": "lo", "lao": "lo", "lo": "lo",
  "tiếng khmer": "km", "khmer": "km", "km": "km",
  "english": "en", "en": "en", "tiếng anh": "en"
};
/** Tìm mã ngôn ngữ từ chuỗi người dùng gõ — khớp chính xác trước, rồi khớp gần đúng */
function resolveLangCode(raw){
  const s = (raw||"").trim().toLowerCase();
  if(!s) return null;
  if(langMap[s]) return langMap[s];
  for(const key in langMap){ if(key.includes(s) || s.includes(key)) return langMap[key]; }
  return null;
}
function renderPhrase(){
  const card = document.getElementById('resultCard');
  if(!treeState.severity){ card.style.display='none'; return; }
  const id = tree[treeState.part][treeState.symptom][treeState.severity];
  const lang = resolveLangCode(document.getElementById('targetLangInput').value);
  card.style.display='block';
  if(!lang || !phrases[id][lang]){
    document.getElementById('phraseOut').textContent = '⚠ Chưa nhận diện được ngôn ngữ này trong gói dữ liệu offline. Hãy chọn từ danh sách gợi ý.';
    document.getElementById('phraseMeta').textContent = '';
    document.getElementById('phraseBin').textContent = '';
    return;
  }
  document.getElementById('phraseOut').textContent = phrases[id][lang];
  document.getElementById('phraseMeta').textContent = `Mã triệu chứng: ${id} · Đường đi: ${treeState.part} → ${treeState.symptom} → ${treeState.severity}`;
  document.getElementById('phraseBin').textContent = 'Mã hóa nhị phân: ' + toBinary(id);
}
function renderAll(){ renderStep1(); renderStep2(); renderStep3(); renderPhrase(); }
function resetTree(){ treeState = {part:null,symptom:null,severity:null}; renderAll(); }
renderAll();

/* ============ MOD 2: RESCUE MAP (Haversine) ============ */
const facilities = [
  // Đông Nam Á
  {name:"BV Chợ Rẫy", type:"Bệnh viện", country:"Việt Nam", lat:10.7539, lng:106.6579},
  {name:"BV Mahosot", type:"Bệnh viện", country:"Lào", lat:17.9667, lng:102.6000},
  {name:"Bangkok Hospital", type:"Bệnh viện", country:"Thái Lan", lat:13.7563, lng:100.5018},
  {name:"BV Calmette", type:"Bệnh viện", country:"Campuchia", lat:11.5564, lng:104.9282},
  {name:"Philippine General Hospital", type:"Bệnh viện", country:"Philippines", lat:14.5789, lng:120.9862},
  {name:"Singapore General Hospital", type:"Bệnh viện", country:"Singapore", lat:1.2789, lng:103.8355},
  {name:"Kuala Lumpur Hospital", type:"Bệnh viện", country:"Malaysia", lat:3.1725, lng:101.6952},
  // Đông Á & Nam Á
  {name:"Peking Union Medical College Hospital", type:"Bệnh viện", country:"Trung Quốc", lat:39.9139, lng:116.4136},
  {name:"St. Luke's International Hospital", type:"Bệnh viện", country:"Nhật Bản", lat:35.6685, lng:139.7797},
  {name:"Seoul National University Hospital", type:"Bệnh viện", country:"Hàn Quốc", lat:37.5796, lng:127.0034},
  {name:"Khunde Hospital", type:"Bệnh viện", country:"Nepal (Himalaya)", lat:27.8270, lng:86.7180},
  {name:"Lukla Health Post", type:"Trạm y tế", country:"Nepal (Himalaya)", lat:27.6869, lng:86.7314},
  // Châu Đại Dương
  {name:"Royal Melbourne Hospital", type:"Bệnh viện", country:"Australia", lat:-37.7986, lng:144.9559},
  // Châu Âu
  {name:"NHS Raigmore Hospital", type:"Bệnh viện", country:"Scotland (Highlands)", lat:57.4715, lng:-4.2296},
  {name:"St Thomas' Hospital", type:"Bệnh viện", country:"Anh (London)", lat:51.4980, lng:-0.1195},
  {name:"Hospital Clínico San Carlos", type:"Bệnh viện", country:"Tây Ban Nha", lat:40.4459, lng:-3.7136},
  {name:"Landspítali Reykjavík", type:"Bệnh viện", country:"Iceland", lat:64.1400, lng:-21.9330},
  // Châu Mỹ
  {name:"Denver Health Medical Center", type:"Bệnh viện", country:"Hoa Kỳ", lat:39.7295, lng:-104.9686},
  {name:"Hospital General de México", type:"Bệnh viện", country:"Mexico", lat:19.4237, lng:-99.1590},
  {name:"Hospital San Ignacio", type:"Bệnh viện", country:"Colombia", lat:4.6294, lng:-74.0656},
  {name:"Hospital Regional Cusco", type:"Bệnh viện", country:"Peru (Andes)", lat:-13.5226, lng:-71.9673},
  {name:"Posta de Salud Ollantaytambo", type:"Trạm y tế", country:"Peru (Andes)", lat:-13.2588, lng:-72.2632},
  // Châu Phi
  {name:"Hôpital Provincial Ouarzazate", type:"Bệnh viện", country:"Morocco (Sahara)", lat:30.9189, lng:-6.9107},
  {name:"Nairobi Hospital", type:"Bệnh viện", country:"Kenya", lat:-1.2921, lng:36.8219}
];
const presets = [
  {lat:27.8000,lng:86.7000}, {lat:-13.3000,lng:-72.0000}, {lat:31.0000,lng:-7.0000}, {lat:57.2000,lng:-4.6000},
  {lat:1.2897,lng:103.8501}, {lat:35.6762,lng:139.6503}, {lat:51.5074,lng:-0.1278}, {lat:4.7110,lng:-74.0721}
];
function applyPreset(){ const v = document.getElementById('userPreset').value; if(v==='custom') return; const p = presets[+v]; document.getElementById('userLat').value = p.lat; document.getElementById('userLng').value = p.lng; findNearest(); }

/** Tự động xác định vị trí thật của người dùng bằng Geolocation API của trình duyệt.
 *  Không cần người dùng nhập bất kỳ thông tin nào — thành công thì tự gọi findNearest(). */
function detectUserLocation(){
  const statusEl = document.getElementById('geoStatus');
  if(!('geolocation' in navigator)){
    statusEl.textContent = '⚠ Thiết bị/trình duyệt không hỗ trợ định vị GPS. Hãy dùng mục "Nâng cao" bên dưới để mô phỏng vị trí.';
    return;
  }
  statusEl.textContent = '📍 Đang tự động xác định vị trí của bạn...';
  navigator.geolocation.getCurrentPosition(
    pos=>{
      document.getElementById('userLat').value = pos.coords.latitude.toFixed(4);
      document.getElementById('userLng').value = pos.coords.longitude.toFixed(4);
      statusEl.textContent = `✅ Đã tự động xác định vị trí (độ chính xác ~${Math.round(pos.coords.accuracy)}m). Đang tìm cơ sở y tế gần nhất...`;
      findNearest();
    },
    err=>{
      statusEl.textContent = `⚠ Không lấy được vị trí tự động (${err.message}). Hãy mở mục "Nâng cao" bên dưới để mô phỏng vị trí, hoặc cấp quyền định vị rồi nhấn "Xác định lại vị trí GPS".`;
    },
    { enableHighAccuracy:true, timeout:10000, maximumAge:60000 }
  );
}
function haversine(lat1,lng1,lat2,lng2){
  const R=6371; const toRad=d=>d*Math.PI/180;
  const dLat=toRad(lat2-lat1), dLng=toRad(lng2-lng1);
  const a=Math.sin(dLat/2)**2 + Math.cos(toRad(lat1))*Math.cos(toRad(lat2))*Math.sin(dLng/2)**2;
  return R*2*Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
}
/** Tính góc phương vị (bearing, độ) từ điểm 1 tới điểm 2 */
function computeBearing(lat1,lng1,lat2,lng2){
  const toRad=d=>d*Math.PI/180, toDeg=r=>r*180/Math.PI;
  const dLng = toRad(lng2-lng1);
  const y = Math.sin(dLng)*Math.cos(toRad(lat2));
  const x = Math.cos(toRad(lat1))*Math.sin(toRad(lat2)) - Math.sin(toRad(lat1))*Math.cos(toRad(lat2))*Math.cos(dLng);
  return (toDeg(Math.atan2(y,x))+360)%360;
}
/** Quy đổi góc độ sang 8 hướng la bàn tiếng Việt */
function bearingToCompass(deg){
  const dirs=['Bắc','Đông Bắc','Đông','Đông Nam','Nam','Tây Nam','Tây','Tây Bắc'];
  return dirs[Math.round(deg/45)%8];
}
let lastDirectionSteps = [];
function findNearest(){
  const lat=parseFloat(document.getElementById('userLat').value), lng=parseFloat(document.getElementById('userLng').value);
  const ranked = facilities.map(f=>({...f, dist:haversine(lat,lng,f.lat,f.lng)})).sort((a,b)=>a.dist-b.dist);
  const top3 = ranked.slice(0,3);
  const list = document.getElementById('facilityList'); list.innerHTML='';
  top3.forEach(f=>{
    const d = document.createElement('div'); d.className='facility-item';
    d.innerHTML = `<div><div>${f.name}</div><div class="type">${f.type} · ${f.country}</div></div><div class="dist">${f.dist.toFixed(1)} km</div>`;
    list.appendChild(d);
  });
  const dest = top3[0];
  // Tạo điểm trung gian (waypoint) lệch nhẹ để mô phỏng một tuyến đường thực tế thay vì đường thẳng tuyệt đối
  const mid = { lat:(lat+dest.lat)/2 + (dest.lng-lng)*0.06, lng:(lng+dest.lng)/2 - (dest.lat-lat)*0.06 };
  drawMap(lat,lng,top3,{mid,dest});

  const leg1Dist = haversine(lat,lng,mid.lat,mid.lng), leg1Bear = computeBearing(lat,lng,mid.lat,mid.lng);
  const leg2Dist = haversine(mid.lat,mid.lng,dest.lat,dest.lng), leg2Bear = computeBearing(mid.lat,mid.lng,dest.lat,dest.lng);
  lastDirectionSteps = [
    `Cơ sở y tế gần nhất là ${dest.name}, cách bạn khoảng ${dest.dist.toFixed(1)} kilômét.`,
    `Bước 1: di chuyển khoảng ${leg1Dist.toFixed(1)} kilômét theo hướng ${bearingToCompass(leg1Bear)}.`,
    `Bước 2: tiếp tục khoảng ${leg2Dist.toFixed(1)} kilômét theo hướng ${bearingToCompass(leg2Bear)} để đến ${dest.name}.`,
    `Bạn đã đến nơi. Hãy tìm nhân viên y tế để được hỗ trợ ngay.`
  ];
  document.getElementById('directionsCard').style.display='block';
  const stepsDiv = document.getElementById('directionSteps'); stepsDiv.innerHTML='';
  lastDirectionSteps.forEach((s,i)=>{
    const seg = document.createElement('div'); seg.className='route-seg';
    seg.innerHTML = `<span class="seg-badge ok">${i===0?'ĐÍCH':'B.'+i}</span><span>${s}</span>`;
    stepsDiv.appendChild(seg);
  });
  queueSync();
}
function speakDirections(){
  if(!('speechSynthesis' in window)){ alert('Trình duyệt này không hỗ trợ đọc chỉ đường bằng giọng nói.'); return; }
  window.speechSynthesis.cancel();
  lastDirectionSteps.forEach(s=>{
    const u = new SpeechSynthesisUtterance(s);
    u.lang = 'vi-VN'; u.rate = 0.95;
    window.speechSynthesis.speak(u);
  });
}
function stopSpeaking(){ if('speechSynthesis' in window) window.speechSynthesis.cancel(); }
function drawMap(userLat,userLng,highlighted,routeInfo){
  const svg = document.getElementById('mapSvg');
  const allLats = facilities.map(f=>f.lat).concat(userLat), allLngs = facilities.map(f=>f.lng).concat(userLng);
  if(routeInfo){ allLats.push(routeInfo.mid.lat, routeInfo.dest.lat); allLngs.push(routeInfo.mid.lng, routeInfo.dest.lng); }
  const minLat=Math.min(...allLats), maxLat=Math.max(...allLats), minLng=Math.min(...allLngs), maxLng=Math.max(...allLngs);
  const pad=20, W=600, H=220;
  const x = lng => pad + (lng-minLng)/(maxLng-minLng||1) * (W-2*pad);
  const y = lat => H-pad - (lat-minLat)/(maxLat-minLat||1) * (H-2*pad);
  let s='';
  facilities.forEach(f=>{
    const isTop = highlighted && highlighted.some(h=>h.name===f.name);
    s += `<circle cx="${x(f.lng)}" cy="${y(f.lat)}" r="${isTop?6:4}" class="dot-fac" opacity="${isTop?1:0.5}"/>`;
    if(isTop) s += `<text x="${x(f.lng)+8}" y="${y(f.lat)+4}" fill="#8992a0" font-size="10">${f.name}</text>`;
  });
  if(routeInfo){
    const ux=x(userLng), uy=y(userLat), mx=x(routeInfo.mid.lng), my=y(routeInfo.mid.lat), dx=x(routeInfo.dest.lng), dy=y(routeInfo.dest.lat);
    s += `<polyline points="${ux},${uy} ${mx},${my} ${dx},${dy}" fill="none" stroke="#ff5a2b" stroke-width="2.2" stroke-dasharray="6 4"/>`;
    s += `<circle cx="${mx}" cy="${my}" r="3" fill="#ff5a2b"/>`;
  }
  s += `<circle cx="${x(userLng)}" cy="${y(userLat)}" r="7" class="dot-user"/><text x="${x(userLng)+9}" y="${y(userLat)-8}" fill="#ff5a2b" font-size="11" font-weight="bold">BẠN</text>`;
  svg.innerHTML = s;
}
drawMap(27.8,86.7,null); // vẽ tạm trong lúc chờ GPS
detectUserLocation(); // tự động xác định vị trí ngay khi tải trang, không cần thao tác

/* ============ MOD 3: SURVIVAL QR ============ */
const qrLabels = {
  // Châu Á
  vn:{blood:"Nhóm máu",allergy:"Dị ứng thuốc",history:"Tiền sử bệnh",contact:"Liên hệ khẩn cấp"},
  la:{blood:"ໝູ່ເລືອດ",allergy:"ແພ້ຢາ",history:"ປະຫວັດການເຈັບປ່ວຍ",contact:"ຕິດຕໍ່ສຸກເສີນ"},
  th:{blood:"กรุ๊ปเลือด",allergy:"แพ้ยา",history:"ประวัติการเจ็บป่วย",contact:"ติดต่อฉุกเฉิน"},
  kh:{blood:"ក្រុមឈាម",allergy:"អាឡែស៊ីនឹងថ្នាំ",history:"ប្រវត្តិជំងឺ",contact:"ទំនាក់ទំនងបន្ទាន់"},
  ph:{blood:"Grupo ng dugo",allergy:"Allergy sa gamot",history:"Kasaysayang medikal",contact:"Pang-emergency na contact"},
  sg:{blood:"Blood type",allergy:"Allergies",history:"Medical history",contact:"Emergency contact"},
  my:{blood:"Kumpulan darah",allergy:"Alahan ubat",history:"Sejarah perubatan",contact:"Hubungan kecemasan"},
  cn:{blood:"血型",allergy:"药物过敏",history:"病史",contact:"紧急联系人"},
  jp:{blood:"血液型",allergy:"薬物アレルギー",history:"既往歴",contact:"緊急連絡先"},
  kr:{blood:"혈액형",allergy:"약물 알레르기",history:"병력",contact:"비상 연락처"},
  np:{blood:"रक्त समूह",allergy:"औषधि एलर्जी",history:"रोगको इतिहास",contact:"आपतकालीन सम्पर्क"},
  // Châu Đại Dương
  au:{blood:"Blood type",allergy:"Allergies",history:"Medical history",contact:"Emergency contact"},
  // Châu Âu
  uk:{blood:"Blood type",allergy:"Allergies",history:"Medical history",contact:"Emergency contact"},
  es:{blood:"Grupo sanguíneo",allergy:"Alergias",history:"Historial médico",contact:"Contacto de emergencia"},
  is:{blood:"Blóðflokkur",allergy:"Ofnæmi",history:"Sjúkrasaga",contact:"Neyðarnúmer"},
  // Châu Mỹ
  us:{blood:"Blood type",allergy:"Allergies",history:"Medical history",contact:"Emergency contact"},
  mx:{blood:"Grupo sanguíneo",allergy:"Alergias",history:"Historial médico",contact:"Contacto de emergencia"},
  co:{blood:"Grupo sanguíneo",allergy:"Alergias",history:"Historial médico",contact:"Contacto de emergencia"},
  pe:{blood:"Grupo sanguíneo",allergy:"Alergias",history:"Historial médico",contact:"Contacto de emergencia"},
  // Châu Phi
  ma:{blood:"فصيلة الدم",allergy:"حساسية الدواء",history:"التاريخ المرضي",contact:"جهة اتصال الطوارئ"},
  ke:{blood:"Kundi la damu",allergy:"Mzio wa dawa",history:"Historia ya matibabu",contact:"Mawasiliano ya dharura"}
};
function generateQR(){
  const payload = { blood: document.getElementById('bloodType').value, allergy: document.getElementById('allergyInput').value, history: document.getElementById('historyInput').value, contact: document.getElementById('contactInput').value, ts: new Date().toISOString() };
  const country = document.getElementById('qrCountry').value;
  const encoded = btoa(unescape(encodeURIComponent(JSON.stringify(payload))));
  document.getElementById('qrResultCard').style.display='block';
  document.getElementById('qrPayload').textContent = 'Payload (Base64): ' + encoded;
  const box = document.getElementById('qrbox'); box.innerHTML = '';
  new QRCode(box, { text: encoded, width:150, height:150 });
  const L = qrLabels[country];
  document.getElementById('qrTranslated').innerHTML = `<b>${L.blood}:</b> ${payload.blood}<br><b>${L.allergy}:</b> ${payload.allergy}<br><b>${L.history}:</b> ${payload.history}<br><b>${L.contact}:</b> ${payload.contact}`;
  queueSync();
}

/* ============ MOD 4: SAFE ROUTE PREDICTOR (Dijkstra) ============ */
const graph = {
  "Lukla": { "Phakding":8 },
  "Phakding": { "Lukla":8, "Namche Bazaar":13 },
  "Namche Bazaar": { "Phakding":13, "Tengboche":10, "Khumjung":3 },
  "Khumjung": { "Namche Bazaar":3, "Tengboche":9 },
  "Tengboche": { "Namche Bazaar":10, "Khumjung":9, "Dingboche":11 },
  "Dingboche": { "Tengboche":11, "Lobuche":9 },
  "Lobuche": { "Dingboche":9, "Everest Base Camp":13 },
  "Everest Base Camp": { "Lobuche":13 }
};
const edgeSignal = {
  "Lukla-Phakding":"ok", "Phakding-Namche Bazaar":"ok", "Namche Bazaar-Khumjung":"ok",
  "Namche Bazaar-Tengboche":"dead", "Khumjung-Tengboche":"dead",
  "Tengboche-Dingboche":"dead", "Dingboche-Lobuche":"dead", "Lobuche-Everest Base Camp":"dead"
};
function getSignal(a,b){ return edgeSignal[a+"-"+b] || edgeSignal[b+"-"+a] || "ok"; }
function dijkstra(graph,start,end){
  const dist={}, prev={}, visited=new Set();
  Object.keys(graph).forEach(n=>dist[n]=Infinity); dist[start]=0;
  while(true){
    let u=null,best=Infinity;
    for(const n in dist){ if(!visited.has(n) && dist[n]<best){ best=dist[n]; u=n; } }
    if(u===null||u===end) break;
    visited.add(u);
    for(const v in graph[u]){ const alt=dist[u]+graph[u][v]; if(alt<dist[v]){ dist[v]=alt; prev[v]=u; } }
  }
  const path=[]; let cur=end; while(cur){ path.unshift(cur); cur=prev[cur]; }
  return { path, total: dist[end] };
}
(function initRouteNodes(){
  const nodes = Object.keys(graph);
  const dl = document.getElementById('routeNodesList');
  nodes.forEach(n=>{ dl.innerHTML += `<option value="${n}">`; });
  document.getElementById('routeFrom').value = "Lukla";
  document.getElementById('routeTo').value = "Everest Base Camp";
})();
/** Khớp tên địa điểm người dùng gõ với node có sẵn trong đồ thị offline (không phân biệt hoa/thường, khoảng trắng) */
function findGraphNode(input){
  const norm = s=>s.trim().toLowerCase();
  const target = norm(input);
  if(!target) return null;
  const keys = Object.keys(graph);
  let match = keys.find(k=>norm(k)===target);
  if(!match) match = keys.find(k=>norm(k).includes(target) || target.includes(norm(k)));
  return match || null;
}
function computeRoute(){
  const fromInput = document.getElementById('routeFrom').value, toInput = document.getElementById('routeTo').value;
  const from = findGraphNode(fromInput), to = findGraphNode(toInput);
  document.getElementById('routeResult').style.display='block';
  const segsDiv = document.getElementById('routeSegs'); segsDiv.innerHTML='';
  const warnDiv = document.getElementById('routeWarn'); warnDiv.innerHTML='';
  if(!from || !to){
    warnDiv.innerHTML = `<div class="warn-banner">⚠ Không tìm thấy "${!from?fromInput:toInput}" trong dữ liệu offline đã tải. Hãy gõ đúng tên điểm gợi ý (vd: Lukla, Namche Bazaar, Everest Base Camp...).</div>`;
    return;
  }
  const { path, total } = dijkstra(graph, from, to);
  let deadKm = 0;
  for(let i=0;i<path.length-1;i++){
    const a=path[i], b=path[i+1], km=graph[a][b], sig=getSignal(a,b);
    if(sig==='dead') deadKm += km;
    const seg = document.createElement('div'); seg.className='route-seg';
    seg.innerHTML = `<span class="seg-badge ${sig==='ok'?'ok':'dead'}">${sig==='ok'?'CÓ SÓNG':'MẤT SÓNG'}</span><span>${a} → ${b}</span><span style="color:var(--text-muted);margin-left:auto;">${km} km</span>`;
    segsDiv.appendChild(seg);
  }
  warnDiv.innerHTML = `<p style="color:var(--text-muted);font-size:13px;margin-top:10px;">Tổng quãng đường: ${total} km</p>`;
  if(deadKm>0) warnDiv.innerHTML += `<div class="warn-banner">⚠ Lộ trình có ${deadKm} km mất sóng. Bấm để tải trước dữ liệu y tế &amp; bản đồ ngoại tuyến cho đoạn này (~5MB).</div>`;
}

/* ============ MOD 5: BUDDY CHECK-IN ============ */
let signalLost = false;
function toggleSignalLoss(){
  signalLost = !signalLost;
  document.getElementById('sigTrack').classList.toggle('on', signalLost);
  document.getElementById('sigLabel').textContent = signalLost ? 'Giả lập: đã mất tín hiệu data' : 'Giả lập: đang có tín hiệu data';
  document.getElementById('sosCard').classList.toggle('urgent', signalLost);
  document.getElementById('sosBtn').classList.toggle('urgent', signalLost);
  document.getElementById('sosHint').textContent = signalLost
    ? '⚠ Bạn đang ở vùng MẤT SÓNG. Nếu gặp nguy hiểm, nhấn nút bên dưới — tín hiệu SOS sẽ được gửi qua SMS ngay cả khi không có data.'
    : 'Nếu bạn gặp nguy hiểm, chỉ cần nhấn nút bên dưới — tín hiệu SOS kèm tọa độ GPS sẽ được gửi ngay tới máy chủ hệ thống NomadCare và liên hệ khẩn cấp (người thân/hostel), kể cả khi đang mất sóng data (qua SMS).';
  const log = document.getElementById('buddyLog');
  if(log.children.length===1 && log.children[0].textContent.includes('Chưa có')) log.innerHTML='';
  const now = new Date().toLocaleTimeString('vi-VN');
  if(signalLost){
    addLog(`📡 <b>${now}</b> — Mất tín hiệu data. Chuyển sang chế độ offline.`, 'warn');
    if(connected) toggleConnection();
    setTimeout(()=>{ addLog(`✉️ <b>${now}</b> — Đã gửi SMS định vị tới liên hệ khẩn cấp đã đăng ký (người thân/hostel): Tọa độ cuối 27.8270, 86.7180.`, 'warn'); queueSync(); }, 1000);
  } else {
    if(!connected) toggleConnection();
    addLog(`🔄 <b>${now}</b> — Có mạng trở lại. Đồng bộ lịch sử GPS &amp; tin nhắn lên máy chủ bảo mật hoàn tất.`, 'ok');
  }
}
/** Nút SOS: chỉ 1 lần nhấn, gửi ngay tọa độ khẩn cấp — hoạt động cả khi mất sóng data (qua kênh SMS/vệ tinh tầm ngắn) */
function sendSOS(){
  const btn = document.getElementById('sosBtn');
  btn.disabled = true;
  const now = new Date().toLocaleTimeString('vi-VN');
  const channel = signalLost ? 'qua SMS/kênh tín hiệu tầm ngắn (không cần data)' : 'qua Internet (kết nối trực tiếp máy chủ)';
  addLog(`🆘 <b>${now}</b> — ĐÃ GỬI TÍN HIỆU SOS KHẨN CẤP ${channel}! Tọa độ hiện tại đã gửi tới máy chủ hệ thống NomadCare và liên hệ khẩn cấp (người thân/hostel).`, 'warn');
  queueSync();
  setTimeout(()=>{ btn.disabled = false; }, 6000);
}
function addLog(html, cls){ const log = document.getElementById('buddyLog'); const d = document.createElement('div'); d.className='log-entry '+cls; d.innerHTML=html; log.prepend(d); }

/* ============ MOD 6: OFFLINE FIRST AID ============ */
// CSDL chấn thương — mỗi mục có id, tên, từ khóa tìm kiếm (tags) và các bước sơ cứu.
// "special:'cpr'" trỏ tới panel hoạt ảnh riêng; các mục còn lại dùng bộ hiển thị chung (generic).
const firstAidDB = {
  cpr: {
    name: "Ngừng tim / Ngừng thở (CPR)",
    tags: ["ngừng tim","ngừng thở","hồi sức","cpr","bất tỉnh không thở","tim ngừng đập"],
    icon: "❤️‍🩹", special: "cpr",
    translations: { en:{ name:"Cardiac / Respiratory arrest (CPR)" }, es:{ name:"Paro cardíaco / respiratorio (RCP)" } }
  },
  bandage: {
    name: "Vết thương hở / Chảy máu",
    tags: ["vết thương","chảy máu","đứt tay","rách da","trầy xước"],
    icon: "🩹",
    steps: [
      "Rửa sạch vết thương bằng nước sạch, tránh chạm trực tiếp bằng tay.",
      "Đặt gạc vô trùng phủ kín vết thương.",
      "Quấn băng vòng quanh, lực vừa phải, không quá chặt.",
      "Cố định đầu băng, kiểm tra tuần hoàn (ngón tay/chân vẫn hồng ấm)."
    ],
    translations: {
      en:{ name:"Open wound / Bleeding", steps:[
        "Rinse the wound with clean water; avoid touching it directly with bare hands.",
        "Cover the wound completely with sterile gauze.",
        "Wrap a bandage around it, firm but not too tight.",
        "Secure the bandage end and check circulation (fingers/toes should stay warm and pink)."
      ]},
      es:{ name:"Herida abierta / Sangrado", steps:[
        "Enjuague la herida con agua limpia; evite tocarla directamente con las manos.",
        "Cubra la herida por completo con una gasa estéril.",
        "Envuelva un vendaje alrededor, firme pero no demasiado apretado.",
        "Asegure el extremo del vendaje y compruebe la circulación (los dedos deben mantenerse cálidos y rosados)."
      ]}
    }
  },
  splint: {
    name: "Gãy xương / Trật khớp",
    tags: ["gãy xương","trật khớp","gãy tay","gãy chân","biến dạng chi"],
    icon: "🦴",
    steps: [
      "Không cố nắn thẳng xương gãy. Giữ nguyên tư thế bị thương.",
      "Đặt vật cứng (que gỗ, thanh nẹp) dọc hai bên chi bị gãy.",
      "Cố định nẹp bằng băng/vải ở phía trên và dưới vị trí gãy.",
      "Kiểm tra không quấn quá chặt, di chuyển nạn nhân đến cơ sở y tế gần nhất."
    ],
    translations: {
      en:{ name:"Broken bone / Dislocation", steps:[
        "Do not try to straighten the broken bone. Keep the injured position as it is.",
        "Place a rigid object (stick, splint) along both sides of the injured limb.",
        "Secure the splint with bandage or cloth above and below the fracture site.",
        "Check it isn't wrapped too tight, then move the victim to the nearest medical facility."
      ]},
      es:{ name:"Fractura / Dislocación", steps:[
        "No intente enderezar el hueso roto. Mantenga la posición de la lesión tal como está.",
        "Coloque un objeto rígido (palo, férula) a ambos lados de la extremidad lesionada.",
        "Asegure la férula con vendas o tela por encima y por debajo del punto de fractura.",
        "Verifique que no esté demasiado apretado y traslade a la víctima al centro médico más cercano."
      ]}
    }
  },
  burn: {
    name: "Bỏng (lửa / nước sôi / hóa chất)",
    tags: ["bỏng","phỏng","bỏng lửa","bỏng nước sôi","bỏng hóa chất"],
    icon: "🔥",
    steps: [
      "Làm mát vết bỏng dưới vòi nước mát (không dùng đá) trong 10–20 phút.",
      "Tháo bỏ trang sức/quần áo gần vùng bỏng trước khi vết bỏng sưng lên.",
      "Che vết bỏng bằng gạc sạch, không dính; không bôi kem đánh răng/bơ/dầu lên vết bỏng.",
      "Nếu bỏng diện rộng, ở mặt/tay, hoặc bỏng sâu — tìm cấp cứu ngay."
    ],
    translations: {
      en:{ name:"Burn (fire / hot water / chemical)", steps:[
        "Cool the burn under cool running water (not ice) for 10–20 minutes.",
        "Remove jewelry/clothing near the burn before it swells.",
        "Cover the burn with clean, non-stick gauze; do not apply toothpaste, butter, or oil.",
        "If the burn is large, on the face/hands, or deep — seek emergency help immediately."
      ]},
      es:{ name:"Quemadura (fuego / agua caliente / química)", steps:[
        "Enfríe la quemadura con agua corriente fresca (no hielo) durante 10 a 20 minutos.",
        "Retire joyas/ropa cerca de la quemadura antes de que se hinche.",
        "Cubra la quemadura con gasa limpia y no adherente; no aplique pasta de dientes, mantequilla ni aceite.",
        "Si la quemadura es extensa, está en la cara/manos, o es profunda, busque ayuda de emergencia de inmediato."
      ]}
    }
  },
  foodpoison: {
    name: "Ngộ độc thực phẩm",
    tags: ["ngộ độc","ngộ độc thực phẩm","đau bụng nôn","tiêu chảy","nôn mửa"],
    icon: "🤢",
    steps: [
      "Ngừng ăn thực phẩm nghi ngờ, giữ lại mẫu nếu có thể để xác định nguyên nhân.",
      "Uống nước/dung dịch bù điện giải từng ngụm nhỏ để tránh mất nước.",
      "Nghỉ ngơi, tránh đồ ăn đặc cho đến khi hết buồn nôn.",
      "Nếu nôn/tiêu chảy kéo dài, sốt cao, hoặc có máu — tìm cấp cứu ngay."
    ],
    translations: {
      en:{ name:"Food poisoning", steps:[
        "Stop eating the suspected food; keep a sample if possible to identify the cause.",
        "Sip water or oral rehydration solution slowly to avoid dehydration.",
        "Rest and avoid solid food until nausea subsides.",
        "If vomiting/diarrhea persists, high fever, or blood appears — seek emergency care immediately."
      ]},
      es:{ name:"Intoxicación alimentaria", steps:[
        "Deje de comer el alimento sospechoso; guarde una muestra si es posible para identificar la causa.",
        "Beba agua o suero de rehidratación oral a sorbos lentos para evitar la deshidratación.",
        "Descanse y evite alimentos sólidos hasta que ceda la náusea.",
        "Si el vómito o la diarrea persisten, hay fiebre alta o sangre, busque atención de emergencia de inmediato."
      ]}
    }
  },
  heatstroke: {
    name: "Sốc nhiệt / Say nắng",
    tags: ["say nắng","sốc nhiệt","kiệt sức vì nóng","trúng nắng","cảm nắng"],
    icon: "☀️",
    steps: [
      "Đưa nạn nhân vào bóng râm hoặc nơi mát ngay lập tức.",
      "Cởi bớt quần áo, làm mát bằng khăn ướt hoặc quạt.",
      "Cho uống nước mát nếu còn tỉnh táo, từng ngụm nhỏ.",
      "Nếu lú lẫn, co giật, hoặc bất tỉnh — đây là cấp cứu, tìm trợ giúp ngay."
    ],
    translations: {
      en:{ name:"Heat stroke / Sunstroke", steps:[
        "Move the person into shade or a cool place immediately.",
        "Remove excess clothing; cool with wet cloths or a fan.",
        "Give cool water in small sips if the person is still conscious.",
        "If confused, seizing, or unconscious — this is an emergency, get help immediately."
      ]},
      es:{ name:"Golpe de calor / Insolación", steps:[
        "Traslade a la persona a la sombra o a un lugar fresco de inmediato.",
        "Quítele el exceso de ropa; enfríe con paños húmedos o un ventilador.",
        "Dele agua fresca en pequeños sorbos si sigue consciente.",
        "Si está confundida, convulsiona o pierde el conocimiento, es una emergencia; busque ayuda de inmediato."
      ]}
    }
  },
  bite: {
    name: "Rắn cắn / Côn trùng cắn",
    tags: ["rắn cắn","côn trùng cắn","ong đốt","bọ cạp cắn","kiến cắn"],
    icon: "🐍",
    steps: [
      "Giữ bình tĩnh, hạn chế cử động vùng bị cắn/đốt để làm chậm lan độc.",
      "Rửa sạch vết cắn bằng nước, tháo trang sức gần vùng bị cắn.",
      "Cố định chi bị cắn ở vị trí thấp hơn tim; không rạch, hút hay garô vết thương.",
      "Tìm cấp cứu y tế ngay; nếu có thể, ghi nhớ đặc điểm con vật để nhận diện."
    ],
    translations: {
      en:{ name:"Snake / Insect bite or sting", steps:[
        "Stay calm and limit movement of the bitten area to slow the spread of venom.",
        "Wash the bite with water; remove jewelry near the area.",
        "Keep the limb lower than the heart; do not cut, suck, or apply a tourniquet to the wound.",
        "Seek medical help immediately; if possible, remember the animal's features for identification."
      ]},
      es:{ name:"Mordedura de serpiente / Picadura de insecto", steps:[
        "Mantenga la calma y limite el movimiento de la zona afectada para retrasar la propagación del veneno.",
        "Lave la mordedura con agua; retire joyas cercanas a la zona.",
        "Mantenga la extremidad más baja que el corazón; no corte, succione ni aplique torniquete a la herida.",
        "Busque ayuda médica de inmediato; si es posible, recuerde las características del animal para identificarlo."
      ]}
    }
  },
  drowning: {
    name: "Đuối nước / Ngạt nước",
    tags: ["đuối nước","ngạt nước","sặc nước","chết đuối"],
    icon: "🌊",
    steps: [
      "Đưa nạn nhân ra khỏi nước, gọi cấp cứu ngay lập tức.",
      "Kiểm tra hô hấp; nếu không thở, bắt đầu hồi sức tim phổi (xem mục CPR).",
      "Nếu còn thở, đặt nạn nhân nằm nghiêng để nước/chất nôn thoát ra ngoài.",
      "Giữ ấm cho nạn nhân, theo dõi liên tục cho đến khi có nhân viên y tế."
    ],
    translations: {
      en:{ name:"Drowning / Near-drowning", steps:[
        "Get the person out of the water and call for emergency help immediately.",
        "Check breathing; if not breathing, begin CPR (see the CPR section).",
        "If breathing, place the person in the recovery position so water/vomit can drain out.",
        "Keep the person warm and monitor continuously until medical help arrives."
      ]},
      es:{ name:"Ahogamiento / Casi ahogamiento", steps:[
        "Saque a la persona del agua y pida ayuda de emergencia de inmediato.",
        "Compruebe la respiración; si no respira, inicie RCP (vea la sección de RCP).",
        "Si respira, colóquela en posición de recuperación para que drene el agua o el vómito.",
        "Manténgala abrigada y vigílela continuamente hasta que llegue ayuda médica."
      ]}
    }
  },
  hypothermia: {
    name: "Hạ thân nhiệt",
    tags: ["hạ thân nhiệt","lạnh cóng","nhiễm lạnh","cóng lạnh"],
    icon: "❄️",
    steps: [
      "Đưa nạn nhân vào nơi kín gió, thay quần áo ướt bằng đồ khô.",
      "Làm ấm dần bằng chăn hoặc thân nhiệt người khác — tránh làm ấm đột ngột.",
      "Cho uống nước ấm nếu còn tỉnh táo; không dùng rượu hoặc caffein.",
      "Nếu run rẩy ngừng đột ngột hoặc lú lẫn — đây là dấu hiệu nặng, cần cấp cứu ngay."
    ],
    translations: {
      en:{ name:"Hypothermia", steps:[
        "Move the person to a sheltered place; replace wet clothing with dry clothing.",
        "Warm up gradually with blankets or body heat — avoid rapid rewarming.",
        "Give warm drinks if conscious; avoid alcohol or caffeine.",
        "If shivering suddenly stops or confusion appears — this is severe, seek emergency help."
      ]},
      es:{ name:"Hipotermia", steps:[
        "Traslade a la persona a un lugar resguardado; cambie la ropa mojada por ropa seca.",
        "Caliéntela gradualmente con mantas o calor corporal; evite el recalentamiento rápido.",
        "Dele bebidas calientes si está consciente; evite el alcohol o la cafeína.",
        "Si los escalofríos cesan de repente o aparece confusión, es grave; busque ayuda de emergencia."
      ]}
    }
  },
  anaphylaxis: {
    name: "Sốc phản vệ / Dị ứng nặng",
    tags: ["sốc phản vệ","dị ứng nặng","khó thở dị ứng","phản ứng dị ứng"],
    icon: "⚠️",
    steps: [
      "Nếu nạn nhân có bút tiêm epinephrine tự động, hỗ trợ họ sử dụng theo hướng dẫn trên bút.",
      "Gọi cấp cứu ngay lập tức — đây là tình huống đe dọa tính mạng.",
      "Đặt nạn nhân nằm ngửa, kê cao chân (trừ khi khó thở thì cho ngồi).",
      "Theo dõi hô hấp liên tục; sẵn sàng hồi sức tim phổi (CPR) nếu ngừng thở."
    ],
    translations: {
      en:{ name:"Anaphylaxis / Severe allergic reaction", steps:[
        "If the person has an epinephrine auto-injector, help them use it as directed on the device.",
        "Call for emergency help immediately — this is a life-threatening situation.",
        "Lay the person on their back with legs raised (unless breathing is difficult, then let them sit).",
        "Monitor breathing continuously; be ready to perform CPR if breathing stops."
      ]},
      es:{ name:"Anafilaxia / Reacción alérgica grave", steps:[
        "Si la persona tiene un autoinyector de epinefrina, ayúdela a usarlo según las instrucciones del dispositivo.",
        "Pida ayuda de emergencia de inmediato: es una situación que pone en peligro la vida.",
        "Recuéstela boca arriba con las piernas elevadas (a menos que le cueste respirar, en cuyo caso siéntela).",
        "Vigile la respiración continuamente; esté listo para realizar RCP si deja de respirar."
      ]}
    }
  }
};
const quickChipIds = ["cpr","bandage","splint","burn","heatstroke","anaphylaxis"];
/** Xây link tìm kiếm video hướng dẫn (mở khi có mạng) — không nhúng video cụ thể để tránh sai lệch nguồn */
function videoSearchUrl(englishName){
  return 'https://www.youtube.com/results?search_query=' + encodeURIComponent(englishName + ' first aid tutorial');
}

/* --- Hình minh họa vector lũy tiến theo từng bước (mỗi bước thêm 1 chi tiết vào hình) --- */
const IL_DARK="#2a323d", IL_BORDER="#3a4351", IL_TEAL="#2bb3a3", IL_ORANGE="#ff5a2b", IL_DANGER="#e0503a", IL_TAN="#8b6f47", IL_MUTED="#8992a0", IL_WATER="#4aa8ff";
const illustrationRenderers = {
  bandage(idx){
    let wraps=''; for(let i=0;i<=idx;i++){ wraps += `<rect x="${45+i*22}" y="35" width="14" height="40" fill="${IL_TEAL}" opacity="0.55"/>`; }
    return `<rect x="20" y="35" width="180" height="40" rx="20" fill="${IL_DARK}" stroke="${IL_BORDER}"/>
      <circle cx="70" cy="55" r="9" fill="${IL_DANGER}" opacity="0.8"/>${wraps}`;
  },
  splint(idx){
    let rods='';
    if(idx>=1) rods += `<rect x="35" y="38" width="150" height="6" rx="3" fill="${IL_TAN}"/><rect x="35" y="72" width="150" height="6" rx="3" fill="${IL_TAN}"/>`;
    if(idx>=2) rods += `<rect x="60" y="30" width="10" height="55" fill="${IL_TEAL}" opacity="0.7"/><rect x="150" y="30" width="10" height="55" fill="${IL_TEAL}" opacity="0.7"/>`;
    if(idx>=3) rods += `<circle cx="110" cy="56" r="4" fill="${IL_ORANGE}"/>`;
    return `<rect x="30" y="45" width="160" height="22" rx="11" fill="${IL_DARK}" stroke="${IL_BORDER}"/>${rods}<circle cx="110" cy="20" r="5" fill="${IL_DANGER}" opacity="0.8"/>`;
  },
  burn(idx){
    const drops = idx>=0 ? `<path d="M100,15 q4,8 0,12 q-4,-4 0,-12" fill="${IL_WATER}"/><path d="M115,10 q4,8 0,12 q-4,-4 0,-12" fill="${IL_WATER}"/>` : '';
    const noJewel = idx>=1 ? `<circle cx="160" cy="55" r="7" fill="none" stroke="${IL_TEAL}" stroke-width="2"/><line x1="155" y1="50" x2="165" y2="60" stroke="${IL_DANGER}" stroke-width="2"/>` : '';
    const gauze = idx>=2 ? `<rect x="85" y="42" width="35" height="26" rx="4" fill="#eae6df" opacity="0.92"/>` : '';
    const warn = idx>=3 ? `<circle cx="190" cy="30" r="9" fill="none" stroke="${IL_DANGER}" stroke-width="2"/><text x="190" y="35" font-size="12" text-anchor="middle" fill="${IL_DANGER}">!</text>` : '';
    return `<rect x="30" y="35" width="160" height="40" rx="18" fill="${IL_DARK}" stroke="${IL_BORDER}"/>
      <ellipse cx="100" cy="55" rx="14" ry="12" fill="${idx>=2?'#eae6df':IL_ORANGE}" opacity="${idx>=2?0.9:0.85}"/>${drops}${noJewel}${gauze}${warn}`;
  },
  foodpoison(idx){
    const waves = `<path d="M60,55 q10,-10 20,0 q10,10 20,0 q10,-10 20,0" stroke="${IL_DANGER}" stroke-width="2" fill="none" opacity="0.7"/>`;
    const glass = idx>=1 ? `<rect x="150" y="40" width="16" height="24" fill="none" stroke="${IL_TEAL}" stroke-width="2"/><rect x="151" y="50" width="14" height="13" fill="${IL_TEAL}" opacity="0.5"/>` : '';
    const zzz = idx>=2 ? `<text x="35" y="30" font-size="14" fill="${IL_MUTED}">Z z z</text>` : '';
    const warn = idx>=3 ? `<circle cx="190" cy="75" r="9" fill="none" stroke="${IL_DANGER}" stroke-width="2"/><text x="190" y="80" font-size="12" text-anchor="middle" fill="${IL_DANGER}">!</text>` : '';
    return `<circle cx="90" cy="55" r="30" fill="${IL_DARK}" stroke="${IL_BORDER}"/>${waves}${glass}${zzz}${warn}`;
  },
  heatstroke(idx){
    const sun = `<circle cx="50" cy="40" r="16" fill="${IL_ORANGE}" opacity="${idx>=0?0.4:0.9}"/>`;
    const shade = idx>=0 ? `<rect x="28" y="28" width="42" height="30" fill="${IL_DARK}" opacity="0.65"/>` : '';
    const fan = idx>=1 ? `<path d="M108,45 q6,-6 12,0 q6,6 12,0" stroke="${IL_WATER}" stroke-width="2" fill="none"/>` : '';
    const glass = idx>=2 ? `<rect x="160" y="40" width="16" height="24" fill="none" stroke="${IL_TEAL}" stroke-width="2"/>` : '';
    const warn = idx>=3 ? `<circle cx="190" cy="70" r="9" fill="none" stroke="${IL_DANGER}" stroke-width="2"/><text x="190" y="75" font-size="12" text-anchor="middle" fill="${IL_DANGER}">!</text>` : '';
    return `<rect x="10" y="75" width="200" height="12" fill="${IL_BORDER}" opacity="0.4"/>${sun}${shade}${fan}${glass}${warn}`;
  },
  bite(idx){
    const limb = `<rect x="30" y="45" width="160" height="22" rx="11" fill="${IL_DARK}" stroke="${IL_BORDER}"/>`;
    const mark = `<circle cx="100" cy="56" r="3" fill="${IL_DANGER}"/><circle cx="110" cy="56" r="3" fill="${IL_DANGER}"/>`;
    const lock = idx>=0 ? `<rect x="18" y="22" width="14" height="10" rx="2" fill="none" stroke="${IL_TEAL}" stroke-width="2"/><path d="M21,22 v-4 a4,4 0 0 1 8,0 v4" fill="none" stroke="${IL_TEAL}" stroke-width="2"/>` : '';
    const drop = idx>=1 ? `<path d="M155,15 q4,8 0,12 q-4,-4 0,-12" fill="${IL_WATER}"/>` : '';
    const arrow = idx>=2 ? `<path d="M100,80 v14 m-5,-6 l5,6 l5,-6" stroke="${IL_ORANGE}" stroke-width="2" fill="none"/>` : '';
    const warn = idx>=3 ? `<circle cx="190" cy="30" r="9" fill="none" stroke="${IL_DANGER}" stroke-width="2"/><text x="190" y="35" font-size="12" text-anchor="middle" fill="${IL_DANGER}">!</text>` : '';
    return limb+mark+lock+drop+arrow+warn;
  },
  drowning(idx){
    const wave = `<path d="M10,72 q15,-15 30,0 q15,15 30,0 q15,-15 30,0 q15,15 30,0 q15,-15 30,0 q15,15 30,0" stroke="${IL_WATER}" stroke-width="3" fill="none"/>`;
    const arrowUp = idx>=0 ? `<path d="M110,40 v-14 m-5,6 l5,-6 l5,6" stroke="${IL_ORANGE}" stroke-width="2" fill="none"/>` : '';
    const person = `<circle cx="110" cy="55" r="10" fill="${idx>=1?IL_TEAL:IL_ORANGE}"/>`;
    const heart = idx>=1 ? `<text x="145" y="35" font-size="16">❤</text>` : '';
    const recovery = idx>=2 ? `<ellipse cx="65" cy="30" rx="14" ry="7" fill="${IL_TEAL}" opacity="0.6"/>` : '';
    const blanket = idx>=3 ? `<rect x="90" y="60" width="40" height="14" rx="4" fill="${IL_ORANGE}" opacity="0.6"/>` : '';
    return wave+arrowUp+person+heart+recovery+blanket;
  },
  hypothermia(idx){
    const snow = `<text x="30" y="35" font-size="18" fill="${IL_WATER}">❄</text>`;
    const roof = idx>=0 ? `<path d="M80,25 l30,-15 l30,15" stroke="${IL_TEAL}" stroke-width="2" fill="none"/>` : '';
    const person = `<circle cx="110" cy="55" r="12" fill="${idx>=1?IL_ORANGE:IL_WATER}" opacity="0.8"/>`;
    const blanket = idx>=1 ? `<rect x="95" y="65" width="30" height="10" rx="3" fill="${IL_ORANGE}" opacity="0.6"/>` : '';
    const cup = idx>=2 ? `<rect x="160" y="45" width="14" height="14" fill="none" stroke="${IL_TEAL}" stroke-width="2"/>` : '';
    const warn = idx>=3 ? `<circle cx="190" cy="30" r="9" fill="none" stroke="${IL_DANGER}" stroke-width="2"/><text x="190" y="35" font-size="12" text-anchor="middle" fill="${IL_DANGER}">!</text>` : '';
    return snow+roof+person+blanket+cup+warn;
  },
  anaphylaxis(idx){
    const person = `<circle cx="90" cy="55" r="12" fill="${IL_DANGER}" opacity="0.7"/>`;
    const injector = idx>=0 ? `<rect x="125" y="45" width="8" height="24" rx="3" fill="${IL_TEAL}"/>` : '';
    const phone = idx>=1 ? `<text x="160" y="35" font-size="16">📞</text>` : '';
    const recline = idx>=2 ? `<ellipse cx="55" cy="78" rx="20" ry="8" fill="${IL_TEAL}" opacity="0.5"/>` : '';
    const monitor = idx>=3 ? `<path d="M150,70 h10 l4,-10 l6,20 l4,-10 h10" stroke="${IL_ORANGE}" stroke-width="2" fill="none"/>` : '';
    return person+injector+phone+recline+monitor;
  }
};
function renderIllustration(id, idx){
  const svg = document.getElementById('faIllustration');
  const renderer = illustrationRenderers[id];
  svg.innerHTML = renderer ? renderer(idx) : '';
}

function initFirstAidUI(){
  const chipsWrap = document.getElementById('quickChips'); chipsWrap.innerHTML='';
  quickChipIds.forEach(id=>{
    const b = document.createElement('button'); b.className='chip'; b.textContent = `${firstAidDB[id].icon||''} ${firstAidDB[id].name}`;
    b.onclick = ()=> selectFirstAid(id);
    chipsWrap.appendChild(b);
  });
}

function renderSearchResults(){
  const q = document.getElementById('faSearch').value.trim().toLowerCase();
  const resultsWrap = document.getElementById('searchResults'); resultsWrap.innerHTML='';
  if(!q) return;
  const matches = Object.keys(firstAidDB).filter(id=>{
    const entry = firstAidDB[id];
    return entry.name.toLowerCase().includes(q) || entry.tags.some(t=>t.toLowerCase().includes(q));
  });
  if(matches.length===0){
    resultsWrap.innerHTML = '<span class="desc" style="margin:0;">Không tìm thấy — thử từ khóa khác (vd: "gãy", "bỏng", "dị ứng"...).</span>';
    return;
  }
  matches.forEach(id=>{
    const b = document.createElement('button'); b.className='chip'; b.textContent = `${firstAidDB[id].icon||''} ${firstAidDB[id].name}`;
    b.onclick = ()=> selectFirstAid(id);
    resultsWrap.appendChild(b);
  });
}

function selectFirstAid(id){
  const entry = firstAidDB[id];
  document.getElementById('faEmpty').style.display = 'none';
  document.getElementById('fa-cpr').style.display = 'none';
  document.getElementById('fa-generic').style.display = 'none';
  if(entry.special === 'cpr'){
    document.getElementById('fa-cpr').style.display = 'block';
    document.getElementById('cprVideoLink').href = videoSearchUrl(entry.translations.en.name);
  } else {
    genericState.id = id; genericState.step = 0;
    document.getElementById('fa-generic').style.display = 'block';
    renderGenericStep();
  }
  queueSync();
}

// --- Bộ hiển thị bước dùng chung (generic step viewer), hỗ trợ đa ngôn ngữ ---
const genericState = { id:null, step:0 };
/** Lấy tên + các bước theo ngôn ngữ đang chọn; mặc định về tiếng Việt nếu chưa có bản dịch */
function getLocalizedEntry(entry, lang){
  if(lang==='vi' || !entry.translations || !entry.translations[lang]) return { name:entry.name, steps:entry.steps };
  const t = entry.translations[lang];
  return { name: t.name || entry.name, steps: t.steps || entry.steps };
}
function renderGenericStep(){
  if(!genericState.id) return;
  const entry = firstAidDB[genericState.id];
  const lang = document.getElementById('faLang').value;
  const loc = getLocalizedEntry(entry, lang);
  const idx = genericState.step;
  document.getElementById('faGenericIcon').textContent = entry.icon || '🩹';
  document.getElementById('faGenericTitle').textContent = loc.name.toUpperCase();
  document.getElementById('faGenericText').textContent = `${idx+1}/${loc.steps.length}: ${loc.steps[idx]}`;
  renderIllustration(genericState.id, idx);
  const dotsWrap = document.getElementById('faGenericDots'); dotsWrap.innerHTML = '';
  loc.steps.forEach((_,i)=>{ const d=document.createElement('div'); d.className='step-dot'+(i<=idx?' on':''); dotsWrap.appendChild(d); });
  document.getElementById('faVideoLink').href = videoSearchUrl(entry.translations && entry.translations.en ? entry.translations.en.name : entry.name);
}
function nextGenericStep(){
  const entry = firstAidDB[genericState.id];
  const lang = document.getElementById('faLang').value;
  const loc = getLocalizedEntry(entry, lang);
  if(genericState.step < loc.steps.length-1){ genericState.step++; renderGenericStep(); }
}
function prevGenericStep(){
  if(genericState.step > 0){ genericState.step--; renderGenericStep(); }
}

// --- Nhận diện giọng nói (Web Speech API) ---
function startVoiceSearch(){
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  const statusEl = document.getElementById('voiceStatus');
  if(!SpeechRecognition){
    statusEl.textContent = 'Trình duyệt này không hỗ trợ nhận diện giọng nói. Hãy nhập từ khóa thủ công.';
    return;
  }
  const recognition = new SpeechRecognition();
  recognition.lang = 'vi-VN';
  recognition.interimResults = false;
  recognition.maxAlternatives = 1;
  statusEl.textContent = '🎙️ Đang nghe... hãy nói tên chấn thương.';
  recognition.start();
  recognition.onresult = (e)=>{
    const transcript = e.results[0][0].transcript;
    document.getElementById('faSearch').value = transcript;
    statusEl.textContent = `Đã nghe: "${transcript}"`;
    renderSearchResults();
  };
  recognition.onerror = ()=>{ statusEl.textContent = 'Không nhận diện được giọng nói, hãy thử lại hoặc nhập tay.'; };
  recognition.onend = ()=>{ if(statusEl.textContent.includes('Đang nghe')) statusEl.textContent=''; };
}

// --- CPR (đặc biệt: hoạt ảnh + đếm giờ + âm thanh) ---
let cprInterval=null, cprAudioCtx=null, cprTimeLeft=30, cprCount=0;
function beep(){
  if(!cprAudioCtx) return;
  const osc = cprAudioCtx.createOscillator(), gain = cprAudioCtx.createGain();
  osc.frequency.value = 880; osc.connect(gain); gain.connect(cprAudioCtx.destination);
  gain.gain.setValueAtTime(0.15, cprAudioCtx.currentTime);
  osc.start(); osc.stop(cprAudioCtx.currentTime + 0.08);
}
function startCPR(){
  if(cprInterval) return;
  cprAudioCtx = cprAudioCtx || new (window.AudioContext||window.webkitAudioContext)();
  cprTimeLeft = 30; cprCount = 0;
  document.getElementById('cprHands').style.animationPlayState = 'running';
  updateCprUI();
  cprInterval = setInterval(()=>{
    cprCount++; beep(); cprTimeLeft -= 0.545;
    if(cprTimeLeft <= 0){ stopCPR(true); }
    updateCprUI();
  }, 545);
}
function stopCPR(done){
  clearInterval(cprInterval); cprInterval = null;
  document.getElementById('cprHands').style.animationPlayState = 'paused';
  if(done) queueSync();
}
function updateCprUI(){
  document.getElementById('cprTimer').textContent = Math.max(0, Math.ceil(cprTimeLeft)) + 's';
  document.getElementById('cprCount').textContent = cprCount;
}
initFirstAidUI();
updateCloudBadge();
</script>
</body>
</html>
