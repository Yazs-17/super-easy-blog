<template>
  <div class="blog-container">
    <!-- Header -->
    <header class="header">
      <div class="container">
        <h1 class="blog-title">My Personal Blog</h1>
        <p class="blog-description">A simple but professional blog platform</p>
      </div>
    </header>

    <!-- Main Content -->
    <main class="container main-content">
      <!-- New Post Form -->
      <div class="card new-post-card">
        <h2>Create New Post</h2>
        <form @submit.prevent="createPost">
          <div class="form-group">
            <label for="postTitle">Title</label>
            <input 
              type="text" 
              id="postTitle" 
              v-model="newPost.title" 
              class="form-control" 
              required
            />
          </div>
          <div class="form-group">
            <label for="postContent">Content</label>
            <textarea 
              id="postContent" 
              v-model="newPost.content" 
              class="form-control" 
              rows="5" 
              required
            ></textarea>
          </div>
          <div class="form-group">
            <label for="postAuthor">Your Name</label>
            <input 
              type="text" 
              id="postAuthor" 
              v-model="newPost.author" 
              class="form-control" 
              required
            />
          </div>
          <button type="submit" class="btn btn-primary">Publish Post</button>
        </form>
      </div>

      <!-- Posts List -->
      <div class="posts-container">
        <h2>Recent Posts</h2>
        <div v-if="loading" class="loading">Loading posts...</div>
        <div v-else-if="posts.length === 0" class="no-posts">No posts yet. Be the first to create one!</div>
        
        <div v-for="post in posts" :key="post.id" class="card post-card">
          <div class="post-header">
            <h3 class="post-title">{{ post.title }}</h3>
            <div class="post-meta">
              <span class="post-author">By {{ post.author }}</span>
              <span class="post-date">{{ formatDate(post.createdAt) }}</span>
            </div>
          </div>
          <div class="post-content">{{ post.content }}</div>
          
          <!-- Comments Section -->
          <div class="comments-section">
            <h4>Comments ({{ post.comments.length }})</h4>
            <ul class="comments-list">
              <li v-for="comment in post.comments" :key="comment.id" class="comment">
                <div class="comment-header">
                  <span class="comment-author">{{ comment.author }}</span>
                  <span class="comment-date">{{ formatDate(comment.createdAt) }}</span>
                </div>
                <div class="comment-content">{{ comment.content }}</div>
              </li>
            </ul>
            
            <!-- New Comment Form -->
            <div class="new-comment-form">
              <h5>Add a Comment</h5>
              <form @submit.prevent="addComment(post.id)">
                <div class="form-group">
                  <label for="commentAuthor">Your Name</label>
                  <input 
                    type="text" 
                    id="commentAuthor" 
                    v-model="newComment.author" 
                    class="form-control" 
                    required
                  />
                </div>
                <div class="form-group">
                  <label for="commentContent">Comment</label>
                  <textarea 
                    id="commentContent" 
                    v-model="newComment.content" 
                    class="form-control" 
                    rows="3" 
                    required
                  ></textarea>
                </div>
                <button type="submit" class="btn btn-secondary">Submit Comment</button>
              </form>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- Footer -->
    <footer class="footer">
      <div class="container">
        <p>&copy; {{ new Date().getFullYear() }} My Personal Blog. All rights reserved.</p>
      </div>
    </footer>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import axios from 'axios';

export default {
  name: 'BlogApp',
  setup() {
    const posts = ref([]);
    const loading = ref(true);
    const newPost = ref({
      title: '',
      content: '',
      author: ''
    });
    const newComment = ref({
      author: '',
      content: ''
    });

    // API base URL
    const API_URL = 'http://localhost:8080/api';

    // Fetch all posts
    const fetchPosts = async () => {
      try {
        loading.value = true;
        const response = await axios.get(`${API_URL}/posts`);
        posts.value = response.data;
      } catch (error) {
        console.error('Error fetching posts:', error);
      } finally {
        loading.value = false;
      }
    };

    // Create a new post
    const createPost = async () => {
      try {
        await axios.post(`${API_URL}/posts`, newPost.value);
        // Reset form
        newPost.value = {
          title: '',
          content: '',
          author: ''
        };
        // Refresh posts
        fetchPosts();
      } catch (error) {
        console.error('Error creating post:', error);
      }
    };

    // Add a comment to a post
    const addComment = async (postId) => {
      try {
        await axios.post(`${API_URL}/posts/${postId}/comments`, newComment.value);
        // Reset form
        newComment.value = {
          author: '',
          content: ''
        };
        // Refresh posts
        fetchPosts();
      } catch (error) {
        console.error('Error adding comment:', error);
      }
    };

    // Format date
    const formatDate = (dateString) => {
      const options = { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit' };
      return new Date(dateString).toLocaleDateString(undefined, options);
    };

    // Fetch posts on component mount
    onMounted(fetchPosts);

    return {
      posts,
      loading,
      newPost,
      newComment,
      createPost,
      addComment,
      formatDate
    };
  }
}
</script>

<style>
/* Global Styles */
:root {
  --primary-color: #3a6ea5;
  --secondary-color: #004e98;
  --accent-color: #ff6b6b;
  --background-color: #f8f9fa;
  --card-background: #ffffff;
  --text-color: #333333;
  --text-light: #6c757d;
  --border-color: #e9ecef;
  --shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  line-height: 1.6;
  color: var(--text-color);
  background-color: var(--background-color);
  margin: 0;
  padding: 0;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 15px;
}

/* Header Styles */
.header {
  background-color: var(--primary-color);
  color: white;
  padding: 2rem 0;
  margin-bottom: 2rem;
  box-shadow: var(--shadow);
}

.blog-title {
  margin: 0;
  font-size: 2.5rem;
  font-weight: 700;
}

.blog-description {
  margin-top: 0.5rem;
  font-size: 1.2rem;
  opacity: 0.9;
}

/* Main Content Styles */
.main-content {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  padding-bottom: 3rem;
}

/* Card Styles */
.card {
  background-color: var(--card-background);
  border-radius: 8px;
  box-shadow: var(--shadow);
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.new-post-card {
  border-left: 4px solid var(--accent-color);
}

.post-card {
  border-left: 4px solid var(--primary-color);
}

/* Form Styles */
.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-control {
  width: 100%;
  padding: 0.75rem;
  font-size: 1rem;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  transition: border-color 0.2s;
}

.form-control:focus {
  outline: none;
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(58, 110, 165, 0.2);
}

.btn {
  display: inline-block;
  font-weight: 500;
  text-align: center;
  white-space: nowrap;
  vertical-align: middle;
  user-select: none;
  border: 1px solid transparent;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: 4px;
  transition: all 0.2s;
  cursor: pointer;
}

.btn-primary {
  color: #fff;
  background-color: var(--primary-color);
  border-color: var(--primary-color);
}

.btn-primary:hover {
  background-color: var(--secondary-color);
  border-color: var(--secondary-color);
}

.btn-secondary {
  color: #fff;
  background-color: var(--text-light);
  border-color: var(--text-light);
}

.btn-secondary:hover {
  background-color: #5a6268;
  border-color: #5a6268;
}

/* Posts Styles */
.posts-container {
  margin-top: 1rem;
}

.post-header {
  margin-bottom: 1rem;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 0.5rem;
}

.post-title {
  margin-top: 0;
  margin-bottom: 0.5rem;
  color: var(--primary-color);
}

.post-meta {
  color: var(--text-light);
  font-size: 0.9rem;
  display: flex;
  gap: 1rem;
}

.post-content {
  margin-bottom: 1.5rem;
  white-space: pre-line;
}

/* Comments Styles */
.comments-section {
  margin-top: 1.5rem;
  padding-top: 1rem;
  border-top: 1px solid var(--border-color);
}

.comments-list {
  list-style: none;
  padding: 0;
  margin: 0 0 1.5rem 0;
}

.comment {
  padding: 1rem;
  background-color: #f8f9fa;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.comment-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
}

.comment-author {
  font-weight: 600;
}

.comment-date {
  color: var(--text-light);
}

.new-comment-form {
  background-color: #f8f9fa;
  padding: 1rem;
  border-radius: 4px;
}

/* Footer Styles */
.footer {
  background-color: var(--primary-color);
  color: white;
  padding: 1.5rem 0;
  text-align: center;
  margin-top: 2rem;
}

/* Responsive Styles */
@media (max-width: 768px) {
  .blog-title {
    font-size: 2rem;
  }
  
  .blog-description {
    font-size: 1rem;
  }
  
  .card {
    padding: 1rem;
  }
}

/* Loading and Empty States */
.loading, .no-posts {
  text-align: center;
  padding: 2rem;
  color: var(--text-light);
}
</style>