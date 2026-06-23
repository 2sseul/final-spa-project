<script setup>
import { onMounted } from 'vue';
import { useMovieStore } from '@/stores/movieStore';

const store = useMovieStore();

onMounted(() => {
    document.title = '❤️ 나의 찜 목록';
});
</script>

<template>
    <main class="page">
        <div class="header-section">
            <h1>❤️ 나의 찜 목록</h1>
            <p class="sub">내가 찜한 영화들을 한눈에 모아봐요</p>
        </div>

        <div v-if="store.favorites.length === 0" class="empty">
            <p>😢 아직 찜한 영화가 없어요.</p>
            <p class="empty-sub">영화 목록에서 마음에 드는 작품을 찜해 보세요!</p>
            <RouterLink to="/movies" class="go-btn">영화 목록 보러 가기 →</RouterLink>
        </div>

        <div v-else>
            <p class="count-info">총 <strong>{{ store.favorites.length }}편</strong>을 찜했습니다.</p>
            <div class="movie-list">
                <div v-for="movie in store.favorites" :key="movie.id" class="movie-card">
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
                            class="fav-btn active"
                        >❤️ 찜 해제</button>
                    </div>
                    <RouterLink
                        :to="`/movies/${movie.id}`"
                        class="stretched-link"
                        :aria-label="`${movie.title} 상세 정보 보기`"
                    ></RouterLink>
                </div>
            </div>
        </div>
    </main>
</template>

<style scoped>
.page { padding: 40px; min-height: 100vh; background: #f8f9fa; }
.header-section { text-align: center; margin-bottom: 30px; color: #2c3e50; }
.sub { font-size: 14px; color: #7f8c8d; margin-top: 6px; }

.empty {
    text-align: center;
    padding: 80px 20px;
    color: #888;
    font-size: 18px;
}
.empty-sub { font-size: 14px; margin-top: 10px; color: #aaa; }
.go-btn {
    display: inline-block;
    margin-top: 24px;
    padding: 12px 28px;
    background: #ff4757;
    color: white;
    text-decoration: none;
    border-radius: 10px;
    font-weight: 700;
    font-size: 15px;
    transition: 0.2s;
}
.go-btn:hover { background: #e84049; }

.count-info { font-size: 15px; color: #555; margin-bottom: 24px; }

.movie-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 30px; }
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
.poster { width: 100%; height: 360px; object-fit: cover; }
.poster-placeholder { width: 100%; height: 360px; background: #ddd; display: flex; align-items: center; justify-content: center; color: #999; }
.card-content { padding: 18px; display: flex; flex-direction: column; flex-grow: 1; }
.title { font-size: 17px; font-weight: bold; color: #333; margin: 0 0 6px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.release-date { font-size: 13px; color: #7f8c8d; margin-bottom: 8px; }
.rating { font-size: 15px; font-weight: bold; color: #f39c12; margin-bottom: 8px; }
.overview { font-size: 13px; color: #555; line-height: 1.4; flex-grow: 1; margin-bottom: 16px; }
.fav-btn {
    position: relative;
    z-index: 2;
    width: 100%;
    padding: 11px;
    border: none;
    border-radius: 8px;
    background: #ecf0f1;
    font-weight: bold;
    font-size: 14px;
    cursor: pointer;
    transition: 0.3s;
}
.fav-btn.active { background: #ff4757; color: white; }
.stretched-link { position: absolute; top: 0; left: 0; right: 0; bottom: 0; z-index: 1; }
</style>
