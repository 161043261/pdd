<template>
  <div id="app">
    <main-content></main-content>
  </div>
</template>

<script>
import mainContent from "./components/main-content";

export default {
  name: "app",
  components: {
    "main-content": mainContent,
  },
};
</script>
