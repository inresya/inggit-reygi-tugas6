<template>
  <div style="margin: 20px; background-color: #f9f9f9; padding: 20px; border-radius: 8px;">
    <form @submit.prevent="save" style="margin-bottom: 20px;">
      <div style="margin-bottom: 10px;">
        <label for="title">Title:</label><br />
        <input id="title" type="text" v-model="form.title" style="width: 100%; padding: 8px;" placeholder="Enter title" /><br />
      </div>
      <div style="margin-bottom: 10px;">
        <label for="content">Content:</label><br />
        <textarea id="content" v-model="form.content" style="width: 100%; padding: 8px;" placeholder="Enter content"></textarea><br />
      </div>
      <button type="submit" style="padding: 10px 20px;">Save</button>
      <button @click.prevent="clearForm" style="padding: 10px 20px; margin-left: 10px;">Clear</button>
    </form>
    <button @click="load" style="padding: 10px 20px; margin-bottom: 20px;">Load Articles</button>
    <ul style="list-style: none; padding: 0;">
      <li v-for="article in articles.slice(0, 3)" :key="article.id" style="margin-bottom: 20px; border-bottom: 1px solid #ccc; padding-bottom: 10px;">
        <strong>{{ article.title }}</strong><br />
        <p>{{ article.content }}</p>
        <button @click="edit(article)" style="padding: 5px 10px; margin-right: 10px;">Edit</button>
        <button @click="remove(article.id)" style="padding: 5px 10px;">Delete</button>
      </li>
    </ul>
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
        const response = await axios.get('http://localhost:5173/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:5173/articles/${form.id}`
          : 'http://localhost:5173/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        clearForm();
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:5173/articles/${id}`);
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

    function clearForm() {
      form.id = null;
      form.title = '';
      form.content = '';
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load, clearForm };
  },
};
</script>

<style scoped>
body {
  background-color: #e0f7fa;
}
</style>
