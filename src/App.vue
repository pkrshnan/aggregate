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
        <h3 slot="title"><a :href="'http://www.reddit.com' + post.data.permalink" target="_blank">{{ post.data.title }}</a></h3>
        <li class="article-source" slot="source"><a :href="'http://www.reddit.com/r/'+ post.data.subreddit" target="_blank">r/{{ post.data.subreddit }}</a></li>
        <li class="article-author" slot="author"><a :href="'http://www.reddit.com/u/' + post.data.author" target="_blank">u/{{ post.data.author }}</a></li>
        <li class="article-time" slot="time">{{ Date(post.data.created).slice(0,15) }}</li>
        <div slot="image" v-if='post.data.hasOwnProperty("preview")'>
          <img v-if="! post.data.preview.images[0].source.url.includes('gif')" class="image" v-bind:src="post.data.preview.images[0].source.url"></img>
          <img class="image" v-else v-bind:src="post.data.preview.images[0].variants.gif.source.url">
        </div>
        <div v-else slot="excerpt">
          <span v-html="parseMarkdown(post.data.selftext)"></span>
        </div>
      </RedditCard>
    </div>
    <div v-if="selected == 'Hacker News'">
      <RedditCard v-for="post in hackerNewsPosts">
        <h3 slot="title"><a :href="'https://news.ycombinator.com/item?id='+ post.id" target="_blank">{{ post.title }}</a></h3>
        <li class="article-author" slot="author"><a :href="'https://news.ycombinator.com/user?id='+ post.by" target="_blank">{{ post.by }}</a></li>
        <li class="article-time" slot="time">{{ Date(post.time).slice(0, 15) }}</li>
        <p slot="excerpt"><a :href="post.url" target="_blank">{{ post.url }}</a></p>
      </RedditCard>
    </div>
    <div v-if="selected == 'Google News'">
      <RedditCard v-for="post in googlePosts">
        <h3 slot="title"><a :href="post.url" target="_blank">{{ post.title }}</a></h3>
        <li class="article-author" slot="author">{{ post.source.name }}</li>
        <li class="article-time" slot="time">{{ Date(post.publishedAt.slice(0,10)).slice(0,15) }}</li>
        <div v-if="post.urlToImage != null" slot="image">
          <img class="image" v-bind:src="post.urlToImage"></img>
        </div>
        <p v-if="post.description != null"slot="excerpt">{{ post.description }}</p>
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
    },

    parseMarkdown(text) {
      var markdown = require("markdown").markdown;
      return markdown.toHTML(text);
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
  padding: 2em;
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
  color: #3073FF;
}

a {
  color: #2c2c2c;
}
a:hover {
  color: #3073FF;
}
</style>
