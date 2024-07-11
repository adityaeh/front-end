<template>
  <div class="toast-container">
    <div v-for="error in errors" :key="error.id" class="toast-error">
      {{ error }}
    </div>
  </div>
</template>

<script>
export default {
  name: "ToastErrors",
  props: {
    errors: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {};
  },
};
</script>

<style scoped>
.toast-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 9999;
}

.toast-error {
  padding: 10px;
  margin-bottom: 10px;
  background-color: #f44336;
  color: #fff;
  border-radius: 4px;
}
</style>
