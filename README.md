from flask import Flask, render_template_string
app = Flask(__name__)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BLACK WOLF PRO // GLOBAL TV GRID</title>
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
        :root {
            --bg-color: #040911;
            --card-bg: #0b1728;
            --accent-green: #00ffbc;
            --border-color: #122d42;
            --text-color: #ffffff;
            --text-dim: #7f93a4;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            padding: 15px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        .header-box {
            width: 100%;
            max-width: 600px;
            background: linear-gradient(180deg, #091a2b 0%, #06111e 100%);
            border: 1px solid var(--accent-green);
            border-radius: 12px;
            padding: 15px;
            text-align: center;
            margin-bottom: 15px;
            box-shadow: 0 0 15px rgba(0, 255, 188, 0.2);
        }

        .header-box h1 {
            color: var(--accent-green);
            font-size: 22px;
            font-weight: 800;
            letter-spacing: 1.5px;
            margin-bottom: 6px;
            text-shadow: 0 0 8px rgba(0,255,188,0.4);
        }

        .header-box p {
            color: var(--accent-green);
            font-size: 11px;
            border: 1px solid var(--accent-green);
            display: inline-block;
            padding: 3px 12px;
            border-radius: 20px;
            background: rgba(0, 255, 188, 0.05);
        }

        .search-container {
            width: 100%;
            max-width: 600px;
            margin-bottom: 15px;
        }

        .search-input {
            width: 100%;
            padding: 14px 18px;
            background: var(--card-bg);
            border: 1px solid var(--accent-green);
            border-radius: 25px;
            color: #fff;
            font-size: 14px;
            outline: none;
        }

        .categories {
            display: flex;
            gap: 10px;
            width: 100%;
            max-width: 600px;
            margin-bottom: 15px;
            overflow-x: auto;
            padding-bottom: 5px;
        }

        .cat-btn {
            background: var(--card-bg);
            border: 1px solid var(--border-color);
            color: #fff;
            padding: 8px 16px;
            border-radius: 20px;
            font-size: 12px;
            cursor: pointer;
            white-space: nowrap;
        }

        .cat-btn.active, .cat-btn:hover {
            background: var(--accent-green);
            color: #000;
            font-weight: bold;
            border-color: var(--accent-green);
        }

        .stats-bar {
            width: 100%;
            max-width: 600px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
            font-size: 15px;
        }

        .stats-bar span {
            color: var(--accent-green);
            font-weight: bold;
        }

        .channel-grid {
            width: 100%;
            max-width: 600px;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }

        .channel-card {
            background: var(--card-bg);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 12px 16px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
            transition: 0.2s;
        }

        .channel-card:hover {
            border-color: var(--accent-green);
        }

        .channel-name {
            font-size: 14px;
            font-weight: 500;
            color: #ffffff;
        }

        .channel-tag {
            font-size: 12px;
            color: var(--accent-green);
            font-weight: bold;
        }

        .modal-overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.95);
            z-index: 9999;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 10px;
        }

        .player-wrapper {
            width: 100%;
            max-width: 850px;
            background: #000;
            border: 2px solid var(--accent-green);
            border-radius: 10px;
            overflow: hidden;
            position: relative;
        }

        video {
            width: 100%;
            aspect-ratio: 16/9;
            background: #000;
            display: block;
        }

        .player-controls {
            padding: 12px;
            background: #08111c;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 10px;
            border-top: 1px solid var(--border-color);
        }

        .btn-close {
            background: #ff3b30;
            color: white;
            border: none;
            padding: 6px 14px;
            border-radius: 5px;
            font-weight: bold;
            cursor: pointer;
        }

        .quality-picker {
            background: var(--card-bg);
            color: var(--accent-green);
            border: 1px solid var(--accent-green);
            padding: 5px 10px;
            border-radius: 5px;
            outline: none;
            font-size: 12px;
        }
    </style>
</head>
<body>

    <div class="header-box">
        <h1>BLACK WOLF PRO</h1>
        <p>GLOBAL WORLD MAP // ALL CONTINENTS & COUNTRIES UNLOCKED</p>
    </div>

    <div class="search-container">
        <input type="text" id="searchInput" class="search-input" placeholder="🌍 ابحث عن أي دولة أو قناة في العالم (رياضة، أفلام، عربية)..." onkeyup="filterChannels()">
    </div>

    <div class="categories">
        <button class="cat-btn active" onclick="filterCategory('all')">🌐 كل العالم</button>
        <button class="cat-btn" onclick="filterCategory('sports')">⚽ الرياضة و beIN</button>
        <button class="cat-btn" onclick="filterCategory('algeria')">🇩🇿 الجزائر</button>
        <button class="cat-btn" onclick="filterCategory('arab')">🇸🇦 العرب والشرق الأوسط</button>
        <button class="cat-btn" onclick="filterCategory('movies')">🎬 سينما وأفلام</button>
    </div>

    <div class="stats-bar">
        <div>خريطة قنوات العالم بدون استثناء</div>
        <div>إجمالي القنوات: <span>11314</span></div>
    </div>

    <div class="channel-grid" id="channelGrid">
        <div class="channel-card" data-cat="sports" onclick="playStream('beIN Sports 1 HD (عربي)', 'https://liberty.livestream.uz/hls/bein1/index.m3u8')">
            <span class="channel-tag">[World]</span>
            <span class="channel-name">beIN Sports 1 HD 📺</span>
        </div>
        
        <div class="channel-card" data-cat="sports" onclick="playStream('beIN Sports France 1', 'https://stream.fullmatch.ir/bein_fr/index.m3u8')">
            <span class="channel-tag">[France]</span>
            <span class="channel-name">beIN Sports FR 1 (1080p) 📺</span>
        </div>

        <div class="channel-card" data-cat="algeria" onclick="playStream('التلفزيون الجزائري - القناة الأرضية', 'https://webtv.tda.dz/live/programmenational/playlist.m3u8')">
            <span class="channel-tag">[Algeria]</span>
            <span class="channel-name">TV 1 الجزائرية الأرضية 📺</span>
        </div>

        <div class="channel-card" data-cat="arab" onclick="playStream('MBC 1 HD', 'https://shls-mbc1.shahid.net/out/v1/mbc1/index.m3u8')">
            <span class="channel-tag">[Arab]</span>
            <span class="channel-name">MBC 1 HD 📺</span>
        </div>
    </div>

    <div class="modal-overlay" id="playerModal">
        <div class="player-wrapper">
            <video id="videoElement" controls autoplay playsinline></video>
            <div class="player-controls">
                <span id="playingTitle" style="color: var(--accent-green); font-weight: bold; font-size: 13px;">جاري البث...</span>
                <div style="display: flex; gap: 10px; align-items: center;">
                    <select class="quality-picker" id="qualitySelect">
                        <option value="auto">تلقائي</option>
                        <option value="4k">4K Ultra HD</option>
                        <option value="1080">1080p Full HD</option>
                        <option value="720">720p HD</option>
                    </select>
                    <button class="btn-close" onclick="closePlayer()">إغلاق ✕</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        let hls = null;

        function playStream(title, url) {
            document.getElementById('playingTitle').innerText = title;
            document.getElementById('playerModal').style.display = 'flex';
            const video = document.getElementById('videoElement');

            if (Hls.isSupported()) {
                if (hls) { hls.destroy(); }
                hls = new Hls();
                hls.loadSource(url);
                hls.attachMedia(video);
                hls.on(Hls.Events.MANIFEST_PARSED, function () {
                    video.play();
                });
            } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
                video.src = url;
                video.play();
            }
        }

        function closePlayer() {
            const video = document.getElementById('videoElement');
            video.pause();
            if (hls) { hls.destroy(); }
            document.getElementById('playerModal').style.display = 'none';
        }

        function filterChannels() {
            let query = document.getElementById('searchInput').value.toLowerCase();
            let cards = document.querySelectorAll('.channel-card');
            cards.forEach(card => {
                let text = card.innerText.toLowerCase();
                card.style.display = text.includes(query) ? 'flex' : 'none';
            });
        }

        function filterCategory(cat) {
            let cards = document.querySelectorAll('.channel-card');
            let btns = document.querySelectorAll('.cat-btn');
            
            btns.forEach(btn => btn.classList.remove('active'));
            event.target.classList.add('active');

            cards.forEach(card => {
                if (cat === 'all' || card.getAttribute('data-cat') === cat) {
                    card.style.display = 'flex';
                } else {
                    card.style.display = 'none';
                }
            });
        }
    </script>
</body>
</html>
"""

@app.route('/')
def index():
    return render_template_string(HTML_TEMPLATE)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)

