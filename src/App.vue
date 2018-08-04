<template>
  <div id="app">
    <nav class="navbar">
      <div class="container">
        <div class="navbar-brand">
          <h2 class="logo is-size-5-desktop is-size-6-touch">Aggregate</h2>
          <a class="navbar-burger" role="button" aria-label="menu" aria-expanded="false">
            <span aria-hidden="true"></span>
            <span aria-hidden="true"></span>
            <span aria-hidden="true"></span>
          </a>
        </div>
        <div class="navbar-menu">
          <div class="navbar-end">
            <p v-on:click="changeSelected('Reddit')":class="{ 'active': checkSelected('Reddit')}" class="navbar-item" href="">Reddit</p>
            <p v-on:click="changeSelected('Hacker News')" :class="{ 'active': checkSelected('Hacker News')}" class="navbar-item" href="">Hacker News</p>
            <p v-on:click="changeSelected('Google News')" :class="{ 'active': checkSelected('Google News')}" class="navbar-item" href="">Google News</p>
          </div>
        </div>
      </div>
    </nav>
    <div v-if="selected == 'Reddit'">
      <RedditCard v-for="post in redditPosts">
        <h3 slot="title">{{ post.data.title }}</h3>
        <li class="article-source" slot="source">r/{{ post.data.subreddit }}</li>
        <li class="article-author" slot="author">u/{{ post.data.author }}</li>
        <li class="article-time" slot="time">{{ post.data.created }}</li>
        <div slot="image" v-if='post.data.hasOwnProperty("preview")'>
          <img class="is-centered image" v-bind:src="post.data.preview.images[0].source.url"></img>
        </div>
        <p slot="excerpt">{{preview}}</p>
      </RedditCard>
    </div>
    <div v-if="selected == 'Hacker News'">
      <RedditCard v-for="post in hackerNewsPosts">
        <h3 slot="title">{{ post.title }}</h3>
        <li class="article-author" slot="author">{{ post.by }}</li>
        <li class="article-time" slot="time">{{ post.time }}</li>
        <p slot="excerpt">{{ post.url }}</p>
      </RedditCard>
    </div>
    <div v-if="selected == 'Google News'">
      <RedditCard v-for="post in googlePosts">
        <h3 slot="title">{{ post.title }}</h3>
        <li class="article-author" slot="author">{{ post.source.name }}</li>
        <li class="article-time" slot="time">{{ post.publishedAt }}</li>
        <p slot="excerpt">{{ post.url }}</p>
      </RedditCard>
    </div>
  </div>
</template>

<script>
import RedditCard from './components/RedditCard'
import axios from 'axios';

export default {
  name: 'App',

  data() {
    return {
      redditPosts: [],
      googlePosts: [],
      hackerNewsPostIds: [],
      hackerNewsPosts: [],
      selected: 'Reddit'
    }
  },

  components: {
    RedditCard 
  },

  methods: {
    fetchRedditData() {
      this.redditPosts = [];
      axios.get("https://www.reddit.com/.json?limit=100")
        .then(response => {
          this.redditPosts = response.data.data.children;
        })
    },

    fetchGoogleData() {
      this.googlePosts = [];
      axios.get("https://newsapi.org/v2/top-headlines?country=ca&pageSize=20&apiKey=651b4a2d16cc4b92a6cd0f15379a0e85")
        .then(response => {
          this.googlePosts = response.data.articles;
        })
    },

    fetchHackerNewsData() {
      axios.get("https://hacker-news.firebaseio.com/v0/newstories.json")
        .then(response => {
          this.hackerNewsPosts = [];
          this.hackerNewsPostIds = response.data;
          for (var i = 0; i < 100; i++) {
            axios.get("https://hacker-news.firebaseio.com/v0/item/" + this.hackerNewsPostIds[i] + ".json")
              .then(response => {
                this.hackerNewsPosts.push(response.data)
              })
          }
      }); 
    },

    checkSelected(name) {
      if (name == this.selected)  {
        return true;
      } else {
        return false;
      }
    },

    changeSelected(newName) {
      this.selected = newName;
      if (newName == 'Hacker News') {
        this.fetchHackerNewsData();
      } else if (newName == 'Reddit') {
        this.fetchRedditData();
      } else if (newName == 'Google News') {
        this.fetchGoogleData();
      }
    }
  },
 
  created() {
    if (this.selected == 'Reddit') {
      this.fetchRedditData();
    } else if (this.selected = 'Hacker News') {
      this.fetchHackerNewsData();
    } else if (this.selected == 'Google News') {
      this.fetchGoogleData();
    }
  }
}
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Raleway:300,400");

.logo {
  padding-top: 0.75em; 
}

.image {
  margin: auto;
  display: block;
  padding-top: 1em;
}
.navbar {
  padding: 0.5em 0; 
  margin-bottom: 1.1em;
  box-shadow: 0px 1px 5px rgba(0,0,0,0.3)
}

.navbar .container {
  padding-left: 0.5em;
  padding-right: 0.5em;
}

.navbar-item:last-child {
  padding-right: 0em;
}

.navbar-item:hover {
  cursor: pointer;
}
.container {
  font-family: 'Raleway';
  font-weight: 400;
}

.active {
  color: #30B3FF;
}
</style>
