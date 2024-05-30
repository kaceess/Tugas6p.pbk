<template>
  <div>
    <div>
      <h2>Articles</h2>
      <ul>
        <li v-for="article in articles" :key="article.id">
          <h3>{{ article.title }}</h3>
          <p>{{ article.content }}</p>
          <button @click="edit(article)" style="background-color: pink; color: white; margin-right: 5px;">Edit</button>
          <button @click="() => remove(article.id)" style="background-color: pink; color: white;">Delete</button>
        </li>
      </ul>
    </div>
    <div>
      <h2>Add/Edit Article</h2>
      <form @submit.prevent="save">
        <label for="title">Title:</label>
        <input type="text" id="title" v-model="form.title">
        <label for="content">Content:</label>
        <textarea id="content" v-model="form.content"></textarea>
        <button type="submit" style="background-color: pink; color: white;">Save</button>
      </form>
    </div>
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
      content: ''
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get("http://localhost:3000/articles");
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

        // Assuming successful save, update UI and reset form
        articles.value = articles.value.map(article => 
          article.id === response.data.id ? response.data : article
        );
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    onMounted(load);

    return { form, articles, save, edit, remove };
  }
};
</script>

<style>
/* Tambahan CSS untuk mempercantik tampilan */
ul {
  list-style-type: none;
  padding: 0;
}

li {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
}

button {
  cursor: pointer;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  font-weight: bold;
  margin-top: 5px;
}
</style>
