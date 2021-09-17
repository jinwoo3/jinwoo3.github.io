<template>
  <Layout :pageTitle="$page.post.title" :pageSubtitle="$page.post.author.id">
    <div class="box">
      <nav class="breadcrumb is-centered" aria-label="breadcrumbs">
        <ul>
          <li><router-link to="/">Home</router-link></li>
          <li class="is-active">
            <a href="#" aria-current="page">{{ $page.post.title }}</a>
          </li>
        </ul>
      </nav>
    </div>
    <section class="section">
      <div class="container">
        <div class="content">
          <div v-html="$page.post.content"></div>
        </div>
      </div>
    </section>
  </Layout>
</template>

<page-query>
  query blog($path: String){
    post: post(path: $path){
     id
     title
     excerpt
     featuredImage
     content
     tags {
       id
     }
     author {
       id
     }
    }
  }
</page-query>

<script>
export default {
  metaInfo() {
    return {
      title: this.$page.post.title,
    };
  },
};
</script>
