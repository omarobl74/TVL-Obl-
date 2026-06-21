<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>تطبيق البث المباشر الاحترافي الشامل V2</title>
  
  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <script src="https://cdn.jsdelivr.net/npm/dashjs@latest/dist/dash.all.min.js"></script>
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
  
  <style>
    :root {
      --bg-primary: #0A0A0A;
      --bg-surface: #141414;
      --bg-surface-light: #1E1E1E;
      --color-primary: #E50914;
      --color-primary-hover: #B80710;
      --color-text: #FFFFFF;
      --color-text-muted: #AAAAAA;
      --color-accent: #FFD700;
      --color-success: #00CC66;
    }

    * { 
      box-sizing: border-box; 
      -webkit-tap-highlight-color: transparent; 
      font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif; 
    }
    
    body { 
      background-color: var(--bg-primary); 
      color: var(--color-text); 
      margin: 0; 
      padding: 0; 
      padding-bottom: 75px; 
      overflow-x: hidden; 
    }
    
    /* 🔔 شريط التنبيهات */
    .notification-banner {
      position: fixed; top: -100px; left: 10px; right: 10px; 
      background: linear-gradient(90deg, var(--color-primary), var(--bg-surface));
      padding: 12px 15px; border-radius: 8px; display: flex; align-items: center; gap: 10px;
      box-shadow: 0 4px 15px rgba(229, 9, 20, 0.3); z-index: 9999; 
      transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
      border: 1px solid var(--color-primary);
    }
    .notification-banner.show { transform: translateY(115px); }
    .badge-ai { background: var(--color-text); color: var(--color-primary); font-size: 10px; font-weight: bold; padding: 2px 6px; border-radius: 4px; }

    /* الشاشات والتنقل */
    .screen { display: none; padding: 15px; animation: fadeIn 0.3s ease; }
    .screen.active { display: block; }
    @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
    
    .bottom-nav {
      position: fixed; bottom: 0; left: 0; right: 0; height: 65px;
      background-color: var(--bg-surface); display: flex; justify-content: space-around;
      align-items: center; border-top: 2px solid var(--color-primary); z-index: 1000;
    }
    .nav-item { display: flex; flex-direction: column; align-items: center; color: #666; cursor: pointer; font-size: 11px; font-weight: 500; transition: color 0.2s; }
    .nav-item.active { color: var(--color-primary); }
    .nav-item i { font-size: 24px; margin-bottom: 3px; }

    .section-title { color: var(--color-primary); font-size: 18px; font-weight: bold; margin: 25px 0 12px 0; border-right: 4px solid var(--color-primary); padding-right: 8px; display: flex; align-items: center; gap: 6px; }
    
    /* كروت القنوات والمباريات */
    .channel-card, .match-card {
      background: var(--bg-surface); border-radius: 8px; padding: 12px; margin-bottom: 10px;
      display: flex; align-items: center; border: 1px solid #222; transition: border-color 0.2s;
    }
    .channel-card:hover { border-color: #444; }
    .channel-logo { width: 55px; height: 55px; object-fit: contain; background: #fff; border-radius: 6px; margin-left: 12px; border: 1px solid #333; padding: 3px; }
    .channel-info { flex-grow: 1; }
    .channel-name { font-size: 14px; font-weight: bold; color: var(--color-text); }
    .channel-cat { font-size: 11px; color: var(--color-text-muted); margin-top: 4px; display: flex; align-items: center; gap: 5px; }
    .vip-tag { background: var(--color-accent); color: #000; font-size: 9px; font-weight: bold; padding: 1px 4px; border-radius: 3px; }
    .btn-play { color: var(--color-primary); font-size: 30px; cursor: pointer; transition: transform 0.1s; }
    .btn-play:hover { transform: scale(1.1); color: var(--color-primary-hover); }

    /* 📺 منطقة مشغل الفيديو والـ UI State */
    .player-header { background: var(--bg-surface); padding: 12px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #222; border-radius: 8px 8px 0 0; }
    .player-header .title { font-size: 15px; font-weight: bold; display: flex; align-items: center; gap: 8px; }
    .live-badge { background: var(--color-primary); color: #fff; font-size: 11px; font-weight: bold; padding: 3px 8px; border-radius: 4px; display: flex; align-items: center; gap: 4px; }
    .live-badge span { width: 8px; height: 8px; background: #fff; border-radius: 50%; display: inline-block; animation: pulse 1s infinite; }
    
    .media-container { width: 100%; background: #000; aspect-ratio: 16/9; max-height: 240px; position: relative; border-radius: 0 0 8px 8px; overflow: hidden; }
    video, iframe { width: 100%; height: 100%; border: none; display: block; }
    
    /* مؤشرات حالة البث */
    .player-overlay {
      position: absolute; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.75);
      display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; display: none;
    }
    .player-overlay.active { display: flex; }
    .player-overlay .status-text { margin-top: 10px; font-size: 14px; color: var(--color-text); font-weight: 500; }

    .quality-bar { background: var(--bg-surface); padding: 10px; display: flex; justify-content: space-between; align-items: center; font-size: 13px; border-bottom: 2px solid var(--color-primary); }
    .quality-select { background: #000; color: #fff; border: 1px solid var(--color-primary); padding: 4px 8px; border-radius: 4px; width: auto; margin-bottom: 0; }

    .player-actions { display: flex; gap: 10px; padding: 10px; background: var(--bg-surface); }
    .player-actions button { flex: 1; padding: 10px; border: none; border-radius: 6px; font-weight: bold; display: flex; align-items: center; justify-content: center; gap: 5px; cursor: pointer; font-size: 13px; }
    .btn-fav-toggle { background: var(--bg-surface-light); color: var(--color-accent); border: 1px solid #333 !important; }
    .btn-refresh-stream { background: var(--color-primary); color: #fff; }

    /* ⚽ شاشة المباريات */
    .match-card { flex-direction: column; align-items: stretch; padding: 14px; }
    .match-league { font-size: 11px; color: var(--color-primary); font-weight: bold; margin-bottom: 10px; text-align: center; letter-spacing: 0.5px; }
    .match-main { display: flex; justify-content: space-between; align-items: center; }
    .team { display: flex; flex-direction: column; align-items: center; gap: 6px; width: 35%; text-align: center; }
    .team img { width: 40px; height: 40px; object-fit: contain; background: #fff; border-radius: 6px; padding: 3px; border: 1px solid #333; }
    .team span { font-size: 13px; font-weight: bold; }
    .match-center { display: flex; flex-direction: column; align-items: center; gap: 5px; }
    .match-score { font-size: 22px; font-weight: bold; color: var(--color-accent); letter-spacing: 2px; }
    .match-time { background: var(--bg-surface-light); font-size: 11px; padding: 4px 10px; border-radius: 4px; font-weight: bold; border: 1px solid #333; }
    .match-time.live { background: var(--color-success); color: #000; animation: pulse 1.5s infinite; }
    .btn-ai-analyze { background: var(--bg-surface-light); border: 1px dashed var(--color-primary); color: #fff; border-radius: 6px; padding: 8px; margin-top: 12px; font-size: 12px; font-weight: bold; display: flex; align-items: center; justify-content: center; gap: 5px; cursor: pointer; width: 100%; }

    /* 🤖 شاشة الذكاء الاصطناعي AI */
    .ai-container { background: var(--bg-surface); border-radius: 8px; padding: 15px; border: 1px solid var(--color-primary); }
    .ai-chat-box { background: var(--bg-primary); border-radius: 6px; padding: 15px; min-height: 150px; border: 1px solid #222; margin-top: 15px; display: flex; flex-direction: column; justify-content: center; }
    .ai-btn-group { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 15px; }
    .ai-btn { background: var(--color-primary); color: #fff; border: none; padding: 12px; border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 5px; }
    .ai-btn.secondary { background: var(--bg-surface-light); border: 1px solid #333; }
    .ai-prediction-bar { background: #222; border-radius: 10px; height: 12px; margin-top: 15px; overflow: hidden; display: none; }
    .ai-progress { background: linear-gradient(90deg, var(--color-success), var(--color-accent)); height: 100%; width: 0%; transition: width 1s ease-in-out; }

    /* 📝 لوحة التحكم ونماذج الإدخال */
    input, select { width: 100%; padding: 12px; background: var(--bg-surface); border: 1px solid #333; color: #fff; border-radius: 6px; margin-bottom: 12px; font-size: 14px; transition: border-color 0.2s; }
    input:focus, select:focus { border-color: var(--color-primary); outline: none; }
    button.btn-submit { width: 100%; padding: 14px; background: var(--color-primary); color: #fff; border: none; border-radius: 6px; font-size: 15px; font-weight: bold; cursor: pointer; margin-bottom: 15px; transition: background 0.2s; }
    button.btn-submit:hover { background: var(--color-primary-hover); }
    .admin-list-item { background: var(--bg-surface-light); padding: 12px; border-radius: 6px; margin-bottom: 8px; display: flex; justify-content: space-between; align-items: center; border: 1px solid #333; }
    .btn-delete { background: var(--color-primary); color: white; border: none; border-radius: 4px; padding: 5px 8px; cursor: pointer; font-size: 12px; display: flex; align-items: center; }

    /* العناصر الديناميكية كـ الـ Spinners */
    .spinner { border: 3px solid rgba(255,255,255,0.1); border-radius: 50%; border-top: 3px solid var(--color-primary); width: 30px; height: 30px; animation: spin 0.8s linear infinite; }
    .global-loader { display: flex; justify-content: center; padding: 20px 0; }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    @keyframes pulse { 0% { opacity: 0.5; } 50% { opacity: 1; } 100% { opacity: 0.5; } }
  </style>
</head>
<body>

  <div id="appNotification" class="notification-banner">
    <span class="badge-ai">نظام التتبع</span>
    <div id="notificationText" style="font-size: 12px; font-weight: 500;">تم تحميل البيئة البرمجية المستقرة بنجاح.</div>
  </div>

  <div id="screen-home" class="screen active">
    <div class="section-title">⭐ القنوات المميزة وبث التضمين VIP</div>
    <div id="vip-channels-container"></div>
    
    <div class="section-title">🌐 دليل القنوات العام بالتطبيق</div>
    <div id="all-channels-container"></div>
  </div>

  <div id="screen-live" class="screen">
    <div class="player-header">
      <div class="title"><i class="material-icons">tv</i> <span id="current-stream-title">يرجى تحديد بث للتشغيل</span></div>
      <div class="live-badge"><span></span> LIVE</div>
    </div>
    
    <div class="media-container">
      <div id="playerOverlay" class="player-overlay">
        <div class="spinner"></div>
        <div id="overlayStatusText" class="status-text">جاري الاتصال بسيرفر البث...</div>
      </div>
      
      <video id="appVideoPlayer" controls playsinline></video>
      <iframe id="appEmbedPlayer" allowfullscreen style="display: none;"></iframe>
    </div>
    
    <div class="quality-bar">
      <span>تعديل مسار جودة الاتصال</span>
      <select class="quality-select" id="streamQualitySelector">
        <option value="auto">تعديل تلقائي الذكي (Adaptive)</option>
        <option value="high">سيرفر عالي الجودة HD</option>
        <option value="low">سيرفر حفظ البيانات SD</option>
      </select>
    </div>
    
    <div class="player-actions">
      <button class="btn-fav-toggle" onclick="alert('تم التحديث المفضلة بنجاح!')"><i class="material-icons">star</i> حفظ بالمفضلة</button>
      <button class="btn-refresh-stream" id="btnRefreshStream"><i class="material-icons">refresh</i> إعادة الاتصال</button>
    </div>

    <div class="section-title">📺 دليل التنقل السريع بين قنوات البث</div>
    <div id="quick-channels-container"></div>
  </div>

  <div id="screen-matches" class="screen">
    <div class="section-title"><i class="material-icons" style="color:var(--color-success);">sports_soccer</i> مباريات ومواعيد بث اليوم</div>
    <div id="matches-container"></div>
  </div>

  <div id="screen-ai" class="screen">
    <div class="section-title"><i class="material-icons" style="color:var(--color-accent);">psychology</i> محرك التحليل والذكاء الاصطناعي</div>
    <div class="ai-container">
      <label style="font-size: 13px; color: var(--color-text-muted); display:block; margin-bottom:8px;">اختر المباراة المستهدفة للتحليل الرقمي:</label>
      <select id="aiMatchSelector"></select>
      
      <div class="ai-btn-group">
        <button class="ai-btn" id="btnAiPredict"><i class="material-icons">online_prediction</i> توقع النتيجة الإحصائية</button>
        <button class="ai-btn secondary" id="btnAiAnalyze"><i class="material-icons">analytics</i> تحليل التكتيك الفني</button>
      </div>

      <div class="ai-prediction-bar" id="aiPredictionBar">
        <div class="ai-progress" id="aiProgressFill"></div>
      </div>

      <div class="ai-chat-box" id="aiChatBox">
        <div id="aiResponseText" style="text-align: center; color: var(--color-text-muted); font-size: 14px;">حدد لقاءً ثم شغل محرك التحليل لاستخراج البيانات الفورية.</div>
      </div>
    </div>
  </div>

  <div id="screen-dashboard" class="screen">
    <div class="section-title">📺 إضافة قناة أو بث تضمين جديد للتطبيق</div>
    <form id="addChannelForm">
      <input type="text" id="targetName" placeholder="اسم القناة / البث (مثال: قناة الجزيرة مباشر)" required>
      <input type="url" id="targetUrl" placeholder="رابط البث المباشر (M3U8 أو MPD أو رابط YouTube / Facebook)" required>
      <input type="url" id="targetLogo" placeholder="رابط الصورة المباشرة للشعار (اتركه فارغاً للتوليد الذكي)">
      <select id="targetCategory">
        <option value="رياضية">رياضية</option>
        <option value="إخبارية">إخبارية</option>
        <option value="ترفيهية">ترفيهية</option>
        <option value="سوشيال ميديا">سوشيال ميديا (YouTube / Facebook)</option>
        <option value="أخرى">تصنيفات عامة أخرى</option>
      </select>
      <div style="margin-bottom:15px; display: flex; align-items: center; gap: 8px;">
        <input type="checkbox" id="targetIsVip" style="width: auto; margin-bottom: 0;"> 
        <label for="targetIsVip" style="font-size: 14px; user-select: none;">تعيين كقناة مميزة عالية الجودة VIP ⭐</label>
      </div>
      <button type="submit" class="btn-submit">تأكيد وحفظ القناة فوراً</button>
    </form>

    <div class="section-title">⚙️ التحكم بالمسارات الحالية المدمجة</div>
    <div id="admin-channels-container"></div>
  </div>

  <div class="bottom-nav">
    <div class="nav-item active" data-screen="home"><i class="material-icons">home</i>الرئيسية</div>
    <div class="nav-item" data-screen="live"><i class="material-icons">tv</i>البث مباشر</div>
    <div class="nav-item" data-screen="matches"><i class="material-icons">sports_soccer</i>المباريات</div>
    <div class="nav-item" data-screen="ai"><i class="material-icons">psychology</i>المساعد AI</div>
    <div class="nav-item" data-screen="dashboard"><i class="material-icons">settings</i>التحكم</div>
  </div>

  <script>
    /**
     * 🌐 الإعدادات الهيكلية العامة للتطبيق للتحديث المستقبلي والتطوير لـ Backend API
     */
    const AppConfig = {
      STORAGE_VERSION: 'v2.1',
      STORAGE_KEY: 'iptv_app_channels_v2',
      PROXY_SERVER_URL: '', 
      AI_API_URL: '',       
      OBFUSCATE_LINKS: false 
    };

    /**
     * 🔒 وحدة الأمان الأساسية والتشفير المؤقت للروابط لمنع فحص الشفرات العشوائي
     */
    const SecurityUtil = {
      obfuscate: (text) => AppConfig.OBFUSCATE_LINKS ? btoa(encodeURIComponent(text)) : text,
      deobfuscate: (encoded) => {
        if (!AppConfig.OBFUSCATE_LINKS) return encoded;
        try { return decodeURIComponent(atob(encoded)); } catch(e) { return encoded; }
      }
    };

    /**
     * 📦 وحدة إدارة وتأمين الـ LocalStorage (Storage Engine Layer)
     */
    const AppStorage = {
      getDefaultChannels() {
        return [
          { id: "ch_yt_aljazeera", name: "بث مباشر قناة الجزيرة الإخبارية | YouTube", url: SecurityUtil.obfuscate("https://www.youtube.com/watch?v=bNYXQDZ66Og"), logo: "https://upload.wikimedia.org/wikipedia/commons/thumb/f/f2/Al_Jazeera_領و.svg/512px-Al_Jazeera_領و.svg.png", category: "إخبارية", isVip: true },
          { id: "ch_fb_sample", name: "بث مباشر فيسبوك التجريبي | Facebook", url: SecurityUtil.obfuscate("https://www.facebook.com/facebook/videos/10153231379946729/"), logo: "https://upload.wikimedia.org/wikipedia/commons/thumb/0/05/Facebook_Logo_%282019%29.png/512px-Facebook_Logo_%282019%29.png", category: "سوشيال ميديا", isVip: true },
          { id: "ch_2m_maroc", name: "دوزيم تيفي المغربية | 2M TV Live", url: SecurityUtil.obfuscate("https://stream-lb.livemediama.com/2m/hls/master.m3u8"), logo: "https://upload.wikimedia.org/wikipedia/commons/thumb/c/cd/2M_Maroc_logo.svg/512px-2M_Maroc_logo.svg.png", category: "مغربية", isVip: true },
          { id: "ch_alkass_one", name: "قناة الكأس 1 الرياضية | Al Kass HD", url: SecurityUtil.obfuscate("https://liveeu-gcp.alkassdigital.net/alkass1-p/main.m3u8"), logo: "https://upload.wikimedia.org/wikipedia/commons/thumb/a/ad/Al_Kass_Sports_Channels_logo.svg/512px-Al_Kass_Sports_Channels_logo.svg.png", category: "رياضية", isVip: false }
        ];
      },
      
      getDefaultMatches() {
        return [
          { id: "match_01", league: "الدوري الاسباني", t1: "ريال مدريد", t1Logo: "https://upload.wikimedia.org/wikipedia/en/thumb/5/56/Real_Madrid_CF.svg/512px-Real_Madrid_CF.svg.png", t2: "برشلونة", t2Logo: "https://upload.wikimedia.org/wikipedia/en/thumb/4/47/FC_Barcelona_%28crest%29.svg/512px-FC_Barcelona_%28crest%29.svg.png", s1: "2", s2: "1", time: "LIVE" },
          { id: "match_02", league: "دوري أبطال أوروبا", t1: "ليفربول", t1Logo: "https://upload.wikimedia.org/wikipedia/en/thumb/0/0c/Liverpool_FC.svg/512px-Liverpool_FC.svg.png", t2: "مانشستر سيتي", t2Logo: "https://upload.wikimedia.org/wikipedia/en/thumb/e/eb/Manchester_City_FC_badge.svg/512px-Manchester_City_FC_badge.svg.png", s1: "0", s2: "0", time: "22:00" }
        ];
      },

      loadChannels() {
        try {
          const stored = localStorage.getItem(`${AppConfig.STORAGE_KEY}_data`);
          const version = localStorage.getItem(`${AppConfig.STORAGE_KEY}_ver`);
          if (stored && version === AppConfig.STORAGE_VERSION) {
            return JSON.parse(stored);
          }
          this.saveChannels(this.getDefaultChannels());
          localStorage.setItem(`${AppConfig.STORAGE_KEY}_ver`, AppConfig.STORAGE_VERSION);
          return this.getDefaultChannels();
        } catch (e) {
          console.error("Storage load failure, recovery initialized", e);
          return this.getDefaultChannels();
        }
      },

      saveChannels(data) {
        try {
          localStorage.setItem(`${AppConfig.STORAGE_KEY}_data`, JSON.stringify(data));
        }  catch(e) {
          console.error("Failed to save state to localStorage", e);
        }
      }
    };

    /**
     * 📺 وحدة مشغل البث الاحترافي - مع معالجة الأخطاء والتنظيف من الذاكرة (Memory Management Player)
     */
    class StreamPlayer {
      constructor(videoElement, embedElement, overlayElement, statusTextElement) {
        this.video = videoElement;
        this.embed = embedElement;
        this.overlay = overlayElement;
        this.statusText = statusTextElement;
        this.hlsInstance = null;
        this.dashPlayer = null;
        this.currentUrl = "";
      }

      showStatus(text) {
        this.overlay.classList.add('active');
        this.statusText.innerText = text;
      }

      hideStatus() {
        this.overlay.classList.remove('active');
      }

      resetEngines() {
        // تفكيك محركات HLS لمنع الـ Memory Leaks المتراكمة في المتصفح
        if (this.hlsInstance) {
          this.hlsInstance.destroy();
          this.hlsInstance = null;
        }
        // تفكيك مشغل DASH
        if (this.dashPlayer) {
          this.dashPlayer.reset();
          this.dashPlayer = null;
        }
        // إيقاف وتصفير عناصر الـ DOM الأساسية
        this.video.pause();
        this.video.src = "";
        this.video.load();
        this.video.style.display = "none";
        this.embed.src = "";
        this.embed.style.display = "none";
      }

      // تحليل الروابط الذكي (YouTube / Facebook Embed Hooks)
      parseEmbedUrl(url) {
        if (url.includes('youtube.com/watch?v=')) {
          const id = url.split('v=')[1].split('&')[0];
          return `https://www.youtube.com/embed/${id}?autoplay=1`;
        }
        if (url.includes('youtu.be/')) {
          const id = url.split('youtu.be/')[1].split('?')[0];
          return `https://www.youtube.com/embed/${id}?autoplay=1`;
        }
        if (url.includes('facebook.com/')) {
          return `https://www.facebook.com/plugins/video.php?href=${encodeURIComponent(url)}&show_text=0&autoplay=1`;
        }
        return null;
      }

      play(rawUrl, streamTitle) {
        this.currentUrl = SecurityUtil.deobfuscate(rawUrl);
        document.getElementById('current-stream-title').innerText = streamTitle;
        this.resetEngines();
        this.showStatus("جاري تحضير مسار البث...");

        const embedUrl = this.parseEmbedUrl(this.currentUrl);
        if (embedUrl) {
          // تشغيل روابط تضمين الشبكات الاجتماعية
          this.embed.src = embedUrl;
          this.embed.style.display = "block";
          this.hideStatus();
          return;
        }

        // تشغيل البث المباشر المباشر (HLS / DASH / MP4)
        this.video.style.display = "block";

        if (this.currentUrl.endsWith('.m3u8')) {
          if (Hls.isSupported()) {
            this.hlsInstance = new Hls({ maxBufferSize: 0, maxBufferLength: 10, liveSyncDuration: 3 });
            this.hlsInstance.loadSource(this.currentUrl);
            this.hlsInstance.attachMedia(this.video);
            this.hlsInstance.on(Hls.Events.MANIFEST_PARSED, () => {
              this.video.play().catch(() => {});
              this.hideStatus();
            });
            this.hlsInstance.on(Hls.Events.ERROR, (event, data) => {
              if (data.fatal) this.showStatus("خطأ في معالجة الإشارة. حاول إعادة الاتصال.");
            });
          } else if (this.video.canPlayType('application/vnd.apple.mpegurl')) {
            // دعم Safari الحصري لروابط HLS الأصلية
            this.video.src = this.currentUrl;
            this.video.addEventListener('loadedmetadata', () => {
              this.video.play().catch(() => {});
              this.hideStatus();
            });
          } else {
            this.showStatus("هذا المتصفح لا يدعم بثوث HLS.");
          }
        } else if (this.currentUrl.endsWith('.mpd')) {
          // معالجة تدفقات MPEG-DASH الاحترافية
          this.dashPlayer = dashjs.MediaPlayer().create();
          this.dashPlayer.initialize(this.video, this.currentUrl, true);
          this.dashPlayer.on(dashjs.MediaPlayer.events.STREAM_INITIALIZED, () => this.hideStatus());
          this.dashPlayer.on(dashjs.MediaPlayer.events.ERROR, () => this.showStatus("فشل تشغيل مسار DASH."));
        } else {
          // الروابط المباشرة العادية MP4 / WebM
          this.video.src = this.currentUrl;
          this.video.play()
            .then(() => this.hideStatus())
            .catch(() => this.showStatus("فشل العرض المباشر للمصدر."));
        }
      }
    }

    /**
     * 🧠 المحرك الكلي لإدارة وتوجيه عناصر الـ UI (Core Application Shell)
     */
    const AppEngine = {
      channels: [],
      matches: [],
      player: null,

      init() {
        this.channels = AppStorage.loadChannels();
        this.matches = AppStorage.getDefaultMatches();
        
        // ربط محرك الـ Player بعناصر الـ DOM
        this.player = new StreamPlayer(
          document.getElementById('appVideoPlayer'),
          document.getElementById('appEmbedPlayer'),
          document.getElementById('playerOverlay'),
          document.getElementById('overlayStatusText')
        );

        this.registerGlobalEvents();
        this.renderAllUI();
        this.triggerSystemNotification("مرحباً بك في منصة IPTV V2. البيئة سريعة ومستقرة.");
      },

      registerGlobalEvents() {
        // 🗺️ نظام التنقل السلس عبر الـ Tabs (Routing Matrix)
        document.querySelectorAll('.bottom-nav .nav-item').forEach(btn => {
          btn.addEventListener('click', (e) => {
            const targetScreen = btn.getAttribute('data-screen');
            
            document.querySelectorAll('.bottom-nav .nav-item').forEach(b => b.classList.remove('active'));
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            
            btn.classList.add('active');
            document.getElementById(`screen-${targetScreen}`).classList.add('active');
          });
        });

        // زر إعادة الاتصال الذكي بالمشغل
        document.getElementById('btnRefreshStream').addEventListener('click', () => {
          if (this.player.currentUrl) {
            this.player.play(SecurityUtil.obfuscate(this.player.currentUrl), document.getElementById('current-stream-title').innerText);
          } else {
            alert("يرجى تحديد قناة أولاً لإعادة الاتصال بالمسار.");
          }
        });

        // معالجة نموذج لوحة تحكم الأدمن (إضافة قناة جديدة)
        document.getElementById('addChannelForm').addEventListener('submit', (e) => {
          e.preventDefault();
          const name = document.getElementById('targetName').value.trim();
          const url = document.getElementById('targetUrl').value.trim();
          let logo = document.getElementById('targetLogo').value.trim();
          const category = document.getElementById('targetCategory').value;
          const isVip = document.getElementById('targetIsVip').checked;

          if (!logo) logo = "https://images.unsplash.com/photo-1594909122845-11baa439b7bf?q=80&w=200&auto=format&fit=crop";

          const newChannel = {
            id: 'ch_' + Date.now(),
            name,
            url: SecurityUtil.obfuscate(url),
            logo,
            category,
            isVip
          };

          this.channels.push(newChannel);
          AppStorage.saveChannels(this.channels);
          this.renderAllUI();
          
          document.getElementById('addChannelForm').reset();
          this.triggerSystemNotification(`تمت إضافة القناة [ ${name} ] بنجاح للتطبيق.`);
        });

        // أزرار محاكاة شاشة المساعد AI
        document.getElementById('btnAiPredict').addEventListener('click', () => this.simulateAiExecution('predict'));
        document.getElementById('btnAiAnalyze').addEventListener('click', () => this.simulateAiExecution('analyze'));
      },

      triggerSystemNotification(text) {
        const banner = document.getElementById('appNotification');
        document.getElementById('notificationText').innerText = text;
        banner.classList.add('show');
        setTimeout(() => banner.classList.remove('show'), 4000);
      },

      createChannelCard(ch) {
        return `
          <div class="channel-card">
            <img class="channel-logo" src="${ch.logo}" onerror="this.src='https://placehold.co/100x100?text=TV'" alt="Logo">
            <div class="channel-info">
              <div class="channel-name">${ch.name}</div>
              <div class="channel-cat">
                ${ch.isVip ? '<span class="vip-tag">VIP ⭐</span>' : ''}
                <i class="material-icons" style="font-size:12px;">layers</i> ${ch.category}
              </div>
            </div>
            <i class="material-icons btn-play" onclick="AppEngine.launchStream('${ch.url}', '${ch.name}')">play_circle_filled</i>
          </div>
        `;
      },

      launchStream(obfuscatedUrl, name) {
        // تحويل المستخدم لشاشة البث تلقائياً وتشغيل المسار فوراً
        document.querySelector('[data-screen="live"]').click();
        this.player.play(obfuscatedUrl, name);
      },

      deleteChannel(id) {
        if(confirm("هل أنت متأكد من رغبتك في حذف هذه القناة من النظام؟")) {
          this.channels = this.channels.filter(c => c.id !== id);
          AppStorage.saveChannels(this.channels);
          this.renderAllUI();
          this.triggerSystemNotification("تم إقصاء وحذف المسار البرمجي للقناة.");
        }
      },

      simulateAiExecution(mode) {
        const selector = document.getElementById('aiMatchSelector');
        if (selector.options.length === 0) {
          alert("لا يوجد لقاءات متاحة للتحليل حالياً.");
          return;
        }
        const matchName = selector.options[selector.selectedIndex].text;
        
        const bar = document.getElementById('aiPredictionBar');
        const fill = document.getElementById('aiProgressFill');
        const box = document.getElementById('aiChatBox');
        
        bar.style.display = "block";
        fill.style.width = "0%";
        box.innerHTML = `<div class="global-loader"><div class="spinner"></div><span style="margin-right:10px;">جاري فحص التكتيكات الفنية وحساب الاحتمالات الإحصائية...</span></div>`;
        
        setTimeout(() => fill.style.width = "100%", 50);

        setTimeout(() => {
          bar.style.display = "none";
          if (mode === 'predict') {
            box.innerHTML = `
              <div style="border-right: 3px solid var(--color-success); padding-right:10px;">
                <h4 style="color:var(--color-success); margin:0 0 5px 0;">🔮 توقعات التقييم الرقمي الفوري لـ:</h4>
                <p style="margin:5px 0; font-size:14px; font-weight:bold;">${matchName}</p>
                <p style="margin:5px 0 0 0; color:var(--color-text-muted); font-size:13px;">بناءً على تتبع أداء آخر 5 مواجهات: فرصة فوز الفريق المستضيف هي <b>64%</b>، ونسبة التعادل الرقمي <b>21%</b>. النتيجة المتوقعة هى (2 - 1).</p>
              </div>`;
          } else {
            box.innerHTML = `
              <div style="border-right: 3px solid var(--color-accent); padding-right:10px;">
                <h4 style="color:var(--color-accent); margin:0 0 5px 0;">📊 التحليل الفني والعمق التكتيكي:</h4>
                <p style="margin:5px 0; font-size:14px; font-weight:bold;">${matchName}</p>
                <p style="margin:5px 0 0 0; color:var(--color-text-muted); font-size:13px;">التحليل الإحصائي يوضح اعتماد الفريقين على تكتيك (4-3-3) الممتد مع الضغط العالي من وسط الملعب. الثغرة الأساسية تقع خلف الأجنحة الدفاعية المتقدمة.</p>
              </div>`;
          }
        }, 1500);
      },

      renderAllUI() {
        const vipContainer = document.getElementById('vip-channels-container');
        const allContainer = document.getElementById('all-channels-container');
        const quickContainer = document.getElementById('quick-channels-container');
        const adminContainer = document.getElementById('admin-channels-container');
        const matchesContainer = document.getElementById('matches-container');
        const aiSelector = document.getElementById('aiMatchSelector');

        // إعادة تصفير الحاويات
        vipContainer.innerHTML = "";
        allContainer.innerHTML = "";
        quickContainer.innerHTML = "";
        adminContainer.innerHTML = "";
        matchesContainer.innerHTML = "";
        aiSelector.innerHTML = "";

        // 1. رندرة القنوات بمختلف الشاشات
        this.channels.forEach(ch => {
          const htmlCard = this.createChannelCard(ch);
          quickContainer.insertAdjacentHTML('beforeend', htmlCard);

          if (ch.isVip) {
            vipContainer.insertAdjacentHTML('beforeend', htmlCard);
          } else {
            allContainer.insertAdjacentHTML('beforeend', htmlCard);
          }

          // قائمة التحكم (الأدمن)
          const adminItem = `
            <div class="admin-list-item">
              <span style="font-size:13px; font-weight:500;">${ch.name} [ ${ch.category} ]</span>
              <button class="btn-delete" onclick="AppEngine.deleteChannel('${ch.id}')"><i class="material-icons" style="font-size:16px;">delete</i> حذف</button>
            </div>
          `;
          adminContainer.insertAdjacentHTML('beforeend', adminItem);
        });

        // 2. رندرة المباريات وجدول البيانات والـ AI Selector
        this.matches.forEach(m => {
          const isLive = m.time === "LIVE";
          const matchHtml = `
            <div class="match-card">
              <div class="match-league">${m.league}</div>
              <div class="match-main">
                <div class="team">
                  <img src="${m.t1Logo}" onerror="this.src='https://placehold.co/100x100?text=T1'" alt="Team 1">
                  <span>${m.t1}</span>
                </div>
                <div class="match-center">
                  <div class="match-score">${m.s1} : ${m.s2}</div>
                  <div class="match-time ${isLive ? 'live' : ''}">${m.time}</div>
                </div>
                <div class="team">
                  <img src="${m.t2Logo}" onerror="this.src='https://placehold.co/100x100?text=T2'" alt="Team 2">
                  <span>${m.t2}</span>
                </div>
              </div>
              <button class="btn-ai-analyze" onclick="document.querySelector('[data-screen=\\'ai\\']').click(); document.getElementById('aiMatchSelector').value='${m.id}'; AppEngine.simulateAiExecution('predict');">
                <i class="material-icons" style="font-size:15px; color:var(--color-accent);">psychology</i> قراءة البيانات فورياً عبر الذكاء الاصطناعي
              </button>
            </div>
          `;
          matchesContainer.insertAdjacentHTML('beforeend', matchHtml);

          // إدراج خيارات الفحص في شاشة الـ AI
          const opt = document.createElement('option');
          opt.value = m.id;
          opt.text = `${m.t1} ضد ${m.t2} (${m.league})`;
          aiSelector.add(opt);
        });
      }
    };

    // إطلاق البنية الأساسية للمنصة عند اكتمال تحميل الصفحة
    window.addEventListener('DOMContentLoaded', () => AppEngine.init());
  </script>
</body>
</html>
