<template>
  <div id="app">
    <nav class="navbar">
      <div class="container">
        <div class="navbar-brand">
          <h2 class="logo is-size-5-desktop is-size-6-touch">Aggregate</h2>
          <a class="navbar-burger" @click="navBurgerToggle" :class="{'is-active': navIsActive}" role="button" aria-label="menu" aria-expanded="false">
            <span aria-hidden="true"></span>
            <span aria-hidden="true"></span>
            <span aria-hidden="true"></span>
         </a>
        </div>
        <div class="navbar-menu" @click="navBurgerToggle" :class="{'is-active': navIsActive}">
          <div class="navbar-end">
            <p v-on:click="changeSelected('Reddit')":class="{ 'active': checkSelected('Reddit')}" class="navbar-item" href="">Reddit</p>
            <p v-on:click="changeSelected('Hacker News')" :class="{ 'active': checkSelected('Hacker News')}" class="navbar-item" href="">Hacker News</p>
            <p v-on:click="changeSelected('Google News')" :class="{ 'active': checkSelected('Google News')}" class="navbar-item" href="">Google News</p>
          </div>
        </div>
      </div>
    </nav>
    <div v-if="selected == 'Reddit'">
      <Modal v-if="showModal" @close="showModal=false">
        <h3 class="modalTitle" slot="title">{{modalTitle}}</h3>
        <iframe class="centered-image" v-if="modalType =='embed'" :src="modalSource" height="430" width="710"></iframe>
        <img class="centered-image" :src="modalSource" v-else></img>
      </Modal>
      <div class="container">
        <input v-model="subreddit" class="searchbar input is-rounded" type="text" placeholder="Enter Subreddit" v-on:keyup="fetchRedditData()">
      </div>
      <RedditCard v-for="post in redditPosts">
        <h3 slot="title"><a :href="'http://reddit.com' + post.data.permalink" target="_blank">{{ post.data.title }}</a></h3>
        <li class="article-source" slot="source"><a  :href="'http://www.reddit.com/r/'+ post.data.subreddit" target="_blank">r/{{ post.data.subreddit }}</a></li>
        <li class="article-author" slot="author"><a  :href="'http://www.reddit.com/u/' + post.data.author" target="_blank">u/{{ post.data.author }}</a></li>
        <li class="article-time" slot="time">{{ Date(post.data.created).slice(0,15) }}</li>
        <div slot="image" v-if='post.data.hasOwnProperty("preview")'>
          <img v-if="post.data.thumbnail == 'default'" class="link-image image" src="./assets/link.png">
          <h3 v-else-if="post.data.over_18 == true" class="title nsfw">NSFW</h3>
          <h3 v-else-if="post.data.thumbnail == 'spoiler'" class="title nsfw">SPOILER</h3>
          <a v-else-if="post.data.thumbnail == 'image' || post.data.thumbnail == 'self'" @click="showRedditModal(post)"><img class="image" v-bind:src="post.data.preview.images[0].source.url"></a>
          <a v-else @click="showRedditModal(post)"><img class="image" :src="post.data.thumbnail"></a>
        </div>
        <div slot="excerpt">
          <span v-html="parseMarkdown(post.data.selftext)"></span>
        </div>
      </RedditCard>
    </div> 
    <div v-if="selected == 'Hacker News'">
      <RedditCard v-for="post in hackerNewsPosts">
        <h3 slot="title"><a :href="'https://news.ycombinator.com/item?id='+ post.id" target="_blank">{{ post.title }}</a></h3>
        <li class="article-author" slot="author"><a :href="'https://news.ycombinator.com/user?id='+ post.by" target="_blank">{{ post.by }}</a></li>
        <li class="article-time" slot="time">{{ Date(post.time).slice(0, 15) }}</li>
        <div v-if="post.hasOwnProperty('url')" slot="excerpt">
          <p><a :href="post.url" target="_blank">{{ post.url }}</a></p>
        </div>
        <div v-if="post.hasOwnProperty('text')" slot="excerpt">
          <span v-html="post.text"></span>
        </div>
      </RedditCard>
    </div>
    <div v-if="selected == 'Google News'">
      <div class="container">
        <div class="buttons columns">
          <button @click="googleSelected('World')" :class="{'is-selected': googleCategorySelected('World')}" class="button column category">World</button>
          <button @click="googleSelected('Canada')" :class="{'is-selected': googleCategorySelected('Canada')}" class="button column category">Canada</button>
          <button @click="googleSelected('Sports')" :class="{'is-selected': googleCategorySelected('Sports')}" class="button column category">Sports</button>
          <button @click="googleSelected('Technology')" :class="{'is-selected': googleCategorySelected('Technology')}" class="button column category">Technology</button>
        </div>
      </div>
      <RedditCard v-for="post in googlePosts">
        <h3 slot="title"><a :href="post.url" target="_blank">{{ post.title }}</a></h3>
        <li class="article-author" slot="author">{{ post.source.name }}</li>
        <li class="article-time" slot="time">{{ Date(post.publishedAt.slice(0,10)).slice(0,15) }}</li>
        <div v-if="post.urlToImage != null" slot="image">
          <img class="image" v-bind:src="post.urlToImage"></img>
        </div>
        <p v-if="post.description != null" slot="excerpt">{{ post.description }}</p>
      </RedditCard>
    </div>
  </div>
</template>

<script>
import RedditCard from './components/RedditCard'
import Modal from './components/Modal'
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
      googleIsSelected: 'World',
      subreddit: null,
      showModal: false,
      modalSource: '',
      modalTitle: '',
      embedHeight: '',
      embedWidth: '',
      modalType: '',
    }
  },

  components: {
    RedditCard,
    Modal
  },

  methods: {
    fetchRedditData() {
      this.redditPosts = [];
      if (this.subreddit) {
        axios.get("https://www.reddit.com/r/" + this.subreddit + '.json?limit=100')          
          .then(response => {
            this.redditPosts = response.data.data.children;
          }) 
      } else {
        axios.get("https://www.reddit.com/.json?limit=100")
          .then(response => {
            this.redditPosts = response.data.data.children;
        }) 
      }
    },

    fetchGoogleData() {
      this.googlePosts = [];

      if (this.googleIsSelected == 'Canada') {
        axios.get("https://newsapi.org/v2/top-headlines?country=ca&pageSize=20&apiKey=651b4a2d16cc4b92a6cd0f15379a0e85")
          .then(response => {
            this.googlePosts = response.data.articles;
        })
      } else if (this.googleIsSelected == 'World') {
        axios.get("https://newsapi.org/v2/top-headlines?language=en&apiKey=651b4a2d16cc4b92a6cd0f15379a0e85")
          .then(response => {
            this.googlePosts = response.data.articles;
        }) 
      } else if (this.googleIsSelected == 'Technology') {
        axios.get("https://newsapi.org/v2/top-headlines?country=us&category=technology&apiKey=651b4a2d16cc4b92a6cd0f15379a0e85")
          .then(response => {
            console.log("Works!")
            this.googlePosts = response.data.articles;
        })
      } else if (this.googleIsSelected == 'Sports') {
        axios.get("https://newsapi.org/v2/top-headlines?country=ca&category=sports&apiKey=651b4a2d16cc4b92a6cd0f15379a0e85")
          .then(response => {
            this.googlePosts = response.data.articles;
        })
      }
    },

    fetchHackerNewsData() {
      axios.get("https://hacker-news.firebaseio.com/v0/newstories.json")
        .then(response => {
          this.hackerNewsPosts = [];
          this.hackerNewsPostIds = response.data;
          for (var i = 0; i < 100; i++) {
            axios.get("https://hacker-news.firebaseio.com/v0/item/" + this.hackerNewsPostIds[i] + ".json?print=pretty")
              .then(response => {
                console.log(response.data)
                if (response.data != null) {
                  this.hackerNewsPosts.push(response.data)

                }
              })
          }
      }); 
    },

    showRedditModal(post) {
      if (post.data.post_hint === "image") {
        if (post.data.url.indexOf('gif') != -1) {
          this.modalSource = post.data.url;
          this.modalType = "image";
        } else {
          this.modalSource = post.data.preview.images[0].source.url;
          this.modalType = 'image';
        }
      } else if (post.data.post_hint==="rich:video") {
        this.modalType = 'embed';
        var link = post.data.media_embed.content;
        var width = link.slice(link.indexOf("width") + 7, link.indexOf("width") + 10)
        var height = link.slice(link.indexOf("height") + 8, link.indexOf("height") + 11)
        link = link.slice(link.indexOf("src") + 5, link.length - 1);
        link = link.slice(0, link.indexOf('"'));

        while (link.indexOf("amp") != -1) {
          link = link.replace("amp;", "")
        }
        this.modalSource = link; 
      } else if (post.data.post_hint == "hosted:video") {
        this.modalSource = post.data.media.reddit_video.fallback_url;
        this.modalType = 'embed';
      } else if (post.data.post_hint === "link") {
        console.log(post.data.preview.images[0].variants.hasOwnProperty('gif'))
        if (post.data.preview.images[0].variants.hasOwnProperty('gif')) {
          this.modalSource = post.data.preview.images[0].variants.gif.source.url;
          console.log(post.data.preview.images[0].variants.gif.source.url);
        } else {
          this.modalSource = post.data.preview.images[0].source.url;
        }
        this.modalType= 'image';
      }
      this.modalTitle = post.data.title;
      this.showModal=true;
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
    },

    googleCategorySelected(name) {
      if (name == this.googleIsSelected) {
        return true;
      }
    },

    googleSelected(name) {
      this.googleIsSelected = name;
      this.fetchGoogleData();
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
  margin: 0.5em auto;
  display: block;
  max-width: 140px;
  max-height: 140px;
}

.link-image {
  border: 1px solid #2c2c2c;
}

.excerpt{
  padding-bottom: 1em;
  margin-top: 1em;
}

.nsfw {
  text-align: center;
  padding: 2em 0;
  font-weight: 400 !important;
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

.buttons {
  text-align: center;
  margin: 1em 0em !important;
}

.category {
  width: 100%;
  padding: 0.25em !important;
  height: 2.5em !important;
  font-family: Raleway;
  font-weight: 400;
}

.category.is-selected {
  color: #3073FF;
  border-color: #3073FF;
}

.searchbar {
  margin: 0.5 0em !important;
  font-family: 'Raleway' !important;
}

@media screen and (max-width: 768px) {
  .modal-card {
    max-width: 600px;
  }
}

@media screen and (max-width: 1024px) {
  .modal-card {
    max-width: 700px;
  }
}

@media screen and (max-width: 1215px) {
  .modal-card {
    width: 900px;
  }
}

@media screen and (max-width: 1407px) {
  .modal-card {
    width: 1100px;
  }
}

.modal-card {
  width: 1300px;
}

.centered-image {
  display: block;
  margin: auto;
}
</style>
