<template>
  <div>
    <nav class="navbar sticky-top navbar-expand-lg custom-navbar">
      <div class="container d-flex justify-content-between align-items-center">
        <NuxtLink class="navbar-brand text-white" to="/">News Portal</NuxtLink>
        <form class="d-flex search-form" @submit.prevent="onSearch">
          <input
            class="form-control me-2"
            type="search"
            placeholder="Search news"
            aria-label="Search"
            v-model="searchQuery"
          />
          <button class="btn btn-custom" type="submit">Search</button>
        </form>
      </div>
    </nav>
    <main>
      <slot />
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const searchQuery = ref('')
const router = useRouter()

function onSearch() {
  const query = searchQuery.value.trim()
  if (query) {
    router.push({ name: 'index', query: { q: query } })
  }
}
</script>

<style scoped>
.custom-navbar {
  position: sticky;
  top: 0;
  z-index: 1030;
  background: linear-gradient(90deg, #6200ea 0%, #673ab7 100%);
  padding: 0.5rem 0;
}

.navbar-brand {
  font-size: 1.8rem;
  font-weight: 700;
}

.search-form {
  max-width: 400px;
  width: 100%;
}

.form-control {
  border-radius: 20px;
  font-size: 1rem;
}

.btn-custom {
  background-color: #fff;
  color: #6200ea;
  border-radius: 20px;
  font-weight: 600;
  border: none;
  transition: background-color 0.2s ease;
}

.btn-custom:hover {
  background-color: #f1f1f1;
}

@media (max-width: 576px) {
  .container {
    flex-direction: column;
    align-items: stretch;
  }
  .navbar-brand {
    text-align: center;
    margin-bottom: 0.5rem;
  }
  .search-form {
    max-width: 100%;
  }
}
</style>
