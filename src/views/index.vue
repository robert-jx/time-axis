<template>
  <section class="time-axis">
    <section class="content">
      <timeAxisView title="星期日" :isRedTitle="true"></timeAxisView>
      <timeAxisView title="星期一" :isRedTitle="true"></timeAxisView>
      <timeAxisView title="星期二" :isRedTitle="true"></timeAxisView>
      <timeAxisView title="星期三" :isRedTitle="true"></timeAxisView>
      <timeAxisView title="星期四" :isRedTitle="true"></timeAxisView>
      <timeAxisView title="星期五" :isRedTitle="true"></timeAxisView>
      <timeAxisView title="星期六" :isRedTitle="true"></timeAxisView>
    </section>
  </section>
</template>

<script>
// @ is an alias to /src
import timeAxisView from '@/components/time-axis/index.vue'

export default {
  name: 'timeAxis',
  components: {
    timeAxisView
  }
}
</script>
<style lang="scss" scoped>
.time-axis {
  width: 100%;
  height: 100%;
  box-sizing: border-box;

  .content {
    width: 50%;
    margin: auto;
  }
}
</style>
