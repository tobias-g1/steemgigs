<template>
  <page :pageClasses="['categories__view', 'row']">
    <cat-nav />
    <el-main>
        <el-row :gutter="15">
            <categoryPreview mode="pages" :post_type="pageDetails.post_type" :limit="8" :header="pageDetails.title" :description="pageDetails.description"></categoryPreview>
        </el-row>
    </el-main>
  </page>
</template>

<script>
import Page from '@/components/page'
import CatNav from '@/components/layout/catNav'
import CategoryPreview from '@/components/snippets/category-preview'

export default {
  components: {
    Page,
    CatNav,
    CategoryPreview
  },
  computed: {
    pageDetails () {
      switch (this.$route.path) {
        case '/steemgigs':
          return {
            title: 'Steem Gigs',
            description: 'See what the SteemGigs community has to offer',
            post_type: 'steemgigs_post'
          }
        case '/requested_gigs':
          return {
            title: 'Requested Gigs',
            description: 'Gig requests provided by our community',
            post_type: 'steemgigs_request'
          }
        case '/testimonials':
          return {
            title: 'Testimonials',
            description: 'SteemGigs success stories written by our users',
            post_type: 'steemgigs_testimonial'
          }
        case '/micro_tasks':
          return {
            title: 'Micro Tasks',
            description: 'A microtask can take different forms. Participants are required to carry out a simple action e.g join a telegram, subscribe to my YouTube, etc',
            post_type: 'microtask_post'
          }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
