<template>
  <div class="watchlist">
    <nav class="navbar">...</nav>

    <div class="watchlist-container">
      <h1>Смотреть позже</h1>

      <div v-if="watchlist.length === 0" class="empty">
        <p>У вас нет фильмов в списке "Смотреть позже"</p>
        <router-link to="/" class="btn-primary">На главную</router-link>
      </div>

      <div v-else class="watchlist-grid">
        <div v-for="item in watchlist" :key="item.movieId" class="watchlist-card">
          <img :src="item.poster" :alt="item.title" class="card-poster">
          <div class="card-info">
            <h3>{{ item.title }}</h3>
            <div class="card-meta">{{ item.year }} · {{ item.duration }} мин</div>
            <div class="card-actions">
              <button @click="watchNow(item.movieId)" class="btn-watch">Смотреть</button>
              <button @click="removeFromWatchlist(item.movieId)" class="btn-remove">Удалить</button>
            </div>
          </div>
        </div>
      </div>

      <div class="watchlist-actions">
        <button @click="clearAll" class="btn-clear">Очистить все</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { collection, getDocs, deleteDoc, doc, query, where } from 'firebase/firestore'
import { db, auth } from '../firebase/config'
import { onAuthStateChanged } from 'firebase/auth'
import { useRouter } from 'vue-router'

const router = useRouter()
const user = ref(null)
const watchlist = ref([])

onAuthStateChanged(auth, (currentUser) => {
  user.value = currentUser
  if (currentUser) loadWatchlist()
})

const loadWatchlist = async () => {
  const q = query(collection(db, 'watchlist'), where('userId', '==', user.value.uid))
  const snapshot = await getDocs(q)
  watchlist.value = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }))
}

const removeFromWatchlist = async (movieId) => {
  const q = query(collection(db, 'watchlist'), where('userId', '==', user.value.uid), where('movieId', '==', movieId))
  const snapshot = await getDocs(q)
  snapshot.forEach(async (doc) => await deleteDoc(doc.ref))
  await loadWatchlist()
}

const watchNow = (movieId) => router.push(`/movie/${movieId}`)

const clearAll = async () => {
  const q = query(collection(db, 'watchlist'), where('userId', '==', user.value.uid))
  const snapshot = await getDocs(q)
  snapshot.forEach(async (doc) => await deleteDoc(doc.ref))
  await loadWatchlist()
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.watchlist {
  min-height: 100vh;
  background: #0a0a0a;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', sans-serif;
  color: #ffffff;
}

.watchlist-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 80px 24px 48px;
}

.watchlist-container h1 {
  font-size: 32px;
  font-weight: 600;
  margin-bottom: 32px;
  color: #ffffff;
  letter-spacing: -0.5px;
}

/* Empty State */
.empty {
  text-align: center;
  padding: 80px 24px;
  background: #1a1a1a;
  border-radius: 12px;
  border: 1px solid #2a2a2a;
}

.empty p {
  font-size: 18px;
  color: #888;
  margin-bottom: 24px;
}

.btn-primary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 10px 24px;
  background: #e50914;
  color: white;
  text-decoration: none;
  border-radius: 4px;
  font-weight: 500;
  font-size: 14px;
  transition: all 0.2s ease;
  border: none;
  cursor: pointer;
}

.btn-primary:hover {
  background: #f40612;
  transform: translateY(-1px);
}

/* Watchlist Grid */
.watchlist-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
  margin-bottom: 48px;
}

/* Card Styles */
.watchlist-card {
  background: #1a1a1a;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  border: 1px solid #2a2a2a;
  cursor: pointer;
}

.watchlist-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.3);
  border-color: #e50914;
}

.card-poster {
  width: 100%;
  height: 360px;
  object-fit: cover;
  display: block;
}

.card-info {
  padding: 16px;
}

.card-info h3 {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  min-height: 44px;
}

.card-meta {
  font-size: 13px;
  color: #888;
  margin-bottom: 16px;
}

.card-actions {
  display: flex;
  gap: 12px;
}

.btn-watch,
.btn-remove {
  flex: 1;
  padding: 8px 0;
  border: none;
  border-radius: 4px;
  font-weight: 500;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-watch {
  background: #e50914;
  color: white;
}

.btn-watch:hover {
  background: #f40612;
}

.btn-remove {
  background: #2a2a2a;
  color: #e50914;
  border: 1px solid #e50914;
}

.btn-remove:hover {
  background: #e50914;
  color: white;
}

/* Actions Section */
.watchlist-actions {
  text-align: center;
  padding-top: 24px;
  border-top: 1px solid #2a2a2a;
}

.btn-clear {
  padding: 10px 24px;
  background: transparent;
  border: 1px solid #e50914;
  color: #e50914;
  border-radius: 4px;
  font-weight: 500;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-clear:hover {
  background: #e50914;
  color: white;
}

/* Responsive Design */
@media (max-width: 768px) {
  .watchlist-container {
    padding: 70px 16px 32px;
  }

  .watchlist-container h1 {
    font-size: 28px;
    margin-bottom: 24px;
  }

  .watchlist-grid {
    grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
    gap: 16px;
  }

  .card-poster {
    height: 320px;
  }

  .card-info {
    padding: 12px;
  }

  .card-info h3 {
    font-size: 14px;
    min-height: 40px;
  }
}

@media (max-width: 480px) {
  .watchlist-grid {
    grid-template-columns: 1fr;
    gap: 16px;
  }

  .card-poster {
    height: 280px;
  }

  .empty {
    padding: 48px 16px;
  }

  .empty p {
    font-size: 16px;
  }

  .card-actions {
    flex-direction: column;
    gap: 8px;
  }

  .btn-watch,
  .btn-remove {
    width: 100%;
  }
}
</style>