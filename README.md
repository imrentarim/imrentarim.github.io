<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>İmren Tarım Depo Canlı Test Yayını</title>

  <style>
    html,body {height:100%; margin:0; background:#000;}
    #wrap {position:fixed; inset:0;}
    iframe {width:100%; height:100%; border:0;}

    /* Offline overlay */
    #offline {
      position:absolute; inset:0;
      display:none; place-items:center;
      color:#aaa; font:16px/1.4 system-ui, sans-serif;
      text-align:center; padding:24px; z-index:10;
      background:rgba(0,0,0,0.3);
    }

    /* Loader overlay */
    #loader {
      position:absolute; inset:0;
      display:grid; place-items:center;
      background:rgba(0,0,0,0.5);
      z-index:20;
    }
    .spinner {
      width:56px; height:56px;
      border:4px solid #666;
      border-top-color:#fff;
      border-radius:50%;
      animation: spin 0.9s linear infinite;
    }
    @keyframes spin { to { transform: rotate(360deg); } }

    /* Respect reduced motion */
    @media (prefers-reduced-motion: reduce) {
      .spinner { animation: none; border-top-color:#999; }
    }
  </style>
</head>
<body>
  <div id="wrap">
    <div id="loader" aria-live="polite" aria-busy="true">
      <div class="spinner" role="img" aria-label="Yükleniyor"></div>
    </div>

    <div id="offline">Yayın offline görünüyor. (Sayfa açılışında tek kontrol yapılır.)</div>

    <iframe id="player"
      allow="autoplay; encrypted-media; picture-in-picture"
      allowfullscreen></iframe>
  </div>

  <script>
    // === CONFIG ===
    const CHANNEL_ID = "UCfO4zU-8bFQXyX4fE6eY-mQ";
    const API_KEY    = "AIzaSyBMT-m7UyRnYLvTtD7dJAftOG-CPMipDys";

    // DOM
    const iframe  = document.getElementById('player');
    const offline = document.getElementById('offline');
    const loader  = document.getElementById('loader');

    // Optional manual override: ?video=VIDEO_ID (skips API)
    const forced = new URLSearchParams(location.search).get('video');
    if (forced) {
      setEmbed(forced);
    } else {
      initOnce();
    }

    function showLoader()  { loader.style.display  = 'grid';  }
    function hideLoader()  { loader.style.display  = 'none';  }
    function showOffline(msg) {
      if (msg) console.warn("[YT] Offline:", msg);
      offline.style.display = 'grid';
      hideLoader();
      iframe.removeAttribute('src');
    }
    function hideOffline() { offline.style.display = 'none'; }

    function setEmbed(videoId) {
      showLoader();
      hideOffline();
      const url =
        `https://www.youtube.com/embed/${videoId}` +
        `?autoplay=1&mute=1&playsinline=1&modestbranding=1&rel=0&cb=${Date.now()}`;
      iframe.src = url;

      // Hide loader when the iframe finishes loading
      const onLoad = () => {
        hideLoader();
        iframe.removeEventListener('load', onLoad);
      };
      iframe.addEventListener('load', onLoad);

      // Safety timeout (in case 'load' is blocked by extensions)
      setTimeout(hideLoader, 4000);
    }

    async function ytSearch(eventType) {
      const url = new URL('https://www.googleapis.com/youtube/v3/search');
      url.search = new URLSearchParams({
        part: 'id',
        channelId: CHANNEL_ID,
        eventType,             // 'live' or 'upcoming'
        type: 'video',
        maxResults: '1',
        order: 'date',
        fields: 'items(id/videoId)',
        key: API_KEY
      });
      const res = await fetch(url, { cache: 'no-store' });
      const data = await res.json().catch(() => ({}));
      if (!res.ok) throw new Error(data?.error?.message || ("API " + res.status));
      return data?.items?.[0]?.id?.videoId || null;
    }

    async function initOnce() {
      try {
        showLoader();
        // Try live; if none, try upcoming (countdown)
        let videoId = await ytSearch('live');
        if (!videoId) videoId = await ytSearch('upcoming');

        if (videoId) setEmbed(videoId);
        else showOffline("no live/upcoming found");
      } catch (e) {
        showOffline(e.message || "error");
      }
    }
  </script>
</body>
</html>
