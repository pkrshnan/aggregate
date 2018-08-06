<template>
  <div class="container">
    <div class="news-card">
      <div class="article-title is-size-5-desktop is-size-6-touch"><slot name="title"></slot></div>
      <ul class="article-info is-size-6-desktop is-size-7-touch">
        <slot name="source"></slot>
        <slot name="author"></slot>
        <slot name="time"></slot>
      </ul>
      <div class="article-image"><slot name="image"></slot></div>
      <div class="excerpt is-size-6-desktop is-size-7-touch"><slot name="excerpt"></slot></div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'RedditCard',
  
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.news-card {
  padding: 1em 2em;
  border: 2px solid rgba(70, 70, 70, 0.3);
  margin: 1em 0;
  font-weight: 400;
}

.article-info {
  display: inline;
  padding: 0.25em 0em;
}

.article-info li {
  display: inline-block;
  font-family: "Raleway";
  font-weight: 300;
}

.article-info li:after {
  content: " | ";
  padding: 0.25em;
}

.article-info li:last-child:after {
  content: "";
}

.excerpt {
  padding: 0.25em 0em;
  font-weight: 300;
}

a {
  color: #2c2c2c;
}

a:hover {
  color: #3073ff;
}

.container {
  margin-top: 1em;
}
</style>
