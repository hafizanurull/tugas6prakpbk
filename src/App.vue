<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <div class="form-group">
        <label for="title">Title</label>
        <input
          type="text"
          id="title"
          v-model="form.title"
          class="form-control"
          placeholder="Enter title"
        />
      </div>
      <div class="form-group">
        <label for="content">Content</label>
        <textarea
          id="content"
          v-model="form.content"
          class="form-control"
          placeholder="Enter content"
        ></textarea>
      </div>
      <button type="submit" class="btn btn-primary">Save</button>
    </form>
    <ul class="articles-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="btn btn-secondary">Edit</button>
        <button @click="remove(article.id)" class="btn btn-danger">Delete</button>
      </li>
    </ul>
    <button @click="load" class="btn btn-info">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    })

    const articles = ref([])

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles')
        articles.value = response.data
      } catch (error) {
        console.error('Error loading articles:', error)
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles'
        const method = form.id ? 'put' : 'post'
        const response = await axios[method](url, form)
        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          )
        } else {
          articles.value.push(response.data)
        }

        form.id = null
        form.title = ''
        form.content = ''
      } catch (error) {
        console.error('Error saving article:', error)
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`)
        articles.value = articles.value.filter((article) => article.id !== id)
      } catch (error) {
        console.error('Error deleting article:', error)
      }
    }

    function edit(article) {
      form.id = article.id
      form.title = article.title
      form.content = article.content
    }

    onMounted(load)

    return { form, articles, save, edit, remove, load }
  }
}
</script>

<style>
.container {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.form {
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
}

.form-control {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
}

.btn {
  padding: 10px 15px;
  margin-right: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-primary {
  background: #007bff;
  color: #fff;
}

.btn-secondary {
  background: #6c757d;
  color: #fff;
}

.btn-danger {
  background: #dc3545;
  color: #fff;
}

.btn-info {
  background: #17a2b8;
  color: #fff;
}

.articles-list {
  list-style: none;
  padding: 0;
}

.article-item {
  background: #fff;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 4px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.article-item h3 {
  margin-top: 0;
}

.article-item p {
  margin-bottom: 10px;
}
</style>
