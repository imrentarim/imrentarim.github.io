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
    #offline {
      position:absolute; inset:0;
      display:grid; place-items:center;
      color:#aaa; font:16px/1.4 system-ui, sans-serif;
      text-align:center; padding:24px;
    }
  </style>
</head>
<body>
  <div id="wrap">
    <div id="offline">Yayın offline görünüyor. (Sayfa açılışında tek kontrol yapılır.) test 3</div>
    <iframe id="player"
      allow="autoplay; encrypted-media; picture-in-picture"
      allowfullscreen
      referrerpolicy="origin-when-cross-origin">
    </iframe>
  </div>

<script>
  // === CONFIG ===
  const CHANNEL_ID = "UCfO4zU-8bFQXyX4fE6eY-mQ";
  const API_KEY    = "AIzaSyBMT-m7UyRnYLvTtD7dJAftOG-CPMipDys"; // keep restricted to your domain

  const iframe  = document.getElementById('player');
  const offline = document.getElementById('offline');

  // Optional manual override: https://site/?video=VIDEO_ID (skips API)
  const forced = new URLSearchParams(location.search).get('video');
  if (forced) {
    console.log("[YT] Forced video via ?video=...", forced);
    setEmbed(forced);
  } else {
    initOnce();
  }

  function setEmbed(videoId) {
    const bust = Date.now();
    const src =
      `https://www.youtube.com/embed/${videoId}` +
      `?autoplay=1&mute=1&playsinline=1&modestbranding=1&rel=0&origin=${location.origin}&cb=${bust}`;
    console.log("[YT] Embedding:", src);
    iframe.src = src;
    offline.style.display = 'none';
  }

  function showOffline(reason) {
    if (reason) console.warn("[YT] Offline:", reason);
    iframe.removeAttribute('src');
    offline.style.display = 'grid';
  }

  async function ytSearch(eventType) {
    const url = new URL('https://www.googleapis.com/youtube/v3/search');
    url.search = new URLSearchParams({
      part: 'snippet',
      channelId: CHANNEL_ID,
      eventType,         // 'live' or 'upcoming'
      type: 'video',
      maxResults: '1',
      order: 'date',
      fields: 'items(id/videoId),error',
      key: API_KEY
    });
    console.log("[YT] Fetch:", url.toString());
    const res = await fetch(url, { cache: 'no-store' });
    console.log("[YT] Status:", res.status);
    let data;
    try { data = await res.json(); } catch { data = {}; }
    console.log("[YT] Data:", data);
    if (!res.ok) throw new Error(data?.error?.message || ("API " + res.status));
    return data?.items?.[0]?.id?.videoId || null;
  }

  async function initOnce() {
    try {
      // 1) Try live
      let videoId = await ytSearch('live');

      // 2) If not live, try upcoming (countdown)
      if (!videoId) videoId = await ytSearch('upcoming');

      if (videoId) setEmbed(videoId);
      else showOffline("no live/upcoming found");
    } catch (e) {
      console.error("[YT] init error:", e);
      showOffline(e.message || "error");
    }
  }
</script>
</body>
</html>
