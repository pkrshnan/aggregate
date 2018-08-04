<template>
  <div id="app" :class="{'is-dark-background': isDark}">
    <nav class="navbar" :class="{'is-dark-background': isDark}">
      <div class="container">
        <div class="navbar-brand" :class="{'extra-padding': navIsActive}">
          <h2 class="logo is-size-5-desktop is-size-6-touch">Aggregate</h2>
          <a class="navbar-burger" @click="navBurgerToggle" :class="{'is-active': navIsActive}" role="button" aria-label="menu" aria-expanded="false">
            <span aria-hidden="true"></span>
            <span aria-hidden="true"></span>
            <span aria-hidden="true"></span>
          </a>
        </div>
        <div class="navbar-menu" :class="{'is-active': navIsActive, 'is-dark-background': isDark}">
          <div class="navbar-end">
            <p v-on:click="changeSelected('Reddit')":class="{ 'active': checkSelected('Reddit'), 'is-dark-background': isDark}" class="navbar-item" href="">Reddit</p>
            <p v-on:click="changeSelected('Hacker News')" :class="{ 'active': checkSelected('Hacker News'), 'is-dark-background': isDark}" class="navbar-item" href="">Hacker News</p>
            <p v-on:click="changeSelected('Google News')" :class="{ 'active': checkSelected('Google News'), 'is-dark-background': isDark}" class="navbar-item" href="">Google News</p>
          </div>
        </div>
      </div>
    </nav>
    <div v-if="selected == 'Reddit'">
      <RedditCard v-for="post in redditPosts" :class="{'is-dark-background': isDark}">
        <h3 slot="title"><a :class="{'is-dark-background': isDark, 'is-light-background': !isDark}":href="'http://www.reddit.com' + post.data.permalink" target="_blank">{{ post.data.title }}</a></h3>
        <li class="article-source" slot="source"><a :class="{'is-dark-background': isDark, 'is-light-background': !isDark}" :href="'http://www.reddit.com/r/'+ post.data.subreddit" target="_blank">r/{{ post.data.subreddit }}</a></li>
        <li class="article-author" slot="author"><a :class="{'is-dark-background': isDark, 'is-light-background': !isDark}" :href="'http://www.reddit.com/u/' + post.data.author" target="_blank">u/{{ post.data.author }}</a></li>
        <li class="article-time" :class="{'is-dark-background': isDark, 'is-light-background': !isDark}" slot="time">{{ Date(post.data.created).slice(0,15) }}</li>
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
    <div class="bottom-footer" :class="{'is-dark-background': isDark, 'is-light-background': !isDark}">
      <button @click="isDark=false" class="light-mode button" :class="{'is-dark': isDark, 'is-white': !isDark}">Light</button>
      <button @click="isDark=true" class="dark-mode button" :class="{'is-dark': isDark, 'is-white': !isDark}">Dark</button>
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
      selected: 'Reddit',
      navIsActive: false,
      isDark: true
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
    },

    parseMarkdown(text) {
      var markdown = require("markdown").markdown;
      return markdown.toHTML(text);
    },

    navBurgerToggle() {
      if (this.navIsActive) {
        this.navIsActive = false;
      } else {
        this.navIsActive = true;
      }
    }
  },
 
  created() {
    this.fetchRedditData();
    this.fetchHackerNewsData();
    this.fetchGoogleData();
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
  margin-bottom: 1.1em;
  box-shadow: 0px 1px 5px rgba(0,0,0,0.3)
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

.navbar-item.active {
  color: #3073FF;
} 

@media screen and (max-width: 1024px) {
  .navbar-brand {
    padding-left: 1em;
  }
}

.is-dark-background {
  background-color: #363636;
  color: #fff;
}

.is-light-background {
  background-color: #fff;
  color: #2c2c2c;
}
.bottom-footer {
  position: fixed;
  bottom: 0;
  width: 100%;
  box-shadow: 5px 1px 0px rgba (0,0,0,0.3);
  text-align: center;
  padding: 0.5em 0;
}

</style>
