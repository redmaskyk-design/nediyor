<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>LyricSync Pro - Spotify Lyrics Translator & Overlay</title>
  
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- React & ReactDOM CDN -->
  <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
  
  <!-- Babel Standalone for JSX Compiling -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <!-- Lucide Icons CDN -->
  <script src="https://unpkg.com/lucide@latest"></script>

  <!-- Google Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;700&family=Space+Grotesk:wght@400;500;700&display=swap" rel="stylesheet">

  <style>
    body {
      font-family: 'Inter', sans-serif;
    }
    .font-space {
      font-family: 'Space Grotesk', sans-serif;
    }
    .font-mono {
      font-family: 'JetBrains Mono', monospace;
    }
    
    /* Scrollbar config */
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: rgba(255, 255, 255, 0.02);
    }
    ::-webkit-scrollbar-thumb {
      background: rgba(255, 255, 255, 0.12);
      border-radius: 4px;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: rgba(255, 255, 255, 0.22);
    }

    @keyframes fade-in {
      from { opacity: 0; transform: translateY(8px); }
      to { opacity: 1; transform: translateY(0); }
    }
    .animate-fade-in {
      animation: fade-in 0.3s ease-out forwards;
    }

    input[type="range"] {
      -webkit-appearance: none;
      appearance: none;
    }
    input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      appearance: none;
      width: 12px;
      height: 12px;
      border-radius: 50%;
      background: #1ed760;
      cursor: pointer;
    }
    input[type="range"]::-moz-range-thumb {
      width: 12px;
      height: 12px;
      border-radius: 50%;
      background: #1ed760;
      cursor: pointer;
      border: none;
    }
  </style>

  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            spotify: {
              DEFAULT: '#1db954',
              bright: '#1ed760',
              dark: '#1aa34a'
            }
          }
        }
      }
    }
  </script>
</head>
<body class="bg-[#09090b] text-gray-200 min-h-screen flex flex-col antialiased selection:bg-[#1db954]/20 selection:text-white">

  <div id="root"></div>

  <!-- React JSX App -->
  <script type="text/babel">
    const { useState, useEffect, useRef } = React;

    // Helper: Parse MM:SS.xx to seconds
    function parseLrcTimeToSeconds(timeStr) {
      const parts = timeStr.split(":");
      if (parts.length !== 2) return 0;
      const minutes = parseInt(parts[0], 10);
      const seconds = parseFloat(parts[1]);
      return (minutes * 60) + seconds;
    }

    const PRESET_SONGS = [
      {
        id: "lose-yourself",
        title: "Lose Yourself",
        artist: "Eminem",
        album: "8 Mile",
        albumArt: "https://images.unsplash.com/photo-1514525253161-7a46d19cd819?w=300&auto=format&fit=crop&q=60",
        duration: 320,
        lyrics: [
          { time: "00:02.00", text: "Look, if you had one shot or one opportunity...", translation: "Bak, eğer tek bir kurşunun ya da tek bir fırsatın olsaydı..." },
          { time: "00:06.00", text: "To seize everything you ever wanted in one moment...", translation: "Hayatın boyunca istediğin her şeyi tek bir anda avuçlarına almak için..." },
          { time: "00:10.50", text: "Would you capture it, or just let it slip?", translation: "Üstüne mi çökerdin yoksa ellerinden akıp gitmesine izin mi verirdin?" },
          { time: "00:14.00", text: "His palms are sweaty, knees weak, arms are heavy", translation: "Avuçları terliyor, dizlerinin bağı çözülmüş, kolları kurşun gibi ağır" },
          { time: "00:18.00", text: "There's vomit on his sweater already, mom's spaghetti", translation: "Kazağına kusmuş bile, anasının yaptığı o meşhur spagetti" },
          { time: "00:21.50", text: "He's nervous, but on the surface he looks calm and ready to drop bombs", translation: "Tırsıyor ama dışarıdan bakınca taş gibi sakin, sahneyi patlatmaya hazır" },
          { time: "00:25.80", text: "But he keeps on forgetting what he wrote down", translation: "Ama karaladığı tüm o satırları unutup duruyor" },
          { time: "00:28.50", text: "The whole crowd goes so loud", translation: "Tüm tayfa bağırmaya başlıyor, kıyamet kopuyor" },
          { time: "00:31.00", text: "He opens his mouth, but the words won't come out", translation: "Ağzını açıyor ama tık yok, sözler boğazında düğümleniyor" },
          { time: "00:34.00", text: "He's choking, how? Everybody's joking now", translation: "Nefesi kesildi, nasıl olur? Şimdi herkes onunla kafa buluyor" },
          { time: "00:37.00", text: "The clock's run out, time's up, over, blaow!", translation: "Zaman daraldı, süre doldu, bitti her şey, güm!" },
          { time: "00:40.00", text: "Snap back to reality, oh, there goes gravity", translation: "Silkelen ve gerçeğe dön, yer çekimi falan yalan oldu" },
          { time: "00:43.00", text: "Oh, there goes Rabbit, he choked, he's so mad", translation: "İşte bizim Rabbit darmadağın oldu, boğuldu hırstan kuduruyor" },
          { time: "00:46.00", text: "But he won't give up that easy, no, he won't have it", translation: "Ama hemen pes etmez öyle kolayına, yediremez kendine" }
        ].map(item => ({ ...item, seconds: parseLrcTimeToSeconds(item.time) }))
      },
      {
        id: "let-me-down-slowly",
        title: "Let Me Down Slowly",
        artist: "Alec Benjamin",
        album: "Narrated for You",
        albumArt: "https://images.unsplash.com/photo-1511671782779-c97d3d27a1d4?w=300&auto=format&fit=crop&q=60",
        duration: 169,
        lyrics: [
          { time: "00:02.00", text: "This night is cold in the kingdom", translation: "Bu krallıkta gece buz kesiyor" },
          { time: "00:06.00", text: "I can feel you fade away", translation: "Yavaşça yok olduğunu hissedebiliyorum" },
          { time: "00:09.50", text: "From the kitchen to the bathroom sink and...", translation: "Mutfaktan banyo lavabosuna kadar her yerde..." },
          { time: "00:13.50", text: "Your steps talk about your departure", translation: "Adımların bana gidişini fısıldıyor" },
          { time: "00:17.50", text: "Don't send me into the dark", translation: "Beni böyle zifiri karanlığa mahkum etme" },
          { time: "00:21.00", text: "If you're leaving, baby, let me down slowly", translation: "Gideceksen bile sevgilim, beni yavaşça terk et" },
          { time: "00:25.00", text: "A little sympathy, I hope you can show me", translation: "Umarım bana birazcık olsun merhamet gösterirsin" }
        ].map(item => ({ ...item, seconds: parseLrcTimeToSeconds(item.time) }))
      },
      {
        id: "blinding-lights",
        title: "Blinding Lights",
        artist: "The Weeknd",
        album: "After Hours",
        albumArt: "https://images.unsplash.com/photo-1470225620780-dba8ba36b745?w=300&auto=format&fit=crop&q=60",
        duration: 200,
        lyrics: [
          { time: "00:06.00", text: "Yeah, I've been on my own for long enough", translation: "Evet, uzun zamandır tek başımayım, canıma tak etti artık" },
          { time: "00:13.00", text: "Maybe you can show me how to love, maybe", translation: "Belki de sevmeyi bana sen öğretirsin, ne dersin?" },
          { time: "00:19.00", text: "I'm going through withdrawals", translation: "Yoksunluk krizlerine giriyorum sensiz" },
          { time: "00:23.00", text: "You don't even have to do too much", translation: "Gözüm yükseklerde değil, çok bir şey yapmana gerek yok" },
          { time: "00:26.50", text: "You can turn me on with just a touch, baby", translation: "Küçük bir dokunuşun bile beni benden almaya yeter, bebeğim" }
        ].map(item => ({ ...item, seconds: parseLrcTimeToSeconds(item.time) }))
      }
    ];

    const CODE_TEMPLATES = {
      overlay: {
        title: "Android Overlay Window Service (Kotlin)",
        desc: "Android'de diğer uygulamaların üzerinde (Draw Over Other Apps) yüzen bir pencere (Floating Widget) açmak için gereken SYSTEM_ALERT_WINDOW izni ve WindowManager servis uygulaması.",
        file: "FloatingOverlayService.kt",
        code: `package com.lyrily.overlay\n\nimport android.app.Service\nimport android.content.Context\nimport android.content.Intent\nimport android.graphics.PixelFormat\nimport android.view.Gravity\nimport android.view.LayoutInflater\nimport android.view.View\nimport android.view.WindowManager\nimport android.widget.TextView\nimport com.lyrily.R\n\nclass FloatingOverlayService : Service() {\n    private lateinit var windowManager: WindowManager\n    private lateinit var overlayView: View\n    private lateinit var params: WindowManager.LayoutParams\n\n    override fun onCreate() {\n        super.onCreate()\n        windowManager = getSystemService(Context.WINDOW_SERVICE) as WindowManager\n        overlayView = LayoutInflater.from(this).inflate(R.layout.overlay_layout, null)\n\n        params = WindowManager.LayoutParams(\n            WindowManager.LayoutParams.WRAP_CONTENT,\n            WindowManager.LayoutParams.WRAP_CONTENT,\n            WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY,\n            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE,\n            PixelFormat.TRANSLUCENT\n        ).apply {\n            gravity = Gravity.TOP or Gravity.START\n            x = 100\n            y = 200\n        }\n        windowManager.addView(overlayView, params)\n    }\n}`
      },
      spotify_broadcast: {
        title: "Spotify Broadcast Receiver (Kotlin)",
        desc: "Spotify uygulamasının Android işletim sistemine yaydığı (Broadcast) müzik değişim, çalma durumu ve süre ilerlemesi olaylarını (Broadcast Receiver) yakalayarak şarkıyı tespit etme yöntemi.",
        file: "SpotifyReceiver.kt",
        code: `package com.lyrily.receiver\n\nimport android.content.BroadcastReceiver\nimport android.content.Context\nimport android.content.Intent\nimport android.util.Log\n\nclass SpotifyReceiver : BroadcastReceiver() {\n    override fun onReceive(context: Context, intent: Intent) {\n        val action = intent.action ?: return\n        if (action == "com.spotify.music.metadatachanged") {\n            val trackName = intent.getStringExtra("track")\n            val artistName = intent.getStringExtra("artist")\n            Log.d("SpotifyReceiver", "Track: $trackName by $artistName")\n        }\n    }\n}`
      },
      react_native: {
        title: "React Native Integration (TS/Native Bridge)",
        desc: "React Native projesinde, Kotlin ile yazdığımız Floating Service'i başlatmak, kapatmak ve o an çalan İngilizce & Türkçe sözleri yerel köprü (Native Modules) üzerinden beslemek için gereken JavaScript API'si.",
        file: "SpotifyOverlayModule.ts",
        code: `import { NativeModules, Platform } from 'react-native';\nconst { SpotifyOverlayModule } = NativeModules;\n\nexport function startOverlayWidget(): void {\n  if (Platform.OS === 'android') {\n    SpotifyOverlayModule.startService();\n  }\n}\n\nexport function updateOverlayLyrics(english: string, turkish: string): void {\n  if (Platform.OS === 'android') {\n    SpotifyOverlayModule.updateLyrics(english, turkish);\n  }\n}`
      },
      flutter: {
        title: "Flutter Integration (MethodChannel & Package)",
        desc: "Flutter projesinde 'system_alert_window' paketini kullanarak veya özel MethodChannel köprüleri ile her zaman üstte kalan Android overlay penceresini tasarlama ve yönetme kodu.",
        file: "spotify_overlay_channel.dart",
        code: `import 'package:flutter/services.dart';\nimport 'package:system_alert_window/system_alert_window.dart';\n\nclass SpotifyOverlayController {\n  static Future<void> updateLyrics(String english, String turkish) async {\n    SystemWindowBody body = SystemWindowBody(\n      rows: [\n        EachRow(columns: [EachColumn(text: SystemWindowText(text: english, fontSize: 16))]),\n        EachRow(columns: [EachColumn(text: SystemWindowText(text: turkish, fontSize: 14))])\n      ]\n    );\n    await SystemAlertWindow.showSystemWindow(body: body);\n  }\n}`
      }
    };

    function App() {
      // Simulator and OS Views
      const [currentView, setCurrentView] = useState("lyrily");
      const [currentTime, setCurrentTime] = useState("13:40");
      
      // Library & Active Track states
      const [songs, setSongs] = useState(PRESET_SONGS);
      const [activeSong, setActiveSong] = useState(PRESET_SONGS[0]);
      
      // Spotify player state emulation
      const [isPlaying, setIsPlaying] = useState(false);
      const [progress, setProgress] = useState(0);

      // Overlay Configuration
      const [overlay, setOverlay] = useState({
        enabled: true,
        opacity: 0.85,
        textSize: "lg",
        theme: "glass",
        translationStyle: "Sokak Türkçesi",
        x: 20,
        y: 110,
        width: 260
      });

      // Drag overlay reference states
      const [isDragging, setIsDragging] = useState(false);
      const dragStartRef = useRef({ x: 0, y: 0 });
      const screenRef = useRef(null);

      // AI Generator States
      const [searchSong, setSearchSong] = useState("");
      const [searchArtist, setSearchArtist] = useState("");
      const [isGenerating, setIsGenerating] = useState(false);
      const [genSuccess, setGenSuccess] = useState(false);
      const [genError, setGenError] = useState("");
      const [isPremium, setIsPremium] = useState(false);
      const [credits, setCredits] = useState(3);
      const [showPaywall, setShowPaywall] = useState(false);
      const [cacheInfo, setCacheInfo] = useState(null);

      // Developer Tab selection
      const [activeDevTab, setActiveDevTab] = useState("overlay");
      const [copied, setCopied] = useState(false);

      // Hydrate Lucide Icons on view updates
      useEffect(() => {
        if (window.lucide) {
          window.lucide.createIcons();
        }
      });

      // System Ticker
      useEffect(() => {
        const updateTime = () => {
          const d = new Date();
          setCurrentTime(`${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}`);
        };
        updateTime();
        const interval = setInterval(updateTime, 60000);
        return () => clearInterval(interval);
      }, []);

      // Spotify audio progress timer emulation
      useEffect(() => {
        let timer;
        if (isPlaying) {
          timer = setInterval(() => {
            setProgress(prev => {
              if (prev >= activeSong.duration) {
                return 0; // Loop song
              }
              return prev + 0.5;
            });
          }, 500);
        }
        return () => clearInterval(timer);
      }, [isPlaying, activeSong]);

      // Handle dragging calculations
      const handleDragStart = (e) => {
        setIsDragging(true);
        const clientX = "touches" in e ? e.touches[0].clientX : e.clientX;
        const clientY = "touches" in e ? e.touches[0].clientY : e.clientY;
        dragStartRef.current = {
          x: clientX - overlay.x,
          y: clientY - overlay.y
        };
        e.preventDefault();
      };

      useEffect(() => {
        const handleMove = (e) => {
          if (!isDragging || !screenRef.current) return;
          const clientX = "touches" in e ? e.touches[0].clientX : e.clientX;
          const clientY = "touches" in e ? e.touches[0].clientY : e.clientY;
          let newX = clientX - dragStartRef.current.x;
          let newY = clientY - dragStartRef.current.y;
          
          const maxW = screenRef.current.clientWidth || 300;
          const maxH = screenRef.current.clientHeight || 600;

          newX = Math.max(8, Math.min(maxW - overlay.width - 8, newX));
          newY = Math.max(30, Math.min(maxH - 120, newY));

          setOverlay(prev => ({ ...prev, x: newX, y: newY }));
        };

        const handleEnd = () => setIsDragging(false);

        if (isDragging) {
          window.addEventListener("mousemove", handleMove);
          window.addEventListener("mouseup", handleEnd);
          window.addEventListener("touchmove", handleMove);
          window.addEventListener("touchend", handleEnd);
        }
        return () => {
          window.removeEventListener("mousemove", handleMove);
          window.removeEventListener("mouseup", handleEnd);
          window.removeEventListener("touchmove", handleMove);
          window.removeEventListener("touchend", handleEnd);
        };
      }, [isDragging]);

      // Locate active lyrics block indexes
      const getActiveLyric = () => {
        if (!activeSong.lyrics || activeSong.lyrics.length === 0) return null;
        let active = activeSong.lyrics[0];
        for (let i = 0; i < activeSong.lyrics.length; i++) {
          if (progress >= activeSong.lyrics[i].seconds) {
            active = activeSong.lyrics[i];
          } else {
            break;
          }
        }
        return active;
      };

      const activeLyric = getActiveLyric();
      const activeIdx = activeSong.lyrics.findIndex(l => l.text === activeLyric?.text);
      const upcomingLyric = activeIdx !== -1 && activeIdx < activeSong.lyrics.length - 1 ? activeSong.lyrics[activeIdx + 1] : null;

      // Real or Standalone Dynamic Fallback Lyric API call
      const handleGenerateLyrics = async () => {
        if (!searchSong.trim() || !searchArtist.trim()) {
          setGenError("Lütfen hem şarkı adını hem de sanatçı adını girin.");
          return;
        }
        if (!isPremium && credits <= 0) {
          setShowPaywall(true);
          return;
        }

        setIsGenerating(true);
        setGenError("");
        setGenSuccess(false);

        try {
          const response = await fetch("/api/generate-lrc", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
              title: searchSong.trim(),
              artist: searchArtist.trim(),
              style: overlay.translationStyle
            })
          });

          if (!response.ok) throw new Error("Server not reachable");
          const data = await response.json();

          if (!data.lyrics || data.lyrics.length === 0) {
            throw new Error("Boş lirik verisi döndü");
          }

          setCacheInfo({
            status: data.cacheStatus || "MISS",
            latency: data.latencyMs || 1800,
            savedCost: data.savedCost || "$0.0025",
            source: data.databaseSource || "Gemini API -> Cloud Firestore"
          });

          const formatted = data.lyrics.map(l => ({
            ...l,
            seconds: parseLrcTimeToSeconds(l.time)
          })).sort((a,b) => a.seconds - b.seconds);

          addNewSong(searchSong.trim(), searchArtist.trim(), formatted);
        } catch (e) {
          // Robust client-side standalone fallback generator (simulates Gemini locally)
          console.log("Running offline dynamic local fallback engine...", e);
          
          setTimeout(() => {
            const simulatedLyrics = [
              { time: "00:02.00", text: `[Playing beautiful "${searchSong.trim()}" by ${searchArtist.trim()}]`, translation: `[${searchArtist.trim()} - "${searchSong.trim()}" keyifle çalıyor]` },
              { time: "00:08.00", text: "Yeah, I hear the music playing in the deep night sky", translation: "Evet, gece gökyüzünün derinliklerinde çalan melodiyi duyuyorum" },
              { time: "00:15.00", text: "We were looking for a sign, searching for reasons why", translation: "Bir işaret arıyorduk, tüm bu olanların sebebini sorguluyorduk" },
              { time: "00:23.00", text: "But you hold my hand, and everything starts to shine", translation: "Ama ellerimi sıkıca tutuyorsun ve her yer ışıl ışıl parlamaya başlıyor" },
              { time: "00:31.00", text: `This is a beautiful custom adaptions: ${overlay.translationStyle}`, translation: `Bu şarkı "${overlay.translationStyle}" tarzına göre uyarlandı` },
              { time: "00:39.00", text: "No more doubts, no more fears, because we are synchronized", translation: "Artık ne bir şüphe ne bir korku kaldı, çünkü tamamen eş zamanlıyız" },
              { time: "00:48.00", text: "[Instrumental solo fading away gracefully]", translation: "[Enstrümantal melodi yavaşça kayboluyor]" }
            ].map(l => ({ ...l, seconds: parseLrcTimeToSeconds(l.time) }));

            setCacheInfo({
              status: "OFFLINE_DYN",
              latency: 450,
              savedCost: "$0.0000 (Yerel Çevrimdışı Simülatör)",
              source: "Local Standalone Fallback Generator"
            });

            addNewSong(searchSong.trim(), searchArtist.trim(), simulatedLyrics);
          }, 1000);
        }
      };

      const addNewSong = (title, artist, lyrics) => {
        if (!isPremium) setCredits(p => Math.max(0, p - 1));
        const newSong = {
          id: `dyn-${Date.now()}`,
          title,
          artist,
          album: "AI Generated",
          albumArt: "https://images.unsplash.com/photo-1618005182384-a83a8bd57fbe?w=300&auto=format&fit=crop&q=60",
          duration: lyrics[lyrics.length - 1].seconds + 10,
          lyrics
        };

        setSongs(prev => [newSong, ...prev]);
        setActiveSong(newSong);
        setProgress(0);
        setIsPlaying(true);
        setGenSuccess(true);
        setSearchSong("");
        setSearchArtist("");
        setCurrentView("spotify");
        setIsGenerating(false);
        setTimeout(() => setGenSuccess(false), 3000);
      };

      const copyToClipboard = (text) => {
        navigator.clipboard.writeText(text);
        setCopied(true);
        setTimeout(() => setCopied(false), 2000);
      };

      return (
        <div className="flex flex-col min-h-screen">
          {/* Header */}
          <header className="border-b border-white/5 bg-[#0e0e11]/90 backdrop-blur-md px-6 py-4 flex items-center justify-between sticky top-0 z-40">
            <div className="flex items-center gap-3">
              <div className="w-10 h-10 rounded-xl bg-gradient-to-tr from-[#1db954] to-[#1ed760] flex items-center justify-center shadow-[0_0_15px_rgba(29,185,84,0.3)]">
                <i data-lucide="layers" class="w-5.5 h-5.5 text-black"></i>
              </div>
              <div>
                <div className="flex items-center gap-2">
                  <span className="font-bold text-lg text-white font-space tracking-tight">LyricSync <span className="text-[#1db954] text-xs font-mono uppercase ml-1">Pro</span></span>
                  <span className="bg-[#1db954]/10 border border-[#1db954]/20 text-[#1db954] text-[10px] font-mono px-2 py-0.5 rounded-full font-bold">
                    STANDALONE ACTIVE
                  </span>
                </div>
                <p className="text-gray-400 text-xs mt-0.5">Spotify Eş Zamanlı İngilizce Şarkı Çevirmeni & Overlay Servisi</p>
              </div>
            </div>

            <div className="hidden md:flex items-center gap-6 text-xs text-gray-400 font-medium">
              <span className="hover:text-white transition-colors cursor-pointer">Rehber</span>
              <span className="hover:text-white transition-colors cursor-pointer">Özellikler</span>
              <span className="hover:text-white transition-colors cursor-pointer">API Sürümleri</span>
              <a href="#developer" className="bg-[#18181b] border border-white/10 text-white px-4 py-2 rounded-xl hover:bg-zinc-800 transition-all font-semibold">
                Mobil Entegrasyon Kodları
              </a>
            </div>
          </header>

          {/* Main Workspace Layout */}
          <main className="flex-1 max-w-7xl w-full mx-auto p-4 lg:p-8 grid grid-cols-1 lg:grid-cols-12 gap-8 items-start">
            
            {/* Left Col: Interactive Virtual Phone Simulator Frame */}
            <div className="lg:col-span-5 flex flex-col items-center gap-4">
              <div className="text-center max-w-sm mb-2">
                <span className="text-[10px] font-bold text-[#1db954] bg-[#1db954]/5 border border-[#1db954]/15 px-3 py-1 rounded-full uppercase tracking-wider">
                  Canlı Simülatör
                </span>
                <h2 className="text-xl font-bold text-white mt-3 font-space tracking-tight">İnteraktif Telefon Ekranı</h2>
                <p className="text-gray-400 text-xs mt-1.5 leading-relaxed">
                  Yüzen pencereyi (Overlay) etkinleştirin, çalma kontrollerini test edin veya Gemini ile kütüphaneyi genişletin!
                </p>
              </div>

              {/* Phone Device Frame */}
              <div className="relative w-[340px] h-[670px] rounded-[48px] border-[10px] border-[#1f1f1f] bg-black shadow-[0_25px_50px_-12px_rgba(0,0,0,0.8)] overflow-hidden ring-4 ring-[#1db954]/15">
                
                {/* Camera Notch */}
                <div className="absolute top-2 left-1/2 -translate-x-1/2 w-28 h-6 bg-black rounded-full z-50 flex items-center justify-between px-3">
                  <div className="w-2.5 h-2.5 bg-[#08080f] rounded-full border border-gray-900"></div>
                  <div className="w-10 h-1 bg-[#101015] rounded-full"></div>
                  <div className="w-1.5 h-1.5 bg-[#151525] rounded-full"></div>
                </div>

                {/* Simulated Screen */}
                <div ref={screenRef} className="relative w-full h-full bg-[#08080c] flex flex-col justify-between text-white overflow-hidden select-none">
                  
                  {/* StatusBar */}
                  <div className="h-10 pt-4 px-6 flex justify-between items-center text-xs font-semibold tracking-tight text-white/90 z-45 bg-gradient-to-b from-black/60 to-transparent">
                    <span>{currentTime}</span>
                    <div className="flex items-center gap-1.5">
                      <i data-lucide="wifi" class="w-3 h-3"></i>
                      <span className="text-[9px] bg-white/10 px-1 py-0.5 rounded font-mono">LTE</span>
                      <div className="flex items-center gap-1 bg-white/10 px-1 py-0.5 rounded text-[9px]">
                        <i data-lucide="battery" class="w-3 h-3 text-emerald-400"></i>
                        <span>84%</span>
                      </div>
                    </div>
                  </div>

                  {/* Dynamic Views inside simulator */}
                  <div className="flex-1 overflow-y-auto px-4 py-2 mt-1 mb-12 relative flex flex-col justify-start">
                    
                    {/* View: Home */}
                    {currentView === "home" && (
                      <div className="flex-1 flex flex-col justify-between pt-6 animate-fade-in">
                        <div className="text-center py-4">
                          <h1 className="text-4xl font-extralight tracking-wide text-white/90">{currentTime}</h1>
                          <p className="text-[10px] text-gray-400 mt-1 font-medium tracking-widest uppercase">TEMMUZ ÇEVRİMDIŞI</p>
                        </div>

                        {/* Grid of Apps */}
                        <div className="grid grid-cols-4 gap-4 px-2 mb-16">
                          <button onClick={() => setCurrentView("lyrily")} className="flex flex-col items-center gap-1.5 group">
                            <div className="w-12 h-12 bg-gradient-to-br from-zinc-800 to-zinc-950 border border-white/5 rounded-2xl flex items-center justify-center shadow-lg active:scale-95 transition-transform">
                              <i data-lucide="sliders" class="w-5 h-5 text-[#1db954]"></i>
                            </div>
                            <span className="text-[9px] text-gray-300 font-medium truncate">Lyrily</span>
                          </button>

                          <button onClick={() => setCurrentView("spotify")} className="flex flex-col items-center gap-1.5 group">
                            <div className="w-12 h-12 bg-gradient-to-br from-[#1ed760] to-[#1aa34a] rounded-2xl flex items-center justify-center shadow-lg active:scale-95 transition-transform">
                              <i data-lucide="music" class="w-5 h-5 text-white"></i>
                            </div>
                            <span className="text-[9px] text-gray-300 font-medium truncate">Spotify</span>
                          </button>

                          <button onClick={() => setCurrentView("instagram")} className="flex flex-col items-center gap-1.5 group">
                            <div className="w-12 h-12 bg-gradient-to-tr from-[#f9ce34] via-[#ee2a7b] to-[#6228d7] rounded-2xl flex items-center justify-center shadow-lg active:scale-95 transition-transform">
                              <i data-lucide="compass" class="w-5 h-5 text-white"></i>
                            </div>
                            <span className="text-[9px] text-gray-300 font-medium truncate">Instagram</span>
                          </button>

                          <button className="flex flex-col items-center gap-1.5 opacity-40">
                            <div className="w-12 h-12 bg-zinc-800 rounded-2xl flex items-center justify-center">
                              <i data-lucide="settings" class="w-5 h-5 text-gray-400"></i>
                            </div>
                            <span className="text-[9px] text-gray-400 truncate">Ayarlar</span>
                          </button>
                        </div>
                      </div>
                    )}

                    {/* View: Lyrily Dashboard */}
                    {currentView === "lyrily" && (
                      <div className="flex-1 flex flex-col gap-4 pt-1 animate-fade-in">
                        <div className="flex items-center justify-between border-b border-white/5 pb-2.5">
                          <div className="flex items-center gap-2">
                            <div className="p-1.5 bg-[#1db954]/10 border border-[#1db954]/20 rounded-lg">
                              <i data-lucide="sliders" class="w-3.5 h-3.5 text-[#1db954]"></i>
                            </div>
                            <span className="font-semibold text-xs tracking-tight text-white">Lyrily Kontrol Paneli</span>
                          </div>
                          <button onClick={() => setCurrentView("home")} className="p-1 hover:bg-white/5 rounded-full text-gray-400">
                            <i data-lucide="x" class="w-3.5 h-3.5"></i>
                          </button>
                        </div>

                        {/* Current Track Banner */}
                        <div className="bg-white/5 border border-white/10 p-2.5 rounded-xl flex items-center gap-2.5">
                          <img src={activeSong.albumArt} alt={activeSong.title} className="w-9 h-9 rounded-lg object-cover" />
                          <div className="flex-1 min-w-0">
                            <p className="text-[9px] text-gray-500 uppercase">Çalan Parça</p>
                            <h4 className="text-xs text-white font-semibold truncate mt-0.5">{activeSong.title}</h4>
                          </div>
                          <button onClick={() => setCurrentView("spotify")} className="bg-[#1ed760] hover:bg-[#1db954] text-black text-[9px] font-bold px-2.5 py-1.5 rounded-full active:scale-95 transition-transform shrink-0">
                            Oynat
                          </button>
                        </div>

                        {/* Interactive Overlay Switch */}
                        <div className="bg-white/5 border border-white/10 p-3 rounded-xl flex flex-col gap-2.5">
                          <div className="flex items-center justify-between">
                            <div>
                              <h4 className="text-xs text-white font-semibold">Yüzen Pencereyi Aç</h4>
                              <p className="text-[9px] text-gray-400 mt-0.5">Ekran üzerinde yüzen karaoke kutusu</p>
                            </div>
                            <button 
                              onClick={() => setOverlay(prev => ({ ...prev, enabled: !prev.enabled }))}
                              className={`w-9 h-5 rounded-full p-0.5 transition-colors relative flex items-center ${overlay.enabled ? "bg-[#1db954]" : "bg-gray-800"}`}
                            >
                              <div className={`w-4 h-4 rounded-full bg-white shadow transform transition-transform ${overlay.enabled ? "translate-x-4" : "translate-x-0"}`} />
                            </button>
                          </div>

                          {overlay.enabled && (
                            <div className="border-t border-white/5 pt-2.5 space-y-2.5">
                              <div>
                                <div className="flex justify-between text-[9px] text-gray-400">
                                  <span>Arka Plan Şeffaflığı</span>
                                  <span className="font-mono text-[#1db954]">{Math.round(overlay.opacity * 100)}%</span>
                                </div>
                                <input 
                                  type="range" min="0.3" max="1.0" step="0.05" value={overlay.opacity}
                                  onChange={(e) => setOverlay(prev => ({ ...prev, opacity: parseFloat(e.target.value) }))}
                                  className="w-full h-1 bg-zinc-900 border border-white/5 rounded-lg appearance-none cursor-pointer mt-1 accent-[#1db954]"
                                />
                              </div>

                              <div className="flex items-center justify-between">
                                <span className="text-[9px] text-gray-400">Yazı Boyutu</span>
                                <div className="flex gap-1">
                                  {["sm", "base", "lg", "xl"].map(sz => (
                                    <button 
                                      key={sz} onClick={() => setOverlay(prev => ({ ...prev, textSize: sz }))}
                                      className={`text-[8px] uppercase font-bold px-1.5 py-0.5 rounded ${overlay.textSize === sz ? "bg-[#1db954]/20 text-[#1db954]" : "bg-[#18181b] text-gray-400"}`}
                                    >
                                      {sz}
                                    </button>
                                  ))}
                                </div>
                              </div>

                              <div className="grid grid-cols-3 gap-1 mt-1">
                                {["glass", "dark", "neon-green"].map(thm => (
                                  <button
                                    key={thm} onClick={() => setOverlay(prev => ({ ...prev, theme: thm }))}
                                    className={`text-[8px] py-1 rounded border capitalize ${overlay.theme === thm ? "bg-[#1db954]/10 text-[#1db954] border-[#1db954]" : "bg-[#09090b] text-gray-400 border-white/5"}`}
                                  >
                                    {thm === "glass" ? "Cam" : thm === "dark" ? "Siyah" : "Neon"}
                                  </button>
                                ))}
                              </div>
                            </div>
                          )}
                        </div>

                        {/* Translation Tone style selector */}
                        <div className="bg-white/5 border border-white/10 p-3 rounded-xl flex flex-col gap-2">
                          <h4 className="text-xs text-white font-semibold flex items-center gap-1.5">
                            <i data-lucide="sparkles" class="w-3.5 h-3.5 text-[#1db954]"></i>
                            Gemini Çeviri Üslubu
                          </h4>
                          <div className="grid grid-cols-2 gap-1.5">
                            {[
                              { id: "Sokak Türkçesi", label: "Sokak Türkçesi", lock: false },
                              { id: "Dramatik & Şiirsel", label: "Dramatik & Şiirsel", lock: false },
                              { id: "Almanca Adaptasyon", label: "Almanca (Premium)", lock: !isPremium },
                              { id: "İspanyolca Adaptasyon", label: "İspanyolca (Premium)", lock: !isPremium }
                            ].map(item => (
                              <button
                                key={item.id}
                                onClick={() => {
                                  if (item.lock) {
                                    setShowPaywall(true);
                                  } else {
                                    setOverlay(prev => ({ ...prev, translationStyle: item.id }));
                                  }
                                }}
                                className={`text-[9px] py-1.5 px-2 rounded-lg text-left border flex items-center justify-between truncate ${overlay.translationStyle === item.id ? "bg-[#1db954]/10 border-[#1db954]/40 text-white" : "bg-[#09090b] border-white/5 text-gray-400"}`}
                              >
                                <span className="truncate">{item.label}</span>
                                {overlay.translationStyle === item.id && <i data-lucide="check" class="w-3 h-3 text-[#1db954]"></i>}
                              </button>
                            ))}
                          </div>
                        </div>

                        {/* AI Generate Segment */}
                        <div className="bg-white/5 border border-white/10 p-3 rounded-xl flex flex-col gap-2">
                          <div className="flex justify-between items-start">
                            <div>
                              <h4 className="text-xs text-white font-semibold flex items-center gap-1">
                                <i data-lucide="sparkles" class="w-3.5 h-3.5 text-[#1db954]"></i>
                                Gemini ile Şarkı Çek
                              </h4>
                              <p className="text-[8px] text-gray-400 mt-0.5">Türkçe çeviri ve milisaniyeli senkron</p>
                            </div>
                            <button onClick={() => setShowPaywall(true)} className="text-[8px] px-1.5 py-0.5 rounded bg-[#1db954]/10 border border-[#1db954]/20 text-[#1db954] font-bold">
                              {isPremium ? "PRO" : `KREDİ: ${credits}/3`}
                            </button>
                          </div>

                          <div className="flex flex-col gap-1.5">
                            <input 
                              type="text" placeholder="Şarkı Adı (örn: Let It Be)" value={searchSong}
                              onChange={(e) => setSearchSong(e.target.value)}
                              className="w-full bg-[#09090b] border border-white/5 rounded-lg px-2.5 py-1.5 text-xs text-white outline-none"
                            />
                            <input 
                              type="text" placeholder="Sanatçı Adı (örn: The Beatles)" value={searchArtist}
                              onChange={(e) => setSearchArtist(e.target.value)}
                              className="w-full bg-[#09090b] border border-white/5 rounded-lg px-2.5 py-1.5 text-xs text-white outline-none"
                            />
                          </div>

                          {genError && <p className="text-[9px] text-red-400 bg-red-950/20 p-1.5 rounded">{genError}</p>}
                          {genSuccess && <p className="text-[9px] text-emerald-400 bg-emerald-950/20 p-1.5 rounded">Başarıyla eklendi!</p>}

                          <button 
                            onClick={handleGenerateLyrics} disabled={isGenerating}
                            className="w-full bg-[#1db954] text-black font-semibold text-xs py-1.5 rounded-lg flex items-center justify-center gap-1 active:scale-95 transition-transform disabled:opacity-50"
                          >
                            {isGenerating ? "Gemini Analiz Ediyor..." : "AI ile Sözleri Üret"}
                          </button>

                          {cacheInfo && (
                            <div className="bg-zinc-950/80 border border-emerald-500/15 rounded-lg p-2 font-mono text-[8px] text-gray-500 space-y-1 mt-1">
                              <div className="flex justify-between text-white font-bold">
                                <span>KAYNAK ANALİZİ</span>
                                <span className="text-emerald-400">{cacheInfo.status}</span>
                              </div>
                              <div className="flex justify-between"><span>Veritabanı:</span><span className="text-gray-300">{cacheInfo.source}</span></div>
                              <div className="flex justify-between"><span>Gecikme:</span><span className="text-gray-300">{cacheInfo.latency}ms</span></div>
                              <div className="flex justify-between"><span>Maliyet:</span><span className="text-emerald-400 font-semibold">{cacheInfo.savedCost}</span></div>
                            </div>
                          )}

                          {/* Quick presets */}
                          <div className="flex gap-1 border-t border-white/5 pt-2 mt-1">
                            <button onClick={() => { setSearchSong("Bohemian Rhapsody"); setSearchArtist("Queen"); }} className="text-[8px] bg-zinc-900 text-gray-300 px-1.5 py-0.5 rounded">Bohemian</button>
                            <button onClick={() => { setSearchSong("Let It Be"); setSearchArtist("The Beatles"); }} className="text-[8px] bg-zinc-900 text-gray-300 px-1.5 py-0.5 rounded">Let It Be</button>
                          </div>
                        </div>

                        {/* Copyright Note */}
                        <div className="bg-zinc-950/40 p-2.5 rounded-lg flex gap-1.5 text-gray-400 border border-white/5">
                          <i data-lucide="info" class="w-3.5 h-3.5 text-[#1db954] shrink-0 mt-0.5"></i>
                          <p className="text-[8px] leading-relaxed">
                            <span className="font-semibold text-gray-200">🛡️ Copyright Koruma Filtresi:</span> Parça lisans haklarını korumak için, sözler o an çalınan saniyeye uygun 1-2 dize şeklinde "kültürel adaptasyon" konseptiyle anlık gösterilir.
                          </p>
                        </div>
                      </div>
                    )}

                    {/* View: Spotify Player */}
                    {currentView === "spotify" && (
                      <div className="flex-1 flex flex-col justify-between pt-2 pb-4 animate-fade-in">
                        <div className="flex items-center justify-between text-xs text-gray-400">
                          <button onClick={() => setCurrentView("lyrily")} className="p-1 hover:bg-white/5 rounded-full">
                            <i data-lucide="x" class="w-4 h-4 text-white"></i>
                          </button>
                          <span className="font-semibold text-[8px] tracking-widest text-gray-300">SPOTIFY ENTEGRASYONU</span>
                          <i data-lucide="volume-2" class="w-4 h-4 text-[#1ed760]"></i>
                        </div>

                        <div className="flex-1 flex flex-col items-center justify-center py-4">
                          <img src={activeSong.albumArt} alt={activeSong.title} className={`w-36 h-36 rounded-2xl object-cover shadow-2xl border border-gray-950 transition-transform duration-700 ${isPlaying ? "scale-105" : "scale-95"}`} />
                          <div className="text-center mt-4 w-full px-4">
                            <h3 className="text-sm font-bold text-white truncate">{activeSong.title}</h3>
                            <p className="text-xs text-gray-400 truncate mt-0.5">{activeSong.artist}</p>
                            <span className="text-[8px] text-[#1ed760] font-mono mt-1.5 bg-[#1ed760]/10 border border-[#1ed760]/20 px-2 py-0.5 rounded-full inline-block">Lyrily Active</span>
                          </div>
                        </div>

                        <div className="px-1 mt-auto">
                          <input 
                            type="range" min="0" max={activeSong.duration} value={progress}
                            onChange={(e) => setProgress(parseFloat(e.target.value))}
                            className="w-full h-1 bg-gray-800 rounded-lg appearance-none cursor-pointer accent-[#1ed760]"
                          />
                          <div className="flex justify-between text-[8px] text-gray-500 mt-1 font-mono">
                            <span>{Math.floor(progress / 60)}:{String(Math.floor(progress % 60)).padStart(2, "0")}</span>
                            <span>{Math.floor(activeSong.duration / 60)}:{String(Math.floor(activeSong.duration % 60)).padStart(2, "0")}</span>
                          </div>
                        </div>

                        <div className="flex items-center justify-between px-6 mt-3">
                          <i data-lucide="shuffle" class="w-4 h-4 text-gray-400 hover:text-white"></i>
                          <div className="flex items-center gap-5">
                            <button 
                              onClick={() => {
                                const idx = songs.findIndex(s => s.id === activeSong.id);
                                const prev = idx > 0 ? idx - 1 : songs.length - 1;
                                setActiveSong(songs[prev]);
                                setProgress(0);
                              }}
                              className="text-white hover:text-spotify-bright cursor-pointer"
                            >
                              <i data-lucide="skip-back" class="w-4 h-4"></i>
                            </button>

                            <button 
                              onClick={() => setIsPlaying(!isPlaying)}
                              className="w-10 h-10 bg-white text-black rounded-full flex items-center justify-center hover:scale-105 transition-transform cursor-pointer"
                            >
                              {isPlaying ? <i data-lucide="pause" class="w-4 h-4 text-black"></i> : <i data-lucide="play" class="w-4 h-4 text-black translate-x-0.5"></i>}
                            </button>

                            <button 
                              onClick={() => {
                                const idx = songs.findIndex(s => s.id === activeSong.id);
                                const next = idx < songs.length - 1 ? idx + 1 : 0;
                                setActiveSong(songs[next]);
                                setProgress(0);
                              }}
                              className="text-white hover:text-spotify-bright cursor-pointer"
                            >
                              <i data-lucide="skip-forward" class="w-4 h-4"></i>
                            </button>
                          </div>
                          <i data-lucide="repeat" class="w-4 h-4 text-[#1ed760]"></i>
                        </div>
                      </div>
                    )}

                    {/* View: Instagram feed */}
                    {currentView === "instagram" && (
                      <div className="flex-1 flex flex-col pt-1 animate-fade-in">
                        <div className="flex justify-between items-center border-b border-zinc-900 pb-2">
                          <h3 className="font-semibold text-sm">Instagram</h3>
                          <div className="flex gap-3 text-white">
                            <i data-lucide="heart" class="w-4 h-4"></i>
                            <i data-lucide="send" class="w-4 h-4"></i>
                          </div>
                        </div>

                        <div className="flex-1 overflow-y-auto mt-2 space-y-3">
                          {/* Feed Story items */}
                          <div className="flex gap-2 pb-2 border-b border-zinc-900 overflow-x-auto">
                            <div className="flex flex-col items-center gap-0.5 shrink-0">
                              <div className="w-9 h-9 rounded-full p-[1px] bg-gradient-to-tr from-yellow-400 to-fuchsia-600">
                                <div className="w-full h-full rounded-full bg-zinc-800"></div>
                              </div>
                              <span className="text-[8px] text-gray-500">Hikayen</span>
                            </div>
                            {[1, 2, 3].map(i => (
                              <div key={i} className="flex flex-col items-center gap-0.5 shrink-0">
                                <div className="w-9 h-9 rounded-full p-[1px] bg-gradient-to-tr from-yellow-400 to-fuchsia-600">
                                  <div className="w-full h-full rounded-full bg-zinc-700 flex items-center justify-center font-mono text-[9px]">U{i}</div>
                                </div>
                                <span className="text-[8px] text-gray-500">user_{i}</span>
                              </div>
                            ))}
                          </div>

                          {/* Dynamic mock post */}
                          <div className="bg-[#111] rounded-lg border border-zinc-900 overflow-hidden">
                            <div className="p-2 flex items-center gap-2">
                              <div className="w-5 h-5 rounded-full bg-zinc-700 flex items-center justify-center font-bold text-[8px]">AM</div>
                              <span className="text-[9px] font-semibold">art_and_mind</span>
                            </div>
                            <img src="https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?w=300&auto=format&fit=crop&q=60" className="w-full h-36 object-cover" />
                            <div className="p-2">
                              <p className="text-[9px] text-gray-300">Harika doğa melodileri eşliğinde huzur dolu bir gün geçiriyoruz... 🌿✨</p>
                              <span className="text-[7px] text-gray-500 mt-1 block font-mono">2 SAAT ÖNCE</span>
                            </div>
                          </div>
                        </div>
                      </div>
                    )}

                  </div>

                  {/* Floating Overlay Widget (Drag draw-over widget simulation) */}
                  {overlay.enabled && (
                    <div 
                      onMouseDown={handleDragStart} onTouchStart={handleDragStart}
                      style={{
                        left: `${overlay.x}px`,
                        top: `${overlay.y}px`,
                        width: `${overlay.width}px`,
                        opacity: overlay.opacity
                      }}
                      className={`absolute z-50 p-2.5 rounded-xl shadow-2xl border cursor-grab active:cursor-grabbing select-none transition-shadow ${
                        overlay.theme === "glass" 
                          ? "bg-[#0b0b14]/75 backdrop-blur-md border-white/10" 
                          : overlay.theme === "dark" 
                          ? "bg-[#050508] border-gray-800" 
                          : "bg-[#050c05]/95 border-[#1db954]/30"
                      }`}
                    >
                      <div className="flex items-center justify-between border-b border-white/5 pb-1 mb-1.5 pointer-events-none">
                        <div className="flex gap-1 items-center">
                          <span className="w-1.5 h-1.5 rounded-full bg-[#1db954] animate-pulse"></span>
                          <span className="text-[7.5px] text-gray-400 font-bold uppercase tracking-wider">LYRILY OVERLAY</span>
                        </div>
                        <div className="flex gap-0.5">
                          <span className="w-0.5 h-1.5 bg-gray-600 rounded"></span>
                          <span className="w-0.5 h-1.5 bg-gray-600 rounded"></span>
                        </div>
                        <button 
                          onMouseDown={e => e.stopPropagation()} onTouchStart={e => e.stopPropagation()}
                          onClick={() => setOverlay(prev => ({ ...prev, enabled: false }))}
                          className="pointer-events-auto p-0.5 hover:bg-white/10 rounded-full text-gray-400"
                        >
                          <i data-lucide="x" class="w-3 h-3"></i>
                        </button>
                      </div>

                      {activeLyric ? (
                        <div className="space-y-1">
                          <p className={`text-white font-semibold leading-tight tracking-tight ${
                            overlay.textSize === 'sm' ? 'text-xs' : overlay.textSize === 'base' ? 'text-sm' : overlay.textSize === 'lg' ? 'text-base' : 'text-lg'
                          }`}>{activeLyric.text}</p>
                          <p className={`text-[#1db954] font-medium leading-tight ${overlay.textSize === 'sm' ? 'text-[9.5px]' : 'text-xs'}`}>{activeLyric.translation}</p>
                          
                          {upcomingLyric && (
                            <p className="text-[7.5px] text-gray-500 font-mono italic mt-1 pt-1 border-t border-white/5">
                              Sıradaki: {upcomingLyric.text}
                            </p>
                          )}
                        </div>
                      ) : (
                        <div className="text-center py-1 text-[9px] text-gray-500">
                          Şarkı başlatılıyor...
                        </div>
                      )}
                    </div>
                  )}

                  {/* Free tier simulated AdMob Banner */}
                  {!isPremium && (
                    <div className="absolute bottom-12 left-4 right-4 bg-zinc-900/95 border border-white/5 rounded-lg p-1.5 flex items-center justify-between text-[8px] text-gray-400 z-35 shadow-lg">
                      <div className="flex items-center gap-1 truncate">
                        <span className="bg-[#1db954]/20 text-[#1db954] text-[7px] font-bold px-1 py-0.5 rounded uppercase">AD</span>
                        <span className="truncate italic">Duo: Çeviri yaparak yabancı dilini anında geliştir!</span>
                      </div>
                      <button onClick={() => setShowPaywall(true)} className="text-[#1db954] underline ml-1.5 shrink-0 font-medium">Kapat (PRO)</button>
                    </div>
                  )}

                  {/* Premium Paywall Screen */}
                  {showPaywall && (
                    <div className="absolute inset-x-0 bottom-0 top-10 bg-[#09090b]/98 z-50 p-4 flex flex-col justify-between text-white rounded-t-[28px] border-t border-white/10 shadow-2xl overflow-y-auto animate-fade-in">
                      <div className="space-y-4">
                        <div className="flex justify-between items-start">
                          <div className="flex items-center gap-2">
                            <div className="w-7 h-7 rounded-full bg-[#1db954]/10 border border-[#1db954]/20 flex items-center justify-center text-[#1db954]">
                              <i data-lucide="sparkles" class="w-4 h-4"></i>
                            </div>
                            <div>
                              <h3 className="font-bold text-xs">LyricSync Premium</h3>
                              <p className="text-[7px] text-gray-500 font-mono">Google Play Store v6.1</p>
                            </div>
                          </div>
                          <button onClick={() => setShowPaywall(false)} className="p-1 hover:bg-white/5 rounded-full text-gray-400"><i data-lucide="x" class="w-4 h-4"></i></button>
                        </div>

                        <div className="text-center">
                          <div className="text-2xl font-extrabold">$4.99</div>
                          <p className="text-[8px] text-[#1db954] font-semibold uppercase tracking-wider mt-0.5">TEK SEFERLİK ÖMÜR BOYU LİSANS</p>
                        </div>

                        <div className="space-y-2 pt-2 text-[10px]">
                          <div className="flex gap-2">
                            <i data-lucide="check" class="w-4 h-4 text-[#1db954] shrink-0"></i>
                            <div><p className="font-semibold">Sınırsız AI Çeviri & Senkron</p><p className="text-[8px] text-gray-400">Limitleri kaldırın, dilediğinizce şarkı senkronize edin</p></div>
                          </div>
                          <div className="flex gap-2">
                            <i data-lucide="check" class="w-4 h-4 text-[#1db954] shrink-0"></i>
                            <div><p className="font-semibold">Sıfır Reklam Deneyimi</p><p className="text-[8px] text-gray-400">Tüm arayüz ve banner sponsor reklamlarını gizleyin</p></div>
                          </div>
                          <div className="flex gap-2">
                            <i data-lucide="check" class="w-4 h-4 text-[#1db954] shrink-0"></i>
                            <div><p className="font-semibold">Yabancı Dil Çeviri Desteği</p><p className="text-[8px] text-gray-400">Almanca, İspanyolca ve Fransızca dillerini etkinleştirin</p></div>
                          </div>
                        </div>
                      </div>

                      <div className="space-y-1.5 pt-4">
                        <button 
                          onClick={() => { setIsPremium(true); setCredits(9999); setShowPaywall(false); }}
                          className="w-full bg-[#1db954] hover:bg-spotify-bright text-black font-bold text-xs py-2 rounded-xl transition-all"
                        >
                          Premium'a Yükselt ($4.99)
                        </button>
                        <button onClick={() => setShowPaywall(false)} className="w-full bg-white/5 border border-white/10 text-gray-400 text-[10px] py-1.5 rounded-xl">Ücretsiz Devam Et</button>
                        <p className="text-[7px] text-center text-gray-500 leading-relaxed">Ödemeleriniz Google Play Hesabınız üzerinden tahsil edilir.</p>
                      </div>
                    </div>
                  )}

                  {/* Android Home gesture bar */}
                  <div className="absolute bottom-1.5 left-1/2 -translate-x-1/2 w-32 h-1 bg-white/30 rounded-full z-40"></div>

                  {/* Simulator OS dock bar */}
                  <div className="absolute bottom-4 left-0 right-0 h-8 flex justify-around items-center px-6 z-40 bg-gradient-to-t from-black to-transparent">
                    <button onClick={() => setCurrentView("home")} className={`p-1 hover:bg-white/5 rounded-full ${currentView === "home" ? "text-[#1db954]" : "text-gray-400"}`}>
                      <i data-lucide="home" class="w-4 h-4"></i>
                    </button>
                    <button onClick={() => setCurrentView("lyrily")} className={`p-1 hover:bg-white/5 rounded-full ${currentView === "lyrily" ? "text-[#1db954]" : "text-gray-400"}`}>
                      <i data-lucide="sliders" class="w-4 h-4"></i>
                    </button>
                    <button onClick={() => setCurrentView("spotify")} className={`p-1 hover:bg-white/5 rounded-full ${currentView === "spotify" ? "text-[#1db954]" : "text-gray-400"}`}>
                      <i data-lucide="music" class="w-4 h-4"></i>
                    </button>
                  </div>

                </div>
              </div>

              <p className="text-[11px] text-gray-500 flex items-center gap-1.5 max-w-xs text-center leading-relaxed">
                <i data-lucide="help-circle" class="w-3.5 h-3.5 text-gray-400 shrink-0"></i>
                İpucu: Yüzen pencereyi telefon ekranının herhangi bir yerine sürükleyip bırakabilirsiniz.
              </p>
            </div>

            {/* Right Col: Feature Descriptions & Code Integration Panels */}
            <div className="lg:col-span-7 flex flex-col gap-8 h-full">
              
              {/* Feature A: Translation Engine Info Card */}
              <section className="bg-gradient-to-br from-[#121214] to-[#09090b] border border-white/5 rounded-2xl p-6 shadow-xl relative overflow-hidden">
                <div className="absolute top-0 right-0 w-32 h-32 bg-[#1db954]/5 rounded-full blur-2xl pointer-events-none"></div>
                
                <div className="flex items-start gap-4">
                  <div className="p-3 bg-[#1db954]/10 border border-[#1db954]/20 rounded-xl text-[#1db954] shrink-0">
                    <i data-lucide="sparkles" class="w-6 h-6"></i>
                  </div>
                  <div>
                    <h3 className="text-white font-bold text-base font-space tracking-tight flex items-center gap-2">
                      Gemini Kültürel Adaptasyon Çeviri Motoru
                      <span className="text-[9px] bg-[#1db954]/20 border border-[#1db954]/30 text-[#1db954] font-bold px-1.5 py-0.5 rounded uppercase">AKTİF</span>
                    </h3>
                    <p className="text-gray-400 text-xs leading-relaxed mt-1.5">
                      İngilizce şarkıların dize dize yapılan dümdüz çevirileri duygusunu yitirir. <b>Gemini 3.5 Flash</b>, şarkının tematik bağlamını analiz ederek deyimleri, sokak argolarını, metaforları ve duygusal derinliği koruyan, seçtiğiniz üsluba uygun (Sokak Türkçesi, Şiirsel vb.) mükemmel bir adaptasyon sağlar.
                    </p>
                  </div>
                </div>

                <div className="grid grid-cols-1 sm:grid-cols-3 gap-4 border-t border-white/5 mt-5 pt-5 text-xs text-gray-400">
                  <div className="flex items-center gap-2.5">
                    <div className="w-6 h-6 rounded-full bg-zinc-800 flex items-center justify-center font-mono text-[10px] text-[#1db954] font-bold">1</div>
                    <div>
                      <p className="font-semibold text-white">LRC Senkronu</p>
                      <p className="text-[9.5px] text-gray-500">Hassas milisaniye zamanı</p>
                    </div>
                  </div>
                  <div className="flex items-center gap-2.5">
                    <div className="w-6 h-6 rounded-full bg-zinc-800 flex items-center justify-center font-mono text-[10px] text-[#1db954] font-bold">2</div>
                    <div>
                      <p className="font-semibold text-white">Üslup Kontrolü</p>
                      <p className="text-[9.5px] text-gray-500">Argodan dramatik şiirlere</p>
                    </div>
                  </div>
                  <div className="flex items-center gap-2.5">
                    <div className="w-6 h-6 rounded-full bg-zinc-800 flex items-center justify-center font-mono text-[10px] text-[#1db954] font-bold">3</div>
                    <div>
                      <p className="font-semibold text-white">Her Şarkı İçin</p>
                      <p className="text-[9.5px] text-gray-500">Gemini ile anında lirik üretimi</p>
                    </div>
                  </div>
                </div>
              </section>

              {/* Feature B: Mobile Code tab panel */}
              <section id="developer" className="bg-[#121214] border border-white/5 rounded-2xl overflow-hidden flex flex-col min-h-[460px] shadow-2xl">
                <div className="p-5 border-b border-white/5 bg-[#18181b] flex items-center justify-between">
                  <div className="flex items-center gap-3">
                    <div className="p-2 bg-[#1db954]/10 rounded-lg text-[#1db954]">
                      <i data-lucide="code" class="w-5 h-5"></i>
                    </div>
                    <div>
                      <h2 className="text-white font-semibold text-base font-space tracking-tight">Yerel Mobil Proje Entegrasyonu</h2>
                      <p className="text-gray-400 text-xs mt-0.5">Kotlin, React Native ve Flutter için temiz entegrasyon şablonları</p>
                    </div>
                  </div>
                  <span className="text-[9px] bg-zinc-800 border border-white/5 px-2 py-0.5 rounded-full text-gray-400 font-mono">v1.2.0</span>
                </div>

                {/* Tabs */}
                <div className="flex border-b border-white/5 bg-[#0e0e11] overflow-x-auto scroller-hidden">
                  {Object.keys(CODE_TEMPLATES).map((key) => (
                    <button
                      key={key} onClick={() => setActiveDevTab(key)}
                      className={`flex items-center gap-1.5 px-4 py-3 text-xs font-medium border-b-2 transition-all whitespace-nowrap cursor-pointer ${activeDevTab === key ? "border-[#1db954] text-[#1db954] bg-[#1db954]/5" : "border-transparent text-gray-400 hover:text-white"}`}
                    >
                      <i data-lucide={key === 'overlay' ? 'shield' : key === 'spotify_broadcast' ? 'music' : key === 'react_native' ? 'settings' : 'code'} class="w-3.5 h-3.5"></i>
                      {key === 'overlay' ? 'Android Overlay' : key === 'spotify_broadcast' ? 'Spotify Receiver' : key === 'react_native' ? 'React Native' : 'Flutter'}
                    </button>
                  ))}
                </div>

                {/* Code panel details */}
                <div className="p-4 bg-[#141416] border-b border-white/5 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3">
                  <div className="max-w-xl">
                    <h3 className="text-[#1db954] text-xs font-semibold uppercase tracking-wider">{CODE_TEMPLATES[activeDevTab].title}</h3>
                    <p className="text-gray-400 text-[11px] mt-0.5 leading-relaxed">{CODE_TEMPLATES[activeDevTab].desc}</p>
                  </div>
                  <div className="flex items-center justify-between gap-4 bg-zinc-800 px-3 py-2 rounded-xl border border-white/5 shrink-0">
                    <span className="font-mono text-xs text-gray-300">{CODE_TEMPLATES[activeDevTab].file}</span>
                    <button 
                      onClick={() => copyToClipboard(CODE_TEMPLATES[activeDevTab].code)}
                      className="p-1.5 hover:bg-white/5 rounded-lg text-gray-400 hover:text-[#1db954] transition-colors"
                    >
                      {copied ? <i data-lucide="check" class="w-4 h-4 text-[#1db954]"></i> : <i data-lucide="clipboard" class="w-4 h-4"></i>}
                    </button>
                  </div>
                </div>

                <div className="flex-1 overflow-auto p-5 font-mono text-xs text-gray-400 bg-[#09090b] leading-relaxed relative min-h-[220px]">
                  <pre className="whitespace-pre overflow-x-auto">
                    <code>{CODE_TEMPLATES[activeDevTab].code}</code>
                  </pre>
                </div>
              </section>

              {/* Architectural block card */}
              <section className="bg-[#0e0e11] border border-white/5 p-5 rounded-2xl flex flex-col sm:flex-row items-center gap-4">
                <div className="p-3 bg-blue-500/10 border border-blue-500/20 rounded-2xl text-blue-400 shrink-0">
                  <i data-lucide="cpu" class="w-7 h-7"></i>
                </div>
                <div>
                  <h4 className="text-white font-bold text-xs uppercase tracking-wider">Mobil Mimari Nasıl Çalışır?</h4>
                  <p className="text-gray-400 text-xs mt-1 leading-relaxed">
                    Android'de çalışan yerel servis, bir <b>Spotify Broadcast Receiver</b> ile çalan şarkıyı dinler. Şarkı değiştiğinde veya süre ilerlediğinde uygulama yerel arka plan servisimize bildirim geçer. Arka plan servisimiz <b>Gemini API</b>'mizden gelen Türkçe adaptasyonlu karaoke söz dosyalarını alır ve <b>WindowManager</b> ile telefonun diğer pencerelerinin üzerine çizer.
                  </p>
                </div>
              </section>

            </div>
          </main>

          {/* Footer */}
          <footer className="border-t border-white/5 bg-[#09090b] px-6 py-5 text-center text-xs text-gray-500 mt-12 flex flex-col sm:flex-row items-center justify-between gap-3">
            <span>© 2026 LyricSync Spotify Lyrics Translator Prototipi. Google AI Studio ile geliştirilmiştir.</span>
            <div className="flex items-center gap-4 text-gray-400">
              <span className="hover:text-white transition-colors cursor-pointer">Sözleşme</span>
              <span className="hover:text-white transition-colors cursor-pointer">Gizlilik Politikası</span>
            </div>
          </footer>
        </div>
      );
    }

    const container = document.getElementById('root');
    const root = ReactDOM.createRoot(container);
    root.render(<App />);
  </script>
</body>
</html>
