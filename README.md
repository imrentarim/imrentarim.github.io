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
    <div id="offline">Yayın offline görünüyor. Canlı başladığında otomatik yenilenecek. Test .</div>
    <iframe id="player"
      allow="autoplay; encrypted-media; picture-in-picture"
      allowfullscreen
      referrerpolicy="origin-when-cross-origin">
    </iframe>
  </div>
<!-- Put this button right under your #offline div -->
<div id="controls" style="position:absolute;bottom:16px;left:0;right:0;display:grid;place-items:center;">
  <button id="retry" style="padding:8px 14px;border:0;border-radius:8px;background:#222;color:#eee;cursor:pointer">
    Yeniden kontrol et (cache’i temizle)
  </button>
</div>

<script>
  // === CONFIG ===
  const CHANNEL_ID = "UCfO4zU-8bFQXyX4fE6eY-mQ";
  const API_KEY    = "AIzaSyBMT-m7UyRnYLvTtD7dJAftOG-CPMipDys";
  const CACHE_TTL  = 5 * 60 * 1000; // 5 minutes

  // === DOM refs ===
  const iframe   = document.getElementById('player');
  const offline  = document.getElementById('offline');
  const retryBtn = document.getElementById('retry');

  // --- Helper: URL param override (no-API fallback) ---
  const urlParams = new URLSearchParams(location.search);
  const forcedVideo = urlParams.get('video'); // e.g. ?video=pph4VJY_VnQ

  function setEmbed(videoId) {
    const bust = Date.now();
    const src =
      `https://www.youtube.com/embed/${videoId}` +
      `?autoplay=1&mute=1&playsinline=1&modestbranding=1&rel=0&enablejsapi=1&origin=${location.origin}&cb=${bust}`;
    console.log('[YT] Embedding', src);
    iframe.src = src;
    offline.style.display = 'none';
  }

  function showOffline(reason) {
    if (reason) console.warn('[YT] Offline reason:', reason);
    iframe.removeAttribute('src');
    offline.style.display = 'grid';
  }

  async function searchVideo(eventType) {
    const url = new URL('https://www.googleapis.com/youtube/v3/search');
    url.search = new URLSearchParams({
      part: 'snippet',
      channelId: CHANNEL_ID,
      eventType,     // 'live' or 'upcoming'
      type: 'video',
      maxResults: '1',
      order: 'date',
      fields: 'items(id/videoId),error',
      key: API_KEY
    });
    const res = await fetch(url, { cache: 'no-store' });
    console.log('[YT] API', eventType, res.status, url.toString());
    if (!res.ok) {
      let j = {};
      try { j = await res.json(); } catch {}
      console.error('[YT] API error', res.status, j);
      throw new Error((j.error && j.error.message) || ('API ' + res.status));
    }
    const data = await res.json();
    return data.items?.[0]?.id?.videoId || null;
  }

  async function fetchVideoId() {
    // Try live first; only if empty, try upcoming (saves quota).
    const liveId = await searchVideo('live');
    if (liveId) return liveId;
    return await searchVideo('upcoming');
  }

  function readCache() {
    try {
      const raw = localStorage.getItem('ytLiveCache');
      if (!raw) return null;
      const obj = JSON.parse(raw);
      if (Date.now() - obj.timestamp < CACHE_TTL) return obj; // valid
    } catch {}
    return null;
  }

  function writeCache(videoId) {
    try {
      localStorage.setItem('ytLiveCache', JSON.stringify({ videoId, timestamp: Date.now() }));
    } catch {}
  }

  async function initOnce({ ignoreCache = false } = {}) {
    try {
      // No-API override path
      if (forcedVideo) {
        console.log('[YT] Forced video via URL param:', forcedVideo);
        setEmbed(forcedVideo);
        return;
      }

      if (!ignoreCache) {
        const cached = readCache();
        if (cached) {
          console.log('[YT] Using cache', cached);
          return cached.videoId ? setEmbed(cached.videoId) : showOffline('cached offline');
        }
      }

      // Fresh API call
      const videoId = await fetchVideoId();
      writeCache(videoId || null);
      if (videoId) setEmbed(videoId);
      else showOffline('no live/upcoming');
    } catch (e) {
      // Quota, referrer, or network problem
      console.error('[YT] init error', e);
      showOffline(e.message || 'error');
    }
  }

  // Run once on page load
  initOnce();

  // Manual refresh button: clears cache and rechecks now
  retryBtn.addEventListener('click', () => {
    localStorage.removeItem('ytLiveCache');
    initOnce({ ignoreCache: true });
  });

  // Optional: when tab becomes visible again, you
</script>


</body>
</html>
