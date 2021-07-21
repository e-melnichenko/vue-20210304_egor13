<template>
  <button @click="handleClick">{{ localCount }}</button>
</template>

<script>
export default {
  name: 'CounterButton',

  props: {
    count: {
      type: Number,
      default: 0,
    }
  },

  model: {
    prop: 'count',
    event: 'increment',
  },

  data() {
    return {
      localCount: this.count,
    }
  },

  watch: {
    count(newValue) {
      this.localCount = newValue;
    }
  },

  methods: {
    handleClick() {
      this.$emit('increment', this.localCount += 1);
    }
  }
};
</script>
