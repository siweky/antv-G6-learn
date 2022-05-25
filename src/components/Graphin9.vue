<template>
  <div class="hello">
    <h4>基础动画</h4>
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
      initData: {
        // 点集
        nodes: [
          {
            id: 'node1',
            x: 100,
            y: 100,
            type: 'circle-animate',
            size: 20,
            label: 'Scale Animation',
            labelCfg: {
              position: 'top',
            },
          },
        ],
        // 边集
        edges: [],
      }
    }
  },
  methods: {
    graphinInit() {
      // 放大、变小动画
      G6.registerNode(
        'circle-animate',
         {
           afterDraw(cfg, group) {
            console.log(cfg, group);
            //  获取该节点上的第一个图形
            const shape = group.get('children')[0];
            // 该图形的动画
            shape.animate(
              (ratio) => {
                // 每一帧的操作，入参 ratio：这一帧的比例值（Number）。返回值：这一帧需要变化的参数集（Object）。
                // 先变大、再变小
                const diff = ratio <= 0.5 ? ratio * 10 : (1 - ratio) * 10;
                let radius = cfg.size;
                // if (isNaN(radius)) radius = radius[0];
                // 返回这一帧需要变化的参数集，这里只包含了半径
                return {
                  r: radius / 2 + diff,
                };
              },
              {
                // 动画重复
                repeat: true,
                duration: 3000,
                easing: 'easeCubic',
              },
            );
          }
        },
        'circle',
      );
      const graph = new G6.Graph({
        container: 'graphin',
        width: 800,
        height: 500,
      });

      // 读取数据
      graph.data(this.initData);
      // 渲染图
      graph.render();
    }
  },
  mounted() {
    this.graphinInit();
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
