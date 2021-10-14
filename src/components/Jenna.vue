<template>
    <div>
        <button @click="test">Click Me </button>
        Data: {{meow}}
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
        meow: null
    }
  },
  methods: {
      abort: function() {

      },
      test: function() {
        let self = this;
        fetch('http://localhost:8080/eth-price')
        .then(response => response.json())
        .then(data => {
          self.meow = data.price
        });
      }
  }
}
</script>

<style scoped>

</style>