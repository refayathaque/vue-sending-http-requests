<template>
  <h1>vue-sending-http-requests</h1>
  <form @submit.prevent="submitForm">
    <label for="greeting">Enter a greeting</label>
    <input id="greeting" name="greeting" type="text" v-model="greeting" />
    <button type="submit">Send this to AWS Lambda</button>
  </form>
</template>

<script>
import { awsResources } from './aws'

export default {
  data() {
    return {
      greeting: ''
    }
  },
  methods: {
    submitForm() {
      console.log(this.greeting)
      fetch(awsResources.apiGatewayEndpoint, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ greeting: this.greeting }),
      })
    }
  }
}
</script>
