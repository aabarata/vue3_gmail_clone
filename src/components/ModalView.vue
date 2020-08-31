<template>
  <div class="modal">
    <div class="overlay" @click="$emit('closeModal')"></div>
    <div class="modal-card">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ModalView',
  setup(props, {emit}) {
    return {
      emit
    }
  }
}
</script>