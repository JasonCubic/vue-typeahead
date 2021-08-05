<template>
  <div ref="click-watcher-wrapper">
    <slot></slot>
  </div>
</template>

<script>
function handleDocumentClick(event) {
  const watchedChildren = Array.prototype.slice.call(this.$refs['click-watcher-wrapper'].children);
  const clickResults = watchedChildren.map((el) => el.contains(event.target));
  if (clickResults.some((result) => result === true)) {
    this.$emit('click-inside');
  } else {
    this.$emit('click-outside');
  }
}
function handleMounted() {
  document.addEventListener('click', this.handleDocumentClick);
}
function handleBeforeDestroy() {
  document.removeEventListener('click', this.handleDocumentClick);
}
export default {
  name: 'ClickWatcher',
  mounted: handleMounted,
  beforeDestroy: handleBeforeDestroy,
  methods: {
    handleDocumentClick,
  },
};
</script>
