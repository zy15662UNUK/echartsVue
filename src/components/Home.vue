<template>
  <div id="home">
    <chart></chart>
  </div>
</template>
<script>
import chart from "./Chart.vue"
export default {
  name: "Chart",
  data: function() {
    return {

    };
  },
  components: {
    "chart": chart
  }
}
</script>
<style scoped>
</style>
