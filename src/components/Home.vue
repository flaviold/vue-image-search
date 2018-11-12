<template>
  <div>
  <div class="flex items-center justify-center bg-gradient h-64">
    <div class="text-white text-center w-full">
      <div class="mb-5 text-5xl">Image Search</div>
      <div><input @keyup.enter="search" class=" w-1/2 p-4 rounded-l" v-model="searchQuery" type="text" placeholder="Search for images..."><button class="p-4 bg-grey-lighter rounded-r" @click="search"><i class="fas fa-search text-grey"></i></button></div>
    </div>
  </div>
  <div class="flex flex-wrap container mt-4 mx-auto">
    <div v-for="image in images" :key="image.id" class="w-full sm:w-1/3 md:w-1/4 lg:w-1/5 xl:w-1/5">
      <div class="mx-1 mb-2">
        <div><img class="" :src="image.urls.regular" alt=""></div>
        <div class="ml-1 text-sm md:text-base">{{ image.user.name }}</div>
        <div class="ml-1 mb-3 sm:text-xs text-sm text-grey">{{ image.likes }} Likes</div>
      </div>
    </div>
  </div>
  <div v-if="showPagination" class="flex justify-center container my-4 mx-auto">
    <div>
      <button @click="pageChange(1)" :class="{ 'pg-btn-disabled': currentPage === 1 }" :disabled="currentPage === 1" class="mr-2 pg-btn pg-btn-blue border-b-4">
        <i class="m-auto fas fa-angle-double-left"></i>
      </button>
      <button @click="pageChange(currentPage - 1)" :class="{ 'pg-btn-disabled': currentPage === 1 }" :disabled="currentPage === 1" class="mr-2 pg-btn pg-btn-blue border-b-4">
        <i class="m-auto fas fa-angle-left"></i>
      </button>
      <div class="mr-2 py-2 bg-blue text-lg text-white pages border-b-4 border-blue-dark rounded">
        <span v-for="page in showingPages" :key="page">
          <button :class="{ 'pg-active': isActive(page) }" @click="pageChange(page)" v-if="typeof(page) === 'number'" class="pg-btn pg-btn-blue">{{ page }}</button><span v-if="typeof(page) !== 'number'" class="pagination-spacer">...</span>
        </span>
      </div>
      <button @click="pageChange(currentPage + 1)" :class="{ 'pg-btn-disabled': currentPage === maxPages }" :disabled="currentPage === maxPages" class="mr-2 pg-btn pg-btn-blue border-b-4">
        <i class="m-auto fas fa-angle-right"></i>
      </button>
      <button @click="pageChange(maxPages)" :class="{ 'pg-btn-disabled': currentPage === maxPages }" :disabled="currentPage === maxPages" class="pg-btn pg-btn-blue border-b-4">
        <i class="m-auto fas fa-angle-double-right"></i>
      </button>
    </div>
  </div>
  </div>
</template>

<script>
import axios from 'axios'
import key from '../assets/js/key.js'

export default {
  data() { 
    return {
      images: [],
      searchQuery: '',
      showingPages: [1,2,3,4,5,6,7,8,9,10],
      showPagination: false,
      currentPage: 4,
      maxPages: 13
    };
  },
  methods: {
    search() {
      this.$router.push({name: '', query: { query: this.searchQuery, page: 1 } });
    },
    pageChange(page) {
      this.$router.push({name: '', query: { query: this.searchQuery, page: page } });
    },
    isActive(page) {
      return this.currentPage === page;
    },
    load(query, page) {
      this.images = [];
      page = page || 1;
      axios('https://api.unsplash.com/search/photos?client_id='+ key +'&query='+ query +'&page='+ page)
        .then(res => {
          console.log(res)
          this.maxPages = res.data['total_pages']
          this.images = res.data.results;
          this.createPagination();
        });
    },
    createPagination() {
      this.showPagination = this.maxPages !== 1;

      if (!this.showPagination) { return; }

      this.showingPages = [];

      if (this.maxPages <= 12) {
        for (let i = 1; i <= this.maxPages; i++) {
          this.showingPages.push(i);
        }
        return;
      }

      let half = Math.floor(this.maxPages/2);
      this.showingPages = [1, 2, 3];
      
      if (this.currentPage === 4) { this.showingPages.push(4); }
      this.showingPages.push('');

      let middleSection = [];
      if (this.currentPage > 4 && this.currentPage < this.maxPages - 3) {
        middleSection = [this.currentPage - 1, this.currentPage, this.currentPage + 1];
      } else {
        middleSection = [half - 1, half, half + 1];
      }
      
      middleSection.forEach(item => {
        this.showingPages.push(item);
      });
      this.showingPages.push('');

      if (this.currentPage === this.maxPages - 3) { this.showingPages.push(this.maxPages - 3); }
      this.showingPages.push(this.maxPages - 2)
      this.showingPages.push(this.maxPages - 1)
      this.showingPages.push(this.maxPages)
    }
  },
  watch: {
    '$route' (to) {
      this.searchQuery = to.query.query || '';
      this.currentPage = parseInt(to.query.page) || 1;
      this.load(this.searchQuery, this.currentPage)
    }
  },
  created () {
    this.searchQuery = this.$route.query.query || '';
    this.currentPage = parseInt(this.$route.query.page) || 1;

    if (this.searchQuery !== '') {
      this.load(this.searchQuery, this.currentPage);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
