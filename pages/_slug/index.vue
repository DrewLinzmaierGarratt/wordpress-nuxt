<template>
  <section div class="page page--home">
    <div class="container">
      <div class="hero flex">
        <h1>{{ page.title }}</h1>
      </div>
    </div>
    <div class="projects container">
      <div class="projects__flex flex">
        <img class="col lazyload" data-src="https://via.placeholder.com/350x150">
        <img class="col" src="https://via.placeholder.com/350x150">
        <img class="col" src="https://via.placeholder.com/350x150">
      </div>
    </div>
  </section>
</template>

<script>
import { mapState } from 'vuex'

export default {
  async asyncData({ app, store, params, error }) {
    const res = await app.$axios.get(
      `${store.state.wordpressAPI}/wp/v2/pages?slug=${
        params.slug
      }&status=publish&_embed`
    )
    if (res.data.length > 0) {
      if (res.data[0].parent === 0) {
        store.commit('setCurrentPage', res.data)
      } else {
        error({ statusCode: 404, message: 'Page not found' })
      }
    } else {
      error({ statusCode: 404, message: 'Page not found' })
    }
  },

  transition() {
    return 'fade'
  },

  head() {
    return {
      title: `${this.page.title} | ${this.$store.state.meta.name}`,
      meta: [{ description: this.page.excerpt }]
    }
  },

  computed: {
    ...mapState({
      page: state => state.currentPage
    })
  },

  mounted() {
    if (this.page.preview) {
      this.preview()
    }
  },

  methods: {
    preview() {
      this.$axios
        .get(
          `${this.$store.state.wordpressAPI}/postlight/v1/post/preview?id=${
            this.$route.query.id
          }&_wpnonce=${this.$route.query.token}`,
          { withCredentials: true }
        )
        .then(response => {
          this.$store.commit('setCurrentPage', response.data)
        })
    }
  }
}
</script>

<style lang="scss" scoped>
.hero {
  width: 100%;
  height: 80vh;
  min-height: em(400px);
  align-items: center;

  h1 {
    margin: auto;
  }
}

.projects {
  &__flex {
    flex-wrap: wrap;
  }

  .col {
    width: 33.33%;
  }
}
</style>
