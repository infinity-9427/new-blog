<template>
  <div class="news-section container-fluid mt-5">
    <h1 class="mb-4 text-center">{{ titleText }}</h1>
    <div v-if="error" class="alert alert-danger">Error loading news: {{ error }}</div>
    <div class="row row-cols-1 row-cols-sm-2 row-cols-md-3 row-cols-lg-4 g-4">
      <div class="col" v-for="(article, index) in finalArticles" :key="index">
        <div class="card h-100 shadow-sm card-hover">
          <NuxtImg :src="article.imageUrl" class="card-img-top article-image" alt="Article Image" />
          <div class="card-body d-flex flex-column">
            <h5 class="card-title">{{ article.title }}</h5>
            <p class="card-text flex-grow-1">{{ article.description }}</p>
            <div class="d-flex align-items-center mt-2">
              <NuxtImg :src="article?.author?.avatar" class="author-avatar rounded-circle" alt="Author Avatar" />
              <p class="card-text mb-0 ms-2">Author: {{ article?.author?.name }}</p>
            </div>
            <a :href="article.url" target="_blank" class="btn btn-primary mt-3">Read More</a>
          </div>
        </div>
      </div>
    </div>
    <nav aria-label="Page navigation example" class="mt-4">
      <ul class="pagination justify-content-center">
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <a href="#" class="page-link" @click.prevent="prevPage">Previous</a>
        </li>
        <li v-for="page in visiblePages" :key="page" class="page-item" :class="{ active: page === currentPage }">
          <a href="#" class="page-link" @click.prevent="goToPage(page)">{{ page }}</a>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === pagination.maxPages }">
          <a href="#" class="page-link" @click.prevent="nextPage">Next</a>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, reactive, onMounted, watch } from 'vue'
import { useRoute } from 'vue-router'

interface Author {
  id: number
  name: string
  email: string
  avatar: string
}

export interface Root {
  author: Author
  description: string
  imageUrl: string
  publishedAt: string
  title: string
  url: string
}

const route = useRoute()
const query = ref(route.query.q || 'Bogota')
const currentPage = ref(1)
const articles = ref<Root[]>([])
const error = ref<string | null>(null)

const pagination = reactive({
  page: 1,
  perPage: 10,
  totalResults: 0,
  maxPages: 1
})

const titleText = computed(() =>
  route.query.q ? `Results for: ${route.query.q}` : 'Latest News'
)

async function fetchNews() {
  error.value = null
  try {
    const res = await fetch(
      `https://news-api-pagination.onrender.com/api/v1/news/search?query=${encodeURIComponent(
        query.value
      )}&page=${currentPage.value}`
    )
    if (!res.ok) throw new Error('Failed to fetch news')
    const data = await res.json()
    articles.value = data.articles || []
    pagination.page = data.page
    pagination.perPage = data.perPage
    pagination.totalResults = data.totalResults
    pagination.maxPages = data.maxPages
  } catch (err: any) {
    error.value = err.message || 'An error occurred'
  }
}

onMounted(() => fetchNews())

watch(() => route.query.q, (newQuery) => {
  query.value = newQuery || 'Bogota'
  currentPage.value = 1
})

watch([query, currentPage], () => fetchNews())

const finalArticles = computed(() =>
  articles.value.map(item => ({
    ...item,
    imageUrl: item.imageUrl || '/no-image.webp'
  }))
)

const visiblePages = computed(() => {
  const delta = 2
  const start = Math.max(1, currentPage.value - delta)
  const end = Math.min(pagination.maxPages, currentPage.value + delta)
  const pages = []
  for (let i = start; i <= end; i++) pages.push(i)
  return pages
})

function goToPage(page: number) {
  if (page !== currentPage.value) currentPage.value = page
}

function prevPage() {
  if (currentPage.value > 1) currentPage.value--
}

function nextPage() {
  if (currentPage.value < pagination.maxPages) currentPage.value++
}
</script>

<style scoped>
.article-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-top-left-radius: 0.25rem;
  border-top-right-radius: 0.25rem;
}
@media (max-width: 576px) {
  .article-image { height: 150px; }
}
@media (min-width: 1200px) {
  .article-image { height: 250px; }
}
.card {
  border: none;
  border-radius: 0.5rem;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.card-hover:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.12);
}
.author-avatar {
  width: 40px;
  height: 40px;
  object-fit: cover;
}
.pagination .page-item.active .page-link {
  background-color: #0d6efd;
  border-color: #0d6efd;
}
.pagination .page-item.disabled .page-link {
  cursor: not-allowed;
}
</style>
