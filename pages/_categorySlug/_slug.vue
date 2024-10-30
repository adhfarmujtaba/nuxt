<template>
  <div v-if="loading" class="loading">Loading...</div>
  <div v-else class="post-container">
    <img :src="post.image" alt="" class="post-image" />
    <h1 class="post-title">{{ post.title }}</h1>
    <div v-html="post.content" class="post-content"></div>
    <nuxt-link to="/" class="back-button">Back to Posts</nuxt-link>
  </div>
</template>

<script>
let cachedPosts = {}; // Local cache for posts

export default {
  data() {
    return {
      post: null,
      loading: true,
    };
  },
  async fetch() {
    const { slug } = this.$route.params;

    // Check if the post is already cached
    if (cachedPosts[slug]) {
      this.post = cachedPosts[slug];
      this.loading = false;
      return;
    }

    try {
      const response = await this.$axios.$get(`https://blog.tourismofkashmir.com/apis?post_slug=${slug}`);
      this.post = response; // Assuming the response contains a single post object
      cachedPosts[slug] = response; // Store in cache
    } catch (error) {
      console.error('Error fetching post:', error);
    } finally {
      this.loading = false;
    }
  },
  head() {
    if (process.client) { // Ensure this runs only in the browser
      const domain = window.location.origin; // Get the current domain
      const postUrl = this.post ? `${domain}/post/${this.post.slug}` : ''; // Adjust the path as needed

      return {
        title: this.post ? this.post.title : 'Loading...',
        meta: [
          { hid: 'description', name: 'description', content: this.post ? this.post.meta_description : 'Loading...' },
          { hid: 'og:title', property: 'og:title', content: this.post ? this.post.title : 'Loading...' },
          { hid: 'og:description', property: 'og:description', content: this.post ? this.post.meta_description : 'Loading...' },
          { hid: 'og:image', property: 'og:image', content: this.post ? this.post.image : '' },
          { hid: 'og:url', property: 'og:url', content: postUrl },
        ],
      };
    }
    // Return an empty object when on the server
    return {};
  },
};
</script>

<style scoped>
.post-container {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.loading {
  text-align: center;
  font-size: 1.5rem;
  margin: 50px 0;
}

.post-title {
  font-size: 2rem;
  margin: 20px 0;
  color: #333;
}

.post-image {
  width: 100%;
  height: auto;
  border-radius: 12px;
  margin-bottom: 20px;
}

.post-content {
  font-size: 1.2rem;
  line-height: 1.6;
  color: #555;
}

.back-button {
  display: inline-block;
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #0070f3;
  color: white;
  text-decoration: none;
  border-radius: 5px;
  transition: background-color 0.3s;
}

.back-button:hover {
  background-color: #005bb5;
}
</style>
