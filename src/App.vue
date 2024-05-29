<template>
  <div class="article-container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input-field" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea-field"></textarea><br />
      <button type="submit" class="btn-save">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-info">
          <h3 class="article-title">{{ article.title }}</h3>
          <p class="article-content">{{ article.content }}</p>
        </div>
        <div class="article-actions">
          <button @click="edit(article)" class="btn-edit">Edit</button>
          <button @click="deleteArticle(article.id)" class="btn-delete">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="btn-load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';

        const response = await axios[method](url, form);

        const savedArticle = response.data;
        const index = articles.value.findIndex(article => article.id === savedArticle.id);
        if (index !== -1) {
          articles.value.splice(index, 1, savedArticle);
        } else {
          articles.value.push(savedArticle);
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        const url = `http://localhost:3000/articles/${id}`;
        await axios.delete(url);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle };
  }
};
</script>

<style scoped>
.article-container {
  max-width: 800px;
  margin: auto;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); 
}

.form {
  margin-bottom: 20px;
}

.input-field,
.textarea-field {
  width: 100%;
  margin-bottom: 10px;
  padding: 12px;
  font-size: 16px;
  border: 1px solid #ced4da; 
  border-radius: 5px;
}

.input-field:focus,
.textarea-field:focus {
  outline: none;
  border-color: #007bff; 
}

.btn-save,
.btn-edit,
.btn-delete,
.btn-load {
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 12px 24px;
  cursor: pointer;
  border-radius: 5px; 
  transition: background-color 0.3s ease; 
}

.btn-save:hover,
.btn-edit:hover,
.btn-delete:hover,
.btn-load:hover {
  background-color: #0056b3;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-bottom: 20px; 
  padding: 20px; 
  background-color: #fff; 
  transition: box-shadow 0.3s ease; 
}

.article-item:hover {
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); 
}

.article-info {
  margin-bottom: 15px; 
}

.article-title {
  margin-top: 0;
  color: #007bff;
  font-size: 20px; 
}

.article-content {
  color: #555;
}

.article-actions {
  text-align: right;
}

.btn-edit,
.btn-delete {
  margin-left: 10px;
}

.btn-delete {
  background-color: #dc3545;
}

.btn-load {
  margin-top: 20px; 
}
</style>
