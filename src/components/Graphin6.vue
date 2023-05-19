<template>
  <div class="hello">
    <h4>图布局 Layout</h4>
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
        // fitView: true,
        // fitViewPadding: [20, 40, 50, 20],
        layout: {
          // Object，可选，布局的方法及其配置项，默认为 random 布局。
          type: 'force', // 指定为力导向布局
          preventOverlap: true, // 防止节点重叠
          // nodeSize: 30        // 节点大小，用于算法中防止节点重叠时的碰撞检测。由于已经在上一节的元素配置中设置了每个节点的 size 属性，则不需要在此设置 nodeSize。
          linkDistance: 100, // 指定边距离为100
        },
        modes: {
          // 支持的behavior
          default: ['drag-canvas', 'zoom-canvas'],
          edit: ['click-select'],
        },
        // 节点在默认状态下的样式配置（style）和其他配置
        defaultNode: {
          size: 30, // 节点大小
          // 节点样式配置
          style: {
            fill: 'steelblue', // 节点填充颜色
            stroke: '#666', // 节点描边颜色
            lineWidth: 1, // 节点描边粗细
          },
          // 节点上的标签文本配置
          labelCfg: {
            style: {
              fill: '#fff', // 节点标签文字颜色
            },
          },
        },
        // 边在默认状态下的样式配置（style）和其他配置
        defaultEdge: {
          // style: {
          //   opacity: 0.6, // 边透明度
          //   stroke: 'grey',
          // },
          labelCfg: {
            autoRotate: true, // 边上的标签文本根据边的方向旋转
          }
        }
      });
      this.render();
    },
    async render() {
      const res = await fetch('/mock/data.json');
      this.initData = await res.json();
      const nodes = this.initData.nodes;
      nodes.forEach((node) => {
        if (!node.style) {
          node.style = {};
        }
        switch (node.class) {
          case 'c0': {
            node.type = 'circle';
            break;
          }
          case 'c1': {
            node.type = 'rect';
            node.size = [35, 20];
            break;
          }
          case 'c2': {
            node.type = 'ellipse';
            node.size = [35, 20];
            break;
          }
        }
      })
      const edges = this.initData.edges;
      edges.forEach((edge) => {
        if (!edge.style) {
          edge.style = {};
        }
        edge.style.lineWidth = edge.weight; // 边的粗细映射边数据中的 weight 属性数值
        edge.style.opacity = 0.6;
        edge.style.stroke = 'grey';
      })
      console.log('res---', this.initData);
      // 读取数据
      this.G6Instance.data(this.initData);
      // 渲染图
      this.G6Instance.render();
      // this.G6Instance.fitView();
    }
  },
  mounted() {
    this.graphinInit();
  },
}
</script>

<style scoped>
</style>
