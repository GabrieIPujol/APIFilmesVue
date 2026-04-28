<template>
    <div class="home">
      <!-- Header -->
      <header class="header">
        <div class="header-top">
          <div class="signal-bars">
            <span v-for="i in 5" :key="i" :class="['bar', `bar-${i}`]"></span>
          </div>
          <div class="logo-wrap">
            <div class="logo-sub">■ SISTEMA DE TRANSMISSÃO ■</div>
            <h1 class="logo glow">CINE<span class="logo-accent">RETRO</span></h1>
            <div class="logo-tagline">[ ARQUIVO DE FILMES — EST. 1984 ]</div>
          </div>
          <div class="status-panel">
            <div class="status-row"><span class="status-dot"></span> ONLINE</div>
            <div class="status-row">{{ currentTime }}</div>
          </div>
        </div>
  
        <!-- Controls -->
        <div class="controls">
          <div class="search-wrap">
            <span class="search-icon">&gt;_</span>
            <input
              v-model="searchQuery"
              @input="handleSearch"
              type="text"
              class="search-input"
              placeholder="BUSCAR TÍTULO..."
              autocomplete="off"
            />
            <span class="blink-cursor">█</span>
          </div>
          <div class="category-tabs">
            <button
              v-for="cat in categories"
              :key="cat.key"
              :class="['cat-btn', { active: activeCategory === cat.key }]"
              @click="setCategory(cat.key)"
            >{{ cat.label }}</button>
          </div>
        </div>
      </header>
  
      <!-- Status bar -->
      <div class="status-bar">
        <span>REGISTROS: {{ totalResults.toLocaleString('pt-BR') }}</span>
        <span>PÁG {{ currentPage }}/{{ totalPages }}</span>
        <span class="status-scroll">▶ SELECIONE UM TÍTULO PARA VER DETALHES ▶ SELECIONE UM TÍTULO ▶</span>
      </div>
  
      <!-- Loading -->
      <div v-if="loading" class="loading-screen">
        <div class="loading-box">
          <div class="pixel-art">▓▓▓░░░░░░░</div>
          <p class="loading-text">CARREGANDO DADOS<span class="dots">...</span></p>
          <div class="loading-bar"><div class="loading-fill" :style="{ width: loadPct + '%' }"></div></div>
        </div>
      </div>
  
      <!-- Error -->
      <div v-else-if="error" class="error-screen">
        <p class="pixel-font">⚠ ERRO DE TRANSMISSÃO</p>
        <p>{{ error }}</p>
        <button class="retro-btn" @click="fetchMovies">TENTAR NOVAMENTE</button>
      </div>
  
      <!-- Grid -->
      <main v-else class="movie-grid">
        <MovieCard
          v-for="movie in movies"
          :key="movie.id"
          :movie="movie"
          @click="goToMovie(movie.id)"
        />
      </main>
  
      <!-- Pagination -->
      <div v-if="!loading && !error && totalPages > 1" class="pagination">
        <button class="pg-btn" :disabled="currentPage === 1" @click="changePage(currentPage - 1)">◄ PREV</button>
        <div class="pg-info">
          <span v-for="p in visiblePages" :key="p">
            <button
              v-if="p !== '...'"
              :class="['pg-num', { active: p === currentPage }]"
              @click="changePage(p)"
            >{{ p }}</button>
            <span v-else class="pg-dots">...</span>
          </span>
        </div>
        <button class="pg-btn" :disabled="currentPage === totalPages" @click="changePage(currentPage + 1)">NEXT ►</button>
      </div>
  
      <footer class="footer">
        <p>CINERETRO © 1984 — DADOS: THE MOVIE DATABASE API — <span class="glow">ALL SYSTEMS NOMINAL</span></p>
      </footer>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted, onUnmounted } from 'vue'
  import { useRouter } from 'vue-router'
  import MovieCard from '../components/MovieCard.vue'
  
  const API_KEY = '4e44d9029b1270a757cddc766a1bcb63'
  const BASE_URL = 'https://api.themoviedb.org/3'
  
  const router = useRouter()
  const movies = ref([])
  const loading = ref(true)
  const error = ref(null)
  const searchQuery = ref('')
  const activeCategory = ref('popular')
  const currentPage = ref(1)
  const totalPages = ref(1)
  const totalResults = ref(0)
  const currentTime = ref('')
  const loadPct = ref(0)
  let searchTimeout = null
  let clockInterval = null
  
  const categories = [
    { key: 'popular', label: 'POPULARES' },
    { key: 'top_rated', label: 'MAIS VOTADOS' },
    { key: 'now_playing', label: 'EM CARTAZ' },
    { key: 'upcoming', label: 'EM BREVE' },
  ]
  
  function updateClock() {
    const now = new Date()
    currentTime.value = now.toLocaleTimeString('pt-BR', { hour12: false })
  }
  
  const visiblePages = computed(() => {
    const total = totalPages.value
    const cur = currentPage.value
    const pages = []
    if (total <= 7) {
      for (let i = 1; i <= total; i++) pages.push(i)
    } else {
      pages.push(1)
      if (cur > 3) pages.push('...')
      for (let i = Math.max(2, cur - 1); i <= Math.min(total - 1, cur + 1); i++) pages.push(i)
      if (cur < total - 2) pages.push('...')
      pages.push(total)
    }
    return pages
  })
  
  async function fetchMovies() {
    loading.value = true
    error.value = null
    loadPct.value = 0
    const interval = setInterval(() => { if (loadPct.value < 85) loadPct.value += 15 }, 200)
  
    try {
      let url
      if (searchQuery.value.trim()) {
        url = `${BASE_URL}/search/movie?api_key=${API_KEY}&language=pt-BR&query=${encodeURIComponent(searchQuery.value)}&page=${currentPage.value}`
      } else {
        url = `${BASE_URL}/movie/${activeCategory.value}?api_key=${API_KEY}&language=pt-BR&page=${currentPage.value}`
      }
      const res = await fetch(url)
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      const data = await res.json()
      movies.value = data.results
      totalPages.value = Math.min(data.total_pages, 500)
      totalResults.value = data.total_results
    } catch (e) {
      error.value = 'Falha ao conectar com o servidor de dados. Verifique sua conexão.'
    } finally {
      clearInterval(interval)
      loadPct.value = 100
      setTimeout(() => { loading.value = false }, 200)
    }
  }
  
  function setCategory(key) {
    activeCategory.value = key
    searchQuery.value = ''
    currentPage.value = 1
    fetchMovies()
  }
  
  function handleSearch() {
    clearTimeout(searchTimeout)
    currentPage.value = 1
    searchTimeout = setTimeout(fetchMovies, 500)
  }
  
  function changePage(p) {
    if (p < 1 || p > totalPages.value) return
    currentPage.value = p
    fetchMovies()
  }
  
  function goToMovie(id) {
    router.push({ name: 'MovieDetail', params: { id } })
  }
  
  onMounted(() => {
    updateClock()
    clockInterval = setInterval(updateClock, 1000)
    fetchMovies()
  })
  
  onUnmounted(() => {
    clearInterval(clockInterval)
    clearTimeout(searchTimeout)
  })
  </script>
  
  <style scoped>
  .home { min-height: 100vh; display: flex; flex-direction: column; }
  
  /* ─── HEADER ─── */
  .header {
    background: var(--bg-panel);
    border-bottom: 2px solid var(--crt-green);
    padding: 1.5rem 2rem 0;
    box-shadow: 0 4px 30px rgba(0, 255, 65, 0.15);
  }
  
  .header-top {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
  }
  
  .logo-wrap { text-align: center; }
  
  .logo {
    font-family: var(--font-display);
    font-size: clamp(3rem, 8vw, 6rem);
    color: var(--crt-green);
    letter-spacing: 0.05em;
    line-height: 1;
    animation: flicker 8s infinite;
  }
  
  .logo-accent { color: var(--crt-amber); }
  
  .logo-sub {
    font-family: var(--font-mono);
    font-size: 0.65rem;
    color: var(--crt-amber);
    letter-spacing: 0.3em;
    margin-bottom: 0.2rem;
  }
  
  .logo-tagline {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: rgba(0,255,65,0.5);
    letter-spacing: 0.2em;
    margin-top: 0.2rem;
  }
  
  /* Signal bars */
  .signal-bars {
    display: flex;
    align-items: flex-end;
    gap: 4px;
    height: 40px;
  }
  .bar {
    width: 8px;
    background: var(--crt-green);
    box-shadow: 0 0 6px var(--crt-green);
  }
  .bar-1 { height: 8px; }
  .bar-2 { height: 14px; }
  .bar-3 { height: 20px; }
  .bar-4 { height: 28px; }
  .bar-5 { height: 36px; opacity: 0.4; animation: blink 1.5s infinite; }
  
  /* Status panel */
  .status-panel {
    text-align: right;
    font-size: 0.75rem;
    color: var(--crt-amber);
    font-family: var(--font-mono);
    letter-spacing: 0.1em;
  }
  .status-row { margin-bottom: 0.25rem; }
  .status-dot {
    display: inline-block;
    width: 8px; height: 8px;
    background: var(--crt-green);
    border-radius: 50%;
    box-shadow: 0 0 8px var(--crt-green);
    animation: blink 1s infinite;
    margin-right: 6px;
  }
  
  /* Controls */
  .controls {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding-bottom: 0;
    flex-wrap: wrap;
  }
  
  .search-wrap {
    display: flex;
    align-items: center;
    flex: 1;
    min-width: 200px;
    background: var(--bg-dark);
    border: 1px solid var(--crt-green);
    padding: 0.5rem 0.75rem;
    gap: 0.5rem;
    box-shadow: inset 0 0 10px rgba(0,255,65,0.05), 0 0 8px rgba(0,255,65,0.1);
  }
  
  .search-icon {
    color: var(--crt-amber);
    font-family: var(--font-mono);
    font-size: 1rem;
    flex-shrink: 0;
  }
  
  .search-input {
    background: transparent;
    border: none;
    outline: none;
    color: var(--crt-green);
    font-family: var(--font-mono);
    font-size: 1rem;
    flex: 1;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    caret-color: transparent;
  }
  
  .search-input::placeholder { color: rgba(0,255,65,0.3); }
  
  .blink-cursor {
    color: var(--crt-green);
    animation: blink 0.8s step-end infinite;
    font-size: 1rem;
  }
  
  .category-tabs {
    display: flex;
    gap: 0;
  }
  
  .cat-btn {
    background: transparent;
    border: 1px solid rgba(0,255,65,0.3);
    color: rgba(0,255,65,0.5);
    font-family: var(--font-mono);
    font-size: 0.7rem;
    letter-spacing: 0.08em;
    padding: 0.5rem 0.75rem;
    cursor: pointer;
    transition: all 0.15s;
    text-transform: uppercase;
    margin-left: -1px;
  }
  
  .cat-btn:hover {
    color: var(--crt-green);
    background: rgba(0,255,65,0.08);
    border-color: var(--crt-green);
  }
  
  .cat-btn.active {
    background: var(--crt-green);
    color: var(--bg-dark);
    border-color: var(--crt-green);
    font-weight: 700;
  }
  
  /* ─── STATUS BAR ─── */
  .status-bar {
    background: var(--crt-green);
    color: var(--bg-dark);
    display: flex;
    align-items: center;
    gap: 2rem;
    padding: 0.25rem 2rem;
    font-family: var(--font-mono);
    font-size: 0.75rem;
    font-weight: 700;
    overflow: hidden;
  }
  
  .status-scroll {
    white-space: nowrap;
    animation: scroll-left 20s linear infinite;
    flex: 1;
  }
  
  @keyframes scroll-left {
    from { transform: translateX(100%); }
    to { transform: translateX(-100%); }
  }
  
  /* ─── LOADING ─── */
  .loading-screen {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .loading-box {
    text-align: center;
    border: 1px solid var(--crt-green);
    padding: 3rem;
    background: var(--bg-panel);
    box-shadow: 0 0 30px rgba(0,255,65,0.2);
  }
  
  .pixel-art {
    font-family: var(--font-mono);
    font-size: 2rem;
    color: var(--crt-amber);
    letter-spacing: 0.2em;
    margin-bottom: 1rem;
  }
  
  .loading-text {
    font-family: var(--font-vt);
    font-size: 1.5rem;
    letter-spacing: 0.2em;
    margin-bottom: 1.5rem;
  }
  
  .dots { animation: blink 0.5s step-end infinite; }
  
  .loading-bar {
    height: 8px;
    background: rgba(0,255,65,0.1);
    border: 1px solid var(--crt-green);
    width: 300px;
    max-width: 100%;
  }
  
  .loading-fill {
    height: 100%;
    background: var(--crt-green);
    box-shadow: 0 0 8px var(--crt-green);
    transition: width 0.2s;
  }
  
  /* ─── ERROR ─── */
  .error-screen {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    color: var(--crt-red);
  }
  
  .pixel-font { font-family: var(--font-pixel); font-size: 0.9rem; }
  
  .retro-btn {
    background: transparent;
    border: 2px solid var(--crt-green);
    color: var(--crt-green);
    padding: 0.75rem 2rem;
    font-family: var(--font-mono);
    font-size: 1rem;
    cursor: pointer;
    letter-spacing: 0.2em;
    transition: all 0.2s;
  }
  .retro-btn:hover {
    background: var(--crt-green);
    color: var(--bg-dark);
  }
  
  /* ─── MOVIE GRID ─── */
  .movie-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 1.5rem;
    padding: 2rem;
    flex: 1;
  }
  
  /* ─── PAGINATION ─── */
  .pagination {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    padding: 2rem;
    border-top: 1px solid var(--border);
  }
  
  .pg-btn {
    background: transparent;
    border: 1px solid var(--crt-green);
    color: var(--crt-green);
    font-family: var(--font-mono);
    font-size: 0.8rem;
    padding: 0.5rem 1rem;
    cursor: pointer;
    letter-spacing: 0.1em;
    transition: all 0.15s;
  }
  .pg-btn:hover:not(:disabled) { background: var(--crt-green); color: var(--bg-dark); }
  .pg-btn:disabled { opacity: 0.3; cursor: default; }
  
  .pg-info { display: flex; align-items: center; gap: 4px; }
  
  .pg-num {
    background: transparent;
    border: 1px solid rgba(0,255,65,0.3);
    color: rgba(0,255,65,0.6);
    font-family: var(--font-mono);
    font-size: 0.8rem;
    width: 36px; height: 36px;
    cursor: pointer;
    transition: all 0.15s;
  }
  .pg-num:hover { background: rgba(0,255,65,0.1); color: var(--crt-green); }
  .pg-num.active { background: var(--crt-green); color: var(--bg-dark); border-color: var(--crt-green); font-weight: 700; }
  
  .pg-dots { color: rgba(0,255,65,0.4); padding: 0 4px; }
  
  /* ─── FOOTER ─── */
  .footer {
    text-align: center;
    padding: 1rem 2rem;
    font-size: 0.65rem;
    color: rgba(0,255,65,0.3);
    border-top: 1px solid var(--border);
    letter-spacing: 0.15em;
  }
  </style>
  