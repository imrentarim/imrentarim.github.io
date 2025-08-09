<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Live</title>
  <style>
    html,body {height:100%; margin:0; background:#000;}
    #wrap {position:fixed; inset:0;}
    iframe {width:100%; height:100%; border:0;}
    #offline {position:absolute; inset:0; display:none; place-items:center; color:#aaa; font:16px/1.4 system-ui, sans-serif; text-align:center; padding:24px;}
  </style>
</head>
<body>
  <div id="wrap">
    <div id="offline">If the stream is offline, it will auto‑refresh and show the next live broadcast once it starts.</div>
    <iframe id="player"
      src="https://www.youtube.com/embed/live_stream?channel=UCfO4zU-8bFQXyX4fE6eY-mQ&autoplay=1&mute=1&playsinline=1&modestbranding=1&rel=0"
      allow="autoplay; encrypted-media; picture-in-picture"
      allowfullscreen
      referrerpolicy="origin-when-cross-origin">
    </iframe>
  </div>

  <script>
    // Basic keep-alive: reload the iframe periodically and on stall states.
    const iframe = document.getElementById('player');
    const offline = document.getElementById('offline');
    let lastReload = Date.now();

    // Reload every 30 minutes to keep the embed fresh (helps long runtimes).
    setInterval(() => {
      if (Date.now() - lastReload > 25 * 60 * 1000) {
        reload();
      }
    }, 5 * 60 * 1000);

    // If the iframe errors, try a fast reload.
    iframe.addEventListener('error', reload);

    function reload() {
      lastReload = Date.now();
      const src = iframe.src;
      // Add a cache-busting query param so browsers don’t reuse the old one.
      const bust = `cb=${Date.now()}`;
      iframe.src = src.includes('?') ? src + '&' + bust : src + '?' + bust;
      offline.style.display = 'grid';
      setTimeout(() => offline.style.display = 'none', 4000);
    }

    // Optional: reload shortly after page becomes visible again (mobile wake).
    document.addEventListener('visibilitychange', () => {
      if (!document.hidden) {
        if (Date.now() - lastReload > 10 * 60 * 1000) reload();
      }
    });
  </script>
</body>
</html>
