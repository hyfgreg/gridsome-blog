<template>
  <div>
    <h1 v-html="$page.post.title" />
    <div v-html="$page.post.content" />
  </div>
</template>

<page-query>
query ($id: ID!) {
  post(id: $id) {
    title
    content
  }
}
</page-query>