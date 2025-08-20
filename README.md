<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>İmren Tarım Depo Canlı Test Yayını</title>

  <!-- Force HTTPS (prevents referrer/key issues and improves embed reliability) -->

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
    <div id="offline">Yayın offline görünüyor. (Sayfa açılışında tek kontrol yapılır.) working </div>
    <iframe id="player"
      allow="autoplay; encrypted-media; picture-in-picture"
      allowfullscreen></iframe>
  </div>

  <script>
    // === CONFIG ===
    const CHANNEL_ID = "UCfO4zU-8bFQXyX4fE6eY-mQ"; // sizin UC… kanal kimliğiniz
    const API_KEY    = "AIzaSyBMT-m7UyRnYLvTtD7dJAftOG-CPMipDys"; // yalnızca domaininize kısıtlı tutun

    const iframe  = document.getElementById('player');
    const offline = document.getElementById('offline');

    // İsteğe bağlı: Manuel geçersiz kılma (API kullanmadan embed) -> ?video=VIDEO_ID
    const forced = new URLSearchParams(location.search).get('video');
    if (forced) {
      setEmbed(forced);
    } else {
      initOnce();
    }

    function setEmbed(videoId) {
      const src =
        `https://www.youtube.com/embed/${videoId}` +
        `?autoplay=1&mute=1&playsinline=1&modestbranding=1&rel=0&cb=${Date.now()}`;
      iframe.src = src;
      offline.style.display = 'none';
    }

    function showOffline(reason) {
      // Konsolda görmek isterseniz:
      if (reason) console.warn("[YT] Offline:", reason);
      iframe.removeAttribute('src');
      offline.style.display = 'grid';
    }

    async function ytSearch(eventType) {
      const url = new URL('https://www.googleapis.com/youtube/v3/search');
      url.search = new URLSearchParams({
        part: 'id',                   // fields ile uyumlu
        channelId: CHANNEL_ID,
        eventType,                    // 'live' veya 'upcoming'
        type: 'video',
        maxResults: '1',
        order: 'date',
        fields: 'items(id/videoId)',  // yalnızca gereken alan
        key: API_KEY
      });
      const res = await fetch(url, { cache: 'no-store' });
      const data = await res.json().catch(() => ({}));
      if (!res.ok) throw new Error(data?.error?.message || ("API " + res.status));
      return data?.items?.[0]?.id?.videoId || null;
    }

    async function initOnce() {
      try {
        // 1) Önce aktif canlı yayını bul
        let videoId = await ytSearch('live');
        // 2) Yoksa planlanmış (upcoming) yayını göm (geri sayım gösterir)
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
