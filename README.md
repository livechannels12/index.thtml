<!DOCTYPE html>
<html lang="ka">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Sport Arena Ultra</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&family=Poppins:wght@600;700;800&display=swap"
      rel="stylesheet"
    />
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
      :root {
        --bg: #0f172a;
        --card: #1e293b;
        --accent: #10b981;
        --hover: #34d399;
        --text: #f9fafb;
        --muted: #94a3b8;
        --border: rgba(255, 255, 255, 0.12);
        --glass: rgba(30, 41, 59, 0.74);
      }
      body.light {
        --bg: #e5e7eb;
        --card: #fff;
        --accent: #059669;
        --hover: #10b981;
        --text: #0f172a;
        --muted: #475569;
        --border: rgba(15, 23, 42, 0.14);
        --glass: rgba(255, 255, 255, 0.82);
      }
      * { box-sizing: border-box; }
      html, body {
        margin: 0; min-height: 100%;
        font-family: Inter, sans-serif;
        color: var(--text);
        background: radial-gradient(circle at 50% -16%, #1d4ed8 0%, var(--bg) 40%);
      }
      .bg {
        position: fixed; inset: 0; z-index: -1; pointer-events: none;
        background:
          linear-gradient(180deg, rgba(15,23,42,.22), rgba(15,23,42,.86)),
          radial-gradient(circle at 50% 0, rgba(16,185,129,.18), transparent 48%),
          repeating-linear-gradient(180deg, rgba(16,185,129,.02), rgba(16,185,129,.02) 4px, transparent 4px, transparent 8px);
      }
      .glass {
        background: var(--glass);
        border: 1px solid var(--border);
        border-radius: 16px;
        backdrop-filter: blur(10px);
        box-shadow: 0 8px 28px rgba(0,0,0,.24);
      }
      .wrap { max-width: 1540px; margin: 0 auto; padding: 14px; }

      .top {
        display: grid;
        grid-template-columns: 1fr minmax(180px,728px) auto;
        gap: 10px; align-items: center;
        padding: 12px;
      }
      .brand { display: flex; align-items: center; gap: 10px; }
      .dot { width: 12px; height: 12px; border-radius: 50%; background: var(--accent); box-shadow: 0 0 16px var(--accent); }
      h1 { margin: 0; font: 800 1.08rem Poppins, sans-serif; }

      .ad-slot {
        min-height: 90px;
        border: 1px dashed rgba(52,211,153,.5);
        border-radius: 12px;
        background: rgba(15,23,42,.35);
        color: var(--muted);
        display: grid; place-content: center;
        font-weight: 700;
      }

      .tools { display: flex; gap: 8px; justify-content: flex-end; }
      button { border: 0; cursor: pointer; font-weight: 700; }
      .pill,.accent {
        min-height: 42px; border-radius: 999px; padding: 0 14px;
      }
      .pill { background: color-mix(in srgb, var(--card), white 8%); color: var(--text); }
      .accent { background: var(--accent); color: #052e1f; }
      .pill:hover,.accent:hover { background: var(--hover); }

      .grid {
        margin-top: 12px;
        display: grid;
        grid-template-columns: minmax(0,1fr) 360px;
        gap: 12px;
      }

      .left-col { display: grid; gap: 12px; }
      .matches,.player-box,.side,.foot { padding: 12px; }

      .h2 { margin: 0 0 10px; font: 700 1.02rem Poppins, sans-serif; }

      .match-list {
        display: grid;
        grid-template-columns: repeat(auto-fit,minmax(180px,1fr));
        gap: 8px;
      }
      .match {
        border: 1px solid var(--border);
        border-radius: 10px;
        padding: 10px;
        background: rgba(15,23,42,.35);
      }
      .match .time { color: var(--muted); font-size: .85rem; }

      .player-shell {
        background: #020617;
        border-radius: 14px;
        overflow: hidden;
        min-height: 260px;
        max-height: 420px;
        display: grid;
        grid-template-columns: 1fr;
        transition: .28s ease;
      }
      .player-shell.split { grid-template-columns: 1fr 1fr; }
      .pane { position: relative; min-height: 240px; }
      video, iframe { width: 100%; height: 100%; border: 0; object-fit: cover; display: block; }
      #fallback { display: none; }

      .hidden { display: none !important; }

      .status {
        position: absolute;
        right: 10px; bottom: 10px;
        border-radius: 10px;
        padding: 8px 10px;
        background: rgba(2,6,23,.85);
        border: 1px solid rgba(255,255,255,.2);
        font-size: .9rem;
      }
      .status.error { left: 10px; right: 10px; top: 10px; bottom: auto; }
      .status.error a { color: var(--hover); }

      .ad-pane { background: #000; position: relative; }
      .ad-badge {
        position: absolute; top: 10px; left: 10px;
        border-radius: 999px; padding: 6px 10px;
        background: rgba(2,6,23,.72);
        border: 1px solid rgba(255,255,255,.3);
        font-size: .82rem;
      }
      .skip {
        position: absolute; right: 10px; bottom: 10px;
        min-height: 36px; border-radius: 10px; padding: 0 10px;
        background: var(--accent); color: #052e1f;
      }

      .below-ad { margin-top: 10px; }
      .side-ad-250 { min-height: 250px; }
      .side-ad-100 { min-height: 100px; margin-top: 10px; }

      .controls { display: grid; gap: 8px; margin: 10px 0; }
      .search, .select, .number {
        width: 100%; min-height: 42px;
        border-radius: 10px;
        border: 1px solid var(--border);
        background: rgba(2,6,23,.5);
        color: var(--text);
        padding: 0 10px;
      }
      .ad-config {
        display: grid;
        grid-template-columns: 1fr 1fr auto;
        gap: 8px;
      }

      .cats { display: flex; flex-wrap: wrap; gap: 8px; }
      .cat {
        min-height: 38px;
        border-radius: 999px;
        padding: 0 12px;
        border: 1px solid var(--border);
        background: rgba(148,163,184,.15);
        color: var(--text);
      }
      .cat.active { background: var(--accent); color: #052e1f; }

      .list {
        list-style: none; margin: 10px 0 0; padding: 0;
        display: grid; gap: 9px; max-height: 460px; overflow: auto;
      }
      .ch button {
        width: 100%; min-height: 60px;
        border-radius: 12px;
        border: 1px solid var(--border);
        background: color-mix(in srgb, var(--card), black 10%);
        color: var(--text);
        display: grid;
        grid-template-columns: 56px 1fr auto;
        gap: 10px; align-items: center;
        text-align: left; padding: 8px;
      }
      .ch button:hover { border-color: var(--hover); transform: scale(1.01); }
      .ch.active button { border-color: var(--accent); }

      .logo {
        width: 56px; height: 34px; border-radius: 10px;
        background: #0b1120;
        display: grid; place-content: center;
        overflow: hidden; color: #d1fae5; font-size: .75rem; font-weight: 800;
      }
      .logo img { width: 100%; height: 100%; object-fit: contain; }

      .fav-btn {
        min-height: 34px;
        border-radius: 10px;
        padding: 0 10px;
        background: rgba(148,163,184,.2);
        color: var(--text);
      }
      .fav-btn.active { background: var(--accent); color: #052e1f; }

      .foot { margin-top: 12px; }
      .copy { margin: 8px 0 0; color: var(--muted); text-align: center; }

      @media (max-width: 1200px) {
        .top { grid-template-columns: 1fr; }
        .tools { justify-content: flex-start; }
        .grid { grid-template-columns: 1fr; }
        .list { max-height: 320px; }
      }
      @media (max-width: 760px) {
        .player-shell { min-height: 200px; max-height: none; }
        .player-shell.split { grid-template-columns: 1fr; }
        .pane { min-height: 200px; }
        .ad-config { grid-template-columns: 1fr; }
      }
    </style>
  </head>
  <body>
    <div class="bg" aria-hidden="true"></div>

    <div class="wrap">
      <header class="top glass">
        <div class="brand"><span class="dot"></span><h1>Sport Arena Ultra</h1></div>
        <div class="ad-slot">HEADER AD 728×90</div>
        <div class="tools">
          <button id="langBtn" class="pill">KA</button>
          <button id="themeBtn" class="pill">Dark</button>
          <button id="reloadBtn" class="pill">Reload API</button>
        </div>
      </header>

      <main class="grid">
        <div class="left-col">
          <section class="matches glass">
            <h2 class="h2">Top Matches</h2>
            <div id="matchList" class="match-list"></div>
          </section>

          <section class="player-box glass">
            <h2 class="h2" data-i18n="live">პირდაპირი ეთერი</h2>
            <div id="playerShell" class="player-shell">
              <div class="pane">
                <video id="video" playsinline muted></video>
                <iframe id="fallback" title="fallback stream"></iframe>

                <div id="loading" class="status hidden">Loading...</div>
                <div id="buffering" class="status hidden">Buffering...</div>
                <div id="error" class="status error hidden">
                  <span id="errorText">Stream unavailable.</span>
                  <a id="externalLink" href="#" target="_blank" rel="noopener">Open externally</a>
                </div>
                <div id="tapPlayBox" class="status hidden">
                  <button id="tapPlay" class="accent">Tap to play</button>
                </div>
              </div>

              <div id="adPane" class="ad-pane hidden">
                <video id="adVideo" playsinline></video>
                <div class="ad-badge" data-i18n="ad">რეკლამა</div>
                <button id="skipBtn" class="skip hidden">
                  <span data-i18n="skip">გამოტოვება</span> <span id="skipSec">5</span>
                </button>
              </div>
            </div>
            <div class="ad-slot below-ad">BELOW PLAYER AD 728×90</div>
          </section>
        </div>

        <aside class="side glass">
          <h2 class="h2" data-i18n="channels">არხების სია</h2>
          <div class="ad-slot side-ad-250">SIDEBAR AD 300×250</div>

          <div class="controls">
            <input id="searchInput" class="search" placeholder="Search channel..." />
            <button id="favFilterBtn" class="pill">☆ Favorites</button>

            <div class="ad-config">
              <input id="skipInput" class="number" type="number" min="1" value="5" title="Pre-roll skip seconds" />
              <input id="hourlyInput" class="number" type="number" min="1" value="60" title="Hourly ad interval minutes" />
              <button id="applyAdConfigBtn" class="accent">Apply Ads</button>
            </div>
          </div>

          <div id="cats" class="cats"></div>
          <ul id="list" class="list"></ul>

          <div class="ad-slot side-ad-100">SIDEBAR AD 300×100</div>
        </aside>
      </main>

      <footer class="foot glass">
        <div class="ad-slot">FOOTER AD 728×90</div>
        <p class="copy">© <span id="year"></span> Sport Arena Ultra</p>
      </footer>
    </div>

    <script>
      const TEXT = {
        ka: { live: "პირდაპირი ეთერი", channels: "არხების სია", ad: "რეკლამა", skip: "გამოტოვება" },
        en: { live: "Live Stream", channels: "Channel List", ad: "Advertisement", skip: "Skip" }
      };

      const DEFAULT_CHANNELS = [
        { name: "1 არხი", category: "GEORGIA", streamUrls: ["https://tv.cdn.xsg.ge/gpb-1tv/index.m3u8"], isIframe: false, logo: "https://livechannels.ge/image/ge1.png" },
        { name: "2 არხი", category: "GEORGIA", streamUrls: ["https://tv.cdn.xsg.ge/gpb-2tv/index.m3u8"], isIframe: false, logo: "https://livechannels.ge/image/ge2.png" },
        { name: "სილქ უნივერსალ", category: "SILK SPORT", streamUrls: ["https://tv-new.silkgo.ge/embed.html?epg=false&ads=false&watermark=false&channels=false&forceDesktop=true&dvr=false&PlayerMuted=false&filterChannels=silk_sport4&hoverControls=true&progressbar=true&controls=true"], isIframe: true, logo: "https://livechannels.ge/image/silk-universal.png" },
        { name: "Eurosport", category: "GERMANY", streamUrls: ["https://bde.online24.pm/play/15/7D6EC0CC5598A28/video.m3u8", "https://bpl.online24.pm/play/15/7D6EC0CC5598A28/video.m3u8"], isIframe: false, logo: "https://livechannels.ge/image/Eurosport-1.png" },
        { name: "Setanta Sports 1 HD", category: "SPORTS", streamUrls: ["https://bde.online24.pm/play/942/7D6EC0CC5598A28/video.m3u8", "https://bpl.online24.pm/play/942/7D6EC0CC5598A28/video.m3u8"], isIframe: false, logo: "https://livechannels.ge/image/setanta-sports1.png" },
        { name: "Setanta Sports 2 HD", category: "SPORTS", streamUrls: ["https://bde.online24.pm/play/943/7D6EC0CC5598A28/video.m3u8", "https://bpl.online24.pm/play/943/7D6EC0CC5598A28/video.m3u8"], isIframe: false, logo: "https://livechannels.ge/image/setanta-sports2.png" },
        { name: "NBA TV", category: "BASKETBALL", streamUrls: ["https://bde.online24.pm/play/9984/7D6EC0CC5598A28/video.m3u8"], isIframe: false, logo: "NB" },
        { name: "UFC ТВ HD", category: "COMBAT", streamUrls: ["https://bde.online24.pm/play/1777/7D6EC0CC5598A28/video.m3u8"], isIframe: false, logo: "UF" }
      ];

      // სურვილის შემთხვევაში აქ ჩაწერე შენი JSON URL (სადაც channels მასივი დაბრუნდება)
      const CHANNELS_API_URL = ""; // მაგალითად: "[https:// ](https://github.com/livechannels12/channels.json"

      const TOP_MATCHES = [
        { teams: "Real Madrid vs Barcelona", time: "21:00", league: "LaLiga" },
        { teams: "Man City vs Arsenal", time: "22:30", league: "Premier League" },
        { teams: "Inter vs Milan", time: "23:00", league: "Serie A" },
        { teams: "Bayern vs Dortmund", time: "20:30", league: "Bundesliga" }
      ];

      const AD_VIDEO_URL = "https://res.cloudinary.com/dqstmljlq/video/upload/v1771688701/reklama_pz6r4y.mp4";

      const STORAGE = {
        lang: "sa_lang",
        theme: "sa_theme",
        favorites: "sa_favorites",
        adSkip: "sa_ad_skip",
        adHourly: "sa_ad_hourly"
      };

      const els = {
        year: document.getElementById("year"),
        matchList: document.getElementById("matchList"),
        langBtn: document.getElementById("langBtn"),
        themeBtn: document.getElementById("themeBtn"),
        reloadBtn: document.getElementById("reloadBtn"),
        searchInput: document.getElementById("searchInput"),
        favFilterBtn: document.getElementById("favFilterBtn"),
        skipInput: document.getElementById("skipInput"),
        hourlyInput: document.getElementById("hourlyInput"),
        applyAdConfigBtn: document.getElementById("applyAdConfigBtn"),
        cats: document.getElementById("cats"),
        list: document.getElementById("list"),

        video: document.getElementById("video"),
        fallback: document.getElementById("fallback"),
        playerShell: document.getElementById("playerShell"),
        loading: document.getElementById("loading"),
        buffering: document.getElementById("buffering"),
        error: document.getElementById("error"),
        errorText: document.getElementById("errorText"),
        externalLink: document.getElementById("externalLink"),
        tapPlay: document.getElementById("tapPlay"),
        tapPlayBox: document.getElementById("tapPlayBox"),

        adPane: document.getElementById("adPane"),
        adVideo: document.getElementById("adVideo"),
        skipBtn: document.getElementById("skipBtn"),
        skipSec: document.getElementById("skipSec")
      };

      let channelsData = [...DEFAULT_CHANNELS];
      let lang = localStorage.getItem(STORAGE.lang) || "ka";
      let theme = localStorage.getItem(STORAGE.theme) || "dark";
      let favorites = new Set(JSON.parse(localStorage.getItem(STORAGE.favorites) || "[]"));
      let searchTerm = "";
      let onlyFavorites = false;
      let activeCategory = "All";
      let activeChannel = null;
      let pendingChannel = null;

      let adConfig = {
        prerollSkip: Number(localStorage.getItem(STORAGE.adSkip) || 5),
        hourlyMinutes: Number(localStorage.getItem(STORAGE.adHourly) || 60)
      };

      let hls = null;
      let adTimer = null;
      let hourlyInterval = null;
      let isHourlyAd = false;

      function t(k) { return TEXT[lang][k] || k; }

      function applyLanguage() {
        document.documentElement.lang = lang;
        localStorage.setItem(STORAGE.lang, lang);
        els.langBtn.textContent = lang.toUpperCase();
        document.querySelectorAll("[data-i18n]").forEach((el) => el.textContent = t(el.dataset.i18n));
      }

      function applyTheme() {
        document.body.classList.toggle("light", theme === "light");
        localStorage.setItem(STORAGE.theme, theme);
        els.themeBtn.textContent = theme === "dark" ? "Dark" : "Light";
      }

      function renderMatches() {
        els.matchList.innerHTML = TOP_MATCHES.map(m => `
          <article class="match">
            <div><strong>${m.teams}</strong></div>
            <div class="time">${m.league} • ${m.time}</div>
          </article>
        `).join("");
      }

      function saveFavorites() {
        localStorage.setItem(STORAGE.favorites, JSON.stringify([...favorites]));
      }

      function categories() {
        return ["All", ...new Set(channelsData.map(c => c.category))];
      }

      function getFilteredChannels() {
        const q = searchTerm.trim().toLowerCase();
        return channelsData.filter(ch => {
          if (activeCategory !== "All" && ch.category !== activeCategory) return false;
          if (onlyFavorites && !favorites.has(ch.name)) return false;
          if (q && !ch.name.toLowerCase().includes(q)) return false;
          return true;
        });
      }

      function logoHTML(ch) {
        if (typeof ch.logo === "string" && /^https?:\/\//i.test(ch.logo)) {
          return `<img src="${ch.logo}" alt="${ch.name}" loading="lazy" onerror="this.style.display='none';this.parentElement.textContent='${ch.name.slice(0,2).toUpperCase()}';">`;
        }
        return ch.logo || ch.name.slice(0,2).toUpperCase();
      }

      function renderCategories() {
        els.cats.innerHTML = "";
        categories().forEach(cat => {
          const b = document.createElement("button");
          b.type = "button";
          b.className = "cat" + (cat === activeCategory ? " active" : "");
          b.textContent = cat;
          b.onclick = () => {
            activeCategory = cat;
            renderCategories();
            renderChannelList();
          };
          els.cats.appendChild(b);
        });
      }

      function toggleFavorite(name) {
        if (favorites.has(name)) favorites.delete(name);
        else favorites.add(name);
        saveFavorites();
        renderChannelList();
      }

      function renderChannelList() {
        const data = getFilteredChannels();
        els.list.innerHTML = "";

        if (!data.length) {
          els.list.innerHTML = `<li class="ch"><button type="button"><span>No channels found</span></button></li>`;
          return;
        }

        const frag = document.createDocumentFragment();
        data.forEach(ch => {
          const li = document.createElement("li");
          li.className = "ch" + (activeChannel?.name === ch.name ? " active" : "");
          const fav = favorites.has(ch.name);

          const btn = document.createElement("button");
          btn.type = "button";
          btn.innerHTML = `
            <span class="logo">${logoHTML(ch)}</span>
            <span>${ch.name}</span>
            <span><button class="fav-btn ${fav ? "active" : ""}" type="button">${fav ? "★" : "☆"}</button></span>
          `;

          const favBtn = btn.querySelector(".fav-btn");
          favBtn.addEventListener("click", (e) => {
            e.stopPropagation();
            toggleFavorite(ch.name);
          });

          btn.addEventListener("click", () => runPreRoll(ch));
          li.appendChild(btn);
          frag.appendChild(li);
        });

        els.list.appendChild(frag);
      }

      function setStatus(name, visible) {
        if (name === "loading") els.loading.classList.toggle("hidden", !visible);
        if (name === "buffering") els.buffering.classList.toggle("hidden", !visible);
        if (name === "error") els.error.classList.toggle("hidden", !visible);
        if (name === "tapplay") els.tapPlayBox.classList.toggle("hidden", !visible);
      }

      function clearPlayerEngine() {
        if (hls) {
          hls.destroy();
          hls = null;
        }
        els.video.pause();
        els.video.removeAttribute("src");
        els.fallback.src = "about:blank";
        els.fallback.style.display = "none";
        els.video.style.display = "block";
      }

      function showError(channel, message) {
        setStatus("loading", false);
        setStatus("buffering", false);
        setStatus("error", true);
        els.errorText.textContent = message;
        els.externalLink.href = channel?.streamUrls?.[0] || "#";
      }

      async function startChannel(channel, index = 0) {
        const url = channel.streamUrls[index];
        if (!url) return showError(channel, "No stream source.");

        clearPlayerEngine();
        setStatus("error", false);
        setStatus("loading", true);
        setStatus("tapplay", false);

        if (channel.isIframe) {
          els.video.style.display = "none";
          els.fallback.style.display = "block";
          els.fallback.src = url;
          setStatus("loading", false);
          activeChannel = channel;
          renderChannelList();
          return;
        }

        try {
          if (els.video.canPlayType("application/vnd.apple.mpegurl")) {
            els.video.src = url;
          } else if (window.Hls?.isSupported()) {
            hls = new window.Hls({
              lowLatencyMode: true,
              enableWorker: true,
              backBufferLength: 10,
              maxBufferLength: 28
            });
            hls.loadSource(url);
            hls.attachMedia(els.video);
            hls.on(window.Hls.Events.ERROR, (_e, data) => {
              if (data.fatal) {
                if (channel.streamUrls[index + 1]) startChannel(channel, index + 1);
                else showError(channel, "Stream failed. Try external link.");
              }
            });
          } else {
            if (channel.streamUrls[index + 1]) startChannel(channel, index + 1);
            else showError(channel, "Browser cannot play stream.");
            return;
          }

          await els.video.play();
          els.video.muted = false;
          setStatus("loading", false);
          activeChannel = channel;
          renderChannelList();
        } catch {
          setStatus("loading", false);
          setStatus("tapplay", true);
        }
      }

      function splitOn() {
        els.playerShell.classList.add("split");
        els.adPane.classList.remove("hidden");
        els.video.muted = true;
      }

      function splitOff() {
        els.playerShell.classList.remove("split");
        els.adPane.classList.add("hidden");
        if (!isHourlyAd) els.video.muted = false;
      }

      function stopAd() {
        clearInterval(adTimer);
        els.adVideo.pause();
        els.adVideo.removeAttribute("src");
        els.skipBtn.classList.add("hidden");
        splitOff();
      }

      function playAd({ skippable }) {
        splitOn();
        els.adVideo.src = AD_VIDEO_URL;
        els.adVideo.currentTime = 0;
        els.adVideo.muted = false;
        els.skipBtn.classList.toggle("hidden", !skippable);

        if (skippable) {
          let sec = adConfig.prerollSkip;
          els.skipSec.textContent = String(sec);
          els.skipBtn.disabled = true;
          clearInterval(adTimer);
          adTimer = setInterval(() => {
            sec -= 1;
            els.skipSec.textContent = String(Math.max(sec, 0));
            if (sec <= 0) {
              clearInterval(adTimer);
              els.skipBtn.disabled = false;
            }
          }, 1000);
        }

        return els.adVideo.play().catch(() => {
          els.adVideo.muted = true;
          return els.adVideo.play();
        });
      }

      async function runPreRoll(channel) {
        pendingChannel = channel;
        isHourlyAd = false;
        clearPlayerEngine();
        await playAd({ skippable: true });
      }

      function scheduleHourlyAd() {
        clearInterval(hourlyInterval);
        hourlyInterval = setInterval(async () => {
          if (!activeChannel) return;
          isHourlyAd = true;
          await playAd({ skippable: false });
        }, adConfig.hourlyMinutes * 60 * 1000);
      }

      function applyAdConfigFromInputs() {
        const skip = Math.max(1, Number(els.skipInput.value) || 5);
        const mins = Math.max(1, Number(els.hourlyInput.value) || 60);

        adConfig.prerollSkip = skip;
        adConfig.hourlyMinutes = mins;

        localStorage.setItem(STORAGE.adSkip, String(skip));
        localStorage.setItem(STORAGE.adHourly, String(mins));

        scheduleHourlyAd();
        alert(`Ad config updated: skip ${skip}s, hourly ${mins}min`);
      }

      async function loadChannelsFromApi() {
        if (!CHANNELS_API_URL) return; // fallback to default
        try {
          const res = await fetch(CHANNELS_API_URL, { cache: "no-store" });
          if (!res.ok) throw new Error("API error");
          const json = await res.json();

          // მხარს უჭერს ფორმატებს: [] ან {channels: []}
          const arr = Array.isArray(json) ? json : json.channels;
          if (!Array.isArray(arr) || !arr.length) throw new Error("Invalid JSON");

          channelsData = arr;
          activeCategory = "All";
          renderCategories();
          renderChannelList();
          if (!activeChannel && channelsData[0]) runPreRoll(channelsData[0]);
        } catch (e) {
          console.warn("Channel API load failed:", e.message);
        }
      }

      function bindEvents() {
        els.langBtn.addEventListener("click", () => {
          lang = lang === "ka" ? "en" : "ka";
          applyLanguage();
        });

        els.themeBtn.addEventListener("click", () => {
          theme = theme === "dark" ? "light" : "dark";
          applyTheme();
        });

        els.reloadBtn.addEventListener("click", async () => {
          await loadChannelsFromApi();
          renderCategories();
          renderChannelList();
        });

        els.searchInput.addEventListener("input", (e) => {
          searchTerm = e.target.value || "";
          renderChannelList();
        });

        els.favFilterBtn.addEventListener("click", () => {
          onlyFavorites = !onlyFavorites;
          els.favFilterBtn.textContent = onlyFavorites ? "★ Favorites" : "☆ Favorites";
          renderChannelList();
        });

        els.applyAdConfigBtn.addEventListener("click", applyAdConfigFromInputs);

        els.video.addEventListener("waiting", () => setStatus("buffering", true));
        els.video.addEventListener("playing", () => setStatus("buffering", false));
        els.video.addEventListener("error", () => {
          if (activeChannel) showError(activeChannel, "Playback error.");
        });

        els.tapPlay.addEventListener("click", async () => {
          try {
            await els.video.play();
            els.video.muted = false;
            setStatus("tapplay", false);
          } catch {
            setStatus("tapplay", true);
          }
        });

        els.skipBtn.addEventListener("click", () => {
          if (els.skipBtn.disabled) return;
          stopAd();
          if (pendingChannel) {
            startChannel(pendingChannel);
            pendingChannel = null;
          } else {
            isHourlyAd = false;
          }
        });

        els.adVideo.addEventListener("ended", () => {
          const wasPreRoll = Boolean(pendingChannel);
          stopAd();
          if (wasPreRoll && pendingChannel) {
            startChannel(pendingChannel);
            pendingChannel = null;
          }
          isHourlyAd = false;
        });
      }

      async function init() {
        els.year.textContent = String(new Date().getFullYear());
        els.skipInput.value = String(adConfig.prerollSkip);
        els.hourlyInput.value = String(adConfig.hourlyMinutes);

        applyTheme();
        applyLanguage();
        renderMatches();

        await loadChannelsFromApi();

        renderCategories();
        renderChannelList();
        bindEvents();
        scheduleHourlyAd();

        if (channelsData[0]) runPreRoll(channelsData[0]);
      }

      init();
    </script>
  </body>
</html>aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
