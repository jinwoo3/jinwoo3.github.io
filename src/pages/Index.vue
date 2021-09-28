<template>
  <Layout pageTitle="Soomgo Tech" pageSubtitle="숨고 테크팀의 기술 블로그">
    <div class="section">
      <div class="container">
        <div class="columns">
          <div class="column cards-grid is-four-fifths">
            <article class="card grid-item" v-for="edge in $page.posts.edges" :key="edge.node.id">
              <div class="card-image">
                <g-link :to="edge.node.path">
                  <figure class="image is-16by9">
                    <img
                      class="content-image"
                      :src="`${edge.node.featuredImage}`"
                      :alt="`${edge.node.title} image`"
                    />
                  </figure>
                </g-link>
              </div>

              <div class="card-content">
                <div class="media">
                  <div class="media-left">
                    <g-link :to="edge.node.author.path">
                      <figure class="image is-48x48">
                        <img
                          class="is-rounded"
                          :src="`../${edge.node.author.image}`"
                          alt="Placeholder image"
                        />
                      </figure>
                    </g-link>
                  </div>
                  <div class="media-content">
                    <h2 class="title is-4">
                      <g-link :to="edge.node.path">
                        {{ edge.node.title }}
                      </g-link>
                    </h2>
                    <p class="subtitle is-6">
                      <g-link :to="edge.node.author.path">
                        {{ edge.node.author.title }}
                      </g-link>
                    </p>
                  </div>
                </div>

                <div class="content">
                  {{ edge.node.excerpt }}
                </div>

                <div class="tags">
                  <span v-for="tag in edge.node.tags" :key="tag.id">
                    <div class="tag">
                      <g-link :to="tag.path">
                        {{ tag.id }}
                      </g-link>
                    </div>
                  </span>
                </div>
              </div>
            </article>
          </div>
        </div>
      </div>
      <Pager class="pager-container" :info="$page.posts.pageInfo" />
    </div>
  </Layout>
</template>

<page-query>
  query($page: Int) {
    posts: allPost(perPage: 9, page: $page) @paginate {
      pageInfo {
        totalPages
        currentPage
      }
      edges {
        node {
          id
          title
          path
          excerpt
          featuredImage
          author {
            id
            title
            path
            image
          }
          tags {
            id
            path
          }
        }
      }
    }
  }
</page-query>

<script>
import { Pager } from 'gridsome';

export default {
  components: {
    Pager,
  },
  metaInfo() {
    return {
      title: '숨고 기술 블로그',
    };
  },
};
</script>

<style>
.pager-container {
  margin: 0 auto;
  width: max-content;
  text-align: center;
  gap: 20px;
  display: flex;
}

.tags {
  gap: 5px;
}

.column {
  width: 100% !important;
}

.title.is-4 a {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

.card {
  margin-bottom: 1.5rem;
}

.content-image {
  object-fit: cover;
}

.cards-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  list-style: none;
  width: 100%;
  justify-content: center;
}

/* Mobile First */
.grid-item {
  width: 100%;
  transition: all 0.2s ease-in-out;
}
/*Medium Width */
@media screen and (min-width: 740px) {
  .grid-item {
    width: calc((100% / 2) - 30px);
  }
}
/*Wide Width */
@media screen and (min-width: 991px) {
  .grid-item {
    width: calc((100% / 3) - 30px);
  }
}
</style>
