<template>
    <div class="movie-card" @click="$emit('click')">
      <div class="poster-wrap">
        <img
          v-if="movie.poster_path"
          :src="`https://image.tmdb.org/t/p/w342${movie.poster_path}`"
          :alt="movie.title"
          class="poster"
          loading="lazy"
        />
        <div v-else class="no-poster">
          <span class="film-icon">▣</span>
          <span>SEM IMAGEM</span>
        </div>
        <div class="poster-overlay">
          <span class="view-label">► VER DETALHES</span>
        </div>
        <div class="rating-badge">
          <span class="rating-star">★</span>
          <span>{{ movie.vote_average ? movie.vote_average.toFixed(1) : 'N/A' }}</span>
        </div>
        <div v-if="movie.release_date" class="year-badge">
          {{ movie.release_date.substring(0, 4) }}
        </div>
      </div>
      <div class="card-info">
        <h3 class="movie-title">{{ movie.title }}</h3>
        <div class="movie-meta">
          <span class="votes">{{ movie.vote_count?.toLocaleString('pt-BR') || 0 }} votos</span>
        </div>
      </div>
      <div class="card-scanline"></div>
    </div>
  </template>
  
  <script setup>
  defineProps({ movie: Object })
  defineEmits(['click'])
  </script>
  
  <style scoped>
  .movie-card {
    background: var(--bg-card);
    border: 1px solid var(--border);
    cursor: pointer;
    transition: all 0.2s;
    position: relative;
    overflow: hidden;
  }
  
  .movie-card:hover {
    border-color: var(--crt-green);
    box-shadow: 0 0 20px rgba(0,255,65,0.25), inset 0 0 20px rgba(0,255,65,0.03);
    transform: translateY(-3px);
  }
  
  .movie-card:hover .poster-overlay { opacity: 1; }
  .movie-card:hover .movie-title { color: var(--crt-green); }
  .movie-card:hover .card-scanline { opacity: 1; }
  .movie-card:active { transform: translateY(-1px); animation: glitch 0.1s; }
  
  .poster-wrap {
    position: relative;
    aspect-ratio: 2/3;
    overflow: hidden;
  }
  
  .poster {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
    filter: contrast(1.05) saturate(0.85);
    transition: filter 0.3s;
  }
  
  .movie-card:hover .poster {
    filter: contrast(1.1) saturate(0.6) sepia(0.15);
  }
  
  .no-poster {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    background: #0a120a;
    color: rgba(0,255,65,0.2);
    font-family: var(--font-mono);
    font-size: 0.7rem;
    letter-spacing: 0.2em;
  }
  
  .film-icon { font-size: 3rem; color: rgba(0,255,65,0.15); }
  
  .poster-overlay {
    position: absolute;
    inset: 0;
    background: rgba(0, 255, 65, 0.12);
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.2s;
    backdrop-filter: brightness(0.8);
  }
  
  .view-label {
    font-family: var(--font-mono);
    font-size: 0.85rem;
    color: var(--crt-green);
    letter-spacing: 0.2em;
    text-shadow: 0 0 10px var(--crt-green);
    border: 1px solid var(--crt-green);
    padding: 0.5rem 1rem;
    background: rgba(5, 10, 5, 0.8);
  }
  
  .rating-badge {
    position: absolute;
    top: 8px;
    right: 8px;
    background: rgba(5,10,5,0.9);
    border: 1px solid var(--crt-amber);
    color: var(--crt-amber);
    font-family: var(--font-mono);
    font-size: 0.75rem;
    padding: 2px 6px;
    display: flex;
    align-items: center;
    gap: 3px;
  }
  
  .rating-star { color: var(--crt-amber); }
  
  .year-badge {
    position: absolute;
    top: 8px;
    left: 8px;
    background: rgba(5,10,5,0.9);
    border: 1px solid rgba(0,255,65,0.4);
    color: rgba(0,255,65,0.7);
    font-family: var(--font-mono);
    font-size: 0.7rem;
    padding: 2px 6px;
  }
  
  .card-info {
    padding: 0.75rem;
    border-top: 1px solid var(--border);
  }
  
  .movie-title {
    font-family: var(--font-mono);
    font-size: 0.8rem;
    color: rgba(0,255,65,0.8);
    text-transform: uppercase;
    letter-spacing: 0.05em;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    transition: color 0.2s;
    margin-bottom: 0.3rem;
  }
  
  .movie-meta {
    font-family: var(--font-mono);
    font-size: 0.65rem;
    color: rgba(0,255,65,0.35);
    letter-spacing: 0.05em;
  }
  
  .card-scanline {
    position: absolute;
    inset: 0;
    pointer-events: none;
    opacity: 0;
    background: repeating-linear-gradient(
      0deg, transparent, transparent 3px,
      rgba(0,255,65,0.02) 3px, rgba(0,255,65,0.02) 4px
    );
    transition: opacity 0.2s;
  }
  </style>
  