<template>
  <div>
    <b-alert
      variant="success"
      dismissible
      v-model="localFlag"
      @dismissed='close()'
    >
      {{ message }}
    </b-alert>
  </div>
</template>

<script>
export default {
  props: ['message', 'value'],
  data() {
    return {
      localFlag: false,
    };
  },
  methods: {
    close() {
      this.$emit('input', false);
    },
  },
  watch: {
    value() {
      this.localFlag = this.value;
    },
  },
  mounted() {
    this.localFlag = this.value;
  },
};
</script>
