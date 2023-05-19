<template>
  <div id="graph-main" class="graph-main">
    <div id="graphin"></div>
    <div id="minimap-box"></div>
    <div class="legend-box">
      <ul>
        <li><i></i><span>人员</span></li>
        <li><i></i><span>应用</span></li>
        <li><i></i><span>厂商</span></li>
      </ul>
    </div>
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
      const container = document.getElementById('graphin');
      const _width = container.clientWidth - 16;
      const _height = container.clientHeight -16;
      // console.log('container', container, container.clientWidth, container.clientHeight);
      // https://antv-g6.gitee.io/zh/docs/manual/tutorial/plugins#minimap
      // 实例化 minimap 插件
      const minimap = new G6.Minimap({
        container: document.getElementById('minimap-box'),
        // className: 'minimap',
        // viewportClassName: 'minimap-viewport',
        type: 'delegate',
        size: [100, 100],
      });
      const grid = new G6.Grid();
      // const toolbar = new G6.ToolBar();
      const toolbar = new G6.ToolBar({
        getContent: () => {
          const outDiv = document.createElement('div');
          outDiv.style.width = '32px';
          outDiv.innerHTML = `
            <ul>
              <li code="ratioUp">+</li>
              <li code="ratioReset">重置</li>
              <li code="ratioDown">-</li>
            </ul>
          `;
          return outDiv
        },
        handleClick: (code) => {
          if (code === 'ratioUp') {
            const zoom = this.G6Instance.getZoom();
            const newRatio = zoom + 0.5
            this.G6Instance.zoomTo(newRatio);
          }
          if (code === 'ratioDown') {
            const zoom = this.G6Instance.getZoom();
            const newRatio = zoom - 0.5
            this.G6Instance.zoomTo(newRatio);
          }
          if (code === 'ratioReset') {
            // this.G6Instance.zoomTo(1, { x: _width / 2, y: _height / 2 });
            this.G6Instance.zoomTo(1);
          }
        }
      });

      // 自定义边： 双线
      G6.registerEdge('double-line', {
        draw(cfg, group) {
          // console.log('cfg', cfg, group);
          const startPoint = cfg.startPoint;
          const endPoint = cfg.endPoint;
          const shape = group.addShape('path', {
            attrs: {
              stroke: '#ABB8D0',
              path: [
                ['M', startPoint.x, startPoint.y - 2],
                ['L', endPoint.x, endPoint.y - 2],
                ['M', startPoint.x, startPoint.y + 2],
                ['L', endPoint.x, endPoint.y + 2],
              ],
              // path: `M ${startPoint.x},${startPoint.y - 1} L ${endPoint.x},${startPoint.y - 1} M ${startPoint.x},${startPoint.y + 1} L ${endPoint.x},${startPoint.y + 1}`
            },
            name: 'path-shape',
          });
          return shape;
        },
        // 响应状态变化
        setState(name, value, item) {
          console.log(555, name, value, item);
          const group = item.getContainer();
          const shape = group.get('children')[0]; // 顺序根据 draw 时确定
          // console.log('shape------', shape);
          if (name === 'click') {
            if (value) {
              shape.attr('stroke', '#215aff');
            } else {
              shape.attr('stroke', '#ABB8D0');
            }
          }
          if (name === 'mouseenter') {
            if (value) {
              shape.attr('lineWidth', 3);
            } else {
              shape.attr('lineWidth', 2);
            }
          }
          if (name === 'mouseleave') {
            if (value) {
              shape.attr('lineWidth', 3);
            } else {
              shape.attr('lineWidth', 2);
            }
          }
        },
      })

      this.G6Instance = new G6.Graph({
        container: 'graphin',
        width: _width,
        height: _height,
        // fitView: true,
        // fitViewPadding: [20, 40, 50, 20],
        layout: {
          // Object，可选，布局的方法及其配置项，默认为 random 布局。
          type: 'gForce', // 指定为辐射布局
          preventOverlap: true, // 防止节点重叠
          // nodeSize: 30        // 节点大小，用于算法中防止节点重叠时的碰撞检测。由于已经在上一节的元素配置中设置了每个节点的 size 属性，则不需要在此设置 nodeSize。
          linkDistance: 100, // 指定边距离为100
        },
        modes: {
          // 支持的behavior
          default: [
            'drag-canvas',
            'drag-node',
            'brush-select',
            'click-select',
            'activate-relations',
            {
              type: 'tooltip', // 提示框
              formatText(model) {
                console.log('model---', model);
                // 提示框文本内容
                const text = 'label:' + model.label + '<br/> class:' + model.class;
                return text;
              }
            },
            {
              type: 'edge-tooltip', // 边提示框
              formatText(model) {
                // 边提示框文本内容
                const text = 'source:' + model.source + '<br/> target:' + model.target + '<br/> weight:' + model.weight;
                return text;
              }
            }
          ],
          edit: ['click-select'],

        },
        // 节点在默认状态下的样式配置（style）和其他配置
        defaultNode: {
          size: 30, // 节点大小
          // 节点样式配置
          style: {
            fill: '#ffffff', // 节点填充颜色
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
          style: {
            opacity: 0.6, // 边透明度
            stroke: 'grey',
          },
          labelCfg: {
            autoRotate: true, // 边上的标签文本根据边的方向旋转
          }
        },
        // 节点不同状态下的样式集合
        nodeStateStyles: {
          // 鼠标 hover 上节点，即 hover 状态为 true 时的样式
          hover: {
            fill: '#ffffff',
            stroke: '#215aff',
            lineWidth: 4,
            shadowColor: 'rgba(129, 174, 239, 0.5)',
            shadowBlur: 16
          },
          // 鼠标点击节点，即 click 状态为 true 时的样式
          click: {
            fill: '#ffffff',
            stroke: '#215aff',
            lineWidth: 4,
            shadowColor: 'rgba(129, 174, 239, 0.5)',
            shadowBlur: 0
          },
          // 鼠标点击节点，即 click 状态为 true 时的样式
          active: {
            fill: '#ffffff',
            // stroke: '#215aff',
            // lineWidth: 4,
            // shadowColor: 'rgba(129, 174, 239, 0.5)',
            // shadowBlur: 16
          },
        },
        // 边不同状态下的样式集合
        edgeStateStyles: {
          // 鼠标点击边，即 click 状态为 true 时的样式
          hover: {
            stroke: '#215aff',
            strokeOpacity: 1,
            opacity: 1,
            lineWidth: 2,
            endArrow: false,
          },
          // inactive: {
          //   stroke: 'red',
          //   strokeOpacity: 1,
          //   opacity: 1,
          //   lineWidth: 2,
          //   endArrow: false,
          // },
        },
        plugins: [minimap, grid, toolbar], // 将 minimap 实例配置到图上
        // plugins: [minimap, grid], // 将 minimap 实例配置到图上
      });
      // this.G6Instance.on('afteractivaterelations', (e) => {
      //   // 当前操作的节点 item
      //   console.log(e.item);
      //   // 当前操作是选中(`'activate'`)还是取消选中(`'deactivate'`)
      //   console.log(e.action);
      // });
      // 鼠标进入节点
      this.G6Instance.on('node:mouseenter', (e) => {
        // console.log('e---', e);
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
        // console.log('click-----', e.item);
        const clickNodes = this.G6Instance.findAllByState('node', 'click');
        clickNodes.forEach((cn) => {
          this.G6Instance.setItemState(cn, 'click', false);
        });
        const nodeItem = e.item; // 获取被点击的节点元素对象
        this.G6Instance.setItemState(nodeItem, 'click', true); // 设置当前节点的 click 状态为 true
      });

      // 点击边
      // this.G6Instance.on('edge:click', (e) => {
      //   console.log('clickEdge---', e.item);
      //   // 先将所有当前是 click 状态的边置为非 click 状态
      //   const clickEdges = this.G6Instance.findAllByState('edge', 'click');
      //   clickEdges.forEach((ce) => {
      //     this.G6Instance.setItemState(ce, 'click', false);
      //   });
      //   const edgeItem = e.item; // 获取被点击的边元素对象
      //   this.G6Instance.setItemState(edgeItem, 'click', true); // 设置当前边的 click 状态为 true
      // });

      // 鼠标移入边
      this.G6Instance.on('edge:mouseenter', (e) => {
        console.log('clickEdge---', e.item);
        const edgeItem = e.item;
        this.G6Instance.setItemState(edgeItem, 'hover', true);
      });

      // 鼠标移出边
      this.G6Instance.on('edge:mouseleave', (e) => {
        const edgeItem = e.item;
        this.G6Instance.setItemState(edgeItem, 'hover', false);
      });

      this.render();
    },
    async render() {
      const res = await fetch('/data.json');
      this.initData = await res.json();
      const nodes = this.initData.nodes;
      nodes.forEach((node) => {
        if (!node.style) {
          node.style = {};
        }
        switch (node.class) {
          case 'c0': { // 厂商
            node.type = 'circle';
            node.size = 60;
            node.label = 'XXXX公司'
            node.style = {
              fill: "#ffffff",
              stroke: "#ABB8D0",
              lineWidth: 2,
              shadowColor: 'rgba(128,139,160,0.24)',
              shadowBlur: 16
            };
            node.labelCfg = {
              position: "center",
              offset: 5,
              style: {
                fill: "#323233",
                stroke: "#323233",
                fontFamily: "PingFangSC-Medium",
                lineWidth: 0.2,
                fontSize: 12
              }
            };
            break;
          }
          case 'c1': { // 应用
            // node.type = 'rect';
            node.type = 'circle';
            node.size = 24;
            node.style = {
              fill: "#63BB7E",
              stroke: "#ffffff",
              lineWidth: 2,
              shadowColor: 'rgba(128,139,160,0.24)',
              shadowBlur: 16
            };
            node.labelCfg = {
              position: "top",
              offset: 3,
              style: {
                fill: "#323233",
                fontFamily: "PingFangSC-Medium",
                fontSize: 12
              }
            };
            break;
          }
          case 'c2': { // 人员
            // node.type = 'ellipse';
            node.type = 'circle';
            node.size = 16;
            node.style = {
              fill: "#81AEEF",
              stroke: "#ffffff",
              lineWidth: 2,
              shadowColor: 'rgba(128,139,160,0.24)',
              shadowBlur: 16
            };
            node.labelCfg = {
              position: "bottom",
              offset: 3,
              style: {
                fill: "#323233",
                fontFamily: "PingFangSC-Medium",
                fontSize: 12
              }
            };
            break;
          }
        }
      })
      const edges = this.initData.edges;
      edges.forEach((edge) => {
        if (!edge.style) {
          edge.style = {};
          edge.label = '';
          // edge.type = 'double-line';

        }
        edge.style = {
          stroke: '#ABB8D0',
          strokeOpacity: 1,
          lineWidth: 1,
          // lineWidth: edge.weight, // 边的粗细映射边数据中的 weight 属性数值
          lineAppendWidth: 10,
        }
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

<style lang="less">
.graph-main {
  position: absolute;
  width: 100%;
  height: 100%;

  #graphin {
    position: relative;
    width: 100%;
    height: 100%;
    .g6-component-toolbar {
      width: 32px;
      height: 98px;
      box-sizing: border-box;
      position: absolute;
      right: 16px;
      bottom: 16px;
      left: inherit;
      top: inherit;
      padding: 0;
      margin: 0;
      z-index: 99;
      background: #FFFFFF;
      border: 1px solid #EBECF0;
      border-radius: 4px;
      box-shadow: -4px -4px 4px 0 rgba(255,255,255,0.50), 4px 4px 8px 0 rgba(128,139,160,0.24);
      ul {
        display: flex;
        flex-direction: column;
        padding: 0;
        margin: 0 auto;
        width: 100%;
        height: 100%;
        li {
          display: block;
          width: 100%;
          height: 32px;

          &:nth-child(2) {
            border-top: 1px solid #EBECF0;
            border-bottom: 1px solid #EBECF0;
          }
        }
      }
    }
  }
  #minimap-box {
    width: 100px;
    height: 100px;
    position: absolute;
    left: 32px;
    bottom: 32px;
    background-color: #ffffff;
    border: 1px solid #e9e9e9;
  }
  /* 提示框的样式 */
  .g6-tooltip {
    border: 1px solid #e2e2e2;
    border-radius: 4px;
    font-size: 12px;
    color: #545454;
    background-color: rgba(255, 255, 255, 0.9);
    padding: 10px 8px;
    box-shadow: rgb(174, 174, 174) 0px 0px 10px;
  }
  .legend-box {
    width: 80px;
    height: 98px;
    box-sizing: border-box;
    background: #FFFFFF;
    border: 1px solid #EBECF0;
    border-radius: 4px;
    box-shadow: -4px -4px 4px 0 rgba(255,255,255,0.50), 4px 4px 8px 0 rgba(128,139,160,0.24);
    position: absolute;
    right: 56px;
    bottom: 16px;
    ul {
      display: flex;
      flex-direction: column;
      width: 100%;
      height: 100%;
      box-sizing: border-box;
      padding: 11px 0 0 14px;
      margin: 0;
      li {
        display: flex;
        width: 100%;
        height: 20px;
        align-items: center;
        box-sizing: border-box;
        padding: 0;
        margin-bottom: 8px;
        i {
          display: block;
          width: 12px;
          height: 12px;
          border-radius: 50%;
          box-sizing: border-box;
          margin-right: 10px;
        }
        &:nth-child(1) i {
          background: #81AEEF;
          border: 2px solid #FFFFFF;
          box-shadow: -4px -4px 8px 0 rgba(255,255,255,0.50), 4px 4px 8px 0 rgba(128,139,160,0.24);
        }
        &:nth-child(2) i {
          background: #63BB7E;
          border: 2px solid #FFFFFF;
          box-shadow: -4px -4px 8px 0 rgba(255,255,255,0.50), 4px 4px 8px 0 rgba(128,139,160,0.24);
        }
        &:nth-child(3) i {
          background: #FFFFFF;
          border: 2px solid #ABB8D0;
          box-shadow: -8px -8px 8px 0 rgba(255,255,255,0.50), 4px 4px 8px 0 rgba(128,139,160,0.24);
        }
        span {
          display: block;
          font-family: PingFangSC-Regular;
          font-size: 12px;
          color: #606061;
          line-height: 20px;
          font-weight: 400;
        }
      }
    }
  }
}


</style>
