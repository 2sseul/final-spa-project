<script setup>
import { ref, computed } from 'vue';
import { RouterLink, RouterView, useRouter } from 'vue-router';
import { useMovieStore } from './stores/movieStore';

const store = useMovieStore();
const router = useRouter();

const searchQuery = ref('');

function goSearch() {
  const q = searchQuery.value.trim();
  if (!q) return;
  router.push({ path: '/search', query: { q } });
  searchQuery.value = '';
}

const totalFavoritesCount = computed(() => {
  return store.favorites.length;
});

const averageFavoritesRating = computed(() => {
  if (store.favorites.length === 0) return '0.0';
  const sum = store.favorites.reduce((acc, m) => acc + m.vote_average, 0);
  return (sum / store.favorites.length).toFixed(1);
});
</script>

<template>
  <div class="app-container">
    <header class="main-header">
      <div class="header-content">
        <RouterLink to="/" class="logo-zone">
          <span class="logo-icon">🍿</span>
          <h1 class="logo-text">NETVUE</h1>
        </RouterLink>

        <nav class="nav-menu">
          <RouterLink to="/" class="nav-item">홈</RouterLink>
          <RouterLink to="/movies" class="nav-item">영화 목록</RouterLink>
        </nav>

        <form @submit.prevent="goSearch" class="search-wrap">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="🔍 영화 검색..."
            class="nav-search"
          />
          <button type="submit" class="search-btn">검색</button>
        </form>

        <div class="header-dashboard">
          <RouterLink to="/favorites" class="dashboard-badge favorite-count">
            <span class="badge-label">❤️ 찜한 작품</span>
            <span class="badge-value">{{ totalFavoritesCount }}개</span>
          </RouterLink>
          <div class="dashboard-badge average-rating">
            <span class="badge-label">⭐️ 평균 평점</span>
            <span class="badge-value">{{ averageFavoritesRating }} / 10</span>
          </div>
        </div>
      </div>
    </header>
    <main class="main-content">
      <RouterView />
    </main>
  </div>
</template>

<style scoped>
.app-container { font-family: "Noto Sans KR", sans-serif; background-color: #f8f9fa; min-height: 100vh; display: flex; flex-direction: column; }
.main-header { background-color: #1e272e; color: #ffffff; position: sticky; top: 0; z-index: 1000; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15); padding: 0 40px; }
.header-content { max-width: 1400px; margin: 0 auto; height: 80px; display: flex; align-items: center; justify-content: space-between; gap: 20px; }
.logo-zone { display: flex; align-items: center; gap: 10px; text-decoration: none; color: #ffffff; flex-shrink: 0; }
.logo-icon { font-size: 28px; }
.logo-text { font-size: 22px; font-weight: 900; letter-spacing: -0.5px; background: linear-gradient(45deg, #ff4757, #ff6b81); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.nav-menu { display: flex; gap: 30px; flex-shrink: 0; }
.nav-item { color: #ced6e0; text-decoration: none; font-size: 16px; font-weight: 700; transition: color 0.2s ease; padding: 8px 12px; border-radius: 6px; }
.nav-item:hover { color: #ffffff; background-color: rgba(255, 255, 255, 0.05); }
.router-link-active.nav-item { color: #ff4757; background-color: rgba(255, 87, 87, 0.1); }

.search-wrap { display: flex; gap: 8px; align-items: center; }
.nav-search {
  padding: 9px 16px;
  border-radius: 20px;
  border: none;
  background: #2f3542;
  color: #ffffff;
  font-size: 14px;
  outline: none;
  width: 200px;
  transition: background 0.2s;
}
.nav-search::placeholder { color: #a4b0be; }
.nav-search:focus { background: #3d4557; }
.search-btn {
  padding: 9px 16px;
  border-radius: 20px;
  border: none;
  background: #ff4757;
  color: white;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.2s;
  flex-shrink: 0;
}
.search-btn:hover { background: #e84049; }

.header-dashboard { display: flex; gap: 15px; flex-shrink: 0; }
.dashboard-badge { background-color: #2f3542; padding: 10px 16px; border-radius: 30px; display: flex; align-items: center; gap: 8px; border: 1px solid #3f4656; text-decoration: none; }
.favorite-count { cursor: pointer; transition: border-color 0.2s, background-color 0.2s; }
.favorite-count:hover { border-color: #ff4757; background-color: rgba(255, 71, 87, 0.15); }
.badge-label { font-size: 13px; color: #a4b0be; font-weight: 500; }
.badge-value { font-size: 14px; font-weight: 800; color: #ffffff; }
.average-rating .badge-value { color: #e1b12c; }
.main-content { flex-grow: 1; width: 100%; box-sizing: border-box; }
</style>
