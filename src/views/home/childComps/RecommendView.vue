<template>
  <div class="recommend">
    <div v-for="item in recommends" class="reitem">
      <a :href="item.link">
        <img :src="item.image" alt />
        <div>{{item.title}}</div>
      </a>
    </div>
  </div>
</template>

<script>
export default {
  naem: "RecommendView",
  props: {
    recommends: {
      type: Array,
      default() {
        return [];
      }
    }
  }
};
</script>

<style scoped>
.recommend {
  display: flex;
  width: 100%;
  text-align: center;
  font-style: 12px;
  padding: 10px 0 20px;
  border-bottom: 110px solid #eee;
}
.reitem {
  flex: 1;
}
.reitem img {
  width: 70px;
  height: 70px;
  margin-bottom: 10px;
}
</style>
