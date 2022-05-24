<template>
  <div>
    <h4>网上例子</h4>
    <button @click="changeDateBtn1">我数树状图一个的按钮(数据切换)</button>
    <button @click="changeDateBtn3">我数树状图三个的按钮(数据切换)</button>
    <div id="container" :style="{ height: '800px', width: '100%' }" />
  </div>
</template>
<script>
import G6 from '@antv/g6'
export default {
  name: 'HellowWord',
  data() {
    return {
      getTreedata: [
        {
          id: 'root',
          label: '汉字',
          name: 1,
          children: []
        }
      ],
      treedata: [
        {
          id: 'root',
          label: 'ASDFSAFFFFFFFFFFFFSAFETWRWR',
          name: 11,
          context: 'root悬框',
          children: [
            {
              id: 'c1',
              label: 'c1',
              name: 2,
              context: 'c1悬框',
              children: [
                {
                  id: 'c1-1',
                  label: 'c1-1',
                  name: 3
                },
                {
                  id: 'c1-2',
                  label: 'c1-2',
                  name: 3,
                  children: [
                    {
                      id: 'c1-2-1',
                      label: 'c1-2-1',
                      name: 4
                    },
                    {
                      id: 'c1-2-2',
                      label: 'c1-2-2',
                      name: 4
                    }
                  ]
                }
              ]
            },
            {
              id: 'c2',
              label: 'c2',
              name: 2
            },
            {
              id: 'c3',
              label: 'c3',
              name: 2,

              children: [
                {
                  id: 'c3-1',
                  label: 'c3-1',
                  name: 3
                },
                {
                  id: 'c3-2',
                  label: 'c3-2',
                  name: 3,

                  children: [
                    {
                      id: 'c3-2-1',
                      label: 'c3-2-1',
                      name: 4
                    },
                    {
                      id: 'c3-2-2',
                      label: 'c3-2-2',
                      name: 4
                    },
                    {
                      id: 'c3-2-3',
                      label: 'c3-2-3',
                      name: 4
                    }
                  ]
                },
                {
                  id: 'c3-3',
                  label: 'c3-3',
                  name: 3
                }
              ]
            }
          ]
        }
      ],
      graph: undefined
    }
  },
  mounted() {
    this.showChart(this.treedata)
  },
  methods: {
    // 接口获取数据重绘graph
    changeDateBtn1() {
      this.graph.data(this.getTreedata[0])
      this.graph.save()
      this.graph.read(this.getTreedata[0])
      this.graph.changeData(this.getTreedata[0], true)
      this.graph.render()
      //让画布内容适应视口
      this.graph.fitView()
    },
    // 接口获取数据重绘graph
    changeDateBtn3() {
      this.graph.data(this.treedata[0])
      this.graph.save()
      this.graph.read(this.treedata[0])
      this.graph.changeData(this.treedata[0], true)
      this.graph.render()
      //让画布内容适应视口
      this.graph.fitView()
    },
    showChart(value) {
      const data = value[0]
      const container = document.getElementById('container')

      if (container) {
        const width = container.clientWidth - 400
        const height = container.clientHeight -400
        const tooltip = new G6.Tooltip({
          offsetX: 70,
          offsetY: 20,
          getContent(e) {
            console.log('e==', e.item._cfg.model.context)
            const outDiv = document.createElement('div')
            outDiv.style.width = '180px'
            outDiv.innerHTML = `
            <ul id="nodeDetails" style="backgroundColor: pink">
              <li style="color:red; font-weight:bold; backgroundColor: pink">名称: ${
                e.item._cfg.model.context
              }</li>
            </ul>`
            return outDiv
          },
          itemTypes: ['node']
        })
        this.graph = new G6.TreeGraph({
          // 图的DOM 容器，可以传入该 DOM 的 id 或者直接传入容器的 HTML 节点对象。
          container: container,
          width: width,
          height: height,
          plugins: [tooltip],
          // 设置画布的模式 包含default 模式和 edit 模式
          modes: {
            // default 模式中包含点击选中节点行为和拖拽画布行为
            default: [
              {
                type: 'collapse-expand',
                trigger: 'click'
              },
              // 拖拽画布 和 缩放画布
              'drag-canvas',
              'zoom-canvas'
            ]
          },
          // 默认的节点设置
          defaultNode: {
            //节点的大小
            type: 'sql',
            // size: 14,
            // // 指定边连入节点的连接点的位置
            // anchorPoints: [[0, 0.5], [1, 0.5]],
            // 节点样式
            style: {
              radius: 4,
              lineWidth: 2,
              fill: '#fff',
              shadowOffsetX: 5,
              shadowOffsetY: 5,
              shadowBlur: 10,
              shadowColor: '#e6f7ff'
            },
            // 节点文字
            labelCfg: {
              style: {
                fontSize: 5,
                nameFontSize: 12,
                childCountWidth: 22,
                countMarginLeft: 0,
                // itemPadding: 16,
                nameMarginLeft: 4,
                rootPadding: 18
              }
            }
          },
          // 默认的配置
          defaultEdge: {
            type: 'cubic-horizontal',
            style: {
              stroke: '#A3B1BF'
            }
          },
          layout: {
            type: 'compactBox',
            // 整个树状图的方向
            direction: 'RL',
            // 指定节点 ID
            getId: function getId(d) {
              return d.id
            },
            // 指定节点高度
            getHeight: function getHeight() {
              return 16
            },
            //指定节点宽度
            getWidth: function getWidth() {
              return 16
            },
            // 指定节点之间的垂直间距
            getVGap: function getVGap() {
              return 10
            },
            // 指定节点之间的水平间距
            getHGap: function getHGap() {
              return 100
            }
          }
        })

        // 为不同节点进行不同的配置 必须在 render 之前调用
        this.graph.node(function(node) {
          return {
            style: {
              fill: node.depth == 2 && node.status ? '#fff' : '#fff',
              stroke: node.depth == 2 && node.status ? 'pink' : 'pink',
              width: node.label.length > 10 ? 100 : 50
            },
            //String 类型。标签文本的文字内容
            label: node.label,
            labelCfg: {
              // 文本的偏移
              offset: 10,
              // 文本相对于节点的位置
              position:
                node.children && node.children.length > 0 ? 'center' : 'center',
              //文本样式
              style: {
                fill: node.depth == 2 && node.status ? '#c1422b' : '#2c3e50'
              }
            }
          }
        })

        // G6.registerNode(
        //   'sql',
        //   {
        //     drawShape(cfg, group) {
        //       console.log('=================0', cfg)
        //       const rect = group.addShape('rect', {
        //         attrs: {
        //           x: -75,
        //           y: -25,
        //           width: 150,
        //           height: 50,
        //           radius: 10,
        //           stroke: '#5B8FF9',
        //           fill: '#C6E5FF',
        //           lineWidth: 3
        //         },
        //         name: 'rect-shape'
        //       })
        //       if (cfg.name) {
        //         group.addShape('text', {
        //           attrs: {
        //             text: cfg.name,
        //             x: 0,
        //             y: 0,
        //             fill: '#00287E',
        //             fontSize: 14,
        //             textAlign: 'center',
        //             textBaseline: 'middle',
        //             fontWeight: 'bold'
        //           },
        //           name: 'text-shape'
        //         })
        //       }
        //       return rect
        //     }
        //   },
        //   'single-node'
        // )

        ///
        // 节点加号图标
        G6.registerNode(
          'sql',
          {
            options: {
              // size: [200, 20],
              stroke: '#91d5ff',
              fill: '#91d5ff'
            },
            drawShape(cfg, group) {
              const styles = this.getShapeStyle(cfg)
              const w = styles.width
              const h = styles.height
              const keyShape = group.addShape('rect', {
                attrs: {
                  x: -w / 2,
                  y: -h / 2,
                  width: w,
                  height: h,
                  // stroke: '#5B8FF9',
                  fill: '#fff',
                  radius: [2, 2, 2, 2],
                  shadowOffsetX: 2,
                  shadowOffsetY: 2,
                  shadowBlur: 10,
                  shadowColor: 'red'
                  // textAlign: 'left'
                },
                name: 'box'
              })
              if (cfg) {
                // title的背景设置
                // group.addShape('rect', {
                //   attrs: {
                //     x: -w / 2,
                //     y: -h / 2,
                //     width: w,
                //     height: h / 3,
                //     fill: '#5B8FF9',
                //     radius: [2, 2, 0, 0],
                //     textAlign: 'center'
                //   },
                //   name: 'title-box',
                //   draggable: true
                // })
                // 有children才会显示这个图标
                // group.addShape('text', {
                //   attrs: {
                //     textBaseline: 'top',
                //     // x: -w / 6,
                //     y: -h / 2.5,
                //     width: w,
                //     height: h / 3,
                //     lineHeight: 20,
                //     textAlign: 'center',
                //     text: cfg.label,
                //     fill: '#fff'
                //   },
                //   name: 'title'
                // })
                // 加号半径
                let xWidth = k => {
                  if (k < 10) {
                    return 25
                  } else if (k >= 10) {
                    return 50
                  }
                }
                // 图标的展示
                cfg.children &&
                  group.addShape('marker', {
                    attrs: {
                      x: xWidth(cfg.label.length),
                      y: 1,
                      r: 6,
                      cursor: 'pointer',
                      // 就是设置折叠展开的样式，但是必须要设置stroke颜色，
                      // 否则 效果就是透明，误以为会没生效，当然x,y,r的定位也很重要
                      symbol:
                        cfg.children.length > 0
                          ? G6.Marker.expand
                          : G6.Marker.collapse,
                      stroke: '#666',
                      lineWidth: 1,
                      fill: '#fff'
                    },
                    name: 'collapse-icon',
                    modelId: cfg.id
                  })
              }
              return keyShape
            },
            update: undefined
            // 设置点击折叠以后显示的图标，这个setState尽量避免使用，因为我在里面的
            // 控制台输出发现这个数据一会儿一输出，像是使用了定时器一样。又似是监听
            // setState(name, value, item) {
            //   console.log(name, value, item)
            //   const group = item.getContainer()
            //   if (name === 'collapsed') {
            //     const marker = item
            //       .get('group')
            //       .find(ele => ele.get('name') === 'collapse-icon')
            //     let icon
            //     if (value === undefined) {
            //       icon = G6.Marker.expand
            //     } else {
            //       icon = G6.Marker.collapse
            //     }
            //     marker.attr('symbol', icon)
            //   }
            // }
          },
          'rect'
        )
        ///
        //初始化数据
        this.graph.data(data)
        //渲染视图
        this.graph.render()
        //让画布内容适应视口
        this.graph.fitView()
      }
    }
  }
}
</script>

<style scoped>
#nodeDetails {
  list-style: none;
  background-color: blue;
}
#nodeDetails > li {
  padding: 5px 0;
  text-align: left;
  background-color: red;
}
::v-deep .g6-component-tooltip {
  background-color: #ccc;
}
</style>