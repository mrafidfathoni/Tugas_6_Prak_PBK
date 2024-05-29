<template>
  <div class="container">
    <!-- Form -->
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>

    <!-- List of Articles -->
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h2>{{ article.title }}</h2>
        <p>{{ article.content }}</p>
        <button @click="edit(article)">Edit</button>
        <button @click="deleteArticle(article.id)">Delete</button>
      </li>
    </ul>

    <!-- Load Button -->
    <button @click="load" class="load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: '',
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
        const response = await axios[method](url, { title: form.title, content: form.content });

        // Assuming successful save, update UI and reset form
        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        // Reset form
        form.id = '';
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        // Hapus artikel dari array lokal terlebih dahulu
        articles.value = articles.value.filter((article) => article.id !== id);
        // Kemudian kirim permintaan hapus ke server
        await axios.delete(`http://localhost:3000/articles/${id}`);
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

    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.form {
  margin-bottom: 20px;
  background: linear-gradient(135deg, #f6f8fa 0%, #e9ebed 100%);
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

input, textarea {
  display: block;
  margin-bottom: 10px;
  width: 100%;
  padding: 12px;
  box-sizing: border-box;
  border: 1px solid #ddd;
  border-radius: 4px;
  color: #19f6f6;
  transition: border-color 0.3s ease;
}

input:focus, textarea:focus {
  border-color: #007BFF;
  outline: none;
}

button {
  margin-right: 10px;
  padding: 10px 20px;
  cursor: pointer;
  background-color: #007BFF;
  color: #fff;
  border: none;
  border-radius: 4px;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

button:hover {
  background-color: #0056b3;
  transform: translateY(-2px);
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  margin-bottom: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: linear-gradient(135deg, #ffffff 0%, #f9f9f9 100%);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.article-item:hover {
  box-shadow: 0 6px 10px rgba(0, 0, 0, 0.15);
}

h2 {
  margin-top: 0;
  font-size: 18px;
  color: #007BFF;
}

p {
  margin: 10px 0;
  color: #1ee442;
}
</style>
