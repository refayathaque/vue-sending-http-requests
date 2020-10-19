<template>
  <h1>vue-sending-http-requests</h1>
  <form @submit.prevent="submitForm">
    <label for="greeting">Enter a greeting</label>
    <input id="greeting" name="greeting" type="text" v-model="greeting" />
    <button type="submit">Send this to AWS Lambda</button>
  </form>
  <button type="button" @click="fetchFromAWSLambda">Fetch from AWS Lambda</button>
  <p>{{ returnedFromAWSLambda }}</p>
</template>

<script>
import { awsResources } from './aws'

export default {
  data() {
    return {
      greeting: '',
      returnedFromAWSLambda: ''
    }
  },
  methods: {
    async fetchFromAWSLambda() {
      console.log('hello')
      const response = await fetch(awsResources.apiGatewayEndpoint)
      // don't need config object for GET, because it's the default and since we're not sending anything we don't need body and therefore don't need headers
      if (response.ok) {
        const data = await response.json();
        console.log(data)
        this.returnedFromAWSLambda = data.body
      } else {
        console.log('Go fix something in AWS')
      }
    },
    async submitForm() {
      console.log(this.greeting)
      const response = await fetch(awsResources.apiGatewayEndpoint, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ greeting: this.greeting }),
      })
      if (response.ok) {
        const data = await response.json();
        console.log(data)
      } else {
        console.log('Go fix something in AWS')
      }
    }
  }
}
</script>
