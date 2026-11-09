# lawrence.github.io
import React, { useState, useEffect, useRef } from 'react';
import { 
  Sparkles, 
  MapPin, 
  Calendar, 
  Shirt, 
  Volume2, 
  VolumeX, 
  CheckCircle, 
  ExternalLink, 
  Gift, 
  Smartphone, 
  ShieldAlert, 
  HelpCircle,
  ChevronRight,
  X,
  User
} from 'lucide-react';

// Web Audio API Synthesizer for ambient tones & interactive SFX
class SoundFX {
  constructor() {
    this.ctx = null;
  }
  init() {
    if (!this.ctx) {
      const AudioCtx = window.AudioContext || window.webkitAudioContext;
      if (AudioCtx) this.ctx = new AudioCtx();
    }
    if (this.ctx && this.ctx.state === 'suspended') {
      this.ctx.resume();
    }
  }
  playPop() {
    this.init();
    if (!this.ctx) return;
    try {
      const osc = this.ctx.createOscillator();
      const gain = this.ctx.createGain();
      osc.type = 'sine';
      osc.frequency.setValueAtTime(580, this.ctx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(140, this.ctx.currentTime + 0.08);
      gain.gain.setValueAtTime(0.12, this.ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.08);
      osc.connect(gain);
      gain.connect(this.ctx.destination);
      osc.start();
      osc.stop(this.ctx.currentTime + 0.08);
    } catch(e) {}
  }
  playChime() {
    this.init();
    if (!this.ctx) return;
    try {
      const now = this.ctx.currentTime;
      [523.25, 659.25, 783.99, 1046.50].forEach((freq, idx) => {
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();
        osc.type = 'triangle';
        osc.frequency.setValueAtTime(freq, now + idx * 0.06);
        gain.gain.setValueAtTime(0.08, now + idx * 0.06);
        gain.gain.exponentialRampToValueAtTime(0.001, now + idx * 0.06 + 0.3);
        osc.connect(gain);
        gain.connect(this.ctx.destination);
        osc.start(now + idx * 0.06);
        osc.stop(now + idx * 0.06 + 0.3);
      });
    } catch(e) {}
  }
}

const sfx = new SoundFX();

// Exact 18 Secret Missions array
const SECRET_MISSIONS = [
  "Film a video asking at least 5 people at the gathering: 'What would you do if you were actually friends with Lawrence?' 🎤",
  "Perform British commentary out loud during 5 random conversations. 🎙️",
  "Record 4 people doing a runway walk towards a phone camera. 👠",
  "Every time someone says something about school, stand up and give them an intense 10-second round of applause. 👏",
  "Start a completely serious debate with 3 different people, pretending you support MAGA. 🇺🇸",
  "Invent a complex 5-step secret handshake and successfully teach it to 3 different people before the night ends. Vlog it. 🤝",
  "Take 5 photos of people in the background where you are secretly holding up a spoon in front of the lens like a frame. 📸",
  "Stand up, clear your throat with a spoon against a glass, and give a 30-second deeply emotional toast to Lawrence using at least three fake words. 🥂",
  "Film a 15-second video getting at least 4 people to react to a completely mundane event (like someone taking a sip of water) in exaggerated, dramatic slow motion. 🎬",
  "Lock eye contact with 3 different people across the room for 20 seconds straight, nod very slowly once, and then immediately look away without smiling. 👁️",
  "Keep a spoon balanced on the tip of your nose for at least 10 unbroken seconds while someone films it. 🥄",
  "Whenever anyone asks you a question, pause for 5 seconds, do a slight idle-animation sway, and answer in a completely flat voice before returning to normal. 🤖",
  "Step in front of someone while they are mid-sentence, put your hand over their phone/face, and tell the group 'My client has no further comments on this matter at this time.' 🛑",
  "Silently hand a fork or spoon to someone who isn't eating or holding food, whisper 'You'll need this later,' and walk away. 🍴",
  "Ask to read the face and palm structure of 3 people and give them absurdly specific predictions about their next 24 hours (e.g., 'You will lose a shoe near a convenience store tomorrow at 3:14 PM'). 🔮",
  "For 3 minutes straight with a group of 3 or more people, repeat the very last word of every sentence someone speaks in a thoughtful whisper. 🗣️",
  "Treat a simple snack or item at the table like a thousand dollar gourmet dish, describing its 'notes and mouthfeel' out loud to at least 3 people. 🍷",
  "Sing happy birthday like the elephant from Sing (text me if udk what this means) 🐘🎶"
];

export default function App() {
  const [screen, setScreen] = useState(1);
  const [guestName, setGuestName] = useState('');
  const [nameError, setNameError] = useState(false);
  const [soundEnabled, setSoundEnabled] = useState(true);
  const [noBtnPos, setNoBtnPos] = useState({ top: 'auto', left: 'auto', absolute: false });
  const [secretMission, setSecretMission] = useState(null);
  
  // Modals state
  const [showMissionModal, setShowMissionModal] = useState(false);
  const [showKahootModal, setShowKahootModal] = useState(false);

  // Background Canvas for ambient particles
  const canvasRef = useRef(null);

  useEffect(() => {
    const saved = localStorage.getItem('lawrence_bday_mission_2026');
    if (saved) {
      setSecretMission(saved);
    }
  }, []);

  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    let animationFrameId;

    let width = canvas.width = window.innerWidth;
    let height = canvas.height = window.innerHeight;

    const handleResize = () => {
      if (!canvas) return;
      width = canvas.width = window.innerWidth;
      height = canvas.height = window.innerHeight;
    };
    window.addEventListener('resize', handleResize);

    const particles = Array.from({ length: 45 }).map(() => ({
      x: Math.random() * width,
      y: Math.random() * height,
      radius: Math.random() * 1.5 + 0.4,
      alpha: Math.random() * 0.45 + 0.2,
      speedX: (Math.random() - 0.5) * 0.3,
      speedY: -Math.random() * 0.4 - 0.1
    }));

    const render = () => {
      ctx.clearRect(0, 0, width, height);

      particles.forEach(p => {
        p.x += p.speedX;
        p.y += p.speedY;

        if (p.y < 0) {
          p.y = height;
          p.x = Math.random() * width;
        }
        if (p.x < 0) p.x = width;
        if (p.x > width) p.x = 0;

        ctx.beginPath();
        ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(255, 255, 255, ${p.alpha})`;
        ctx.shadowBlur = 6;
        ctx.shadowColor = 'rgba(255, 255, 255, 0.3)';
        ctx.fill();
        ctx.shadowBlur = 0;
      });

      animationFrameId = requestAnimationFrame(render);
    };

    render();

    return () => {
      window.removeEventListener('resize', handleResize);
      cancelAnimationFrame(animationFrameId);
    };
  }, []);

  const triggerSound = (type = 'pop') => {
    if (!soundEnabled) return;
    if (type === 'pop') sfx.playPop();
    if (type === 'chime') sfx.playChime();
  };

  const dodgeNoButton = () => {
    if (!guestName.trim()) {
      setNameError(true);
      return;
    }
    triggerSound('pop');
    const rx = Math.floor(Math.random() * 60) + 15;
    const ry = Math.floor(Math.random() * 60) + 20;
    setNoBtnPos({
      top: `${ry}%`,
      left: `${rx}%`,
      absolute: true
    });
  };

  const handleRsvpYes = () => {
    if (!guestName.trim()) {
      setNameError(true);
      triggerSound('pop');
      return;
    }
    setNameError(false);
    triggerSound('chime');
    setScreen(3);
  };

  const getOrGenerateMission = () => {
    triggerSound('chime');
    if (secretMission) {
      setShowMissionModal(true);
      return;
    }
    const randomIndex = Math.floor(Math.random() * SECRET_MISSIONS.length);
    const assigned = SECRET_MISSIONS[randomIndex];
    localStorage.setItem('lawrence_bday_mission_2026', assigned);
    setSecretMission(assigned);
    setShowMissionModal(true);
  };

  return (
    <div className="relative min-h-screen bg-[#09090b] text-zinc-100 font-sans antialiased overflow-x-hidden selection:bg-zinc-700 selection:text-white flex flex-col justify-between">
      {/* Background Canvas Particles */}
      <canvas ref={canvasRef} className="fixed inset-0 pointer-events-none z-0 opacity-40" />

      {/* Glow Orbs Background */}
      <div className="fixed top-[-10%] left-[-10%] w-[450px] h-[450px] bg-zinc-800/20 rounded-full blur-[140px] pointer-events-none" />
      <div className="fixed bottom-[-10%] right-[-10%] w-[450px] h-[450px] bg-indigo-950/20 rounded-full blur-[140px] pointer-events-none" />

      {/* Top Header */}
      <header className="relative z-20 flex items-center justify-between px-6 py-5 max-w-2xl mx-auto w-full border-b border-zinc-800/60 backdrop-blur-md">
        <div className="flex items-center space-x-2">
          <span className="h-2 w-2 rounded-full bg-emerald-400 animate-ping" />
          <span className="text-[11px] font-mono uppercase tracking-[0.2em] text-zinc-400">
            LAWRENCE // 18TH
          </span>
        </div>
        <button
          onClick={() => {
            setSoundEnabled(!soundEnabled);
            if (!soundEnabled) triggerSound('pop');
          }}
          className="p-2 rounded-full border border-zinc-800 bg-zinc-900/60 text-zinc-400 hover:text-white hover:border-zinc-700 transition"
          title="Toggle SFX"
        >
          {soundEnabled ? <Volume2 size={16} /> : <VolumeX size={16} />}
        </button>
      </header>

      {/* MAIN CONTENT AREA */}
      <main className="relative z-10 max-w-xl mx-auto px-4 py-8 flex-1 flex flex-col justify-center w-full">
        {/* ================= SCREEN 1: INTRO PERMIT ================= */}
        {screen === 1 && (
          <div className="space-y-6 bg-zinc-900/60 backdrop-blur-2xl border border-zinc-800/80 rounded-3xl p-8 sm:p-10 shadow-2xl relative overflow-hidden">
            {/* Header Tag */}
            <div className="inline-flex items-center space-x-2 px-3 py-1 rounded-full border border-amber-500/30 bg-amber-500/10 text-amber-400 text-[10px] font-mono tracking-widest uppercase">
              <ShieldAlert size={12} />
              <span>ENTRY PERMIT REQUIRED</span>
            </div>

            {/* Title & Subtext */}
            <div className="space-y-2">
              <h1 className="text-3xl sm:text-4xl font-black tracking-tight text-white leading-tight">
                Lawrence's 18th Gathering
              </h1>
              <p className="text-zinc-400 text-sm leading-relaxed font-light">
                A small gathering for my close friends.
              </p>
            </div>

            {/* Notice Card */}
            <div className="p-5 rounded-2xl border border-zinc-800/80 bg-zinc-950/70 space-y-4 shadow-inner">
              <div className="flex items-start space-x-3 text-zinc-300 text-xs sm:text-sm leading-relaxed">
                <span className="text-base">⚠️</span>
                <div>
                  <p className="font-semibold text-white mb-1">Quick Heads Up</p>
                  <p className="text-zinc-400 leading-normal">
                    This is the very first site I've ever coded from scratch... If a button looks weird or something, pretend it didn't happen and let me know. 😅
                  </p>
                </div>
              </div>

              <div className="pt-3 border-t border-zinc-800/80 text-[11px] text-zinc-500 font-mono">
                // Note: If any wording is cringe, Gemini wrote it.
              </div>
            </div>

            {/* Next Button */}
            <button
              onClick={() => {
                triggerSound('chime');
                setScreen(2);
              }}
              className="w-full group relative flex items-center justify-center space-x-2 py-4 px-6 rounded-2xl bg-zinc-100 text-zinc-950 font-bold text-xs uppercase tracking-wider hover:bg-white transition shadow-xl active:scale-95"
            >
              <span>I PROMISE TO BE NICE</span>
              <ChevronRight size={16} className="group-hover:translate-x-1 transition-transform" />
            </button>
          </div>
        )}

        {/* ================= SCREEN 2: INVITATION & RSVP ================= */}
        {screen === 2 && (
          <div className="space-y-6 bg-zinc-900/60 backdrop-blur-2xl border border-zinc-800/80 rounded-3xl p-8 sm:p-10 shadow-2xl text-center relative">
            <div className="inline-flex items-center space-x-2 px-3 py-1 rounded-full border border-zinc-800 bg-zinc-950/80 text-zinc-300 text-[10px] font-mono tracking-widest uppercase">
              <Sparkles size={12} className="text-amber-400" />
              <span>LAWRENCE 18TH</span>
            </div>

            <div className="space-y-2">
              <h1 className="text-3xl sm:text-4xl font-extrabold tracking-tight text-white">
                You’re Invited.
              </h1>
              <p className="text-zinc-400 text-xs sm:text-sm leading-relaxed font-light max-w-sm mx-auto">
                Joining for an exclusive 18th gathering evening at Central's finest Italian joint.
              </p>
            </div>

            {/* Quick Info Snapshot Cards */}
            <div className="grid grid-cols-1 sm:grid-cols-2 gap-3 text-left">
              <div className="bg-zinc-950/80 p-4 rounded-2xl border border-zinc-800/80 space-y-1">
                <div className="flex items-center space-x-1.5 text-[10px] font-mono text-zinc-400 uppercase tracking-wider">
                  <Calendar size={12} className="text-amber-400" />
                  <span>WHEN</span>
                </div>
                <div className="text-xs font-bold text-white">November 20th</div>
                <div className="text-[11px] text-zinc-400 font-mono">7:00 PM – 9:00 PM</div>
              </div>

              <div className="bg-zinc-950/80 p-4 rounded-2xl border border-zinc-800/80 space-y-1">
                <div className="flex items-center space-x-1.5 text-[10px] font-mono text-zinc-400 uppercase tracking-wider">
                  <MapPin size={12} className="text-amber-400" />
                  <span>WHERE</span>
                </div>
                <div className="text-xs font-bold text-white">Franks New York Italian</div>
                <div className="text-[11px] text-zinc-400">Central, HK</div>
              </div>
            </div>

            {/* Interactive RSVP Box */}
            <div className="bg-zinc-950/90 p-6 rounded-2xl border border-zinc-800/90 space-y-5 text-left">
              <div className="text-center space-y-1">
                <h3 className="text-xs font-mono tracking-widest text-zinc-400 uppercase">
                  CONFIRM YOUR ATTENDANCE
                </h3>
              </div>

              {/* Name Input Field */}
              <div className="space-y-1.5">
                <label className="text-[11px] font-mono text-zinc-400 uppercase flex items-center space-x-1.5">
                  <User size={13} className="text-amber-400" />
                  <span>YOUR NAME / NICKNAME <span className="text-amber-400">*</span></span>
                </label>
                <input
                  type="text"
                  value={guestName}
                  onChange={(e) => {
                    setGuestName(e.target.value);
                    if (e.target.value.trim()) setNameError(false);
                  }}
                  placeholder="e.g. Alex"
                  className={`w-full px-4 py-3 rounded-xl bg-zinc-900 border text-white text-xs focus:outline-none transition ${
                    nameError 
                      ? 'border-red-500 focus:border-red-500 placeholder-red-400/60' 
                      : 'border-zinc-800 focus:border-zinc-500 placeholder-zinc-600'
                  }`}
                />
                {nameError && (
                  <p className="text-[10px] text-red-400 font-mono">Please enter your name first so Lawrence knows who you are!</p>
                )}
              </div>

              <div className="flex flex-col sm:flex-row items-center justify-center gap-3 relative min-h-[50px] pt-1">
                {/* YES BUTTON */}
                <button
                  onClick={handleRsvpYes}
                  className="w-full sm:w-auto flex-1 py-3.5 px-6 rounded-xl bg-gradient-to-r from-zinc-100 to-zinc-300 text-zinc-950 font-bold text-xs uppercase tracking-wider hover:from-white hover:to-zinc-200 transition shadow-lg active:scale-95 flex items-center justify-center gap-2"
                >
                  <span>YES! I'M IN! 🎉</span>
                </button>

                {/* RUNAWAY NO BUTTON */}
                <button
                  onMouseEnter={dodgeNoButton}
                  onClick={dodgeNoButton}
                  style={
                    noBtnPos.absolute
                      ? { position: 'fixed', top: noBtnPos.top, left: noBtnPos.left, zIndex: 50 }
                      : {}
                  }
                  className="w-full sm:w-auto py-3.5 px-5 rounded-xl border border-zinc-800 bg-zinc-900 text-zinc-400 hover:text-white text-xs font-mono transition-all"
                >
                  No, I hate you 😢
                </button>
              </div>
            </div>
          </div>
        )}

        {/* ================= SCREEN 3: DETAILS & EVENT HUB ================= */}
        {screen === 3 && (
          <div className="space-y-4">
            {/* Confirmation Banner */}
            <div className="p-5 rounded-3xl border border-emerald-500/30 bg-emerald-500/10 backdrop-blur-xl flex items-start space-x-3 text-left">
              <CheckCircle size={20} className="text-emerald-400 shrink-0 mt-0.5" />
              <div>
                <h2 className="text-sm font-bold text-white uppercase tracking-wider">
                  YAY! 🥳 RSVP CONFIRMED FOR {guestName.toUpperCase()}
                </h2>
                <p className="text-xs text-emerald-300/80 mt-0.5">Can't wait to see you! Here is everything you need to know.</p>
              </div>
            </div>

            {/* Event Details Grid */}
            <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">
              {/* Where Card */}
              <div className="p-5 rounded-2xl border border-zinc-800/80 bg-zinc-900/60 backdrop-blur-xl space-y-2 flex flex-col justify-between text-left">
                <div>
                  <div className="flex items-center space-x-1.5 text-[10px] font-mono text-zinc-400 tracking-widest uppercase mb-1">
                    <MapPin size={13} className="text-amber-400" />
                    <span>WHERE</span>
                  </div>
                  <h4 className="text-sm font-bold text-white">Franks New York Italian</h4>
                  <p className="text-[11px] text-zinc-400 mt-1 leading-relaxed">
                    79 Wyndham St, Central, Hong Kong Island
                  </p>
                </div>
                <a
                  href="https://www.google.com/maps/place/Franks+New+York+Italian/@22.2818967,114.1531766,17z/data=!3m1!4b1!4m6!3m5!1s0x340400a251d78c47:0x81eaeeedebe98b7d!8m2!3d22.2818967!4d114.1557515!16s%2Fg%2F11fn7v52c4?entry=ttu"
                  target="_blank"
                  rel="noreferrer"
                  onClick={() => triggerSound('pop')}
                  className="inline-flex items-center space-x-1 text-[11px] font-mono text-amber-400 hover:text-amber-300 transition mt-2"
                >
                  <span>Open Google Maps</span>
                  <ExternalLink size={12} />
                </a>
              </div>

              {/* When & Dress Code Card */}
              <div className="p-5 rounded-2xl border border-zinc-800/80 bg-zinc-900/60 backdrop-blur-xl space-y-3 flex flex-col justify-between text-left">
                <div>
                  <div className="flex items-center space-x-1.5 text-[10px] font-mono text-zinc-400 tracking-widest uppercase mb-1">
                    <Calendar size={13} className="text-amber-400" />
                    <span>WHEN</span>
                  </div>
                  <h4 className="text-sm font-bold text-white">November 20th</h4>
                  <p className="text-[11px] font-mono text-zinc-400">7:00 PM – 9:00 PM</p>
                </div>

                <div className="pt-2 border-t border-zinc-800/80">
                  <div className="flex items-center space-x-1.5 text-[10px] font-mono text-zinc-400 tracking-widest uppercase mb-0.5">
                    <Shirt size={13} className="text-amber-400" />
                    <span>DRESS CODE</span>
                  </div>
                  <p className="text-xs font-bold text-zinc-200">Black Polo</p>
                </div>
              </div>
            </div>

            {/* Contact Info Card */}
            <div className="p-4 rounded-2xl border border-zinc-800 bg-zinc-950 text-center space-y-1.5">
              <h3 className="text-[10px] font-mono tracking-widest text-zinc-400 uppercase flex items-center justify-center gap-1.5">
                <HelpCircle size={13} />
                <span>GOT QUESTIONS? 🤔</span>
              </h3>
              <p className="text-xs text-zinc-300 font-mono">
                WhatsApp: <strong className="text-white">56811002</strong> | Instagram: <strong className="text-zinc-400">you should know this</strong>
              </p>
              <p className="text-[11px] text-amber-400/90 italic bg-amber-500/10 py-1 px-3 rounded-lg border border-amber-500/20 inline-block mt-1">
                (Please let me know if you have any food allergies!) 🥜
              </p>
            </div>

            {/* Gathering Prep & Hub */}
            <div className="space-y-2.5 pt-2">
              <h3 className="text-[10px] font-mono uppercase tracking-widest text-zinc-500 text-center">
                GATHERING PREP & LINKS
              </h3>

              <div className="grid grid-cols-1 gap-2.5">
                {/* 1. Secret Mission Button */}
                <button
                  onClick={getOrGenerateMission}
                  className="bg-gradient-to-r from-amber-950/20 to-zinc-900/60 hover:from-amber-950/40 hover:to-zinc-900/80 p-4 rounded-2xl border border-amber-500/30 text-left transition flex items-center justify-between group shadow-md"
                >
                  <div className="flex items-center space-x-3">
                    <div className="w-9 h-9 rounded-xl bg-amber-500/10 border border-amber-500/30 text-amber-400 flex items-center justify-center">
                      <HelpCircle size={18} />
                    </div>
                    <div>
                      <h4 className="text-xs font-bold text-white group-hover:text-amber-300 transition">Your Secret Mission 🕵️‍♀️</h4>
                      <p className="text-[10px] text-zinc-400">Unique secret task saved to your device</p>
                    </div>
                  </div>
                  <ChevronRight size={16} className="text-zinc-600 group-hover:text-amber-400 group-hover:translate-x-1 transition" />
                </button>

                {/* 2. Kahoot Download Card */}
                <button
                  onClick={() => {
                    triggerSound('pop');
                    setShowKahootModal(true);
                  }}
                  className="bg-gradient-to-r from-indigo-950/20 to-zinc-900/60 hover:from-indigo-950/40 hover:to-zinc-900/80 p-4 rounded-2xl border border-indigo-500/30 text-left transition flex items-center justify-between group shadow-md"
                >
                  <div className="flex items-center space-x-3">
                    <div className="w-9 h-9 rounded-xl bg-indigo-500/10 border border-indigo-500/30 text-indigo-400 flex items-center justify-center">
                      <Smartphone size={18} />
                    </div>
                    <div>
                      <h4 className="text-xs font-bold text-white group-hover:text-indigo-300 transition">Download Kahoot! App 📱</h4>
                      <p className="text-[10px] text-zinc-400">Required for trivia mini-games</p>
                    </div>
                  </div>
                  <ChevronRight size={16} className="text-zinc-600 group-hover:text-indigo-400 group-hover:translate-x-1 transition" />
                </button>

                {/* 3. Inline Wishlist Card */}
                <div className="p-4 rounded-2xl border border-zinc-800 bg-zinc-900/40 backdrop-blur-sm space-y-1.5 text-left">
                  <div className="flex items-center space-x-1.5 text-[10px] font-mono text-zinc-400 uppercase tracking-wider">
                    <Gift size={13} className="text-amber-400" />
                    <span>WISHLIST NOTE</span>
                  </div>
                  <p className="text-xs text-zinc-300 leading-relaxed">
                    (Optional) but buy whatever you think I'd like — personal items like a card or handcraft are equally valued because of time and effort.
                  </p>
                </div>
              </div>
            </div>
          </div>
        )}
      </main>

      {/* ================= MODAL 1: SECRET MISSION ================= */}
      {showMissionModal && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/80 backdrop-blur-md">
          <div className="relative w-full max-w-md p-6 rounded-3xl border border-amber-500/40 bg-zinc-900 shadow-2xl space-y-5 text-left">
            <button
              onClick={() => setShowMissionModal(false)}
              className="absolute top-4 right-4 p-1.5 rounded-full text-zinc-400 hover:text-white bg-zinc-800"
            >
              <X size={16} />
            </button>

            <div className="flex items-center space-x-2 text-amber-400 font-mono text-[10px] tracking-widest uppercase">
              <ShieldAlert size={14} />
              <span>YOUR SECRET MISSION</span>
            </div>

            <div className="space-y-1">
              <h3 className="text-xl font-bold text-white">Confidential Task 🕵️‍♀️</h3>
              <p className="text-xs text-zinc-400">Screenshot this and don't tell anyone!</p>
            </div>

            <div className="p-4 rounded-2xl bg-zinc-950 border border-zinc-800 text-amber-200 font-mono text-xs leading-relaxed">
              "{secretMission}"
            </div>

            <div className="p-3.5 rounded-xl border border-amber-500/30 bg-amber-500/10 text-xs text-amber-300 space-y-1">
              <span className="font-bold block uppercase tracking-wider text-[10px]">⚠️ CRITICAL RULE:</span>
              <p className="text-[11px] text-amber-200/80">
                ONLY click this button on the phone you are bringing to the party! If you click it on your laptop, your mission will be stuck on your computer.
              </p>
            </div>

            <button
              onClick={() => setShowMissionModal(false)}
              className="w-full py-3 rounded-xl bg-amber-400 text-zinc-950 font-bold text-xs uppercase tracking-wider hover:bg-amber-300 transition"
            >
              GOT IT (SCREENSHOT TAKEN)
            </button>
          </div>
        </div>
      )}

      {/* ================= MODAL 2: KAHOOT DOWNLOAD ================= */}
      {showKahootModal && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/80 backdrop-blur-md">
          <div className="relative w-full max-w-sm p-6 rounded-3xl border border-indigo-500/40 bg-zinc-900 shadow-2xl space-y-5 text-center">
            <button
              onClick={() => setShowKahootModal(false)}
              className="absolute top-4 right-4 p-1.5 rounded-full text-zinc-400 hover:text-white bg-zinc-800"
            >
              <X size={16} />
            </button>

            <div className="mx-auto h-12 w-12 rounded-2xl bg-indigo-500/10 border border-indigo-500/30 flex items-center justify-center text-indigo-400">
              <Smartphone size={24} />
            </div>

            <div className="space-y-1">
              <h3 className="text-lg font-bold text-white">Download Kahoot! 📱</h3>
              <p className="text-xs text-zinc-400 leading-relaxed">
                We'll be playing trivia mini-games! Download Kahoot! on your phone before arriving.
              </p>
            </div>

            <div className="flex flex-col gap-2.5">
              <a
                href="https://apps.apple.com/app/kahoot-play-create-quizzes/id1300720412"
                target="_blank"
                rel="noreferrer"
                className="w-full py-3 rounded-xl bg-indigo-600 hover:bg-indigo-500 text-white font-bold text-xs transition flex items-center justify-center space-x-2"
              >
                <span> iOS App Store</span>
                <ExternalLink size={12} />
              </a>
              <a
                href="https://play.google.com/store/apps/details?id=no.mobitroll.kahoot.android"
                target="_blank"
                rel="noreferrer"
                className="w-full py-3 rounded-xl border border-zinc-700 bg-zinc-800 hover:bg-zinc-700 text-zinc-200 font-bold text-xs transition flex items-center justify-center space-x-2"
              >
                <span>🤖 Android Google Play</span>
                <ExternalLink size={12} />
              </a>
            </div>
          </div>
        </div>
      )}

      {/* Footer */}
      <footer className="relative z-10 max-w-2xl mx-auto px-6 py-4 w-full text-center text-[10px] font-mono text-zinc-600 uppercase tracking-widest">
        LAWRENCE 18TH GATHERING • ALL RIGHTS RESERVED
      </footer>
    </div>
  );
}
