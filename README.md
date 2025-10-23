<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>M có thích t không?</title>
  <style>
    :root{
      --bg1:#ffe6f0;
      --bg2:#fff1f6;
      --card:#ffffff;
      --accent:#ff6b9a;
      --muted:#6b6b6b;
    }
    html,body{
      height:100%;
      margin:0;
      font-family: "Helvetica Neue", Arial, sans-serif;
      background: linear-gradient(135deg,var(--bg1),var(--bg2));
      -webkit-font-smoothing:antialiased;
      -moz-osx-font-smoothing:grayscale;
    }

    .wrap{
      min-height:100%;
      display:flex;
      align-items:center;
      justify-content:center;
      padding:24px;
      box-sizing:border-box;
    }

    .card{
      width:min(720px, 96%);
      max-width:720px;
      background:var(--card);
      border-radius:18px;
      padding:32px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.08);
      text-align:center;
      position:relative;
      overflow:hidden;
    }

    h1{
      margin:0 0 18px 0;
      font-size:28px;
      color:#222;
      line-height:1.2;
      letter-spacing:0.3px;
      opacity:0;
      transform: translateY(8px);
      animation: appear 450ms forwards;
    }

    p.sub{
      margin:0 0 22px 0;
      color:var(--muted);
      font-size:14px;
    }

    .btn-row{
      display:flex;
      gap:18px;
      justify-content:center;
      margin-top:12px;
      position:relative;
      height:64px;
      align-items:center;
    }

    button{
      border:0;
      padding:12px 22px;
      border-radius:999px;
      font-weight:700;
      cursor:pointer;
      outline:none;
      box-shadow: 0 6px 18px rgba(0,0,0,0.08);
      transition: transform 170ms ease, box-shadow 170ms ease;
      user-select:none;
    }

    button:active{ transform: translateY(2px); }

    .yes{
      background: linear-gradient(90deg, #ff8fbf, #ff6b9a);
      color:white;
      font-size:16px;
    }

    .no{
      background:transparent;
      border:2px solid rgba(0,0,0,0.06);
      color:#333;
      font-size:16px;
      position:relative;
    }

    .hint{
      font-size:13px;
      color:#999;
      margin-top:14px;
    }

    .small-heart{
      position:absolute;
      bottom:12px;
      right:12px;
      font-size:18px;
      opacity:0.12;
    }

    @keyframes appear{
      to{ opacity:1; transform:translateY(0); }
    }

    /* floating heart animation container */
    .hearts {
      position:absolute;
      left:0; right:0; top:0; bottom:0;
      pointer-events:none;
      overflow:hidden;
    }

    .heart {
      position:absolute;
      font-size:20px;
      animation: floatUp linear forwards;
      opacity:0.8;
      transform-origin:center;
    }

    @keyframes floatUp {
      0% { transform: translateY(0) scale(0.8); opacity:0.9; }
      100% { transform: translateY(-380px) scale(1.2); opacity:0; }
    }

    /* final message style */
    .final {
      margin-top:18px;
      font-size:20px;
      font-weight:700;
      color:var(--accent);
      animation: appear 400ms forwards;
    }

    /* small responsive tweaks */
    @media (max-width:460px){
      h1{ font-size:22px; }
      button{ font-size:15px; padding:10px 16px; }
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card" id="card">
      <div class="hearts" id="hearts"></div>

      <h1 id="prompt">M có thích t không?</h1>
      <p class="sub" id="sub">Bấm thử xem nha 😳</p>

      <div class="btn-row" id="btnRow">
        <button class="yes" id="yesBtn">Có</button>
        <button class="no" id="noBtn">Không</button>
      </div>

      <div class="hint" id="hint">Bạn ấn "Không" nhiều quá thì mình sẽ buồn đó...</div>
      <div class="small-heart">💌</div>
    </div>
  </div>

  <script>
    // --- Nội dung câu cho mỗi lần bấm "Không" ---
    const noMessages = [
      // Lần 1 (nếu bạn muốn đổi, sửa trực tiếp ở đây)
      "Suy nghĩ kỹ chưa đó 😳",
      // Lần 2..10 theo bạn cung cấp:
      "chắc chx",
      "có lần 1 thôi đó",
      "đừng mà",
      "t ko vui nha",
      "t mệt á",
      "nghỉ kỹ chx",
      "thoi mà",
      "nút có kìa",
      "thoi vậy"
    ];

    const finalNoMessage = "M ko thích t nhưng t vẫn thích m 💔";
    const finalYesMessage = "Cuối năm t tỏ tình m ❤️";

    // state
    let noCount = 0; // đã bấm "Không" bao nhiêu lần

    const promptEl = document.getElementById('prompt');
    const subEl = document.getElementById('sub');
    const yesBtn = document.getElementById('yesBtn');
    const noBtn = document.getElementById('noBtn');
    const heartsEl = document.getElementById('hearts');
    const btnRow = document.getElementById('btnRow');

    // helper: tạo tim bay (mỗi lần bấm có/không sẽ tạo)
    function spawnHeart(x, y){
      const el = document.createElement('div');
      el.className = 'heart';
      el.textContent = ['💖','💘','💕','💝'][Math.floor(Math.random()*4)];
      // randomize size and horizontal drift
      const size = 14 + Math.random()*26;
      el.style.fontSize = size + 'px';
      el.style.left = (x - size/2) + 'px';
      el.style.top = (y - size/2) + 'px';
      el.style.opacity = 0.9;
      el.style.animationDuration = (3000 + Math.random()*1600) + 'ms';
      heartsEl.appendChild(el);
      setTimeout(()=> el.remove(), 5200);
    }

    // move the "No" button to a random position inside btnRow
    function jumpNoButton(){
      // get dimensions
      const rect = btnRow.getBoundingClientRect();
      const noRect = noBtn.getBoundingClientRect();

      // available area (within btnRow)
      const pad = 6;
      const maxX = rect.width - noRect.width - pad;
      const maxY = rect.height - noRect.height - pad;

      // random position
      const rx = Math.max(0, Math.floor(Math.random() * (maxX + 1)));
      const ry = Math.max(0, Math.floor(Math.random() * (maxY + 1)));

      // apply transform to noBtn (relative to its container)
      noBtn.style.position = 'relative';
      noBtn.style.transform = `translate(${rx}px, ${ry}px)`;
      noBtn.style.transition = 'transform 260ms cubic-bezier(.2,.9,.3,1)';
    }

    // reset positions (for final states)
    function resetButtonPositions(){
      noBtn.style.transform = '';
      noBtn.style.position = '';
    }

    // handle No click
    noBtn.addEventListener('click', (e) => {
      noCount++;
      // spawn heart from click position (nice effect)
      const rect = card.getBoundingClientRect ? card.getBoundingClientRect() : {left:0,top:0};
      spawnHeart(e.clientX - rect.left, e.clientY - rect.top);

      if (noCount <= 10){
        // show corresponding message (lần 1 -> noMessages[0], ...)
        const msg = noMessages[noCount - 1] || "Hmmm...";
        promptEl.textContent = msg;
        subEl.textContent = `Lần ${noCount} rồi đó...`;
        // move the button (nghịch ngợm)
        jumpNoButton();
      } else {
        // lần thứ 11 -> final message
        promptEl.textContent = finalNoMessage;
        subEl.textContent = "";
        resetButtonPositions();
        // disable no button
        noBtn.disabled = true;
        yesBtn.disabled = true;
        noBtn.style.cursor = 'default';
        yesBtn.style.cursor = 'default';
        // small heart shower
        for(let i=0;i<12;i++){
          setTimeout(()=> spawnHeart(Math.random()*btnRow.clientWidth, btnRow.clientHeight), i*120);
        }
      }
    });

    // handle Yes click
    yesBtn.addEventListener('click', (e) => {
      spawnHeart(e.clientX - card.getBoundingClientRect().left, e.clientY - card.getBoundingClientRect().top);
      promptEl.textContent = finalYesMessage;
      subEl.textContent = "Đợi cuối năm nha 🥺✨";
      // show some hearts
      for(let i=0;i<10;i++){
        setTimeout(()=> spawnHeart(Math.random()*btnRow.clientWidth, btnRow.clientHeight), i*90);
      }
      // disable buttons to avoid further interaction
      noBtn.disabled = true;
      yesBtn.disabled = true;
      noBtn.style.cursor = 'default';
      yesBtn.style.cursor = 'default';
      resetButtonPositions();
    });

    // small UX: if user moves mouse near "Không", move it away a bit (playful)
    noBtn.addEventListener('mouseenter', (e) => {
      // only nudge if not yet final
      if (noCount < 11){
        // 40% chance to jump to avoid being too mean
        if (Math.random() < 0.6) jumpNoButton();
      }
    });

    // accessible: allow keyboard pressing of Yes
    yesBtn.addEventListener('keyup', (e) => {
      if (e.key === 'Enter') yesBtn.click();
    });

    // reference to card for spawn offsets
    const card = document.getElementById('card');

    // Small initial pop hearts
    setTimeout(()=> spawnHeart(60, 160), 400);
    setTimeout(()=> spawnHeart(120, 120), 650);

  </script>
</body>
</html>
