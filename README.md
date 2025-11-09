# geo-tour-v1
Jeodinamik zincir üzerinden çoktan seçmeli quiz sistemi: Engabreen'den Simav'a interaktif öğrenme.
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>JeoTurizm: Engabreen → Simav</title>

    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"/>
    <style>
      :root { 
        --bg: #f0f4f8; --text: #333; --card: #fff; --quiz: #fffbe6; 
        --flow1: #3a8fb7; --flow2: #c62828; --flow3: #1565c0; --flow4: #7b1fa2; --flow5: #ff6f00; --flow6: #2e7d32;
        --btn: #0055a5; --font-h: 'EB Garamond', serif; --font-b: 'Inter', sans-serif;
      }
      * { box-sizing: border-box; margin:0; padding:0; }
      body { font-family: var(--font-b); line-height: 1.6; color: var(--text); background: var(--bg); padding: 15px; }
      .jeoturizm-wrapper { font-family: var(--font-b); line-height: 1.6; color: var(--text); background: var(--bg); }
      .jeoturizm-wrapper .container { max-width: 800px; margin: 0 auto; }
      .jeoturizm-wrapper .card { padding: 15px; border-radius: 8px; background: var(--card); border: 1px solid #ddd; margin-bottom: 20px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
      .jeoturizm-wrapper #map { height: 350px; border-radius: 10px; margin-bottom: 20px; border: 2px solid var(--btn); }
      .jeoturizm-wrapper h1, .jeoturizm-wrapper h2, .jeoturizm-wrapper h3 { font-family: var(--font-h); margin-bottom: 10px; }
      .jeoturizm-wrapper button, .jeoturizm-wrapper a.jeo-btn { padding: 8px 15px; border: none; border-radius: 5px; cursor: pointer; margin: 5px 2px; transition: 0.2s; text-decoration: none; display: inline-block; font-size: 0.9em; }
      .jeoturizm-wrapper button:hover, .jeoturizm-wrapper a.jeo-btn:hover { filter: brightness(1.1); }
      .jeoturizm-wrapper .card-menu { padding: 15px; border-radius: 8px; background: var(--card); border: 1px solid #ddd; margin-bottom: 15px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); cursor: pointer; transition: 0.2s; }
      .jeoturizm-wrapper .card-menu:hover { box-shadow: 0 4px 8px rgba(0,0,0,0.1); transform: translateY(-2px); }
      .jeoturizm-wrapper .card-menu h3 { margin-bottom: 5px; }
      .jeoturizm-wrapper .card-menu p { font-size: 0.9em; color: #555; }
      .jeoturizm-wrapper .quiz-option { background: #eee; padding: 10px; margin: 5px 0; border-radius: 5px; cursor: pointer; transition: 0.2s; }
      .jeoturizm-wrapper .quiz-option:hover { background: #ddd; }
      .jeoturizm-wrapper .quiz-option.selected { background: #f7c5c5; }
      .jeoturizm-wrapper .quiz-option.correct { background: #c8f7c5; }
      .jeoturizm-wrapper .quiz-option.disabled { cursor: default; }
      .jeoturizm-wrapper .quiz-option.correct.disabled { background: #c8f7c5; font-weight: bold; }
      .jeoturizm-wrapper .quiz-option.selected.disabled { background: #f7c5c5; }
      .jeoturizm-wrapper .quiz-option.disabled:not(.correct):not(.selected) { background: #f0f0f0; color: #888; }
      .jeoturizm-wrapper .flow-map { background: #e9ecef; padding: 20px 10px; border-radius: 10px; margin-bottom: 30px; border: 1px solid #dee2e6; display: flex; flex-direction: column; align-items: center; gap: 15px; }
      .jeoturizm-wrapper .flow-node { width: 90%; max-width: 250px; padding: 10px 15px; border-radius: 8px; text-align: center; cursor: pointer; font-weight: 600; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1); transition: 0.2s; }
      .jeoturizm-wrapper .flow-node:hover { transform: translateY(-2px); box-shadow: 0 6px 10px rgba(0,0,0,0.15); }
      .jeoturizm-wrapper .flow-arrow { font-size: 24px; color: #495057; }
      .jeoturizm-wrapper .flow-node-1 { background: var(--flow1); } .jeoturizm-wrapper .flow-node-2 { background: var(--flow2); } .jeoturizm-wrapper .flow-node-3 { background: var(--flow3); }
      .jeoturizm-wrapper .flow-node-4 { background: var(--flow4); } .jeoturizm-wrapper .flow-node-5 { background: var(--flow5); } .jeoturizm-wrapper .flow-node-6 { background: var(--flow6); }
      .jeoturizm-wrapper .ge-embed { display: none; margin-top: 10px; }
      .jeoturizm-wrapper .ge-embed.open { display: block; }
      .jeoturizm-wrapper .lang-btn { float: right; background: var(--btn); color: white; }
      .jeoturizm-wrapper .image-container { position: relative; overflow: hidden; border-radius: 10px; margin: 15px 0; box-shadow: 0 4px 8px rgba(0,0,0,0.1); transition: all 0.3s ease; cursor: pointer; }
      .jeoturizm-wrapper .image-container img { width: 100%; height: auto; display: block; transition: transform 0.4s ease; }
      .jeoturizm-wrapper .image-container:hover { transform: translateY(-3px); box-shadow: 0 8px 16px rgba(0,0,0,0.15); }
      .jeoturizm-wrapper .image-container:hover img { transform: scale(1.05); }
      .jeoturizm-wrapper .image-overlay { position: absolute; bottom: 0; left: 0; right: 0; background: linear-gradient(transparent, rgba(0,0,0,0.7)); color: white; padding: 20px 15px 15px; font-size: 0.9em; opacity: 0; transition: opacity 0.3s ease; }
      .jeoturizm-wrapper .image-container:hover .image-overlay { opacity: 1; }
      .jeoturizm-wrapper #lightbox { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); justify-content: center; align-items: center; z-index: 9999; }
      .jeoturizm-wrapper #lightbox.open { display: flex; }
      .jeoturizm-wrapper .lightbox-content { position: relative; background: #fff; border-radius: 12px; max-width: 90%; max-height: 90%; overflow: auto; padding: 20px; text-align: center; box-shadow: 0 10px 30px rgba(0,0,0,0.3); }
      .jeoturizm-wrapper #lightbox .close { position: absolute; top: 10px; right: 15px; font-size: 2em; cursor: pointer; color: #aaa; }
      .jeoturizm-wrapper #lightbox .close:hover { color: #000; }
      .jeoturizm-wrapper #lb-img { max-width: 100%; height: auto; border-radius: 8px; margin: 10px 0; }
      .jeoturizm-wrapper #lb-title { margin: 0 0 10px; color: var(--btn); font-family: var(--font-h); }
      .jeoturizm-wrapper #lb-desc { margin-top: 10px; font-size: 0.95em; color: #555; }
      .jeoturizm-wrapper .earth-btn { background: #1a73e8; color: white; }
      .jeoturizm-wrapper .voice-btn { background: #6a1b9a; color: white; }
    </style>
</head>
<body>

    <div class="jeoturizm-wrapper">
      <div class="container">
        <button class="lang-btn" id="lang-btn">English</button>
        <h1 id="title" style="text-align:center;color:var(--btn);font-size:2em;"></h1>
        
        <div id="main"></div>
        
      </div>

      <div id="lightbox">
        <div class="lightbox-content">
          <span class="close" onclick="window.jeotur.closeLightbox()">×</span>
          <h2 id="lb-title"></h2>
          <img id="lb-img" src="" alt="">
          <p id="lb-desc"></p>
        </div>
      </div>
    </div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
    (function() {
      // Global bir wrapper kullanarak Blogger'ın global 'window' alanı ile çakışmayı önleyelim
      window.jeotur = {
        lang: 'tr', // init fonksiyonunda localStorage'dan güncellenecek
        map: null,
        quizIcons: {
          engabreen: "📈",
          simav: "⇲",
          eynal: "♨️",
          nasa: "🏞️",
          citgol: "⚗️",
          sismik: "📡"
        },
        data: {
          title: { tr: "Engabreen → Simav: Buzuldan Sıcak Suya", en: "Engabreen → Simav: Glacier to Hot Spring" },
          desc: { tr: "Bu jeodinamik tur, Norveç'teki buzul kalkmasından Batı Anadolu'daki jeotermal alanlara uzanan nedensel zinciri inceler.", en: "This geodynamic tour explores the causal chain from glacial rebound in Norway to geothermal fields in Western Anatolia." },
          stops: [
            { 
              id: "engabreen", lat: 66.65, lng: 13.85, order: 1,
              name: { tr: "Engabreen Buzulu", en: "Engabreen Glacier" }, color: "#3a8fb7",
              img: "https://upload.wikimedia.org/wikipedia/commons/2/2a/Engabreen_glacier.jpg",
              overlay: { tr: "Svartisen Buzulu'nun kolu. İzostatik yükselme burada başlar.", en: "Branch of Svartisen. Isostatic rebound starts here." },
              desc: { tr: "Engabreen, Svartisen buzullarının bir koludur. Buzul, kayayı törpüleyerek vadileri genişletir.", en: "Engabreen is part of Svartisen. Glacier erodes rock, widens valleys." },
              earth: "https://earth.google.com/web/@66.660,13.610,1500a,4000d,55y,10h,50t,0r",
              voice: { tr: "Engabreen’de kaya ile buz arasındaki yavaş ama güçlü ilişkiyi gözlemleyin.", en: "Observe the slow but powerful interaction between ice and rock." },
              quiz: { 
                q_tr: "Buzul kalkması ne yaratır?", q_en: "What does glacier unloading cause?",
                a_tr: "Yüzey yükselmesi", a_en: "Surface uplift",
                options_tr: ["Yüzey yükselmesi", "Deniz çekilmesi", "Fay oluşturur"],
                options_en: ["Surface uplift", "Sea retreat", "Creates fault"]
              }
            },
            { 
              id: "simav", lat: 39.0894, lng: 28.9761, order: 2,
              name: { tr: "Simav Fay Zonu", en: "Simav Fault Zone" }, color: "#c62828",
              img: "https://upload.wikimedia.org/wikipedia/commons/thumb/9/9f/Simav_Fault_Zone.jpg/800px-Simav_Fault_Zone.jpg",
              overlay: { tr: "Batı Anadolu uzanma rejiminde normal faylar.", en: "Normal faults in West Anatolia extension." },
              desc: { tr: "Normal faylar gerilme altında açılır.", en: "Normal faults open under tension." },
              earth: "https://earth.google.com/web/@39.089,28.976,100a,1000d,35y,0h,0t,0r",
              voice: { tr: "Simav’da gerilme fayları tetikler.", en: "Stress activates faults in Simav." },
              quiz: { 
                q_tr: "Fay tipi nedir?", q_en: "What type of fault is it?",
                a_tr: "Normal fay", a_en: "Normal fault",
                options_tr: ["Normal fay", "Ters fay", "Doğrultu atımlı fay"],
                options_en: ["Normal fault", "Reverse fault", "Strike-slip fault"]
              }
            },
            { 
              id: "eynal", lat: 39.0948, lng: 28.9584, order: 3,
              name: { tr: "Eynal Kaplıcası", en: "Eynal Hot Spring" }, color: "#1565c0",
              img: "https://upload.wikimedia.org/wikipedia/commons/thumb/2/2a/Eynal_Kapl%C4%B1cas%C4%B1.jpg/800px-Eynal_Kapl%C4%B1cas%C4%B1.jpg",
              overlay: { tr: "60°C termal su faydan çıkar.", en: "60°C water rises through fault." },
              desc: { tr: "Fay zonu sıcak suyun yolunu açar.", en: "Fault zone opens path for hot water." },
              earth: "https://earth.google.com/web/@39.095,28.958,100a,1000d,35y,0h,0t,0r",
              voice: { tr: "Eynal’da faylar sıcak suyun yolunu açar.", en: "Faults open pathways for hot water in Eynal." },
              quiz: { 
                q_tr: "Sıcak su nasıl çıkar?", q_en: "How does hot water rise?",
                a_tr: "Fay çatlaklarından", a_en: "From fault cracks",
                options_tr: ["Fay çatlaklarından", "Volkanik bacadan", "Yeraltı gölünden"],
                options_en: ["From fault cracks", "From volcanic vent", "From underground lake"]
              }
            },
            { 
              id: "nasa", lat: 39.106, lng: 28.987, order: 4,
              name: { tr: "Naşa Termal Alanı", en: "Naşa Thermal Area" }, color: "#7b1fa2",
              img: "https://upload.wikimedia.org/wikipedia/commons/thumb/3/3f/Simav_hot_spring_Kutahya.jpg/800px-Simav_hot_spring_Kutahya.jpg",
              overlay: { tr: "Fay kenarlarında doğal termal havuzlar.", en: "Natural thermal pools along fault edges." },
              desc: { tr: "Fay kenarlarında doğal termal havuzlar oluşur. Jeolojik laboratuvar.", en: "Natural thermal pools form along faults. Geological laboratory." },
              earth: "https://earth.google.com/web/@39.106,28.987,50a,500d,35y,-20h,60t,0r",
              voice: { tr: "Naşa’da fay kenarları doğal termal havuzlar oluşturur. Doğanın jeolojik laboratuvarı.", en: "Fault edges form natural pools. Nature's geological lab." },
              quiz: { 
                q_tr: "Termal havuzlar nerede oluşur?", q_en: "Where do thermal pools form?",
                a_tr: "Fay kenarlarında", a_en: "Along fault edges",
                options_tr: ["Fay kenarlarında", "Volkanik kraterde", "Dağ zirvesinde"],
                options_en: ["Along fault edges", "In volcanic crater", "On mountain summit"]
              }
            },
            { 
              id: "citgol", lat: 39.10, lng: 28.98, order: 5,
              name: { tr: "Çitgöl Kaplıcası", en: "Çitgöl Hot Spring" }, color: "#ff6f00",
              img: "https://via.placeholder.com/800x400?text=Citgol+Hot+Spring", // GÜNCELLENECEK
              overlay: { tr: "Yüksek mineral içerikli sular.", en: "High mineral content waters." },
              desc: { tr: "Çitgöl’de mineral içeriği yüksek sular yüzeye çıkar.", en: "High mineral waters surface at Çitgöl." },
              earth: "https://earth.google.com/web/@39.10,28.98,100a,1000d,35y,0h,0t,0r",
              voice: { tr: "Çitgöl’de mineral içeriği yüksek sular yüzeye çıkar.", en: "High mineral waters rise in Çitgöl." },
              quiz: { 
                q_tr: "Suyun özelliği nedir?", q_en: "What is the property of the water?",
                a_tr: "Mineral içerikli", a_en: "Mineral-rich",
                options_tr: ["Sıcak", "Mineral içerikli", "Tatlı su"],
                options_en: ["Hot", "Mineral-rich", "Freshwater"]
              }
            },
            { 
              id: "sismik", lat: 39.09, lng: 28.97, order: 6,
              name: { tr: "Sismik İstasyon", en: "Seismic Station" }, color: "#2e7d32",
              img: "https://via.placeholder.com/800x400?text=Seismic+Station", // GÜNCELLENECEK
              overlay: { tr: "Fay hareketlerini izler.", en: "Monitors fault movements." },
              desc: { tr: "Sismik istasyon fay hareketlerini 7/24 izler.", en: "Seismic station monitors faults 24/7." },
              earth: "https://earth.google.com/web/@39.09,28.97,100a,1000d,35y,0h,0t,0r",
              voice: { tr: "Sismik istasyon fay hareketlerini sürekli izler.", en: "Seismic station continuously monitors faults." },
              quiz: { 
                q_tr: "Ne izlenir?", q_en: "What is monitored?",
                a_tr: "Mikro-depremler", a_en: "Micro-earthquakes",
                options_tr: ["Mikro-depremler", "Volkanik gazlar", "Yeraltı suyu"],
                options_en: ["Micro-earthquakes", "Volcanic gases", "Groundwater"]
              }
            }
          ]
        },

        // --- YARDIMCI FONKSİYONLAR ---
        cleanText: function(text) {
          if (typeof text !== 'string') return text;
          const txt = document.createElement('textarea');
          txt.innerHTML = text;
          return txt.value.replace(/[\u200B-\u200D\uFEFF\u00A0→]/g, ' ').replace(/\s+/g, ' ').trim();
        },
        t: function(key) { return this.cleanText(this.data[key][this.lang]); },
        tt: function(obj, key) { return this.cleanText(obj[key][this.lang]); },

        // --- HARİTA VE AKIŞ (SENİN KODUNDAN) ---
        initMap: function() {
          if (document.getElementById('map')) {
            if (this.map) this.map.remove();
            this.map = L.map('map').setView([52, 21], 3);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '© OpenStreetMap' }).addTo(this.map);
            this.data.stops.forEach(s => {
              L.marker([s.lat, s.lng]).addTo(this.map)
                .bindPopup(`<b onclick="window.jeotur.renderStop('${s.id}')" style="cursor:pointer;">${this.tt(s, 'name')}</b>`)
                .on('click', () => this.renderStop(s.id));
            });
            this.map.fitBounds(this.data.stops.map(s => [s.lat, s.lng]), { padding: [50, 50] });
          }
        },
        renderFlow: function() {
          const sorted = [...this.data.stops].sort((a,b) => a.order - b.order);
          let html = `<h3 style="color:var(--btn);text-align:center;">${this.lang==='tr'?'Nedensel Zincir':'Causal Chain'}</h3>`;
          sorted.forEach((n, i) => {
            html += `<div class="flow-node flow-node-${i+1}" onclick="window.jeotur.renderStop('${n.id}')">${i+1}. ${this.tt(n, 'name')}</div>`;
            if (i < sorted.length-1) html += `<div class="flow-arrow"><i class="fa-solid fa-arrow-down"></i></div>`;
          });
          return html;
        },

        // --- QUIZ FONKSİYONLARI (SENİN KODUNDAN) ---
        renderQuizOptions: function(s) {
          const savedAnswer = localStorage.getItem("quiz_" + s.id);
          const correctAnswer = (this.lang === 'tr') ? s.quiz.a_tr : s.quiz.a_en;
          const options = (this.lang === 'tr') ? s.quiz.options_tr : s.quiz.options_en;
          return options.map(opt => {
            let className = 'quiz-option';
            let onClick = `window.jeotur.checkAnswer('${s.id}', '${opt}')`;
            if (savedAnswer) {
              const isCorrect = (opt === correctAnswer);
              const isSelected = (opt === savedAnswer);
              className += ' disabled';
              if (isCorrect) className += ' correct';
              if (isSelected && !isCorrect) className += ' selected';
              onClick = '';
            }
            return `<div class="${className}" onclick="${onClick}">${opt}</div>`;
          }).join("");
        },
        checkAnswer: function(id, selected) {
          try {
            localStorage.setItem("quiz_" + id, selected);
          } catch (e) {
            console.error("LocalStorage yazma hatası (İzleme Engellemesi açık olabilir):", e);
          }
          this.renderStop(id); // Kartı yenilemek için (senin kodundaki gibi)
        },
        resetQuiz: function() {
          if (!confirm(this.lang === 'tr' ? 'Tüm quiz yanıtlarınızı silmek istediğinizden emin misiniz?' : 'Are you sure you want to delete all your quiz answers?')) {
            return;
          }
          try {
            this.data.stops.forEach(s => {
              localStorage.removeItem("quiz_" + s.id);
            });
            alert(this.lang === 'tr' ? 'Quiz yanıtları sıfırlandı!' : 'Quiz answers have been reset!');
            this.renderMenu(); // Menüyü yenile
          } catch (e) {
            console.error(e);
          }
        },

        // --- DURAK SAYFASI RENDER (SENİN KODUNDAN) ---
        renderStop: function(id) {
          const s = this.data.stops.find(x => x.id === id);
          if (!s) return;

          const quizQ = (this.lang === 'tr') ? s.quiz.q_tr : s.quiz.q_en;
          const desc = this.tt(s, 'desc');
          const overlay = this.tt(s, 'overlay');

          document.getElementById('main').innerHTML = `
            <div class="card">
              <h2 style="color:${s.color};">${this.tt(s, 'name')}</h2>
              <div class="image-container" onclick="window.jeotur.openLightbox('${s.id}')">
                <img src="${s.img}" alt="${this.tt(s, 'name')}" onerror="this.src='https://via.placeholder.com/800x400?text=${encodeURIComponent(this.tt(s, 'name'))}'">
                <div class="image-overlay">${overlay}</div>
              </div>
              <p>${desc}</p>

              <button class="jeo-btn earth-btn" onclick="window.open('${s.earth}','_blank')">
                <i class="fa-solid fa-globe"></i> ${this.lang==='tr'?'Google Earth':'Open in Earth'}
              </button>

              <button class="jeo-btn voice-btn" onclick="window.jeotur.playVoice('${s.id}')">
                <i class="fa-solid fa-volume-high"></i> ${this.lang==='tr'?'Sesli Anlat':'Audio Guide'}
              </button>
            </div>

            <div class="card" style="background:var(--quiz);">
              <h3 style="color:${s.color};">${this.quizIcons[s.id] || '❓'} ${quizQ}</h3>
              ${this.renderQuizOptions(s)}
            </div>

            <div style="text-align:center;margin:15px 0;">
              <button class="jeo-btn" style="background:var(--btn);color:white;" onclick="window.jeotur.renderMenu()">
                &larr; ${this.lang==='tr'?'Geri Dön':'Back'}
              </button>
            </div>
          `;
          
          // Durak sayfasında haritayı tekrar başlatmıyoruz (senin kodundaki gibi)
          if (this.map) {
             this.map.remove();
             this.map = null;
          }
        },

        // --- ANA MENÜ RENDER (SENİN KODUNDAN) ---
        renderMenu: function() {
          let html = `
            <div class="card" id="map"></div>
            <div class="flow-map">${this.renderFlow()}</div>
          `;

          this.data.stops.forEach(s => {
            html += `
              <div class="card-menu" onclick="window.jeotur.renderStop('${s.id}')">
                <h3>${this.tt(s, 'name')} ${this.quizIcons[s.id] || ''}</h3>
                <p>${this.tt(s,'overlay')}</p>
              </div>
            `;
          });

          html += `
            <button onclick="window.jeotur.resetQuiz()" class="jeo-btn" style="background:#b23; color:white; width:100%; margin-top:10px;">
              <i class="fa-solid fa-trash-can"></i> ${this.lang==='tr'?'Quizleri Sıfırla':'Reset Quizzes'}
            </button>
          `;

          document.getElementById('main').innerHTML = html;
          this.initMap(); // Haritayı menüde başlat
        },

        // --- LIGHTBOX (SENİN KODUNDAN) ---
        openLightbox: function(id) {
          const s = this.data.stops.find(x => x.id === id);
          document.getElementById('lb-title').textContent = this.tt(s,'name');
          document.getElementById('lb-img').src = s.img;
          document.getElementById('lb-desc').textContent = this.tt(s,'desc');
          document.getElementById('lightbox').classList.add('open');
        },
        closeLightbox: function() {
          document.getElementById('lightbox').classList.remove('open');
        },

        // --- SES (BROWSER TTS) (SENİN KODUNDAN) ---
        playVoice: function(id) {
          const s = this.data.stops.find(x => x.id === id);
          const text = s.voice[this.lang];
          if ('speechSynthesis' in window) {
            speechSynthesis.cancel(); // Önceki konuşmaları durdur
            const msg = new SpeechSynthesisUtterance(text);
            msg.lang = (this.lang === 'tr') ? 'tr-TR' : 'en-US';
            msg.rate = 0.9;
            speechSynthesis.speak(msg);
          }
        },

        // --- DİL DEĞİŞTİRME (SENİN KODUNDAN) ---
        toggleLang: function() {
          this.lang = (this.lang === 'tr') ? 'en' : 'tr';
          try { localStorage.setItem('jeotur_lang', this.lang); } catch(e){}
          document.getElementById('lang-btn').textContent = (this.lang==='tr'?'English':'Türkçe');
          document.getElementById('title').textContent = this.t('title');
          this.renderMenu();
        },

        // --- BAŞLANGIÇ (SENİN KODUNDAN) ---
        init: function() {
          try {
            const savedLang = localStorage.getItem('jeotur_lang');
            if (savedLang) this.lang = savedLang;
          } catch(e){
            console.warn("localStorage erişimi engellendi (İzleme Engellemesi?).");
          }
          document.getElementById('lang-btn').textContent = (this.lang==='tr'?'English':'Türkçe');
          document.getElementById('title').textContent = this.t('title');
          this.renderMenu();
          
          // Lightbox kapatma event'lerini ekle
          const lightbox = document.getElementById('lightbox');
          if (lightbox) {
            lightbox.onclick = (e) => {
              if (e.target === lightbox) this.closeLightbox();
            };
          }
          document.addEventListener('keydown', (e) => {
            if (e.key === 'Escape') this.closeLightbox();
          });
        }
      };

      // Uygulamayı Başlat
      document.getElementById('lang-btn').onclick = () => window.jeotur.toggleLang();
      window.addEventListener('load', () => {
        window.jeotur.init();
      });

    })();
    </script>

</body>
</html>
