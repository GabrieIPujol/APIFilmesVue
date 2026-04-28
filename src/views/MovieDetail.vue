<template>
    <div class="detail-page">
      <!-- Back nav -->
      <nav class="topnav">
        <button class="back-btn" @click="router.back()">
          ◄ VOLTAR AO ARQUIVO
        </button>
        <div class="breadcrumb">
          <span>CINERETRO</span>
          <span class="sep">//</span>
          <span>DETALHES</span>
          <span class="sep">//</span>
          <span class="crumb-title">{{ movie?.title || '...' }}</span>
        </div>
      </nav>
  
      <!-- Loading -->
      <div v-if="loading" class="center-state">
        <div class="terminal-loader">
          <div class="term-line" v-for="(l, i) in termLines" :key="i" :style="{ opacity: i < visibleLines ? 1 : 0 }">
            {{ l }}
          </div>
        </div>
      </div>
  
      <!-- Error -->
      <div v-else-if="error" class="center-state error">
        <p class="err-code">ERR::404</p>
        <p>{{ error }}</p>
        <button class="retro-btn" @click="fetchDetail">RETRY</button>
      </div>
  
      <!-- Content -->
      <div v-else-if="movie" class="detail-content">
        <!-- Hero -->
        <div class="hero" :style="backdropStyle">
          <div class="hero-overlay"></div>
          <div class="hero-content">
            <div class="poster-col">
              <div class="poster-frame">
                <img
                  v-if="movie.poster_path"
                  :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
                  :alt="movie.title"
                  class="hero-poster"
                />
                <div v-else class="no-img">▣<br>SEM IMAGEM</div>
              </div>
              <div class="poster-badge-row">
                <div v-if="movie.vote_average" class="big-score">
                  <span class="score-label">NOTA</span>
                  <span class="score-num">{{ movie.vote_average.toFixed(1) }}</span>
                  <span class="score-max">/10</span>
                </div>
              </div>
            </div>
            <div class="info-col">
              <div class="title-area">
                <div class="title-pre">■ ARQUIVO Nº {{ movie.id }}</div>
                <h1 class="movie-title glow">{{ movie.title }}</h1>
                <div v-if="movie.tagline" class="tagline">"{{ movie.tagline }}"</div>
              </div>
  
              <div class="meta-grid">
                <div class="meta-item">
                  <span class="meta-label">ANO</span>
                  <span class="meta-val">{{ movie.release_date?.substring(0,4) || 'N/D' }}</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">DURAÇÃO</span>
                  <span class="meta-val">{{ runtime }}</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">IDIOMA</span>
                  <span class="meta-val">{{ movie.original_language?.toUpperCase() }}</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">POPULARIDADE</span>
                  <span class="meta-val">{{ Math.round(movie.popularity) }}</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">STATUS</span>
                  <span class="meta-val status-tag">{{ translateStatus(movie.status) }}</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">VOTOS</span>
                  <span class="meta-val">{{ movie.vote_count?.toLocaleString('pt-BR') }}</span>
                </div>
              </div>
  
              <div v-if="movie.genres?.length" class="genres">
                <span v-for="g in movie.genres" :key="g.id" class="genre-tag">{{ g.name.toUpperCase() }}</span>
              </div>
  
              <div class="overview-section">
                <div class="section-label">■ SINOPSE</div>
                <p class="overview">{{ movie.overview || 'Sinopse não disponível.' }}</p>
              </div>
            </div>
          </div>
        </div>
  
        <!-- Details panel -->
        <div class="panels">
          <!-- Cast -->
          <div v-if="cast.length" class="panel">
            <div class="panel-header">
              <span class="panel-icon">◈</span> ELENCO PRINCIPAL
              <span class="panel-count">[{{ cast.length }}]</span>
            </div>
            <div class="cast-list">
              <div v-for="actor in cast" :key="actor.id" class="cast-item">
                <div class="cast-photo-wrap">
                  <img
                    v-if="actor.profile_path"
                    :src="`https://image.tmdb.org/t/p/w185${actor.profile_path}`"
                    :alt="actor.name"
                    class="cast-photo"
                  />
                  <div v-else class="cast-no-photo">{{ initials(actor.name) }}</div>
                </div>
                <div class="cast-info">
                  <div class="cast-name">{{ actor.name }}</div>
                  <div class="cast-role">{{ actor.character }}</div>
                </div>
              </div>
            </div>
          </div>
  
          <!-- Production info -->
          <div class="panel info-panel">
            <div class="panel-header"><span class="panel-icon">◈</span> DADOS DE PRODUÇÃO</div>
            <div class="info-rows">
              <div v-if="movie.budget" class="info-row">
                <span class="irow-label">ORÇAMENTO</span>
                <span class="irow-val">{{ formatMoney(movie.budget) }}</span>
              </div>
              <div v-if="movie.revenue" class="info-row">
                <span class="irow-label">RECEITA</span>
                <span class="irow-val amber">{{ formatMoney(movie.revenue) }}</span>
              </div>
              <div v-if="movie.production_countries?.length" class="info-row">
                <span class="irow-label">PAÍSES</span>
                <span class="irow-val">{{ movie.production_countries.map(c => c.name).join(', ') }}</span>
              </div>
              <div v-if="movie.production_companies?.length" class="info-row">
                <span class="irow-label">PRODUTORAS</span>
                <span class="irow-val">{{ movie.production_companies.slice(0,3).map(c => c.name).join(' / ') }}</span>
              </div>
              <div v-if="movie.spoken_languages?.length" class="info-row">
                <span class="irow-label">IDIOMAS</span>
                <span class="irow-val">{{ movie.spoken_languages.map(l => l.name).join(', ') }}</span>
              </div>
            </div>
  
            <!-- Rating bar -->
            <div class="rating-visual">
              <div class="rv-label">APROVAÇÃO DA AUDIÊNCIA</div>
              <div class="rv-bar-wrap">
                <div class="rv-bar">
                  <div class="rv-fill" :style="{ width: (movie.vote_average * 10) + '%' }"></div>
                </div>
                <span class="rv-pct">{{ Math.round(movie.vote_average * 10) }}%</span>
              </div>
            </div>
          </div>
  
          <!-- Similar movies -->
          <div v-if="similar.length" class="panel similar-panel">
            <div class="panel-header"><span class="panel-icon">◈</span> TÍTULOS SIMILARES</div>
            <div class="similar-grid">
              <div
                v-for="s in similar"
                :key="s.id"
                class="similar-item"
                @click="router.push({ name: 'MovieDetail', params: { id: s.id } })"
              >
                <img
                  v-if="s.poster_path"
                  :src="`https://image.tmdb.org/t/p/w154${s.poster_path}`"
                  :alt="s.title"
                  class="similar-poster"
                />
                <div v-else class="similar-no-img">▣</div>
                <div class="similar-title">{{ s.title }}</div>
                <div class="similar-year">{{ s.release_date?.substring(0,4) }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted, watch } from 'vue'
  import { useRouter, useRoute } from 'vue-router'
  
  const props = defineProps({ id: [String, Number] })
  const router = useRouter()
  const route = useRoute()
  
  const API_KEY = '4e44d9029b1270a757cddc766a1bcb63'
  const BASE = 'https://api.themoviedb.org/3'
  
  const movie = ref(null)
  const cast = ref([])
  const similar = ref([])
  const loading = ref(true)
  const error = ref(null)
  const visibleLines = ref(0)
  
  const termLines = [
    '> INICIALIZANDO CONEXÃO...',
    '> ACESSANDO BANCO DE DADOS...',
    '> LOCALIZANDO REGISTRO...',
    '> DESCRIPTOGRAFANDO DADOS...',
    '> CARREGANDO MÍDIA...',
    '> PRONTO.',
  ]
  
  const backdropStyle = computed(() => {
    if (!movie.value?.backdrop_path) return {}
    return {
      '--backdrop': `url(https://image.tmdb.org/t/p/w1280${movie.value.backdrop_path})`
    }
  })
  
  const runtime = computed(() => {
    if (!movie.value?.runtime) return 'N/D'
    const h = Math.floor(movie.value.runtime / 60)
    const m = movie.value.runtime % 60
    return h > 0 ? `${h}h ${m}min` : `${m}min`
  })
  
  function translateStatus(s) {
    const map = {
      Released: 'LANÇADO', 'In Production': 'EM PRODUÇÃO',
      'Post Production': 'PÓS-PRODUÇÃO', Planned: 'PLANEJADO',
      Rumored: 'RUMOR', Canceled: 'CANCELADO'
    }
    return map[s] || s?.toUpperCase() || 'N/D'
  }
  
  function formatMoney(n) {
    if (!n) return 'N/D'
    return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 }).format(n)
  }
  
  function initials(name) {
    return name?.split(' ').slice(0,2).map(w => w[0]).join('') || '?'
  }
  
  async function fetchDetail() {
    loading.value = true
    error.value = null
    movie.value = null
    visibleLines.value = 0
  
    // Terminal typing effect
    const lineInterval = setInterval(() => {
      if (visibleLines.value < termLines.length) visibleLines.value++
      else clearInterval(lineInterval)
    }, 250)
  
    try {
      const movieId = props.id || route.params.id
      const [movieRes, creditsRes, similarRes] = await Promise.all([
        fetch(`${BASE}/movie/${movieId}?api_key=${API_KEY}&language=pt-BR`),
        fetch(`${BASE}/movie/${movieId}/credits?api_key=${API_KEY}&language=pt-BR`),
        fetch(`${BASE}/movie/${movieId}/similar?api_key=${API_KEY}&language=pt-BR&page=1`)
      ])
      if (!movieRes.ok) throw new Error(`Filme não encontrado (${movieRes.status})`)
      const [m, c, s] = await Promise.all([movieRes.json(), creditsRes.json(), similarRes.json()])
      movie.value = m
      cast.value = c.cast?.slice(0, 12) || []
      similar.value = s.results?.slice(0, 8) || []
    } catch (e) {
      error.value = e.message
    } finally {
      clearInterval(lineInterval)
      visibleLines.value = termLines.length
      loading.value = false
    }
  }
  
  onMounted(fetchDetail)
  watch(() => route.params.id, fetchDetail)
  </script>
  
  <style scoped>
  .detail-page { min-height: 100vh; display: flex; flex-direction: column; }
  
  /* ─── NAV ─── */
  .topnav {
    background: var(--bg-panel);
    border-bottom: 1px solid var(--border);
    padding: 0.75rem 2rem;
    display: flex;
    align-items: center;
    gap: 2rem;
  }
  
  .back-btn {
    background: transparent;
    border: 1px solid var(--crt-green);
    color: var(--crt-green);
    font-family: var(--font-mono);
    font-size: 0.8rem;
    padding: 0.4rem 1rem;
    cursor: pointer;
    letter-spacing: 0.1em;
    transition: all 0.15s;
    white-space: nowrap;
  }
  .back-btn:hover { background: var(--crt-green); color: var(--bg-dark); }
  
  .breadcrumb {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: rgba(0,255,65,0.4);
    letter-spacing: 0.1em;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    overflow: hidden;
  }
  .sep { color: rgba(0,255,65,0.2); }
  .crumb-title {
    color: var(--crt-amber);
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  
  /* ─── CENTER STATES ─── */
  .center-state {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 4rem;
  }
  .center-state.error { flex-direction: column; gap: 1rem; color: var(--crt-red); }
  .err-code { font-family: var(--font-pixel); font-size: 1.5rem; }
  
  .terminal-loader {
    background: var(--bg-panel);
    border: 1px solid var(--crt-green);
    padding: 2rem 3rem;
    box-shadow: 0 0 30px rgba(0,255,65,0.15);
    min-width: 340px;
  }
  
  .term-line {
    font-family: var(--font-mono);
    font-size: 1rem;
    color: var(--crt-green);
    letter-spacing: 0.1em;
    margin-bottom: 0.5rem;
    transition: opacity 0.3s;
  }
  
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
  .retro-btn:hover { background: var(--crt-green); color: var(--bg-dark); }
  
  /* ─── HERO ─── */
  .hero {
    position: relative;
    padding: 3rem 2rem;
    background-color: var(--bg-dark);
    background-image: var(--backdrop, none);
    background-size: cover;
    background-position: center top;
  }
  
  .hero-overlay {
    position: absolute;
    inset: 0;
    background: linear-gradient(
      to right,
      rgba(5,10,5,0.98) 0%,
      rgba(5,10,5,0.85) 50%,
      rgba(5,10,5,0.75) 100%
    );
    backdrop-filter: grayscale(0.4) contrast(1.05);
  }
  
  .hero-content {
    position: relative;
    display: flex;
    gap: 3rem;
    max-width: 1200px;
    margin: 0 auto;
    align-items: flex-start;
  }
  
  /* Poster */
  .poster-col { flex-shrink: 0; }
  
  .poster-frame {
    width: 220px;
    border: 2px solid var(--crt-green);
    box-shadow: 0 0 20px rgba(0,255,65,0.3), 4px 4px 0 rgba(0,255,65,0.15);
    overflow: hidden;
  }
  
  .hero-poster { width: 100%; display: block; filter: contrast(1.05) saturate(0.85); }
  
  .no-img {
    width: 100%;
    aspect-ratio: 2/3;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: var(--bg-card);
    color: rgba(0,255,65,0.2);
    font-family: var(--font-mono);
    font-size: 0.7rem;
    letter-spacing: 0.2em;
    gap: 0.5rem;
  }
  
  .poster-badge-row { margin-top: 0.75rem; }
  
  .big-score {
    background: var(--bg-panel);
    border: 1px solid var(--crt-amber);
    padding: 0.5rem;
    display: flex;
    align-items: baseline;
    gap: 4px;
    justify-content: center;
    box-shadow: 0 0 10px rgba(255,176,0,0.2);
  }
  
  .score-label {
    font-family: var(--font-mono);
    font-size: 0.6rem;
    color: var(--crt-amber);
    letter-spacing: 0.2em;
  }
  .score-num {
    font-family: var(--font-vt);
    font-size: 2rem;
    color: var(--crt-amber);
    line-height: 1;
    text-shadow: 0 0 10px var(--crt-amber);
  }
  .score-max {
    font-family: var(--font-mono);
    font-size: 0.8rem;
    color: rgba(255,176,0,0.5);
  }
  
  /* Info col */
  .info-col { flex: 1; }
  
  .title-pre {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: var(--crt-amber);
    letter-spacing: 0.3em;
    margin-bottom: 0.5rem;
  }
  
  .movie-title {
    font-family: var(--font-display);
    font-size: clamp(2rem, 5vw, 4rem);
    color: var(--crt-green);
    letter-spacing: 0.05em;
    line-height: 1.1;
    margin-bottom: 0.5rem;
  }
  
  .tagline {
    font-family: var(--font-vt);
    font-size: 1.3rem;
    color: rgba(0,255,65,0.5);
    font-style: italic;
    margin-bottom: 1.5rem;
    letter-spacing: 0.05em;
  }
  
  /* Meta grid */
  .meta-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 0.75rem 1.5rem;
    margin-bottom: 1.5rem;
    border: 1px solid var(--border);
    padding: 1rem;
    background: rgba(0,255,65,0.02);
  }
  
  .meta-item { display: flex; flex-direction: column; gap: 2px; }
  
  .meta-label {
    font-family: var(--font-mono);
    font-size: 0.6rem;
    color: rgba(0,255,65,0.4);
    letter-spacing: 0.2em;
  }
  
  .meta-val {
    font-family: var(--font-vt);
    font-size: 1.2rem;
    color: var(--crt-green);
    letter-spacing: 0.05em;
  }
  
  .status-tag { color: var(--crt-amber); }
  
  /* Genres */
  .genres {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    margin-bottom: 1.5rem;
  }
  
  .genre-tag {
    border: 1px solid rgba(0,255,65,0.4);
    color: rgba(0,255,65,0.7);
    font-family: var(--font-mono);
    font-size: 0.65rem;
    padding: 0.25rem 0.75rem;
    letter-spacing: 0.1em;
  }
  
  /* Overview */
  .section-label {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: var(--crt-amber);
    letter-spacing: 0.3em;
    margin-bottom: 0.5rem;
  }
  
  .overview {
    font-family: var(--font-vt);
    font-size: 1.1rem;
    color: rgba(0,255,65,0.7);
    line-height: 1.8;
    max-width: 680px;
    letter-spacing: 0.02em;
  }
  
  /* ─── PANELS ─── */
  .panels {
    display: flex;
    flex-direction: column;
    gap: 0;
    max-width: 1200px;
    margin: 0 auto;
    width: 100%;
    padding: 2rem;
    gap: 2rem;
  }
  
  .panel {
    background: var(--bg-panel);
    border: 1px solid var(--border);
  }
  
  .panel-header {
    font-family: var(--font-mono);
    font-size: 0.8rem;
    letter-spacing: 0.2em;
    color: var(--crt-amber);
    padding: 0.75rem 1.25rem;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
  
  .panel-icon { color: var(--crt-green); }
  .panel-count { color: rgba(0,255,65,0.4); margin-left: auto; }
  
  /* Cast */
  .cast-list {
    display: flex;
    gap: 0;
    overflow-x: auto;
    padding: 1rem;
    gap: 1rem;
    scrollbar-width: thin;
    scrollbar-color: var(--crt-green) var(--bg-dark);
  }
  
  .cast-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.5rem;
    flex-shrink: 0;
    width: 90px;
  }
  
  .cast-photo-wrap {
    width: 70px;
    height: 70px;
    border: 1px solid var(--border);
    overflow: hidden;
    border-radius: 50%;
  }
  
  .cast-photo {
    width: 100%;
    height: 100%;
    object-fit: cover;
    filter: grayscale(0.3) contrast(1.05);
  }
  
  .cast-no-photo {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: var(--bg-card);
    color: rgba(0,255,65,0.4);
    font-family: var(--font-mono);
    font-size: 1.1rem;
  }
  
  .cast-info { text-align: center; }
  
  .cast-name {
    font-family: var(--font-mono);
    font-size: 0.6rem;
    color: var(--crt-green);
    letter-spacing: 0.05em;
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    width: 90px;
  }
  
  .cast-role {
    font-family: var(--font-mono);
    font-size: 0.55rem;
    color: rgba(0,255,65,0.4);
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    width: 90px;
  }
  
  /* Info panel */
  .info-panel { padding-bottom: 1rem; }
  
  .info-rows { padding: 1rem 1.25rem; }
  
  .info-row {
    display: flex;
    align-items: baseline;
    gap: 1rem;
    padding: 0.4rem 0;
    border-bottom: 1px solid rgba(0,255,65,0.05);
    font-family: var(--font-mono);
    font-size: 0.8rem;
  }
  
  .irow-label {
    color: rgba(0,255,65,0.4);
    letter-spacing: 0.15em;
    font-size: 0.65rem;
    min-width: 120px;
    flex-shrink: 0;
  }
  
  .irow-val { color: rgba(0,255,65,0.8); }
  .irow-val.amber { color: var(--crt-amber); }
  
  .rating-visual { padding: 1rem 1.25rem; }
  
  .rv-label {
    font-family: var(--font-mono);
    font-size: 0.65rem;
    color: rgba(0,255,65,0.4);
    letter-spacing: 0.2em;
    margin-bottom: 0.5rem;
  }
  
  .rv-bar-wrap { display: flex; align-items: center; gap: 1rem; }
  
  .rv-bar {
    flex: 1;
    height: 10px;
    background: rgba(0,255,65,0.05);
    border: 1px solid rgba(0,255,65,0.2);
  }
  
  .rv-fill {
    height: 100%;
    background: linear-gradient(to right, var(--crt-green), var(--crt-amber));
    box-shadow: 0 0 6px var(--crt-green);
    transition: width 1s ease;
  }
  
  .rv-pct {
    font-family: var(--font-vt);
    font-size: 1.5rem;
    color: var(--crt-amber);
    min-width: 50px;
    text-align: right;
  }
  
  /* Similar */
  .similar-grid {
    display: flex;
    gap: 1rem;
    overflow-x: auto;
    padding: 1rem;
    scrollbar-width: thin;
    scrollbar-color: var(--crt-green) var(--bg-dark);
  }
  
  .similar-item {
    flex-shrink: 0;
    width: 110px;
    cursor: pointer;
    transition: all 0.2s;
  }
  
  .similar-item:hover { transform: translateY(-4px); }
  .similar-item:hover .similar-title { color: var(--crt-green); }
  
  .similar-poster {
    width: 100%;
    aspect-ratio: 2/3;
    object-fit: cover;
    display: block;
    border: 1px solid var(--border);
    filter: grayscale(0.3) contrast(1.05);
    transition: all 0.2s;
  }
  
  .similar-item:hover .similar-poster {
    border-color: var(--crt-green);
    filter: none;
    box-shadow: 0 0 10px rgba(0,255,65,0.3);
  }
  
  .similar-no-img {
    width: 100%;
    aspect-ratio: 2/3;
    display: flex;
    align-items: center;
    justify-content: center;
    background: var(--bg-card);
    border: 1px solid var(--border);
    color: rgba(0,255,65,0.2);
    font-size: 2rem;
  }
  
  .similar-title {
    font-family: var(--font-mono);
    font-size: 0.6rem;
    color: rgba(0,255,65,0.6);
    margin-top: 0.4rem;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    letter-spacing: 0.05em;
    transition: color 0.2s;
  }
  
  .similar-year {
    font-family: var(--font-mono);
    font-size: 0.55rem;
    color: rgba(0,255,65,0.3);
    letter-spacing: 0.1em;
  }
  
  @media (max-width: 768px) {
    .hero-content { flex-direction: column; align-items: center; }
    .poster-frame { width: 180px; }
    .meta-grid { grid-template-columns: repeat(2, 1fr); }
    .info-col { width: 100%; }
    .movie-title { font-size: 2rem; }
  }
  </style>
  