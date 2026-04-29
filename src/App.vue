<template>
  <div id="cineretro">
    <div class="scanlines"></div>
    <div class="noise"></div>
    <router-view v-slot="{ Component }">
      <transition name="fade" mode="out-in">
        <component :is="Component" />
      </transition>
    </router-view>
  </div>
</template>

<style>
  :root {
    --crt-green: #00ff41;
    --crt-amber: #ffb000;
    --crt-red: #ff2244;
    --crt-blue: #00cfff;
    --bg-dark: #050a05;
    --bg-panel: #0a140a;
    --bg-card: #0d1a0d;
    --border: #1a3a1a;
    --font-display: 'Bebas Neue', sans-serif;
    --font-mono: 'Share Tech Mono', monospace;
    --font-pixel: 'Press Start 2P', monospace;
    --font-vt: 'VT323', monospace;
  }

  *, *::before, *::after { 
    box-sizing: border-box; 
    margin: 0; 
    padding: 0; 
  }

  html { 
    background: var(--bg-dark); 
  }

  body {
    background: var(--bg-dark);
    color: var(--crt-green);
    font-family: var(--font-mono);
    min-height: 100vh;
    overflow-x: hidden;
  }

  #cineretro {
    position: relative;
    min-height: 100vh;
  }

  .scanlines {
    pointer-events: none;
    position: fixed;
    inset: 0;
    z-index: 9999;
    background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(0, 0, 0, 0.15) 2px,
      rgba(0, 0, 0, 0.15) 4px
    );
  }

  .noise {
    pointer-events: none;
    position: fixed;
    inset: -200%;
    z-index: 9998;
    width: 400%;
    height: 400%;
    opacity: 0.03;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
    animation: noise-drift 8s steps(10) infinite;
  }

  @keyframes noise-drift {
    0% { transform: translate(0, 0); }
    10% { transform: translate(-5%, -5%); }
    20% { transform: translate(-10%, 5%); }
    30% { transform: translate(5%, -10%); }
    40% { transform: translate(-5%, 15%); }
    50% { transform: translate(-10%, 5%); }
    60% { transform: translate(15%, 0); }
    70% { transform: translate(0, 10%); }
    80% { transform: translate(-15%, 0); }
    90% { transform: translate(10%, 5%); }
    100% { transform: translate(5%, 0); }
  }

  ::-webkit-scrollbar { 
    width: 6px; 
  }

  ::-webkit-scrollbar-track { 
    background: var(--bg-dark); 
  }

  ::-webkit-scrollbar-thumb {
    background: var(--crt-green); 
  }

  .fade-enter-active, .fade-leave-active { 
    transition: opacity 0.3s ease; 
  }

  .fade-enter-from, .fade-leave-to { 
    opacity: 0; 
  }

  .glow {
    text-shadow: 0 0 8px currentColor, 0 0 20px currentColor;
  }

  @keyframes flicker {
    0%, 95%, 100% { opacity: 1; }
    96% { opacity: 0.85; }
    97% { opacity: 0.95; }
    98% { opacity: 0.75; }
    99% { opacity: 0.9; }
  }

  @keyframes blink {
    0%, 49% { opacity: 1; }
    50%, 100% { opacity: 0; }
  }

  @keyframes glitch {
    0% { transform: translate(0); }
    20% { transform: translate(-2px, 2px); }
    40% { transform: translate(-2px, -2px); }
    60% { transform: translate(2px, 2px); }
    80% { transform: translate(2px, -2px); }
    100% { transform: translate(0); }
  }
</style>
