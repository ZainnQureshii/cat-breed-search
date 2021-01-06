<template>
  <div id="app">
    <h2 class="main-heading">Cat Breed Search</h2>
    <hr />
    <div class="search">
      <p>Search here</p>
      <input
        type="text"
        v-model="searchString"
        @input="search(searchString)"
        placeholder="Enter Cat's Breed"
      />
    </div>
    <div v-if="isLoading" class="loader">
      <img src="./assets/images/loader.gif" alt="loader-img" />
    </div>
    <div class="result">
      <div v-if="isCatsFound === true && !isLoading" class="cats-list">
        <h4>Search results for "{{ searchString }}"</h4>
        <table>
          <thead>
            <tr>
              <th>S.no</th>
              <th>Name</th>
              <th>Lifespan</th>
              <th>Temperament</th>
              <th>Origin</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(cat, index) in cats[currentPage]" :key="index">
              <td>{{ index + 1 }}</td>
              <td>{{ cat.name ? cat.name : "-" }}</td>
              <td>{{ cat.life_span ? cat.life_span : "-" }}</td>
              <td>{{ cat.temperament ? cat.temperament : "-" }}</td>
              <td>{{ cat.origin ? cat.origin : "-" }}</td>
            </tr>
          </tbody>
        </table>

        <div class="pagination">
          <ul>
            <li @click="currentPage = 0">&laquo;</li>
            <li
              @click="currentPage = index"
              v-for="(page, index) in totalPages"
              :key="index"
              v-bind:class="{ active: currentPage === index }"
            >
              {{ index + 1 }}
            </li>
            <li @click="currentPage = cats.length - 1">&raquo;</li>
          </ul>
        </div>
      </div>
      <div v-if="isCatsFound === false && !isLoading">No results found.</div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  components: {},
  data() {
    return {
      searchString: "",
      isLoading: false,
      cats: [],
      isCatsFound: 0,
      totalPages: 0,
      currentPage: 0,
      api: {
        baseUrl: "https://api.thecatapi.com/v1/breeds/search?",
        q: "",
      },
    };
  },
  methods: {
    search(searchParams) {
      if (searchParams.length > 0) {
        this.isLoading = true;
        let apiSearchString = searchParams.split(" ");
        this.api.q = apiSearchString.join("+");
        const { baseUrl, q } = this.api;
        const apiUrl = `${baseUrl}q=${q}`;
        this.getData(apiUrl);
      } else {
        this.cats = [];
        this.isCatsFound = "";
      }
    },
    async getData(apiUrl) {
      setTimeout(async () => {
        try {
          const response = await axios.get(apiUrl);
          this.cats = response.data;
          if (this.cats.length > 0) {
            this.cats = this.createPagination(this.cats);
            this.isCatsFound = true;
            this.currentPage = 0;
          } else this.isCatsFound = false;
          this.isLoading = false;
        } catch (e) {
          console.log(e);
        }
      }, 300);
    },
    createPagination(cats) {
      let paginationLength = cats.length / 8;
      let paginatedCats = [];
      let count = 0;
      for (let i = 0; i < paginationLength; i++) {
        let batch = cats.slice(count, count + 8);
        paginatedCats.push(batch);
        count += 8;
      }
      this.totalPages = paginatedCats.length;
      return paginatedCats;
    },
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
.main-heading {
  margin: 20px 0 15px;
  text-align: center;
}
.search {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.search input {
  margin-left: 10px;
  padding: 5px 8px;
  font-size: 15px;
  border-radius: 5px;
}
.loader img {
  max-width: 70px;
  display: block;
  margin: 0 auto;
  margin-top: 40px;
}
.result {
  margin-top: 20px;
  padding: 20px;
}
.result h4 {
  margin-bottom: 20px;
}
.result table {
  width: 100%;
}
.result table tr:nth-child(even) {
  background-color: #f1f1f1;
}
.result table tr td,
.result table tr th {
  padding: 8px 5px;
  font-size: 14px;
}
.result table tr td:first-child {
  text-align: center;
}
.result .pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.result .pagination ul {
  list-style: none;
  display: flex;
}
.result .pagination ul li {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 11px 15px 7px;
  cursor: pointer;
  font-weight: 500;
  box-shadow: 1px 2px 5px 0px rgba(0, 0, 0, 0.3);
}
.result .pagination ul li.active {
  background: #000;
  color: #fff;
}
</style>
