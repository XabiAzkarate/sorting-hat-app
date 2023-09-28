<template>
  <div :class="['message-container', sender]">
    <div :class="`message-content ${sender}`">
      {{ content }}
    </div>
  </div>
</template>

<script>
export default {
  props: ['content', 'sender']
}
</script>

<style scoped>
.message-container {
  display: flex;
  justify-content: flex-end;
  margin: 5px;
}

.message-container.hat {
  justify-content: flex-start; /* Align to left for hat messages */
}

.message-content {
  display: inline-block;
  padding: 10px 20px;
  max-width: 80%;
}

.message-content.user {
  background-color: #270d3b;
  color: white;
  align-self: flex-end;
  border-radius: 10px 10px 0 10px;
}

.message-content.hat {
  background-color: #9e9c9c9d;
  color: black;
  align-self: flex-start;
  justify-content: flex-start;
  border-radius: 10px 10px 10px 0;
}
</style>
