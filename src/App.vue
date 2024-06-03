<template>
     <div class="container mt-5">
    <div class="card">
      <div class="card-header">
        <h3>Article Form</h3>
      </div>
      <div class="card-body">
        <form @submit.prevent="save">
          <div class="form-group">
            <label for="title">Title</label>
            <input type="text" class="form-control" id="title" v-model="form.title" placeholder="Enter title" />
          </div>
          <div class="form-group">
            <label for="content">Content</label>
            <textarea class="form-control" id="content" v-model="form.content" rows="3" placeholder="Enter content"></textarea>
          </div>
          <button type="submit" class="btn btn-primary">Save</button>
        </form>
      </div>
    </div>
    <div class="mt-4">
      <h3>Articles</h3>
      <ul class="list-group">
        <li class="list-group-item" v-for="article in articles" :key="article.id">
          <h5>{{ article.title }}</h5>
          <p>{{ article.content }}</p>
          <button class="btn btn-secondary btn-sm" @click="edit(article)">Edit</button>
          <button class="btn btn-danger btn-sm" @click="del(article.id)">Delete</button>
        </li>
      </ul>
      <button class="btn btn-info mt-3" @click="load">Load Articles</button>
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
          
          if (method === 'post') {
            articles.value.push(response.data);
          } else {
            articles.value = articles.value.map(article =>
              article.id === response.data.id ? response.data : article
            );
          }
  
         
          form.id = null;
          form.title = '';
          form.content = '';
        } catch (error) {
          console.error('Error saving article:', error);
        }
      }
  
      async function del(id) {
        try {
          await axios.delete(`http://localhost:3000/articles/${id}`);
       
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
  
      return { form, articles, save, load, edit, del };
    },
  };
  </script>
  