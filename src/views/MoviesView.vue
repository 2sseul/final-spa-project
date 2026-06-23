<script setup>
    import { ref, computed, onMounted } from 'vue';
    import { useMovieStore } from '@/stores/movieStore';

    const store = useMovieStore();

    const sortBy = ref(null);
    const sortDir = ref('asc');
    const currentPage = ref(1);
    const perPage = 10;

    onMounted(() => {
        document.title = '🍿 국내 극장 화제작 (인기순)';
        if (store.movies.length === 0) {
            store.fetchMovies();
        }
    });

    const sortedMovies = computed(() => {
        if (!sortBy.value) return [...store.movies];

        const list = [...store.movies];
        const dir = sortDir.value === 'asc' ? 1 : -1;

        if (sortBy.value === 'title') {
            list.sort((a, b) => dir * a.title.localeCompare(b.title, 'ko'));
        } else if (sortBy.value === 'release_date') {
            list.sort((a, b) => dir * (new Date(a.release_date) - new Date(b.release_date)));
        } else if (sortBy.value === 'rating') {
            list.sort((a, b) => dir * (a.vote_average - b.vote_average));
        }
        return list;
    });

    const totalPages = computed(() => Math.ceil(sortedMovies.value.length / perPage));

    const pagedMovies = computed(() => {
        const start = (currentPage.value - 1) * perPage;
        return sortedMovies.value.slice(start, start + perPage);
    });

    function handleSort(type) {
        if (sortBy.value === type) {
            if (sortDir.value === 'asc') {
                sortDir.value = 'desc';
            } else {
                sortBy.value = null;
                sortDir.value = 'asc';
            }
        } else {
            sortBy.value = type;
            sortDir.value = 'asc';
        }
        currentPage.value = 1;
    }

    function goToPage(p) {
        currentPage.value = p;
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    function dirIcon(type) {
        if (sortBy.value !== type) return '';
        return sortDir.value === 'asc' ? ' ▲' : ' ▼';
    }
</script>

<template>
    <main class="page">
        <div class="header-section">
            <h1>🍿 국내 극장 화제작 (인기순)</h1>
            <p class="sub-title">2025년 이후 국내 정식 개봉한 실시간 인기 상영작</p>
        </div>

        <div v-if="store.isLoading" class="status-message loading">⌛️ 실시간 국내 개봉작 데이터를 싣고 오는 중입니다 ...</div>

        <div v-else-if="store.errorMessage" class="status-message error">🚨 {{ store.errorMessage }}</div>

        <div v-else>
            <div class="sort-bar">
                <span class="sort-label">정렬 :</span>
                <button
                    @click="handleSort('title')"
                    :class="['sort-btn', { active: sortBy === 'title' }]"
                >제목 순{{ dirIcon('title') }}</button>
                <button
                    @click="handleSort('release_date')"
                    :class="['sort-btn', { active: sortBy === 'release_date' }]"
                >개봉일 순{{ dirIcon('release_date') }}</button>
                <button
                    @click="handleSort('rating')"
                    :class="['sort-btn', { active: sortBy === 'rating' }]"
                >평점 순{{ dirIcon('rating') }}</button>
            </div>

            <div class="movie-list">
                <div v-for="movie in pagedMovies" :key="movie.id" class="movie-card">
                    <img
                        v-if="movie.poster_path"
                        :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
                        :alt="movie.title"
                        class="poster"
                    />
                    <div v-else class="poster-placeholder">이미지 준비 중</div>
                    <div class="card-content">
                        <h3 class="title">{{ movie.title }}</h3>
                        <p class="release-date" v-if="movie.release_date">📆 개봉일: {{ movie.release_date }}</p>
                        <p class="rating">⭐️ {{ movie.vote_average.toFixed(1) }} / 10</p>
                        <p class="overview">
                            {{ movie.overview ? movie.overview.substring(0, 60) + '...' : '국내에 등록된 줄거리 요약 정보가 없습니다.'}}</p>
                        <button
                            @click.prevent="store.toggleFavorite(movie.id)"
                            :class="{active: movie.isFavorite}"
                            class="fav-btn">{{ movie.isFavorite ? '❤️ 찜 해제' : '🤍 찜하기' }}</button>
                    </div>
                    <RouterLink
                        :to="`/movies/${movie.id}`"
                        class="stretched-link"
                        :aria-label="`${movie.title} 상세 정보 보기`"></RouterLink>
                </div>
            </div>

            <div class="pagination" v-if="totalPages > 1">
                <button
                    @click="goToPage(currentPage - 1)"
                    :disabled="currentPage === 1"
                    class="page-btn arrow"
                >이전</button>
                <button
                    v-for="p in totalPages"
                    :key="p"
                    @click="goToPage(p)"
                    :class="['page-btn', { active: currentPage === p }]"
                >{{ p }}</button>
                <button
                    @click="goToPage(currentPage + 1)"
                    :disabled="currentPage === totalPages"
                    class="page-btn arrow"
                >다음</button>
            </div>
        </div>
    </main>
</template>

<style scoped>
.page { padding: 40px; background-color: #f8f9fa; min-height: 100vh; }
.header-section { margin-bottom: 20px; text-align: center; color: #2c3e50; }
.sub-title { font-size: 14px; color: #7f8c8d; margin-top: 5px; }
.status-message { text-align: center; font-size: 20px; font-weight: bold; padding: 50px; border-radius: 12px; }
.loading { color: #3498db; background-color: #e3f2fd; }
.error { color: #e74c3c; background-color: #fdeaea; }

.sort-bar {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 28px;
}
.sort-label { font-size: 14px; color: #555; font-weight: 600; }
.sort-btn {
    padding: 7px 18px;
    border: 2px solid #ddd;
    border-radius: 20px;
    background: white;
    cursor: pointer;
    font-size: 13px;
    font-weight: 600;
    color: #555;
    transition: 0.2s;
}
.sort-btn:hover { border-color: #ff4757; color: #ff4757; }
.sort-btn.active { background: #ff4757; border-color: #ff4757; color: white; }

.movie-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 30px; }

.movie-card {
    position: relative;
    border-radius: 12px;
    overflow: hidden;
    background: white;
    text-align: left;
    box-shadow: 0 4px 15px rgba(0,0,0,0.05);
    transition: transform 0.2s ease;
    display: flex;
    flex-direction: column;
}
.movie-card:hover { transform: translateY(-5px); }
.poster { width: 100%; height: 380px; object-fit: cover; }
.poster-placeholder { width: 100%; height: 380px; background-color: #ddd; display: flex; align-items: center; justify-content: center; color: #7f8c8d; font-weight: bold; }
.card-content { padding: 20px; display: flex; flex-direction: column; flex-grow: 1; }
.title { font-size: 18px; color: #333; margin: 0 0 6px 0; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; font-weight: bold; }
.release-date { font-size: 13px; color: #7f8c8d; margin-bottom: 10px; font-weight: 500; }
.rating { font-weight: bold; color: #f39c12; margin-bottom: 10px; font-size: 16px; }
.overview { font-size: 13px; color: #555; line-height: 1.4; margin-bottom: 20px; flex-grow: 1; }

.fav-btn {
    position: relative;
    z-index: 2;
    width: 100%;
    padding: 12px;
    cursor: pointer;
    border: none;
    background: #ecf0f1;
    color: #333;
    border-radius: 8px;
    font-weight: bold;
    font-size: 14px;
    transition: 0.3s;
    margin-top: auto;
}
.fav-btn.active { background: #ff4757; color: white; }

.stretched-link {
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    z-index: 1;
}

.pagination {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 8px;
    margin-top: 50px;
}
.page-btn {
    min-width: 40px;
    height: 40px;
    border: 2px solid #ddd;
    background: white;
    border-radius: 8px;
    font-size: 14px;
    font-weight: 700;
    cursor: pointer;
    transition: 0.2s;
    color: #555;
    padding: 0 12px;
}
.page-btn:hover:not(:disabled) { border-color: #ff4757; color: #ff4757; }
.page-btn.active { background: #ff4757; border-color: #ff4757; color: white; }
.page-btn:disabled { opacity: 0.3; cursor: not-allowed; }
.page-btn.arrow { font-size: 13px; }
</style>
