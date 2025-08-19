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
  // === CONFIG ===
  const CHANNEL_ID = "UCfO4zU-8bFQXyX4fE6eY-mQ";   // your UC… channel ID
  const API_KEY    = "AIzaSyBMT-m7UyRnYLvTtD7dJAftOG-CPMipDys"; // keep restricted
  const CACHE_TTL  = 5 * 60 * 1000; // 5 minutes in ms

  // === DOM refs ===
  const iframe  = document.getElementById('player');
  const offline = document.getElementById('offline');

  function setEmbed(videoId) {
    const bust = Date.now();
    iframe.src =
      `https://www.youtube.com/embed/${videoId}` +
      `?autoplay=1&mute=1&playsinline=1&modestbranding=1&rel=0&enablejsapi=1&origin=${location.origin}&cb=${bust}`;
    offline.style.display = 'none';
  }

  function showOffline() {
    iframe.removeAttribute('src');
    offline.style.display = 'grid';
  }

  async function searchVideo(eventType) {
    const url = new URL('https://www.googleapis.com/youtube/v3/search');
    url.search = new URLSearchParams({
      part: 'snippet',
      channelId: CHANNEL_ID,
      eventType,
      type: 'video',
      maxResults: '1',
      order: 'date',
      fields: 'items(id/videoId)',
      key: API_KEY
    });
    const res = await fetch(url, { cache: 'no-store' });
    if (!res.ok) throw new Error('YouTube API ' + res.status);
    const data = await res.json();
    return data.items?.[0]?.id?.videoId || null;
  }

  async function fetchVideoId() {
    // Try live first, then upcoming
    let videoId = await searchVideo('live');
    if (!videoId) videoId = await searchVideo('upcoming');
    return videoId;
  }

  async function initOnce() {
    try {
      // Check localStorage cache
      const cached = localStorage.getItem('ytLiveCache');
      if (cached) {
        const { videoId, timestamp } = JSON.parse(cached);
        if (Date.now() - timestamp < CACHE_TTL) {
          if (videoId) return setEmbed(videoId);
          return showOffline();
        }
      }

      // No valid cache → call API
      const videoId = await fetchVideoId();
      // Save result with timestamp
      localStorage.setItem('ytLiveCache', JSON.stringify({
        videoId,
        timestamp: Date.now()
      }));

      if (videoId) {
        setEmbed(videoId);
      } else {
        showOffline();
      }
    } catch (e) {
      console.error(e);
      showOffline();
    }
  }

  // Run only once on page load
  initOnce();
</script>


</body>
</html>
