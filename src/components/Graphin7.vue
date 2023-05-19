<template>
  <div class="hello">
    <h4>交互Behavior</h4>
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
          default: ['drag-canvas', 'zoom-canvas', 'drag-node'],
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
        },
        // 节点不同状态下的样式集合
        nodeStateStyles: {
          // 鼠标 hover 上节点，即 hover 状态为 true 时的样式
          hover: {
            fill: 'lightsteelblue',
          },
          // 鼠标点击节点，即 click 状态为 true 时的样式
          click: {
            stroke: '#000',
            lineWidth: 3,
          },
        },
        // 边不同状态下的样式集合
        edgeStateStyles: {
          // 鼠标点击边，即 click 状态为 true 时的样式
          click: {
            stroke: 'steelblue',
          }
        }
      });

      // 鼠标进入节点
      this.G6Instance.on('node:mouseenter', (e) => {
        const nodeItem = e.item; // 获取鼠标进入的节点元素对象
        this.G6Instance.setItemState(nodeItem, 'hover', true); // 设置当前节点的 hover 状态为 true
      });

      // 鼠标离开节点
      this.G6Instance.on('node:mouseleave', (e) => {
        const nodeItem = e.item; // 获取鼠标离开的节点元素对象
        this.G6Instance.setItemState(nodeItem, 'hover', false); // 设置当前节点的 hover 状态为 false
      });

      // 点击节点
      this.G6Instance.on('node:click', (e) => {
        // 先将所有当前是 click 状态的节点置为非 click 状态
        const clickNodes = this.G6Instance.findAllByState('node', 'click');
        clickNodes.forEach((cn) => {
          this.G6Instance.setItemState(cn, 'click', false);
        });
        const nodeItem = e.item; // 获取被点击的节点元素对象
        this.G6Instance.setItemState(nodeItem, 'click', true); // 设置当前节点的 click 状态为 true

      })
      // 点击边
      this.G6Instance.on('edge:click', (e) => {
        // 先将所有当前是 click 状态的边置为非 click 状态
        const clickEdges = this.G6Instance.findAllByState('edge', 'click');
        clickEdges.forEach((ce) => {
          this.G6Instance.setItemState(ce, 'click', false);
        });
        const edgeItem = e.item; // 获取被点击的边元素对象
        this.G6Instance.setItemState(edgeItem, 'click', true); // 设置当前边的 click 状态为 true
      })

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
