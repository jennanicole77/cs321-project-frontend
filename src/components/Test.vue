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
       <div>
        <multiselect v-model="value" :options="multiTest">hello</multiselect>
      </div>
    </div>
</template>

<script>
import Multiselect from 'vue-multiselect'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  components: { Multiselect },
  data() {
    return {
        currPrice: null,
        currProfitability: null,
        options: null
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
      },
      multiTest: function() {
        self.options = ['list', 'of', 'options']
      }
  }
}
</script>


<style src="vue-multiselect/dist/vue-multiselect.min.css">
</style>