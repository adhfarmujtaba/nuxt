<template>
  <div>
    <div v-if="loading" class="skeleton-container">
      <div v-for="index in 10" :key="index" class="card skeleton-card">
        <div class="skeleton-image"></div>
        <div class="card-content">
          <div class="skeleton-title"></div>
          <div class="skeleton-content"></div>
          <div style="display: flex; align-items: center;">
            <div class="skeleton-avatar"></div>
            <div class="skeleton-username"></div>
            <div class="skeleton-date"></div>
          </div>
        </div>
      </div></div>
    <div v-else class="news-list">
      <div v-for="post in posts" :key="post.id" class="card" @contextmenu.prevent>
        <nuxt-link :to="`${post.category_slug}/${post.slug}/`" class="card-link">
          <div class="image-container" style="position: relative">
            <img
              :src="post.image"
              :alt="post.title"
              class="news-item-image"
              style="width: 100%; height: 180px; object-fit: cover"
            />
            <div
              style="position: absolute; bottom: 10px; right: 10px; background-color: rgba(0, 0, 0, 0.5); color: white; padding: 5px; border-radius: 5px; font-size: 0.8rem;"
            >
              {{ post.read_time }} min read
            </div>
          </div>
          <div class="card-content">
            <h2>{{ truncateText(post.title, 10) }}</h2>
            <p>{{ truncateText(post.meta_description, 20) }}</p>
          </div>
        </nuxt-link>
        <div style="display: flex; align-items: center; margin-bottom: 5px;">
          <nuxt-link :to="`/profile/${post.username}`" style="text-decoration: none; color: inherit; display: flex; align-items: center;">
            <img
              :src="`https://blog.tourismofkashmir.com/${post.avatar}`"
              alt="Avatar"
              class="avatar"
              style="width: 30px; height: 30px; border-radius: 50%; margin-right: 5px"
            />
            <span class="username">{{ post.username }}</span>
          </nuxt-link>
          <span class="views">. {{ formatViews(post.views) }} views</span>
          <span class="date">{{ formatDate(post.created_at) }}</span>
        </div>
      </div>
      <button v-if="!loading" @click="loadMore" class="load-more">Load More</button>
    </div>
  </div>
</template>

<script>
import '../assets/index.css'; // Check the spelling

let cachedPosts = []; // Local cache for posts

export default {
  data() {
    return {
      posts: [],
      loading: true,
      page: 1,
    };
  },
  async fetch() {
    this.loading = true; // Ensure loading state starts
    // Use cached posts if available
    if (cachedPosts.length > 0) {
      this.posts = cachedPosts;
      this.loading = false;
      return;
    }

    try {
      const response = await this.$axios.$get(`https://blog.tourismofkashmir.com/apis?posts&page=${this.page}`);
      this.posts = response; // Use response directly
      cachedPosts = response; // Store in cache
    } catch (error) {
      console.error('Error fetching posts:', error);
    } finally {
      this.loading = false;
    }
  },
  methods: {
    loadMore() {
      this.page++;
      this.fetch(); // No need to set loading again here
    },
    truncateText(text, limit) {
      const words = text.split(' ');
      return words.length > limit ? words.slice(0, limit).join(' ') + '...' : text;
    },
    formatViews(views) {
      if (views >= 10000000) {
        return Math.floor(views / 10000000) + 'cr';
      } else if (views >= 1000000) {
        return Math.floor(views / 1000000) + 'M';
      } else if (views >= 100000) {
        return Math.floor(views / 100000) + 'L';
      } else if (views >= 1000) {
        return Math.floor(views / 1000) + 'k';
      } else {
        return views;
      }
    },
    formatDate(date) {
      const currentDate = new Date();
      const postDate = new Date(date);

      const timeDifference = currentDate - postDate;
      const seconds = Math.floor(timeDifference / 1000);
      const minutes = Math.floor(seconds / 60);
      const hours = Math.floor(minutes / 60);
      const days = Math.floor(hours / 24);

      if (seconds < 60) {
        return seconds + ' sec ago';
      } else if (minutes < 60) {
        return minutes + ' mins ago';
      } else if (hours < 24) {
        return hours + ' hours ago';
      } else if (days < 7) {
        return days + ' days ago';
      } else {
        const weeks = Math.floor(days / 7);
        return weeks + ' weeks ago';
      }
    },
  },
};
</script>
