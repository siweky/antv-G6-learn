<template>
  <div class="hello">
    <h4>配置</h4>
    <div id="graphin"></div>
  </div>
</template>

<script>
import G6 from '@antv/g6';
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      G6Instance: {},
      initData: {
        // 点集
        nodes: [
          {
            id: 'node1', // 节点的唯一标识
            x: 100, // 节点横坐标
            y: 200, // 节点纵坐标
            label: '起始点1', // 节点文本
          },
          {
            id: 'node2',
            x: 300,
            y: 200,
            label: '目标点',
          },
        ],
        // 边集
        edges: [
          // 表示一条从 node1 节点连接到 node2 节点的边
          {
            source: 'node1', // 起始点 id
            target: 'node2', // 目标点 id
            label: '我是连线', // 边的文本
          },
        ],
      }
    }
  },
  methods: {
    graphinInit() {
      this.G6Instance = new G6.Graph({
        container: 'graphin',
        width: 800,
        height: 500,
        fitView: true,
        fitViewPadding: 20,
        animate: true,
        modes: {
          // 支持的behavior
          default: ['drag-canvas', 'zoom-canvas'],
          edit: ['click-select'],
        }
      });
      this.render();
      // this.G6Instance.fitViewPadding(40);
    },
    async render() {
      const res = await fetch('/mock/data.json');
      this.initData = await res.json();
      console.log('res---', this.initData);
      // 读取数据
      this.G6Instance.data(this.initData);
      // 渲染图
      this.G6Instance.render();
      // this.G6Instance.fitViewPadding = 100;
      this.G6Instance.fitView();
    }
  },
  mounted() {
    this.graphinInit();
  },
}
</script>

<style scoped>
</style>
