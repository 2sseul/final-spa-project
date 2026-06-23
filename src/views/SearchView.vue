<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useMovieStore } from '@/stores/movieStore';

const store = useMovieStore();
const route = useRoute();
const router = useRouter();

const inputValue = ref(route.query.q || '');

onMounted(() => {
    if (store.movies.length === 0) store.fetchMovies();
});

watch(() => route.query.q, (q) => {
    inputValue.value = q || '';
    document.title = q ? `🔍 "${q}" 검색 결과` : '🔍 영화 검색';
}, { immediate: true });

const searchResults = computed(() => {
    const q = (route.query.q || '').trim().toLowerCase();
    if (!q) return [];
    return store.movies.filter(m => m.title.toLowerCase().includes(q));
});

function doSearch() {
    const q = inputValue.value.trim();
    if (!q) return;
    router.push({ path: '/search', query: { q } });
}
</script>

<template>
    <main class="page">
        <div class="search-header">
            <h1>🔍 영화 검색</h1>
        </div>

        <form @submit.prevent="doSearch" class="search-box">
            <input
                v-model="inputValue"
                type="text"
                placeholder="영화 제목을 입력하세요..."
                class="search-input"
            />
            <button type="submit" class="search-btn">검색</button>
        </form>

        <div v-if="store.isLoading" class="status loading">⌛️ 데이터 로딩 중...</div>

        <div v-else-if="route.query.q">
            <p class="result-info">
                <strong>"{{ route.query.q }}"</strong> 검색 결과 — 총 <strong>{{ searchResults.length }}</strong>편
            </p>

            <div v-if="searchResults.length === 0" class="no-result">
                😅 검색 결과가 없습니다. 다른 키워드로 찾아보세요!
            </div>

            <div v-else class="movie-list">
                <div v-for="movie in searchResults" :key="movie.id" class="movie-card">
                    <img
                        v-if="movie.poster_path"
                        :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
                        :alt="movie.title"
                        class="poster"
                    />
                    <div v-else class="poster-placeholder">이미지 없음</div>
                    <div class="card-content">
                        <h3 class="title">{{ movie.title }}</h3>
                        <p class="release-date" v-if="movie.release_date">📆 {{ movie.release_date }}</p>
                        <p class="rating">⭐️ {{ movie.vote_average.toFixed(1) }} / 10</p>
                        <p class="overview">
                            {{ movie.overview ? movie.overview.substring(0, 60) + '...' : '줄거리 정보 없음' }}
                        </p>
                        <button
                            @click.prevent="store.toggleFavorite(movie.id)"
                            :class="{ active: movie.isFavorite }"
                            class="fav-btn"
                        >{{ movie.isFavorite ? '❤️ 찜 해제' : '🤍 찜하기' }}</button>
                    </div>
                    <RouterLink
                        :to="`/movies/${movie.id}`"
                        class="stretched-link"
                        :aria-label="`${movie.title} 상세 정보 보기`"
                    ></RouterLink>
                </div>
            </div>
        </div>

        <div v-else class="guide">
            영화 제목의 일부만 입력해도 검색할 수 있습니다 🎬
        </div>
    </main>
</template>

<style scoped>
.page { padding: 40px; min-height: 100vh; background: #f8f9fa; }
.search-header { text-align: center; margin-bottom: 24px; color: #2c3e50; }

.search-box {
    display: flex;
    gap: 10px;
    max-width: 600px;
    margin: 0 auto 36px;
}
.search-input {
    flex: 1;
    padding: 14px 18px;
    border: 2px solid #ddd;
    border-radius: 10px;
    font-size: 15px;
    outline: none;
    transition: border-color 0.2s;
}
.search-input:focus { border-color: #ff4757; }
.search-btn {
    padding: 14px 24px;
    background: #ff4757;
    color: white;
    border: none;
    border-radius: 10px;
    font-size: 15px;
    font-weight: 700;
    cursor: pointer;
    transition: 0.2s;
}
.search-btn:hover { background: #e84049; }

.status { text-align: center; padding: 40px; font-size: 18px; color: #3498db; }
.result-info { font-size: 15px; color: #555; margin-bottom: 24px; }
.no-result { text-align: center; padding: 60px; font-size: 18px; color: #888; }
.guide { text-align: center; padding: 60px; font-size: 16px; color: #aaa; }

.movie-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 28px; }
.movie-card {
    position: relative;
    border-radius: 12px;
    overflow: hidden;
    background: white;
    box-shadow: 0 4px 15px rgba(0,0,0,0.05);
    transition: transform 0.2s;
    display: flex;
    flex-direction: column;
}
.movie-card:hover { transform: translateY(-5px); }
.poster { width: 100%; height: 320px; object-fit: cover; }
.poster-placeholder { width: 100%; height: 320px; background: #ddd; display: flex; align-items: center; justify-content: center; color: #999; }
.card-content { padding: 16px; display: flex; flex-direction: column; flex-grow: 1; }
.title { font-size: 16px; font-weight: bold; color: #333; margin: 0 0 6px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.release-date { font-size: 12px; color: #7f8c8d; margin-bottom: 6px; }
.rating { font-size: 14px; font-weight: bold; color: #f39c12; margin-bottom: 8px; }
.overview { font-size: 12px; color: #555; line-height: 1.4; flex-grow: 1; margin-bottom: 14px; }
.fav-btn {
    position: relative;
    z-index: 2;
    width: 100%;
    padding: 10px;
    border: none;
    border-radius: 8px;
    background: #ecf0f1;
    font-weight: bold;
    font-size: 13px;
    cursor: pointer;
    transition: 0.3s;
}
.fav-btn.active { background: #ff4757; color: white; }
.stretched-link { position: absolute; top: 0; left: 0; right: 0; bottom: 0; z-index: 1; }
</style>
