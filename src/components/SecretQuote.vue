<template>
  <div class="col-sm-6 col-sm-offset-3">
    <h1>Get a Secret Chuck Norris Quote!</h1>
    <button class="btn btn-warning" v-on:click="getQuote()">Get a Quote</button>
    <div class="quote-area" v-if="quote">
      <h2><blockquote>{{ quote }}</blockquote></h2>      
    </div>
  </div>
</template>

<script>
import auth from '../auth'

export default {

  data() {
    return {
      quote: ''
    }
  },

  methods: {
    getQuote() {
      this.$http
        .get('http://localhost:3001/api/protected/random-quote', (data) => {
          this.quote = data;
        }, { 
          headers: auth.getAuthHeader()
        })
        .error((err) => console.log(err))
    }
  },

  route: {
    canActivate() {
      return auth.user.authenticated
    }
  }

}
</script>