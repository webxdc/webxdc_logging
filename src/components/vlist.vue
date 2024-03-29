<!-- Taken from https://github.com/waningflow/vue3-virtual-list -->

<template>
  <div
    class="vue3-virtual-list-container"
    ref="root"
    @scroll.passive="handleScroll"
  >
    <table
      class="vue3-virtual-list-scroll"
      :style="`height: ${scrollHeight}px;padding-top: ${paddingTop}px`"
    >
      <tr
        v-for="item in pool"
        :key="item[dataKey]"
        :style="`height: ${itemSize}px`"
      >
        <slot :item="item" :index="item._index"></slot>
      </tr>
    </table>
  </div>
</template>

<script lang="ts">
import { ref, toRefs, defineComponent, onMounted, watch, Ref } from "vue";

interface Props {
  data: any[];
  dataKey: string;
  itemSize: number;
  poolBuffer: number;
}

export default defineComponent({
  name: "VirtualList",
  props: {
    data: {
      type: Array,
      default: () => []
    },
    dataKey: {
      type: String,
      default: () => "id"
    },
    itemSize: {
      type: Number,
      default: () => 40
    },
    poolBuffer: {
      type: Number,
      default: () => 30
    }
  },
  setup(props: Props): any {
    const { data, poolBuffer, itemSize } = toRefs(props);
    const root = ref<HTMLElement>() as Ref<HTMLElement>;
    const pool = ref<any[]>([]);
    const scrollHeight = ref(data.value.length * itemSize.value);
    watch(data, cData => {
      scrollHeight.value = cData.length * itemSize.value;
    });

    watch(itemSize, cItemSize => {
      scrollHeight.value = data.value.length * cItemSize;
    });

    let containerSize = 0;
    const paddingTop = ref(0);
    let isScrollBusy = false;

    const handleScroll = () => {
      if (isScrollBusy) return;
      isScrollBusy = true;

      requestAnimationFrame(() => {
        isScrollBusy = false;
        const range: number[] = [];
        range[0] =
          Math.floor(root.value.scrollTop / itemSize.value) -
          Math.floor(poolBuffer.value / 2);
        range[0] = Math.max(range[0], 0);
        range[1] =
          range[0] +
          Math.floor(root.value.clientHeight / itemSize.value) +
          poolBuffer.value;
        range[1] = Math.min(range[1], data.value.length);
        pool.value = data.value
          .slice(range[0], range[1])
          .map((v, i) => ({ ...v, _index: range[0] + i }));
        paddingTop.value = range[0] * itemSize.value;
      });
    };
    onMounted(() => {
      containerSize = root.value.clientHeight;
      const contentLines = Math.ceil(containerSize / itemSize.value);
      const totalLines = contentLines + poolBuffer.value;
      const range = [0, totalLines];
      pool.value = data.value
        .slice(range[0], range[0] + range[1])
        .map((v, i) => ({ ...v, _index: range[0] + i }));
    });

    return { pool, scrollHeight, root, handleScroll, paddingTop };
  }
});
</script>

<style scoped lang="sass">
.vue3-virtual-list-container 
  width: 100%
  height: 100%
  min-width: 100px
  min-height: 100px
  overflow: hidden
  @apply p-2

.vue3-virtual-list-item-container 
  overflow: hidden


</style>