<template>
  <b-row>
    <b-col>
      <b-row class="border-bottom sticky-top page-header bg-white">
        <div class="d-flex align-items-center w-100 mx-3">
          <b-link @click="$router.go(-1)">
            <i class="bi-arrow-left" />
          </b-link>
          <h5 class="text-uppercase p-0 my-3 mx-2 flex-grow-1">
            {{ $t("general.bookmarks") }}
          </h5>
        </div>
      </b-row>
      <div v-if="posts[0] == null && !isLoading" class="empty-section"> No bookmarks yet!</div>
      <ui-posts v-model="posts" />
    </b-col>
  </b-row>
</template>
<style scoped lang="scss">
.empty-section{
   text-align: center;margin-top: 50%;font-weight: bold;color: #8a96a3c4;font-size: 1.2rem;line-height: 1.4rem;font-weight: 500;
}
</style>
<script>
import Post from "../models/Post";
import UiPosts from "../ui/UiPosts.vue";
export default {
  data: function () {
    return {
      posts: [],
      page: 1,
      hasMore: false,
      isLoading: false,
    };
  },
  mounted() {
    this.loadPosts();
    window.addEventListener("scroll", this.updateScroll);
  },
  components: {
    UiPosts,
  },
  methods: {
    updateScroll() {
      const scrollPosition = window.innerHeight + window.scrollY;
      if (
        document.body.offsetHeight &&
        scrollPosition &&
        document.body.offsetHeight - scrollPosition <= 1000 &&
        !this.isLoading &&
        this.hasMore
      ) {
        this.loadMore();
      }
    },
    loadPosts() {
      this.isLoading = true;
      if (this.page == 1) {
        this.$showSpinner();
      }
      this.$get(
        "/bookmarks?page=" + this.page,
        (data) => {
          let posts = [...this.posts];
          for (let obj of data.data) {
            posts.push(new Post(obj));
          }
          this.posts = posts;
          this.isLoading = false;
          this.hasMore = data.next_page_url != null;
        },
        (errors) => {
          console.log(errors);
        }
      );
    },
    loadMore() {
      if (this.hasMore) {
        this.page = this.page + 1;
        this.loadPosts();
      }
    },
  },
};
</script>
