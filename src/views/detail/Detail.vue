<template>
  <div id="detail">
    <detail-nav></detail-nav>
  </div>
</template>

<script>
import DetailNav from "./childComps/DetailNav";

export default {
  name: "Detail",
  components: {
    DetailNav
  },
  data() {
    return {
      iid: null
    };
  },
  created() {
    this.iid = this.$route.params.iid;
  }
};
</script>

<style scoped></style>
