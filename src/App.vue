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
            <a class="navbar-item" href="">Reddit</a>
            <a class="navbar-item" href="">Hacker News</a>
            <a class="navbar-item" href="">Google News</a>
          </div>
        </div>
      </div>
    </nav>
    <RedditCard>
      <h3 slot="title">Council votes to oppose PCs' bill to shrink size</h3>
      <li class="article-source" slot="source">Google News</li>
      <li class="article-author" slot="author">Toronto Sun</li>
      <li class="article-time" slot="time">2 hours ago</li>
      <p slot="excerpt">Toronto city council voted Monday to express its opposition to a controversial bill that would cut the council nearly in half to the Ontario government. Council also voted to request that the provincial government conduct a referendum on the number of ...</p> 

    </RedditCard>
    <RedditCard>
      <h3 slot="title">Council votes to oppose PCs' bill to shrink size</h3>
      <li class="article-source" slot="source">Google News</li>
      <li class="article-author" slot="author">Toronto Sun</li>
      <li class="article-time" slot="time">2 hours ago</li>
      <p slot="excerpt">Toronto city council voted Monday to express its opposition to a controversial bill that would cut the council nearly in half to the Ontario government. Council also voted to request that the provincial government conduct a referendum on the number of ...</p> 

    </RedditCard>
    <RedditCard>
      <h3 slot="title">Council votes to oppose PCs' bill to shrink size</h3>
      <li class="article-source" slot="source">Google News</li>
      <li class="article-author" slot="author">Toronto Sun</li>
      <li class="article-time" slot="time">2 hours ago</li>
      <p slot="excerpt">Toronto city council voted Monday to express its opposition to a controversial bill that would cut the council nearly in half to the Ontario government. Council also voted to request that the provincial government conduct a referendum on the number of ...</p> 

    </RedditCard>
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
    }
  },

  components: {
    RedditCard 
  },

  methods: {
    fetchRedditData() {
      axios.get("https://www.reddit.com/.json?limit=100")
        .then(response => {
          this.redditPosts = response.data.data.children;
        })
    },

    fetchGoogleData() {
      axios.get("https://newsapi.org/v2/top-headlines?country=ca&pageSize=20&apiKey=651b4a2d16cc4b92a6cd0f15379a0e85")
        .then(response => {
          this.googlePosts = response.data.articles;
        })
    },

    fetchHackerNewsData() {
      axios.get("https://hacker-news.firebaseio.com/v0/newstories.json")
        .then(response => {
          this.hackerNewsPostIds = response.data;
          for (var i = 0; i < 100; i++) {
            axios.get("https://hacker-news.firebaseio.com/v0/item/" + this.hackerNewsPostIds[i] + ".json")
              .then(response => {
                this.hackerNewsPosts.push(response.data)
              })
          }
      }); 
    }
  },
  
  created() {
    this.fetchRedditData();
    this.fetchGoogleData();
    this.fetchHackerNewsData();
  }
}
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Raleway:300,400");

.logo {
  padding-top: 0.75em; 
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

.container {
  font-family: 'Raleway';
  font-weight: 400;
}


</style>
