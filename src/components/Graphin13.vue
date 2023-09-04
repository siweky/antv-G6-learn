<template>
  <div class="container">
    <div id="graphin-main"></div>
    <div id="minimap-box"></div>
  </div>
</template>

<script>
import G6 from '@antv/g6';
import { GraphLayoutPredict } from '@antv/vis-predict-engine';
export default {
  name: 'DataFlowLinkDepict',
  data() {
    return {
      icon: {
        account: require('@/assets/icon/account.svg'),
        accountAct: require('@/assets/icon/account-act.svg'),
        app: require('@/assets/icon/app.svg'),
        appAct: require('@/assets/icon/app-act.svg'),
        net: require('@/assets/icon/net.svg'),
        netAct: require('@/assets/icon/net-act.svg'),
        db: require('@/assets/icon/db.svg'),
        dbAct: require('@/assets/icon/db-act.svg'),
        avatar: require('@/assets/icon/avatar.jpg'),
      },
      G6Instance: {},
      initData: {
        // 点集
        nodes: [
          {
            id: 'node1',
            x: 100,
            y: 200,
          },
          {
            id: 'node2',
            x: 300,
            y: 200,
          },
        ],
        // 边集
        edges: [
          // 表示一条从 node1 节点连接到 node2 节点的边
          {
            source: 'node1',
            target: 'node2',
            label: '我是连线', // 边的文本
          },
        ],
      }
    }
  },
  methods: {
    async graphinInit() {
      // const _this = this;
      const container = document.getElementById('graphin-main');
      const w = container.clientWidth - 8;
      const h = container.clientHeight - 8;

      // 创建minimap实例
      const minimap = new G6.Minimap({
        container: 'minimap-box',
        size: [100, 100],
        // className: 'minimap',
        type: 'delegate',
      })
      // 创建grid实例
      const grid = new G6.Grid();
      // predictLayout 表示预测的布局，如 force 或 radial
      // confidence 表示预测的可信度
      const { predictLayout, confidence } = await GraphLayoutPredict.predict(this.initData);
      console.log("----predictLayout---", predictLayout);
      console.log("----confidence---", confidence);
      // 创建 G6 图实例
      this.G6Instance = new G6.Graph({
        container: 'graphin-main', // 指定图画布的容器 id
        // 画布宽高
        width: w,
        height: h,
        renderer: 'canvas',
        layout: {
          type: predictLayout,
          linkDistance: 200,         // 可选，边长
          // maxIteration: 1000,       // 可选
          // unitRadius: 200,          // 可选
          nodeSize: 50,             // 可选
          preventOverlap: true,     // 可选，必须配合 nodeSize
          nodeSpacing: 0,
          strictRadial: true,       // 可选
          workerEnabled: true,       // 可选，开启 web-worker
        },
        // fitView: true,
        // fitViewPadding: [0, 0, 0, 0],
        animate: true,
        plugins: [minimap, grid],
        modes: {
          default: [
            'drag-node',
            'drag-canvas',
            'activate-relations',
            {
              type: 'zoom-canvas',
              minZoom: 0.2,
              maxZoom: 2,
              enableOptimize: true,
            },
            {
              type: 'tooltip', // 提示框
              formatText(model) {
                // console.log('tooltip:', model)
                // 提示框文本内容
                const text = 'label: ' + model.label + '<br/> class: ' + model.class;
                return text;
              },
            },
            {
              type: 'edge-tooltip', // 边提示框
              formatText(model) {
                // 边提示框文本内容
                const text = 'source: ' + model.source + '<br/> target: ' + model.target + '<br/> weight: ' + model.weight;
                return text;
              },
            },
          ]
        },
        defaultNode: {
          type: 'circle',
          style: {
            lineWidth: 1,
            stroke: '#8297AF',
            fill: '#f2f4f7',
            opacity: 1,
            // shadowColor: '#ffffff',
            // shadowBlur: 5,
          },
        },
        defaultEdge: {
          type: 'line',
          size: 1,
          // color: '#8297AF',
          labelCfg: {
            // 边上的标签文本配置
            autoRotate: true, // 边上的标签文本根据边的方向旋转
          },
          style: {
            endArrow: {
              // path: 'M 0,0 L 4,2 L 4,-2 Z',
              path: G6.Arrow.triangle(5, 5, 0),
              lineWidth: 3,
            },
            lineAppendWidth: 10,
          },
        },
        nodeStateStyles: {
          hover: {
            // stroke: '#EF3E3E',
            // fill: '#fdebeb',
            // lineWidth: 1,
          },
          click: {
            // stroke: '#EF3E3E',
            // fill: '#fdebeb',
            // lineWidth: 1,
          },
          select: {},
          // acctive: {
          //   opacity: 1,
          // },
          // inactive: {
          //   opacity: 0.2,
          // },
        },
        edgeStateStyles: {
          // hover: {
          //   stroke: 'steelblue'
          // },
          // click: {
          //   stroke: 'steelblue'
          // },
          // select: {},

        }
      });


      // 鼠标进入节点
      this.G6Instance.on('node:mouseenter', (e) => {
        console.log('node:mouseenter', e, { ...e.item._cfg.model.icon });
        const nodeItem = e.item; // 获取鼠标进入的节点元素对象
        // const itemModel = nodeItem._cfg.model;
        this.G6Instance.setItemState(nodeItem, 'hover', true); // 设置当前节点的 hover 状态为 true
        // console.log('--------------', nodeItem);
        // this.G6Instance.updateItem(nodeItem, this.setActNodeStateIcon(nodeItem._cfg.model.class, true, 'hover'));
        // this.G6Instance.updateItem(nodeItem, this.setActNodeStateIcon(itemModel.class, true));
      });
      // 鼠标离开节点
      this.G6Instance.on('node:mouseleave', (e) => {
        console.log('node:mouseleave', e, { ...e.item._cfg.model.icon });
        const nodeItem = e.item; // 获取鼠标离开的节点元素对象
        // const itemModel = nodeItem._cfg.model;
        this.G6Instance.setItemState(nodeItem, 'hover', false); // 设置当前节点的 hover 状态为 false
        // if (itemModel.icon.activeWay === 'click') {
        //   this.G6Instance.updateItem(nodeItem, this.setActNodeStateIcon(itemModel.class, true, 'hover'));
        // } else {
        //   this.G6Instance.setItemState(nodeItem, 'click', false); // 设置当前节点的 click 状态为 false
        //   this.G6Instance.updateItem(nodeItem, this.setActNodeStateIcon(itemModel.class, false, 'hover'));
        // }
        // this.G6Instance.updateItem(nodeItem, this.setActNodeStateIcon(itemModel.class, false));
      });
      // 点击节点
      this.G6Instance.on('node:click', (e) => {
        console.log('node:click', e, { ...e.item._cfg.model.icon });
        // 先将所有当前是 click 状态的节点置为非 click 状态
        const clickNodes = this.G6Instance.findAllByState('node', 'click');
        clickNodes.forEach((cn) => {
          console.log(cn._cfg.model.icon)
          this.G6Instance.setItemState(cn, 'click', false);
        });
        const nodeItem = e.item; // 获取被点击的节点元素对象
        this.G6Instance.setItemState(nodeItem, 'click', true); // 设置当前节点的 click 状态为 true

        // 聚焦点击的节点到画布中心
        this.G6Instance.focusItem(e.item, true, {
          easing: 'easeCubic',
          duration: 500,
        });
      });
      // 点击边
      this.G6Instance.on('edge:click', (e) => {
        console.log('edge:click', e);
        // 先将所有当前是 click 状态的边置为非 click 状态
        const clickEdges = this.G6Instance.findAllByState('edge', 'click');
        clickEdges.forEach((ce) => {
          this.G6Instance.setItemState(ce, 'click', false);
        });
        const edgeItem = e.item; // 获取被点击的边元素对象
        this.G6Instance.setItemState(edgeItem, 'click', true); // 设置当前边的 click 状态为 true
      });

      this.render();
      // 读取数据
      // this.G6Instance.data(this.initData);
      // 渲染图
      // this.G6Instance.render();
    },
    setActNodeStateIcon(nodeClass, isActive, activeWay) {
      console.log(nodeClass, isActive);
      const iconName = isActive ? `${nodeClass}Act` : nodeClass;
      const styleModel = {
        icon: {
          activeWay: activeWay,
          show: true,
          width: 21,
          height: 21,
          img: this.icon[iconName]
        },
      };
      return styleModel;
    },
    // 初始化节点样式
    setOriginNodeStyle(node) {
      if (!node.style) {
        node.style = {};
      }
      node.type = 'circle';
      node.size = 30;
      node.labelCfg = {
        position: 'bottom',
        style: {
          fill: '#262626',
          fontSize: 12,
        }
      };
      if (node.dangerNode) {
        node.style.stroke = '#EF3E3E';
        node.style.fill = '#fdebeb';
        node.style.lineWidth = 1;
        node.icon = {
          show: true,
          width: 21,
          height: 21,
          img: this.icon[`${node.class}Act`]
        };
      } else {
        node.icon = {
          show: true,
          width: 21,
          height: 21,
          img: this.icon[node.class]
        };
      }
    },
    async render() {
      const res = await fetch('/mock/data1.json');
      const remoteData = await res.json();
      let dangerTargetNode = [];
      remoteData.nodes.forEach((node) => {
        this.setOriginNodeStyle(node);
        const dangerNodeEdge = remoteData.edges.find((v) => v.source === node.id && node.dangerNode);
        if (dangerNodeEdge) {
          if (!dangerTargetNode.includes(dangerNodeEdge.target)) {
            dangerTargetNode.push(dangerNodeEdge.target);
          }
        }
      });
      remoteData.nodes.forEach((node) => {
        if (dangerTargetNode.includes(node.id)) {
          node.style.stroke = '#EF3E3E';
          node.style.fill = '#fdebeb';
          node.style.lineWidth = 1;
          node.icon = {
            show: true,
            width: 21,
            height: 21,
            img: this.icon[`${node.class}Act`]
          };
        }
      })
      remoteData.edges.forEach((edge) => {
        if (!edge.style) {
          edge.style = {};
        }
        // edge.label = '';
        // edge.style.lineWidth = edge.weight;
        edge.style.opacity = 0.8;
        const isDanger = remoteData.nodes.find((v) => v.id === edge.source).dangerNode;
        if (edge.apiFlowLine) {
          edge.size = 2;
          edge.style.stroke = '#00CD69';
          edge.style.fill = '#00CD69';
        }
        if (!edge.apiFlowLine) {
          edge.size = 1;
          edge.style.stroke =  '#d0d9e2';
          edge.style.fill = '#d0d9e2';
        }
        if (isDanger) {
          edge.size = 2;
          edge.style.stroke = '#EF3E3E';
          edge.style.fill = '#EF3E3E';
        }
      })
      console.log('res---', remoteData);
      this.initData = remoteData;
      this.G6Instance.data(this.initData); // 读取数据
      this.G6Instance.render(); // 渲染图
    }
  },
  mounted() {
    this.graphinInit();
  }
}
</script>

<style lang="less">
.container {
  width: 100vw;
  height: 100vh;
  position: relative;
}
#graphin-main {
  width: 100%;
  height: 100%;
  /* 节点提示框的样式 */
  .g6-tooltip {
    border-radius: 4px;
    font-size: 12px;
    color: #fff;
    background-color: rgba(0, 0, 0, 0.75);
    padding: 10px 8px;
    box-shadow:  0px 2px 8px rgba(0, 0, 0, 0.15);
  }
}
#minimap-box {
  position: absolute;
  bottom: 12px;
  left: 12px;
  background-color: #ffffff;
  border: 1px solid #e0e0e0;
}
</style>