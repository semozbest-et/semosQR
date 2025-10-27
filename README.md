<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>QR Replacer — download with QR filename</title>
<style>
  body { font-family: "Poppins", Arial, sans-serif; background:#f6faf8; display:flex; justify-content:center; padding:40px 20px; color:#333; }
  .card { background:#fff; border-radius:14px; box-shadow:0 6px 20px rgba(0,0,0,0.08); max-width:620px; width:100%; padding:24px; text-align:center; }
  h2 { color:#007a3a; margin-bottom:18px; font-weight:600; }
  .controls { display:flex; gap:12px; flex-wrap:wrap; justify-content:center; margin-bottom:12px; align-items:center; }
  label{color:#006644;font-weight:500;}
  input[type="file"], input[type="range"] { padding:8px; border-radius:8px; border:1px solid #ddd; background:#fafafa; }
  #sizeValue{font-weight:600;color:#007a3a;}
  button{ padding:10px 20px; background:#00a65a;color:#fff;border:none;border-radius:8px;cursor:pointer;font-weight:600; }
  .canvas-wrapper { margin-top:14px; background:#f7f7f7; padding:10px; border-radius:10px; }
  canvas{ max-width:100%; border-radius:8px; cursor:grab; }
  canvas:active{ cursor:grabbing; }
  .instructions { text-align: left; margin-bottom: 20px; padding: 15px; background: #f0f8f4; border-radius: 8px; }
  .instructions h3 { margin-top: 0; color: #007a3a; }
  .instructions ol { padding-left: 20px; }
  .instructions li { margin-bottom: 10px; }
  @media (max-width:520px){ .controls{ flex-direction:column; } input[type="file"], input[type="range"]{ width:90%; } }
</style>
</head>
<body>
  <div class="card">
    <h2>🖼️ Replace & Move QR Code</h2>
    
    <div class="instructions">
      <h3>How to use:</h3>
      <ol>
        <li>Step 1: Upload the original image</li>
        <li>Step 2: Upload the QR code</li>
        <li>Step 3: Click Download JPG</li>
      </ol>
    </div>

    <div class="controls">
      <label for="originalUpload">Upload Original Image</label>
      <input id="originalUpload" type="file" accept="image/*">
    </div>

    <div class="controls">
      <label for="qrUpload">Upload New QR Code</label>
      <input id="qrUpload" type="file" accept="image/*">
    </div>

    <div class="controls">
      <label for="qrSize">QR Size</label>
      <input id="qrSize" type="range" min="1" max="100" value="55">
      <span id="sizeValue">55%</span>
    </div>

    <div class="controls">
      <button id="downloadBtn">Download JPG</button>
    </div>

    <div class="canvas-wrapper">
      <canvas id="canvas"></canvas>
    </div>
  </div>

<script>
  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');
  const qrUpload = document.getElementById('qrUpload');
  const originalUpload = document.getElementById('originalUpload');
  const qrSize = document.getElementById('qrSize');
  const sizeValue = document.getElementById('sizeValue');
  const downloadBtn = document.getElementById('downloadBtn');

  const baseImg = new Image();
  const qrImg = new Image();

  let qrScale = 0.55;
  let qrX = 0, qrY = 0;
  let isDragging = false, lastX = 0, lastY = 0;
  let qrFileNameBase = 'replaced-qr';
  let originalFileName = 'original';

  baseImg.src = "C:/Users/info/Downloads/ArifQR/Original.jpg";
  baseImg.onload = () => {
    canvas.width = baseImg.width;
    canvas.height = baseImg.height;
    centerQR();
    drawImages();
  };
  baseImg.onerror = () => {
    if (!canvas.width) {
      canvas.width = 800;
      canvas.height = 600;
      ctx.fillStyle = "#f0f0f0";
      ctx.fillRect(0,0,canvas.width,canvas.height);
      ctx.fillStyle = "#999";
      ctx.font = "18px Arial";
      ctx.fillText("Can't load Original.jpg from path — please upload a background image or check path.", 20, 40);
    }
  };

  qrSize.addEventListener('input', () => {
    qrScale = qrSize.value / 100;
    sizeValue.textContent = qrSize.value + '%';
    centerQR();
    drawImages();
  });

  qrUpload.addEventListener('change', (e) => {
    const file = e.target.files && e.target.files[0];
    if (!file) return;
    const rawName = file.name || 'replaced-qr';
    const nameWithoutExt = rawName.replace(/\.[^/.]+$/, "");
    const safeBase = nameWithoutExt.replace(/[^\w\-]/g, '_').slice(0, 200) || 'replaced-qr';
    qrFileNameBase = safeBase;
    const reader = new FileReader();
    reader.onload = (evt) => { qrImg.src = evt.target.result; };
    reader.readAsDataURL(file);
  });

  originalUpload.addEventListener('change', (e) => {
    const file = e.target.files && e.target.files[0];
    if (!file) return;
    const rawName = file.name || 'original';
    const nameWithoutExt = rawName.replace(/\.[^/.]+$/, "");
    originalFileName = nameWithoutExt.replace(/[^\w\-]/g, '_').slice(0, 200) || 'original';
    const reader = new FileReader();
    reader.onload = (evt) => {
      baseImg.src = evt.target.result;
      baseImg.onload = () => {
        canvas.width = baseImg.width;
        canvas.height = baseImg.height;
        centerQR();
        drawImages();
      };
    };
    reader.readAsDataURL(file);
  });

  qrImg.onload = () => {
    if (!canvas.width) {
      canvas.width = Math.max(qrImg.width * 2, 800);
      canvas.height = Math.max(qrImg.height * 2, 600);
    }
    centerQR();
    drawImages();
  };

  function centerQR() {
    if (qrImg.complete && qrImg.naturalWidth) {
      const w = qrImg.width * qrScale;
      const h = qrImg.height * qrScale;
      qrX = (canvas.width - w) / 2;
      qrY = (canvas.height - h) / 2;
    }
  }

  function drawImages() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    if (baseImg && baseImg.complete && baseImg.naturalWidth) {
      ctx.drawImage(baseImg, 0, 0, canvas.width, canvas.height);
    } else {
      ctx.fillStyle = "#f7f7f7";
      ctx.fillRect(0,0,canvas.width,canvas.height);
    }
    if (qrImg && qrImg.complete && qrImg.naturalWidth) {
      const w = qrImg.width * qrScale;
      const h = qrImg.height * qrScale;
      ctx.drawImage(qrImg, qrX, qrY, w, h);
    }
  }

  canvas.addEventListener('mousedown', (e) => {
    if (!qrImg.src) return;
    isDragging = true;
    const r = canvas.getBoundingClientRect();
    lastX = e.clientX - r.left;
    lastY = e.clientY - r.top;
  });
  window.addEventListener('mousemove', (e) => {
    if (!isDragging) return;
    const r = canvas.getBoundingClientRect();
    const x = e.clientX - r.left;
    const y = e.clientY - r.top;
    const dx = x - lastX;
    const dy = y - lastY;
    qrX += dx; qrY += dy;
    lastX = x; lastY = y;
    drawImages();
  });
  window.addEventListener('mouseup', () => { isDragging = false; });

  canvas.addEventListener('touchstart', (e) => {
    if (!qrImg.src) return;
    isDragging = true;
    const t = e.touches[0];
    const r = canvas.getBoundingClientRect();
    lastX = t.clientX - r.left;
    lastY = t.clientY - r.top;
  }, {passive: true});
  canvas.addEventListener('touchmove', (e) => {
    if (!isDragging) return;
    const t = e.touches[0];
    const r = canvas.getBoundingClientRect();
    const x = t.clientX - r.left;
    const y = t.clientY - r.top;
    const dx = x - lastX;
    const dy = y - lastY;
    qrX += dx; qrY += dy;
    lastX = x; lastY = y;
    drawImages();
  }, {passive: true});
  canvas.addEventListener('touchend', () => { isDragging = false; });

  // ✅ Download using the QR filename
  downloadBtn.addEventListener('click', () => {
    drawImages();
    canvas.toBlob((blob) => {
      if (!blob) {
        alert('Failed to create image blob.');
        return;
      }
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.download = qrFileNameBase + '.jpg'; // ✅ Use QR file name
      link.href = url;
      document.body.appendChild(link);
      link.click();
      setTimeout(() => {
        document.body.removeChild(link);
        URL.revokeObjectURL(url);
      }, 500);
    }, 'image/jpeg', 0.95);
  });
</script>
</body>
</html>
