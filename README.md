<!DOCTYPE html>
<html lang="ka">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Sport Arena Live</title>
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
      :root{--bg:#0f172a;--card:#1e293b;--accent:#10b981;--hover:#34d399;--text:#f9fafb;--muted:#94a3b8;--border:rgba(255,255,255,.12)}
      *{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--text);font-family:Inter,Arial,sans-serif}
      .wrap{max-width:1400px;margin:0 auto;padding:12px}
      .top,.box{background:var(--card);border:1px solid var(--border);border-radius:14px}
      .top{padding:12px;display:flex;justify-content:space-between;align-items:center;gap:8px}
      .grid{margin-top:12px;display:grid;grid-template-columns:1fr 340px;gap:12px}
      .box{padding:12px}
      .player{background:#000;border-radius:12px;overflow:hidden;min-height:280px;display:grid;grid-template-columns:1fr}
      .player.split{grid-template-columns:1fr 1fr}
      .pane{min-height:280px;position:relative}
      video,iframe{width:100%;height:100%;border:0;display:block;object-fit:cover}
      #fallback{display:none}
      .hidden{display:none!important}
      .status{position:absolute;right:10px;bottom:10px;background:rgba(0,0,0,.7);padding:8px 10px;border-radius:8px}
      .adtag{position:absolute;left:10px;top:10px;background:rgba(0,0,0,.7);padding:6px 10px;border-radius:999px}
      .skip{position:absolute;right:10px;bottom:10px;background:var(--accent);border:0;border-radius:8px;padding:8px 10px;font-weight:700}
      .controls{display:grid;gap:8px;margin:10px 0}
      input,button,select{min-height:40px;border-radius:10px;border:1px solid var(--border)}
      input{background:#0b1220;color:var(--text);padding:0 10px}
      button{background:#0b1220;color:var(--text);padding:0 12px;cursor:pointer}
      button:hover{border-color:var(--hover)}
      .cats{display:flex;gap:8px;flex-wrap:wrap}
      .cat.active{background:var(--accent);color:#052e1f}
      .list{list-style:none;padding:0;margin:10px 0 0;display:grid;gap:8px;max-height:520px;overflow:auto}
      .ch button{width:100%;display:grid;grid-template-columns:56px 1fr auto;align-items:center;gap:10px;text-align:left;background:#111b30}
      .logo{width:56px;height:34px;border-radius:8px;background:#0b1220;display:grid;place-content:center;overflow:hidden;font-size:12px;font-weight:700}
      .logo img{width:100%;height:100%;object-fit:contain}
      .fav.active{background:var(--accent);color:#052e1f}
      @media(max-width:1000px){.grid{grid-template-columns:1fr}.player{min-height:220px}.pane{min-height:220px}.player.split{grid-template-columns:1fr}}
    </style>
  </head>
  <body>
    <div class="wrap">
      <header class="top">
        <strong>Sport Arena Live</strong>
        <div style="display:flex;gap:8px">
          <button id="langBtn">KA</button>
          <button id="themeBtn">Dark</button>
        </div>
      </header>

      <main class="grid">
        <section class="box">
          <h3 id="liveTitle">პირდაპირი ეთერი</h3>
          <div id="player" class="player">
            <div class="pane">
              <video id="video" playsinline muted></video>
              <iframe id="fallback"></iframe>
              <div id="loading" class="status hidden">Loading...</div>
              <div id="error" class="status hidden">Stream error</div>
            </div>
            <div id="adPane" class="pane hidden">
              <video id="adVideo" playsinline></video>
              <div class="adtag">რეკლამა</div>
              <button id="skipBtn" class="skip hidden">გამოტოვება <span id="skipSec">5</span></button>
            </div>
          </div>
        </section>

        <aside class="box">
          <h3 id="listTitle">არხების სია</h3>
          <div class="controls">
            <input id="searchInput" placeholder="Search..." />
            <button id="favFilterBtn">☆ Favorites</button>
          </div>
          <div id="cats" class="cats"></div>
          <ul id="list" class="list"></ul>
        </aside>
      </main>
    </div>

    <script>
      const CHANNELS_API_URL = "livechannels12/channels.json";
      const AD_VIDEO_URL = "https://res.cloudinary.com/dqstmljlq/video/upload/v1771688701/reklama_pz6r4y.mp4";

      const I18N = {
        ka: { live: "პირდაპირი ეთერი", list: "არხების სია", skip: "გამოტოვება" },
        en: { live: "Live Stream", list: "Channel List", skip: "Skip" }
      };

      let channels = [];
      let lang = localStorage.getItem("lang") || "ka";
      let theme = localStorage.getItem("theme") || "dark";
      let favorites = new Set(JSON.parse(localStorage.getItem("favorites") || "[]"));
      let onlyFav = false;
      let search = "";
      let activeCategory = "All";
      let activeChannel = null;
      let pendingChannel = null;
      let hls = null;
      let skipTimer = null;

      const el = {
        langBtn: document.getElementById("langBtn"),
        themeBtn: document.getElementById("themeBtn"),
        liveTitle: document.getElementById("liveTitle"),
        listTitle: document.getElementById("listTitle"),
        searchInput: document.getElementById("searchInput"),
        favFilterBtn: document.getElementById("favFilterBtn"),
        cats: document.getElementById("cats"),
        list: document.getElementById("list"),
        player: document.getElementById("player"),
        video: document.getElementById("video"),
        fallback: document.getElementById("fallback"),
        loading: document.getElementById("loading"),
        error: document.getElementById("error"),
        adPane: document.getElementById("adPane"),
        adVideo: document.getElementById("adVideo"),
        skipBtn: document.getElementById("skipBtn"),
        skipSec: document.getElementById("skipSec")
      };

      function applyLang() {
        el.langBtn.textContent = lang.toUpperCase();
        el.liveTitle.textContent = I18N[lang].live;
        el.listTitle.textContent = I18N[lang].list;
        el.skipBtn.childNodes[0].textContent = I18N[lang].skip + " ";
      }

      function applyTheme() {
        document.body.style.background = theme === "dark" ? "#0f172a" : "#e5e7eb";
        document.body.style.color = theme === "dark" ? "#f9fafb" : "#0f172a";
        el.themeBtn.textContent = theme === "dark" ? "Dark" : "Light";
      }

      function saveFavorites() {
        localStorage.setItem("favorites", JSON.stringify([...favorites]));
      }

      function getCategories() {
        return ["All", ...new Set(channels.map(c => c.category))];
      }

      function filteredChannels() {
        const q = search.toLowerCase().trim();
        return channels.filter(c => {
          if (activeCategory !== "All" && c.category !== activeCategory) return false;
          if (onlyFav && !favorites.has(c.name)) return false;
          if (q && !c.name.toLowerCase().includes(q)) return false;
          return true;
        });
      }

      function renderCats() {
        el.cats.innerHTML = "";
        getCategories().forEach(cat => {
          const b = document.createElement("button");
          b.className = "cat" + (cat === activeCategory ? " active" : "");
          b.textContent = cat;
          b.onclick = () => { activeCategory = cat; renderCats(); renderList(); };
          el.cats.appendChild(b);
        });
      }

      function logoHTML(ch) {
        if (typeof ch.logo === "string" && /^https?:\/\//i.test(ch.logo)) {
          return `<img src="${ch.logo}" alt="${ch.name}" onerror="this.style.display='none';this.parentElement.textContent='TV'">`;
        }
        return ch.logo || "TV";
      }

      function renderList() {
        const data = filteredChannels();
        el.list.innerHTML = "";
        data.forEach(ch => {
          const li = document.createElement("li");
          li.className = "ch" + (activeChannel?.name === ch.name ? " active" : "");
          const fav = favorites.has(ch.name);
          li.innerHTML = `<button>
            <span class="logo">${logoHTML(ch)}</span>
            <span>${ch.name}</span>
            <span><button class="fav ${fav ? "active" : ""}">${fav ? "★" : "☆"}</button></span>
          </button>`;
          li.querySelector(".fav").onclick = (e) => {
            e.stopPropagation();
            if (favorites.has(ch.name)) favorites.delete(ch.name); else favorites.add(ch.name);
            saveFavorites(); renderList();
          };
          li.firstElementChild.onclick = () => runPreRoll(ch);
          el.list.appendChild(li);
        });
      }

      function clearEngine() {
        if (hls) { hls.destroy(); hls = null; }
        el.video.pause();
        el.video.removeAttribute("src");
        el.fallback.src = "about:blank";
        el.fallback.style.display = "none";
        el.video.style.display = "block";
      }

      function showLoading(v) { el.loading.classList.toggle("hidden", !v); }
      function showError(v, msg = "Stream error") { el.error.textContent = msg; el.error.classList.toggle("hidden", !v); }

      async function playChannel(ch, idx = 0) {
        const url = ch.streamUrls[idx];
        if (!url) return showError(true, "No source");
        clearEngine(); showError(false); showLoading(true);

        if (ch.isIframe) {
          el.video.style.display = "none";
          el.fallback.style.display = "block";
          el.fallback.src = url;
          showLoading(false);
          activeChannel = ch; renderList();
          return;
        }

        try {
          if (el.video.canPlayType("application/vnd.apple.mpegurl")) {
            el.video.src = url;
          } else if (window.Hls?.isSupported()) {
            hls = new Hls({ lowLatencyMode: true, enableWorker: true });
            hls.loadSource(url);
            hls.attachMedia(el.video);
            hls.on(Hls.Events.ERROR, (_e, data) => {
              if (data.fatal) {
                if (ch.streamUrls[idx + 1]) playChannel(ch, idx + 1);
                else showError(true, "Stream failed");
              }
            });
          } else {
            if (ch.streamUrls[idx + 1]) return playChannel(ch, idx + 1);
            return showError(true, "Not supported");
          }

          await el.video.play();
          el.video.muted = false;
          showLoading(false);
          activeChannel = ch;
          renderList();
        } catch {
          showLoading(false);
          showError(true, "Tap/click to play");
        }
      }

      function adOn() {
        el.player.classList.add("split");
        el.adPane.classList.remove("hidden");
        el.video.muted = true;
      }
      function adOff() {
        el.player.classList.remove("split");
        el.adPane.classList.add("hidden");
        el.video.muted = false;
      }

      function playAd(skippable = true) {
        adOn();
        el.adVideo.src = AD_VIDEO_URL;
        el.adVideo.currentTime = 0;
        el.adVideo.muted = false;
        el.skipBtn.classList.toggle("hidden", !skippable);

        if (skippable) {
          let sec = 5;
          el.skipSec.textContent = sec;
          el.skipBtn.disabled = true;
          clearInterval(skipTimer);
          skipTimer = setInterval(() => {
            sec--;
            el.skipSec.textContent = Math.max(sec, 0);
            if (sec <= 0) {
              clearInterval(skipTimer);
              el.skipBtn.disabled = false;
            }
          }, 1000);
        }
        return el.adVideo.play().catch(() => {
          el.adVideo.muted = true;
          return el.adVideo.play();
        });
      }

      async function runPreRoll(ch) {
        pendingChannel = ch;
        clearEngine();
        await playAd(true);
      }

      function bind() {
        el.langBtn.onclick = () => {
          lang = lang === "ka" ? "en" : "ka";
          localStorage.setItem("lang", lang);
          applyLang();
        };
        el.themeBtn.onclick = () => {
          theme = theme === "dark" ? "light" : "dark";
          localStorage.setItem("theme", theme);
          applyTheme();
        };
        el.searchInput.oninput = (e) => { search = e.target.value || ""; renderList(); };
        el.favFilterBtn.onclick = () => {
          onlyFav = !onlyFav;
          el.favFilterBtn.textContent = onlyFav ? "★ Favorites" : "☆ Favorites";
          renderList();
        };
        el.skipBtn.onclick = () => {
          if (el.skipBtn.disabled) return;
          el.adVideo.pause(); el.adVideo.removeAttribute("src");
          adOff();
          if (pendingChannel) { playChannel(pendingChannel); pendingChannel = null; }
        };
        el.adVideo.onended = () => {
          el.adVideo.removeAttribute("src");
          adOff();
          if (pendingChannel) { playChannel(pendingChannel); pendingChannel = null; }
        };
      }

      async function loadChannels() {
        const r = await fetch(CHANNELS_API_URL, { cache: "no-store" });
        const json = await r.json();
        channels = Array.isArray(json) ? json : (json.channels || []);
      }

      async function init() {
        applyLang();
        applyTheme();
        bind();
        await loadChannels();
        renderCats();
        renderList();
        if (channels[0]) runPreRoll(channels[0]);
      }

      init();
    </script>
  </body>
</html>
