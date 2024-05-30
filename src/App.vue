<template>
  <div class="container">
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit" class="save-button">
        <i class="fas fa-save"></i> Save
      </button>
    </form>
    <ul>
      <li v-for="article in articles" :key="article.id" class="article">
        <strong>{{ article.title }}</strong><br />
        <p class="article-content">{{ article.content }}</p>
        <div class="article-buttons">
          <button @click="edit(article)" class="edit-button">
            <i class="fas fa-edit"></i> Edit
          </button>
          <button @click="remove(article.id)" class="delete-button">
            <i class="fas fa-trash-alt"></i> Delete
          </button>
        </div>
      </li>
    </ul>
    <button @click="load" class="load-button">
      <i class="fas fa-sync-alt"></i> Load
    </button>
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
        const response = await axios.get('http://localhost:3000/article');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/article/${form.id}`
          : 'http://localhost:3000/article';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (form.id) {
          const index = articles.value.findIndex((article) => article.id === response.data.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      if (confirm('Are you sure you want to delete this article?')) {
        try {
          await axios.delete(`http://localhost:3000/article/${id}`);
          articles.value = articles.value.filter(article => article.id !== id);
        } catch (error) {
          console.error('Error deleting article:', error);
        }
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, remove, edit, load };
  },
};
</script>


<style>
  body {
    background-color: #222;
    color: #fff;
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
  }

  .container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
  }

  form input[type="text"],
  form textarea {
    width: 100%;
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #333;
    background-color: #333;
    color: #fff;
    border-radius: 5px;
  }

  .article-buttons {
    margin-top: 20px;
  }

  .article {
    background-color: #333;
    padding: 20px;
    border-radius: 5px;
    margin-bottom: 10px;
  }

  .article strong {
    color: #4CAF50;
  }

  .article-content {
    margin-bottom: 10px;
  }

    /* CSS untuk tombol Save */
    .save-button {
    background-color: #4CAF50; /* Hijau */
  }

  /* CSS untuk tombol Delete */
  .delete-button {
    background-color: #f44336; /* Merah */
  }

  /* CSS untuk tombol Load */
  .load-button {
    background-color: #2196F3; /* Biru */
  }

  /* CSS untuk tombol Edit */
  .edit-button {
    background-color: #FFC107; /* Kuning */
  }

  /* CSS umum untuk semua tombol */
  button {
    padding: 10px 15px;
    margin-right: 10px;
    border: none;
    border-radius: 5px;
    color: #fff;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  button i {
    margin-right: 5px;
  }

  button:hover {
    opacity: 0.8;
  }

</style>
