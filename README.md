// --- project: video_downloader_webapp ---

// Server side using Node.js and Express with yt-dlp

// 1. Install dependencies: // npm install express cors body-parser

// 2. Make sure yt-dlp is installed on your system // (https://github.com/yt-dlp/yt-dlp)

const express = require('express'); const bodyParser = require('body-parser'); const cors = require('cors'); const { exec } = require('child_process');

const app = express(); const PORT = 3000;

app.use(cors()); app.use(bodyParser.json());

app.post('/get-video-info', (req, res) => { const { url } = req.body; if (!url) return res.status(400).send({ error: 'No URL provided' });

// yt-dlp command to fetch formats in JSON
const command = `yt-dlp -j ${url}`;

exec(command, (err, stdout, stderr) => {
    if (err) return res.status(500).send({ error: 'Failed to fetch video info' });
    try {
        const data = JSON.parse(stdout);
        const { title, thumbnail, formats } = data;
        const downloadable = formats.filter(f => f.url);
        res.send({ title, thumbnail, formats: downloadable });
    } catch (e) {
        res.status(500).send({ error: 'Failed to parse video info' });
    }
});

});

app.listen(PORT, () => { console.log(Server running at http://localhost:${PORT}); });

/* Frontend (index.html):

<!DOCTYPE html><html>
<head>
  <title>ভিডিও ডাউনলোডার</title>
  <style>
    body { font-family: sans-serif; max-width: 600px; margin: auto; padding: 20px; }
    img { max-width: 100%; margin: 10px 0; }
  </style>
</head>
<body>
  <h2>ভিডিও ডাউনলোড করুন</h2>
  <input type="text" id="url" placeholder="ভিডিও লিংক দিন" style="width: 100%; padding: 10px;">
  <button onclick="fetchInfo()">ডাউনলোড অপশন দেখান</button>
  <div id="result"></div>  <script>
    async function fetchInfo() {
      const url = document.getElementById('url').value;
      const res = await fetch('http://localhost:3000/get-video-info', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ url })
      });
      const data = await res.json();
      const container = document.getElementById('result');
      if (data.error) return container.innerHTML = `<p style='color:red;'>${data.error}</p>`;

      container.innerHTML = `<h3>${data.title}</h3><img src="${data.thumbnail}">`;
      data.formats.forEach(f => {
        if (f.format_note && f.url) {
          container.innerHTML += `<p><a href="${f.url}" target="_blank">${f.format_note} - ${f.ext}</a></p>`;
        }
      });
    }
  </script></body>
</html>
*/# SAMAN1
