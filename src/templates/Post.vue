<template>
  <Layout :pageTitle="$page.post.title" :pageSubtitle="$page.post.excerpt">
    <p>{{ $page.allPost.edges }}</p>

    <div class="box breadcrumbs">
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
    <div class="box author">
      <figure class="image is-rounded is-128x128">
        <img class="is-rounded" :src="`../../../../../../${$page.post.author.image}`" />
      </figure>
      <div class="content">
        <h2 class="title is-5">
          Posted by <g-link :to="$page.post.author.path">{{ $page.post.author.title }}</g-link>
        </h2>
        <p>{{ $page.post.author.blurb }}</p>
      </div>
    </div>
  </Layout>
</template>

<page-query>
  query blog ($path: String){
    allPost (filter: {path: {in: [$path]}}) {
      edges {
        previous {
          title
        }
        next {
          title
        }
      }
    }
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
       title
       image
       blurb
     }
    }
  }
</page-query>

<style>
.icon.icon-link {
  display: none;
}

.content div {
  margin: auto;
  width: 60%;
  align-items: center;
}

.content div img {
  min-width: 100%;
}

.box.breadcrumbs {
  background-color: white;
  z-index: 99;
  top: 0;
  position: sticky;
}

.author {
  display: flex;
  flex-direction: column;
  gap: 20px;
  text-align: center;
}

.image {
  margin: auto;
}
</style>

<script>
export default {
  metaInfo() {
    return {
      title: `${this.$page.post.title} by ${this.$page.post.author.title}`,
    };
  },
};
</script>
