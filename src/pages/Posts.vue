<template>
  <div>
      <ul>
        <template v-for="edges in $page.allPost.edges">
            <li :key="edges.node.id">{{ edges.node.title }}</li>
        </template>

      </ul>
      <Pager :info="$page.allPost.pageInfo"/>
  </div>
</template>

<page-query>
query ($page:Int){
allPost(page:$page, perPage:20, sortBy:"date", order: DESC) @paginate {
    pageInfo{
      totalPages
      currentPage
    }
    edges{
      node{
        id
        title
        date
      }
    }
  }
}
</page-query>

<script>
import { Pager } from 'gridsome'

export default {
  metaInfo: {
    title: 'Posts'
  },
  components: {
    Pager
  }
}
</script>

<style>

</style>