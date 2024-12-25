<template>
  <div class="home">
    <drag boundary :border="false" rotatable :width="200" :height="200" :left="0" :top="0">
      <img alt="Vue logo" src="../assets/logo.png" />
    </drag>
    <drag boundary :border="false" rotatable :width="200" :height="200" :left="0" :top="0">
      <img
        style="width:200px;height:200px;"
        src="https://img2.baidu.com/it/u=1178427128,2940697875&fm=253&fmt=auto&app=138&f=JPEG?w=500&h=625"
        alt
      />
    </drag>
    <drag boundary :border="false" rotatable :width="200" :height="200" :left="0" :top="0">这是一条文案</drag>
  </div>
</template>

<script>
// @ is an alias to /src
import drag from "@/components/drag/index.vue";
export default {
  name: "HomeView",
  components: {
    drag,
  },
};
</script>
<style scoped>
.home {
  height: 100vh;
}
</style>