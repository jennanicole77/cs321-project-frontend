<template>
    <div>
      <div>
        <button @click="test1">Click Me For Current Price of Ethereum</button>
        Data: {{currPrice}}
      </div>
       <div>
        <button @click="test2">Click Me For current  profitablility per 100Mhs of ethereum</button>
        Data: {{currProfitability}}
      </div>
    </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
        currPrice: null,
        currProfitability: null
    }
  },
  methods: {
      abort: function() {

      },
      test1: function() {
        let self = this;
        fetch('http://localhost:8080/eth-price')
        .then(response => response.json())
        .then(data => {
          self.currPrice = data.price
        });
      },
      test2: function() {
        let self = this;
        fetch('http://localhost:8080/profitability')
        .then(response => response.json())
        .then(data => {
          self.currProfitability = data.id
        });
      }
  }
}
</script>

<style scoped>

</style>