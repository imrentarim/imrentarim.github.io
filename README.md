<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>İmren Tarım Depo Canlı Test Yayını</title>
  <style>
    html,body {height:100%; margin:0; background:#000;}
    #wrap {position:fixed; inset:0;}
    iframe {width:100%; height:100%; border:0;}
    #offline {position:absolute; inset:0; display:grid; place-items:center; color:#aaa; font:16px/1.4 system-ui, sans-serif; text-align:center; padding:24px;}
  </style>
</head>
<body>
  <div id="wrap">
    <div id="offline">Yayın offline görünüyor. Canlı başladığında otomatik yenilenecek.</div>
    <iframe id="player"
      allow="autoplay; encrypted-media; picture-in-picture"
      allowfullscreen
      referrerpolicy="origin-when-cross-origin">
    </iframe>
  </div>

  <script>
    const CHANNEL_ID = "UCfO4zU-8bFQXyX4fE6eY-mQ";          // e.g. starts with "UC..."
    const API_KEY    = "AIzaSyBMT-m7UyRnYLvTtD7dJAftOG-CPMipDys";             // YouTube Data API v3 key
    const CHECK_MS   = 60_000;                     // re-check every 60s
    const iframe = document.getElementById('player');
    const offline = document.getElementById('offline');

    async function setLiveEmbed() {
      try {
        // Search for an active live video on this channel
        const url = new URL('https://www.googleapis.com/youtube/v3/search');
        url.search = new URLSearchParams({
          part: 'snippet',
          channelId: CHANNEL_ID,
          eventType: 'live',
          type: 'video',
          maxResults: '1',
          order: 'date',
          key: API_KEY
        });

        const res = await fetch(url);
        if (!res.ok) throw new Error('API error ' + res.status);
        const data = await res.json();

        if (data.items && data.items.length) {
          const videoId = data.items[0].id.videoId;
          // Use the standard embed for the specific video ID
          iframe.src =
            `https://www.youtube.com/embed/${videoId}` +
            `?autoplay=1&mute=1&playsinline=1&modestbranding=1&rel=0&enablejsapi=1&origin=${location.origin}`;
          offline.style.display = 'none';
        } else {
          // No active live – fall back to channel placeholder (or keep offline message)
          iframe.removeAttribute('src');
          offline.style.display = 'grid';
        }
      } catch (e) {
        console.error(e);
        offline.style.display = 'grid';
      }
    }

    // Initial load + periodic re-check (useful if you start streaming later)
    setLiveEmbed();
    setInterval(setLiveEmbed, CHECK_MS);
  </script>
</body>
</html>
