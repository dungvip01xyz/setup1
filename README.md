<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Hiển thị links từ data.txt</title>
  <style>
    body { font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; padding: 20px; max-width: 900px; margin: auto; }
    h1 { font-size: 1.4rem; margin-bottom: 8px; }
    #status { color: #555; margin-bottom: 12px; }
    ul { padding-left: 1.2rem; }
    li { margin: 6px 0; }
    a { word-break: break-all; text-decoration: none; }
    a:hover { text-decoration: underline; }
    .btn { display:inline-block; margin:8px 0; padding:6px 10px; border-radius:6px; background:#eee; cursor:pointer; }
  </style>
</head>
<body>
  <h1>Links từ <code>data.txt</code></h1>
  <div id="status">Đang tải...</div>
  <div>
    <button id="refresh" class="btn">Tải lại</button>
    <button id="openAll" class="btn">Mở tất cả trong tab mới</button>
  </div>
  <ul id="list"></ul>

  <script>
    // Xử lý an toàn: escape HTML để tránh injection
    function escapeHtml(s) {
      return s.replace(/[&<>"']/g, (c) => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'})[c]);
    }

    // Regex lấy URL (http/https)
    const urlRegex = /https?:\/\/[^\s"'<>`]+/ig;

    async function loadAndShow() {
      const status = document.getElementById('status');
      const listEl = document.getElementById('list');
      status.textContent = 'Đang tải data.txt...';
      listEl.innerHTML = '';
      try {
        const res = await fetch('data.txt', {cache: 'no-store'});
        if (!res.ok) throw new Error('Không thể đọc data.txt — HTTP ' + res.status);
        const text = await res.text();

        // Lấy tất cả URL
        const matches = text.match(urlRegex) || [];

        if (!matches.length) {
          status.textContent = 'Không tìm thấy link trong data.txt';
          return;
        }

        status.textContent = `Tìm thấy ${matches.length} link:`;
        for (const u of matches) {
          const li = document.createElement('li');
          // show text + clickable link mở tab mới, noopener for security
          li.innerHTML = `<a href="${escapeHtml(u)}" target="_blank" rel="noopener noreferrer">${escapeHtml(u)}</a>`;
          listEl.appendChild(li);
        }
      } catch (err) {
        status.textContent = 'Lỗi: ' + err.message + '. Nếu bạn mở file bằng file://, hãy chạy một web server (ví dụ: python -m http.server).';
      }
    }

    document.getElementById('refresh').addEventListener('click', loadAndShow);

    // mở tất cả link (lưu ý: nếu browser coi là popup, có thể bị chặn; vì thế dùng 1 click để kích hoạt)
    document.getElementById('openAll').addEventListener('click', () => {
      const anchors = Array.from(document.querySelectorAll('#list a'));
      if (!anchors.length) return;
      for (const a of anchors) {
        window.open(a.href, '_blank', 'noopener,noreferrer');
      }
    });

    // auto load khi mở trang
    loadAndShow();
  </script>
</body>
</html>
