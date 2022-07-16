<template>
  <div>
    <h1>Movies by tag</h1>
    <h2 v-if="loadingReviews || loading">Loading reviews in #{{ $route.params.tag }}</h2>
    <h2 v-if="result">Reviews in #{{ $route.params.tag }} </h2>
    <p v-if="error">Something went wrong... </p>
    <ReviewList v-if="!loadingReviews && result" :reviews="result.reviewsByTag" />
  </div>
</template>

<script>
import gql from 'graphql-tag'
import { useQuery } from '@vue/apollo-composable'
import { useRoute } from 'vue-router'
import ReviewList from '@/components/ReviewList.vue'

const REVIEWS_BY_TAG_QUERY = gql`
  query ($tag: String!) {
    reviewsByTag(tag: $tag) {
      title
      movie {
        title
        slug
      }
      publishDate
      published
      metaDescription
      slug
      tags {
        name
      }
    }
  }`

export default {
  name: 'ReviewsByTag',
  components: {
    ReviewList,
  },
  data () {
    return {
      reviews: [],
      loadingReviews: false,
    }
  },
  apollo: {
    reviews: {
      query: REVIEWS_BY_TAG_QUERY,
        variables() {
          return { tag: this.$route.params.tag };
        }, 
    },
  },
  created() {
    this.$watch(
      () => this.$route.params,
      (toParams, previousParams) => {
        if (toParams !== previousParams) {
          this.loadingReviews = true;
          //This is incredibly hacky and should be fixed.
          setTimeout(location.reload(), 2000);
        }
      }
    )
  },
  setup() {
    const tag = useRoute().params.tag.toString();
    const { result, loading, error } = useQuery(REVIEWS_BY_TAG_QUERY, () => ({
      tag: tag,
    }));
    return {
      result,
      loading,
      error
    };
  },
}
</script>