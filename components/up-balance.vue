<template>
  <div class="up-balance">
    <div class="icon-box">
      <img
        class="icon"
        :src="require(`@/assets/img/${name}.svg`)"
        :alt="name"
      />
      <img class="chain" src="@/assets/img/BSC.svg" alt="BSC" />
    </div>
    <span>{{ name }}</span>
  </div>
</template>
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  props: {
    name: {
      type: String,
      // BNB USDT USDC ETH
      default: 'BNB',
    },
  },
})
</script>

<style lang="scss">
.up-balance {
  display: flex;
  align-items: center;
  .icon-box {
    position: relative;
    width: 28px;
    height: 28px;
    margin-right: 8px;
    .icon {
      width: 100%;
      height: 100%;
    }
    .chain {
      position: absolute;
      right: -4px;
      bottom: 0;
      width: 16px;
      height: 16px;
    }
  }
}
</style>
