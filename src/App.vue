<template>
  <div>
    <form @submit.prevent="save" class="form-container">
      <input type="text" v-model="form.title" placeholder="Masukkan judul" /><br />
      <textarea v-model="form.content" placeholder="Masukkan isi konten"></textarea><br />
      <button type="submit">Simpan</button>
    </form>
    <table class="article-table">
      <thead>
        <tr>
          <th>Judul</th>
          <th>Konten</th>
          <th>Aksi</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="article in articles" :key="article.id">
          <td>{{ article.title }}</td>
          <td>{{ article.content }}</td>
          <td>
            <button @click="edit(article)">Edit</button>
            <button @click="remove(article.id)">Hapus</button>
          </td>
        </tr>
      </tbody>
    </table>
    <button @click="load" class="load-button">Muat</button>
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

        if (response.status === 201 || response.status === 200) {
          // Reload articles after saving
          load();
          resetForm();
        }
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        if (!id) {
          console.error('Invalid article ID');
          return;
        }
        const response = await axios.delete(`http://localhost:3000/articles/${id}`);
        if (response.status === 200) {
          articles.value = articles.value.filter((article) => article.id !== id);
        } else {
          console.error('Error deleting article:', response.statusText);
        }
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    function resetForm() {
      form.id = null;
      form.title = '';
      form.content = '';
    }

    onMounted(load);

    return { form, articles, save, edit, remove };
  },
};
</script>

<style scoped>
/* Form Styling */
.form-container {
  margin-bottom: 20px;
  box-shadow:  0px   10px 200px red; /* Added box-shadow */
}

.form-container input[type="text"],
.form-container textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;

}

.form-container button[type="submit"] {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

/* Table Styling */
.article-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
  box-shadow:  0px   10px 200px red; /* Added box-shadow */
}

.article-table th,
.article-table td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: left;
}

.article-table th {
  background-color: #d08009;
}

.article-table td {
  background-color: #201f1f;
}

.article-table button {
  padding: 5px 10px;
  margin-right: 5px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.article-table button:hover {
  background-color: #0056b3;
}

.load-button {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 20px;
}
</style>
