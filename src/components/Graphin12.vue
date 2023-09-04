<template>
  <div id="graphin-container" :class="['container', { 'full-screen': isFullScreen }]">
    <div id="graphin-main"></div>
    <div id="minimap-box"></div>
  </div>
</template>

<script>
import G6 from '@antv/g6';

export default {
  name: 'DataFlowLinkDepict',
  data() {
    return {
      rootItemId: '0',
      isFullScreen: false,
      minZoom: 0.2, // 最小缩放比例
      maxZoom: 2, // 最大缩放比例
      originSize: { // 画布初始化尺寸
        width: 0,
        height: 0,
      },
      animateDuration: 200,
      gridImg: require('@/assets/icon/grid.png'),
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
        optPlus: require('@/assets/icon/opt-plus.svg'),
        optMinus: require('@/assets/icon/opt-minus.svg'),
        optSmallScreen: require('@/assets/icon/opt-smallscreen.svg'),
        optFullScreen: require('@/assets/icon/opt-fullscreen.svg'),
        optFocus: require('@/assets/icon/opt-focus.svg'),
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
    graphinInit() {
      const _this = this;
      const container = document.getElementById('graphin-main');
      const w = container.clientWidth - 8;
      const h = container.clientHeight - 8;
      this.originSize = {
        width: w,
        height: h
      }
      // 创建minimap实例
      const minimap = new G6.Minimap({
        container: 'minimap-box',
        size: [100, 100],
        // className: 'minimap',
        type: 'delegate',
      })
      // 创建grid实例
      // const grid = new G6.Grid({ img: this.gridImg });
      const grid = new G6.Grid();
      // 辅助线
      // const snapLine = new G6.SnapLine({
      //   line: {
      //     stroke: '#000000',
      //     lineWidth: 0.5,
      //   },
      //   itemAlignType: true
      // })

      // 视口操作工具栏
      const toolbar = new G6.ToolBar({
        getContent: () => {
          const outDiv = document.createElement('div');
          outDiv.style.width = '32px';
          outDiv.innerHTML = `
            <div class='toolbar-inner-container'>
              <ul class='toollist-1'>
                <li code='optPlus' txt='放大'><img src='${this.icon.optPlus}'/></li>
                <li code='optMinus' txt='缩小'><img src='${this.icon.optMinus}'/></li>
                <li code='optSmallScreen' txt='小窗显示'><img src='${this.icon.optSmallScreen}'/></li>
              </ul>
              <ul class='toollist-2'>
                <li code='optFullScreen' txt='全屏显示'><img src='${this.icon.optFullScreen}'/></li>
                <li code='optFocus' txt='中心聚焦'><img src='${this.icon.optFocus}'/></li>
              </ul>
            </div>
          `
          return outDiv
        },
        handleClick: (code, graph) => {
          console.log(code, graph)
          // 返回值zoom表示当前视口的缩放比例
          const zoom = graph.getZoom();
          const { innerWidth: full_W, innerHeight: full_H } = window;
          const newCenter = { x: w / 2, y: h / 2 }; // 缩放中心
          const animateCfg = {
            duration: this.animateDuration, // Number，一次动画的时长
            easing: 'easeQuadInOut', // String，动画函数
          };
          if (code === 'optPlus') {
            if (zoom < _this.maxZoom) {
              const newRatio = (zoom + 0.2).toFixed(1);
              graph.zoomTo(newRatio, newCenter, true, animateCfg);
            }
          }
          if (code === 'optMinus') {
            if (zoom > _this.minZoom) {
              const newRatio = (zoom - 0.2).toFixed(1);
              graph.zoomTo(newRatio, newCenter, true, animateCfg);
            }
          }
          if (code === 'optSmallScreen') {
            this.isFullScreen = false;
            const { width: o_W, height: o_H } = this.originSize;
            graph.changeSize(o_W, o_H);
            graph.focusItem(this.rootItemId, true, animateCfg);
          }
          if (code === 'optFullScreen') {
            this.isFullScreen = true;
            graph.changeSize(full_W, full_H);
            graph.focusItem(this.rootItemId, true, animateCfg);
          }
          if (code === 'optFocus') {
            graph.zoomTo(1, newCenter, true, animateCfg);
            setTimeout(() => {
              graph.focusItem(this.rootItemId, true, animateCfg);
            }, this.animateDuration);
          }
        }
      });
      // 自定义节点
      G6.registerNode('nodeWithBorder', {
        // 绘制节点
        draw(cfg, group) {
          const keyShape = group.addShape('circle', {
            attrs: {
              r: cfg.size / 2,
              stroke: cfg.style.stroke,
              fill: cfg.style.fill,
              lineWidth: cfg.style.lineWidth
            },
            name: 'circle-shape',
            draggable: true,
          });
          group.addShape('circle', {
            attrs: {
              r: cfg.size / 2 + cfg.style.lineWidth,
              stroke: '#ffffff',
              lineWidth: cfg.style.lineWidth
            },
            name: 'circle-border',
            draggable: true,
          });
          group.addShape('image', {
            attrs: {
              x: -cfg.icon.width / 2,
              y: -cfg.icon.height / 2,
              width: cfg.icon.width,
              height: cfg.icon.height,
              img: cfg.icon.img,
              // img: cfg.dangerNode ? _this.icon[`${cfg.class}Act`] : _this.icon[cfg.class]
            },
            name: 'image-shape',
            draggable: true,
          });
          group.addShape('text', {
            attrs: {
              x: 0, // 居中
              y: cfg.size - 5,
              textAlign: 'center',
              textBaseline: 'middle',
              text: cfg.label,
              fill: '#000000',
            },
            name: 'text-shape',
            draggable: true,
          });
          return keyShape;
        },
      }, 'circle')

      // 创建 G6 图实例
      this.G6Instance = new G6.Graph({
        container: 'graphin-main', // 指定图画布的容器 id
        // 画布宽高
        width: w,
        height: h,
        renderer: 'canvas',
        layout: {
          type: 'radial',
          linkDistance: 500,         // 可选，边长
          maxIteration: 1000,       // 可选
          unitRadius: 200,          // 可选
          nodeSize: 50,             // 可选
          preventOverlap: true,     // 可选，必须配合 nodeSize
          nodeSpacing: 0,
          strictRadial: true,       // 可选
          workerEnabled: true,       // 可选，开启 web-worker
        },
        // fitView: true,
        // fitViewPadding: [0, 0, 0, 0],
        animate: true,
        plugins: [minimap, grid, toolbar],
        modes: {
          default: [
            {
              type: 'drag-node',
              // enableDelegate: true,
              // enableDebounce: true,
              enableOptimize: true,
            },
            {
              type: 'drag-canvas',
              enableOptimize: true,
            },
            'activate-relations',
            {
              type: 'zoom-canvas',
              minZoom: _this.minZoom,
              maxZoom: _this.maxZoom,
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
              path: G6.Arrow.triangle(5, 5, 2),
              lineWidth: 3,
            },
            lineWidth: 1,
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
        // console.log('node:mouseenter', e, { ...e.item._cfg.model.icon });
        const nodeItem = e.item; // 获取鼠标进入的节点元素对象
        // const itemModel = nodeItem._cfg.model;
        this.G6Instance.setItemState(nodeItem, 'hover', true); // 设置当前节点的 hover 状态为 true
        // console.log('--------------', nodeItem);
        // this.G6Instance.updateItem(nodeItem, this.setActNodeStateIcon(nodeItem._cfg.model.class, true, 'hover'));
        // this.G6Instance.updateItem(nodeItem, this.setActNodeStateIcon(itemModel.class, true));
      });
      // 鼠标离开节点
      this.G6Instance.on('node:mouseleave', (e) => {
        // console.log('node:mouseleave', e, { ...e.item._cfg.model.icon });
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
        // console.log('node:click', e, { ...e.item._cfg.model.icon });
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
      node.type = 'nodeWithBorder';
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
  width: 70vw;
  height: 70vh;
  position: relative;
  // background: #ffffff;
  &.full-screen {
    position: fixed;
    width: 100vw;
    height: 100vh;
    z-index: 999;
  }
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
  .g6-component-toolbar {
    width: 32px;
    background-color: rgba(0, 0, 0, 0);
    border: none;
    top: 72px;
    left: 16px;
    padding: 0;
    .toolbar-inner-container {
      display: flex;
      flex-direction: column;
      gap: 8px;
      ul {
        display: flex;
        flex-direction: column;
        background-color: #ffffff;
        border: 1px solid #D7DADF;
        border-radius: 3px;
        margin: 0;
        padding: 0;
        position: relative;
        li {
          display: flex;
          justify-content: center;
          align-items: center;
          align-content: center;
          width: 100%;
          height: 32px;
          line-height: 32px;
          cursor: pointer;
          position: relative;
          &:hover {
            &::before {
              display: block;
              content: attr(txt);
              white-space: nowrap;
              font-size: 12px;
              line-height: 1;
              padding: 6px;
              color: #ffffff;
              background-color: rgba(0, 0, 0, 0.75);
              position: absolute;
              left: 32px;
              top: 16px;
            }
          }
          img {
            display: block;
            width: 16px;
            height: 16px;
          }
        }
        &.toollist-1 {
          height: 96px;
          &::before, &::after {
            display: block;
            content: ' ';
            width: 20px;
            height: 1px;
            background-color: #D7DADF;
            position: absolute;
          }
          &::before {
            top: 31px;
            left: 6px;
          }
          &::after {
            bottom: 31px;
            left: 6px;
          }
        }
        &.toollist-2 {
          height: 64px;
          &::before {
            display: block;
            content: ' ';
            width: 20px;
            height: 1px;
            background-color: #D7DADF;
            position: absolute;
          }
          &::before {
            top: 31px;
            left: 6px;
          }
        }
      }
    }
  }
}
#minimap-box {
  position: absolute;
  bottom: 16px;
  left: 16px;
  background-color: #ffffff;
  border: 1px solid #e0e0e0;
}
</style>