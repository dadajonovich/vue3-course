<template>
  <div>
    <h1>Страница с постами</h1>
    <my-input v-focus v-model="searchQuery" placeholder="Поиск..." />
    <div class="app_btns">
      <my-button @click="showDialog">Создать пост</my-button>
      <my-select v-model="selectedSort" :options="sortOptions" />
    </div>

    <my-dialog v-model:show="dialogVisible"><post-form @create="createPost" /></my-dialog>

    <post-list :posts="sortedAndSearchPosts" @remove="removePost" v-if="!isPostsLoading" />
    <div v-else>Идет загрузка...</div>
    <div v-intersection="loadMorePosts" class="observer"></div>
    <!-- <div class="page__wrapper">
        <div
          v-for="pageNumber in totalPages"
          :key="pageNumber"
          class="page"
          :class="{ 'current-page': page === pageNumber }"
          @click="changePage(pageNumber)"
        >
          {{ pageNumber }}
        </div>
      </div> -->
  </div>
</template>

<script>
import axios from 'axios';
import PostForm from '../components/PostForm.vue';
import PostList from '../components/PostList.vue';

export default {
  components: {
    PostList,
    PostForm
  },
  data() {
    return {
      posts: [],
      dialogVisible: false,
      isPostsLoading: false,
      selectedSort: '',
      searchQuery: '',
      page: 1,
      totalPages: 0,
      limit: 10,
      sortOptions: [
        { value: 'title', name: 'По названию' },
        { value: 'body', name: 'По содержимому' }
      ]
    };
  },
  methods: {
    createPost(post) {
      this.posts.push(post);
      this.dialogVisible = false;
    },
    removePost(post) {
      this.posts = this.posts.filter((p) => p.id !== post.id);
    },
    showDialog() {
      this.dialogVisible = true;
    },
    // changePage(pageNumber) {
    //   this.page = pageNumber;
    // },
    async fetchPosts() {
      try {
        this.isPostsLoading = true;
        const responce = await axios.get('https://jsonplaceholder.typicode.com/posts', {
          params: {
            _page: this.page,
            _limit: this.limit
          }
        });
        this.totalPages = Math.ceil(responce.headers['x-total-count'] / this.limit);
        this.posts = responce.data;
      } catch (error) {
        alert('Ошибка');
      } finally {
        this.isPostsLoading = false;
      }
    },
    async loadMorePosts() {
      try {
        this.page += 1;
        const responce = await axios.get('https://jsonplaceholder.typicode.com/posts', {
          params: {
            _page: this.page,
            _limit: this.limit
          }
        });
        this.totalPages = Math.ceil(responce.headers['x-total-count'] / this.limit);
        this.posts = [...this.posts, ...responce.data];
      } catch (error) {
        alert('Ошибка');
      }
    }
  },
  mounted() {
    this.fetchPosts();
    // console.log(this.$refs.observer);
    // const options = {
    //   rootMargin: '0px',
    //   threshold: 1.0
    // };
    // const callback = (entries, observer) => {
    //   if (entries[0].isIntersecting && this.page < this.totalPages) {
    //     this.loadMorePosts();
    //   }
    // };
    // const observer = new IntersectionObserver(callback, options);
    // observer.observe(this.$refs.observer);
  },

  computed: {
    sortedPosts() {
      return [...this.posts].sort((post1, post2) => {
        return post1[this.selectedSort]?.localeCompare(post2[this.selectedSort]);
      });
    },
    sortedAndSearchPosts() {
      return this.sortedPosts.filter((post) =>
        post.title.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    }
  },

  watch: {
    // page() {
    //   this.fetchPosts();
    // }
  }
};
</script>

<style>
.app_btns {
  margin: 15px 0;
  display: flex;
  justify-content: space-between;
}

.page__wrapper {
  display: flex;
  margin-top: 15px;
}

.page {
  border: 1px solid black;
  padding: 10px;
}

.current-page {
  border: 2px solid green;
}

.observer {
  height: 30px;
  background: gray;
}
</style>
