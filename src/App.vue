<template>
  <h1>vue-sending-http-requests</h1>
  <form @submit.prevent="submitForm">
    <label for="greeting">Enter a greeting</label>
    <input id="greeting" name="greeting" type="text" v-model="greeting" />
    <button type="submit">Send this to AWS Lambda</button>
  </form>
  <p>{{ greetingSentToAWS }}</p>
  <button type="button" @click="fetchFromAWSLambda">Fetch from AWS Lambda</button>
  <p v-if="isLoading">Loading...</p>
  <p v-else-if="!isLoading && isNetworkError">Network error</p>
  <p v-else-if="!isLoading && isServerSideError">Server-side (AWS) error</p>
  <p v-else-if="!isLoading && !fetchedFromAWS">Nothing returned from AWS Lambda</p>
  <p v-else>{{ fetchedFromAWS }}</p>
</template>

<script>
import { awsResources } from './aws'

export default {
  data() {
    return {
      greeting: '',
      greetingSentToAWS: 'Nothing sent to AWS Lambda yet',
      fetchedFromAWS: '',
      isLoading: false,
      isNetworkError: false,
      isServerSideError: false
    }
  },
  methods: {
    async fetchFromAWSLambda() {
      this.isLoading = true
      this.isNetworkError = false // important to reset for subsequent API calls
      this.isServerSideError = false
      let response
      try {
        response = await fetch(awsResources.apiGatewayEndpoint)
        // don't need config object for GET, because it's the default and since we're not sending anything we don't need body and therefore don't need headers
        if (response.ok) { // the `ok` and `status` keys in the response object can be checked for "server-side" errors, statuses of 400s (client error) and 500s (server error), https://httpstatuses.com/
          const data = await response.json();
          this.isLoading = false
          this.fetchedFromAWS = data.body
        } else { // will run when there is a "server-side"/AWS error, when `response.ok === false`
          this.isLoading = false
          this.isServerSideError = true
          console.log('Go fix something in AWS')
        }
      } catch(e) { // test this by changing `apiGatewayEndpoint`
        this.isLoading = false
        this.isNetworkError = true
        console.log(e, 'Something is wrong with the API call itself')
      }
    },
    // ^ should be design pattern for all RESTful API calls, i.e., use ^ for POSTs like what's below as well
    async submitForm() {
      const response = await fetch(awsResources.apiGatewayEndpoint, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ greeting: this.greeting }),
      })
      console.log(response)
      if (response.ok) {
        const data = await response.json();
        console.log(data)
        this.greetingSentToAWS = data.body
      } else {
        console.log('Go fix something in AWS')
      }
    }
  },
  async mounted() {
    // using ^ lifecycle method we can make API call and load data when the component mounts
    this.fetchFromAWSLambda()
    const response = await fetch('https://jsonplaceholder.typicode.com/todos/gsfdgdg')
    // const response = await fetch('https://jsonplaceholder.typicode.com/todos/thiswillnotwork')
    console.log(response)
    // ^ to test for server-side errors look at `response.ok` and `response.status`
  }
}
</script>
