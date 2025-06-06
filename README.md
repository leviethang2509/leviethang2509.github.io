<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Split Image with Grid and Excel Answer</title>
  <style>
    .grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 0; /* không có khoảng cách giữa các ảnh */
      max-width: fit-content;
      margin-top: 20px;
    }
    
    .grid img {
      width: 100%;
      height: auto;
      display: block;
      object-fit: cover;
      margin: 0;
      padding: 0;
      border: none; /* bỏ viền */
      visibility: hidden;
      image-rendering: pixelated; /* tùy chọn, giúp ảnh cắt ra khớp nhau hơn */
    }
    
    .grid .visible {
      visibility: visible;
    }
    
    .container {
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
    }
    
    #defaultImage {
      display: none;
      width: 40%;
      height: auto;
      object-fit: contain;
    }
    
  </style>

  <style>
    .flying-object {
      position: fixed;
      top: 20%;
      left: -150px;
      width: 120px;
      height: auto;
      z-index: 9999;
      pointer-events: none;
      animation: flyAcross 12s linear infinite;
    }
  
    @keyframes flyAcross {
      0% {
        transform: translateX(0);
      }
      100% {
        transform: translateX(120vw);
      }
    }
  </style>
  
</head>
<body>

<img id="defaultImage" src="./44.jpg" crossOrigin="anonymous">

<h3>Upload Excel chứa đáp án (ví dụ: "1A 10C 25B"):</h3>
<input type="file" id="uploadExcel" accept=".xlsx"><br><br>
<div class="container">
  <div class="grid" id="result"></div>
</div>

<!-- SheetJS -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<!-- Fireworks -->
<script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>

<script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.5.0/dist/confetti.browser.min.js"></script>

<script>
let answerMap = new Map();
let fireworks = null; // giữ hiệu ứng để quản lý
const soSanh = [
  "1A", "2B", "3C", "4D", "5A", "6B", "7C", "8D", "9A", "10B",
  "11C", "12D", "13A", "14B", "15C", "16D", "17A", "18B", "19C", "20D",
  "21A", "22B", "23C", "24D", "25A", "26B", "27C", "28D", "29A", "30B",
  "31C", "32D", "33A"
];

document.getElementById('uploadExcel').addEventListener('change', function (e) {
  const file = e.target.files[0];
  const reader = new FileReader();

  reader.onload = function (event) {
    const data = new Uint8Array(event.target.result);
    const workbook = XLSX.read(data, { type: 'array' });
    const sheetName = workbook.SheetNames[0];
    const worksheet = workbook.Sheets[sheetName];
    const json = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

    const answerString = json.map(row => row[0])
      .filter(cell => !!cell)
      .join(' ')
      .replace(/\s+/g, '')
      .toUpperCase();

    const regex = /(\d{1,2}|100)[A-D]/g;
    const matches = answerString.match(regex);

    answerMap.clear();

    if (matches) {
      matches.forEach(ans => {
        const number = parseInt(ans.match(/\d+/)[0]);
        const choice = ans.match(/[A-D]/)[0];
        answerMap.set(number, choice);
      });

      const img = document.getElementById('defaultImage');
      if (img.complete) {
        processImage(img);
      } else {
        img.onload = () => processImage(img);
      }
    } else {
      document.getElementById('result').innerHTML = 'Không tìm thấy đáp án phù hợp.';
    }
  };

  reader.readAsArrayBuffer(file);
});
function processImage(img) {
  const rows = 11;
  const cols = 3;
  const tileWidth = img.naturalWidth / cols;
  const tileHeight = img.naturalHeight / rows;

  const result = document.getElementById('result');
  result.innerHTML = '';

  let count = 1;
  let correctAnswers = 0;
  let wrongAnswers = [];

  // Duyệt qua các ô ảnh và so sánh đáp án
  for (let y = 0; y < rows; y++) {
    for (let x = 0; x < cols; x++) {
      const canvas = document.createElement('canvas');
      canvas.width = tileWidth;
      canvas.height = tileHeight;
      const ctx = canvas.getContext('2d');

      ctx.drawImage(img, x * tileWidth, y * tileHeight, tileWidth, tileHeight, 0, 0, tileWidth, tileHeight);

      const imgElement = document.createElement('img');
      imgElement.src = canvas.toDataURL();

      const container = document.createElement('div');
      container.appendChild(imgElement);

      const currentAnswer = answerMap.get(count);
      const fullAnswer = `${count}${currentAnswer}`;

      if (currentAnswer && soSanh.includes(fullAnswer)) {
        imgElement.classList.add('visible');
        correctAnswers++;
      } else {
        wrongAnswers.push(count);
      }

      result.appendChild(container);
      count++;
    }
  }

  // Tổng kết và chỉ hiển thị hiệu ứng khi tất cả đáp án đúng
  if (correctAnswers === 33) {
    alert('Chúc mừng! Bạn đã hoàn thành thử thách!');
    launchPlaneFireworks();  // Hiển thị hiệu ứng máy bay và pháo hoa khi đúng
  } else {
    if (wrongAnswers.length > 0) {
      alert(`Bạn đã trả lời sai các câu: ${wrongAnswers.join(', ')}. Hãy cố gắng lần sau nhé!`);
    }
    stopFireworks();  // Dừng hiệu ứng nếu có đang chạy
  }
}

function launchPlaneFireworks() {
  const airplane = document.getElementById('airplane');
  airplane.style.display = 'block';  // Hiển thị máy bay
  
  // Phun pháo hoa phía sau máy bay liên tục
  const duration = 12000;
  const interval = 150;
  const startTime = Date.now();
  
  const intervalId = setInterval(() => {
    const rect = airplane.getBoundingClientRect();
    
    confetti({
      particleCount: 4,
      angle: 180,
      spread: 40,
      startVelocity: 30,
      origin: {
        x: (rect.left + rect.width / 2) / window.innerWidth,
        y: (rect.top + rect.height / 2) / window.innerHeight,
      }
    });

    // Dừng máy bay và hiệu ứng sau thời gian
    if (Date.now() - startTime > duration) {
      clearInterval(intervalId);
      airplane.style.display = 'none';  // Ẩn máy bay
    }
  }, interval);

  // Bắn pháo hoa toàn màn hình khi máy bay xuất hiện
  (function frame() {
    confetti({
      particleCount: 5,
      angle: 60,
      spread: 55,
      origin: { x: 0 },
    });
    confetti({
      particleCount: 5,
      angle: 120,
      spread: 55,
      origin: { x: 1 },
    });

    if (Date.now() - startTime < duration) {
      requestAnimationFrame(frame);
    }
  })();
}

function stopFireworks() {
  // Dừng tất cả hiệu ứng pháo hoa và máy bay nếu sai đáp án
  document.getElementById('airplane').style.display = 'none';
  document.getElementById('helicopter').classList.remove('active');
  confetti.reset();  // Dừng pháo hoa nếu có đang chạy
}

  </script>
</body>
<img src="./—Pngtree—jet fighter illustration_8476956.png" class="flying-object airplane" id="airplane" />
<img src="" class="flying-object helicopter" style="top: 30%;" id="helicopter" />


</html>
