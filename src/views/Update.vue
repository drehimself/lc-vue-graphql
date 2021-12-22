<template>
  <div>
    <div class="error">{{ error }}</div>
    <form action="#" @submit.prevent="updatePost">
      <div>
        <label for="title">Title</label>
        <input v-model="title" type="text" name="title" id="title" />
      </div>
      <div>
        <label for="body">Body</label>
        <textarea
          v-model="body"
          name="body"
          id="body"
          cols="30"
          rows="10"
        ></textarea>
      </div>
      <div>
        <button>Update Post</button>
        <div v-if="loading">Loading...</div>
      </div>
    </form>
  </div>
</template>

<script>
// @ is an alias to /src
import gql from 'graphql-tag'

export default {
  name: 'Post',
  data() {
    return {
      title: '',
      body: '',
      error: null,
      loading: false,
    }
  },
  watch: {
    post(newPost) {
      this.title = newPost.title
      this.body = newPost.body

      if (newPost.user.id !== this.me.id) {
        window.location.href = '/'
      }
    },
  },
  apollo: {
    me: {
      query: gql`
        query {
          me {
            id
          }
        }
      `,
    },
    post: {
      query: gql`
        query getPost($id: ID!) {
          post(id: $id) {
            title
            body
            user {
              id
            }
          }
        }
      `,
      variables() {
        return {
          id: this.$route.params.id,
        }
      },
    },
  },
  methods: {
    updatePost() {
      this.loading = true

      this.$apollo
        .mutate({
          mutation: gql`
            mutation updatePost($id: ID!, $title: String!, $body: String!) {
              updatePost(id: $id, title: $title, body: $body) {
                id
                title
              }
            }
          `,
          variables: {
            id: this.$route.params.id,
            title: this.title,
            body: this.body,
          },
        })
        .then(data => {
          console.log(data)
          this.loading = false
          this.$router.push({ name: 'Home' })
        })
        .catch(error => {
          console.log(error.graphQLErrors)
          this.loading = false
          const key = Object.keys(
            error.graphQLErrors[0].extensions.validation
          )[0]
          this.error = error.graphQLErrors[0].extensions.validation[key][0]
        })
    },
  },
}
</script>
