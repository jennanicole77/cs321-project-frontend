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
          this.meow = new Date();
      }
  }
}
</script>

<style scoped>

</style>