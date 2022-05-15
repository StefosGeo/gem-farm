<template>
  <div class="nes-container huVjiU with-title p-0" >
    <div class="text-center title" :style="{backgroundColor: bg_color}">
      <p>{{ title }}</p>
    </div>
    <slot />
    <div class="flex flex-wrap">
      <NFTCard
        v-for="nft in nfts"
        :key="nft"
        :nft="nft"
        @selected="handleSelected"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import NFTCard from '@/components/gem-bank/NFTCard.vue';

export default defineComponent({
  components: { NFTCard },
  emits: ['selected'],
  props: {
    title: String,
    nfts: Array,
    bg_color: String,
  },
  setup(props, ctx) {
    const handleSelected = (e: any) => {
      ctx.emit('selected', e);
    };
    return {
      handleSelected,
    };
  },
});
</script>

<style>
.title {
  background-color: #474747;
  width: 100%;
  border-top-left-radius: 16px;
  border-top-right-radius: 16px;
  width: 100%;
  color: white;
}
</style>
