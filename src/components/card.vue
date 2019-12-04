<template>
  <div>
    <div class="card">
      <p>{{text.desc}}</p>
      <img :src="text.imgUrl" />
    </div>
  </div>
</template>

<script>
export default {
  props: ["text"]
};
</script>

<style>
.card {
  display: flex;
  padding: 20rpx;
  text-align: center;
  /* height: 570rpx; */
}
.card img {
  width: 640rpx;
  height: 480rpx;
}
</style>

