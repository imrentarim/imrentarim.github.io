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
  const CHANNEL_ID = "UCfO4zU-8bFQXyX4fE6eY-mQ";   // your UC… channel ID
  const API_KEY    = "AIzaSyBMT-m7UyRnYLvTtD7dJAftOG-CPMipDys"; // restrict in GCP!
  const CHECK_MS   = 60_000; // re-check every 60s
  const iframe = document.getElementById('player');
  const offline = document.getElementById('offline');

  async function ytSearch(eventType) {
    const url = new URL('https://www.googleapis.com/youtube/v3/search');
    url.search = new URLSearchParams({
      part: 'snippet',
      channelId: CHANNEL_ID,
      eventType,           // 'live' or 'upcoming'
      type: 'video',
      maxResults: '1',
      order: 'date',
      key: API_KEY,
      // Trim payload a bit
      fields: 'items(id/videoId)'
    });
    const res = await fetch(url);
    if (!res.ok) throw new Error('YouTube API ' + res.status);
    const data = await res.json();
    return (data.items && data.items[0] && data.items[0].id.videoId) || null;
  }

  function setEmbed(videoId) {
    // cache-bust param to avoid stubborn caching on some proxies
    const bust = Date.now();
    iframe.src =
      `https://www.youtube.com/embed/${videoId}` +
      `?autoplay=1&mute=1&playsinline=1&modestbranding=1&rel=0&enablejsapi=1&origin=${location.origin}&cb=${bust}`;
    offline.style.display = 'none';
  }

  async function setLiveEmbed() {
    try {
      // 1) Try currently live
      let videoId = await ytSearch('live');

      // 2) If none live, try upcoming (will show countdown)
      if (!videoId) {
        videoId = await ytSearch('upcoming');
      }

      if (videoId) {
        setEmbed(videoId);
      } else {
        iframe.removeAttribute('src');
        offline.style.display = 'grid';
      }
    } catch (e) {
      console.error(e);
      // API key misconfigured, quota, or network issue
      // Keep last good embed if present; otherwise show offline.
      if (!iframe.src) offline.style.display = 'grid';
    }
  }

  // Initial load + periodic refresh (so it picks up new lives)
  setLiveEmbed();
  setInterval(setLiveEmbed, CHECK_MS);

  // Optional: small periodic re-embed to keep long sessions fresh
  setInterval(() => {
    if (iframe.src) {
      const u = new URL(iframe.src);
      u.searchParams.set('cb', Date.now());
      iframe.src = u.toString();
    }
  }, 30 * 60 * 1000); // every 30 min
</script>

</body>
</html>
