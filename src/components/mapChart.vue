<template>
  <div>
    <div>
      <el-button size="mini" type="primary" @click="backMap()" class="btn">返回概览</el-button>
      <div id="container"></div>
    </div>
  </div>
</template>

<script>
require("echarts/map/js/china");
import axios from "axios";
import { provinces, cityMap } from './map.js';
export default {
  name: "mapChart",
  data() {
    return {
      myChart: null,
    };
  },
  mounted() {
    // 初始化加载
    this.mapChart();
  },

  methods: {
    // 返回全国视图
    backMap() {
      this.mapChart();
    },

    // 配置渲染map
    mapChart() {
      this.myChart = this.$echarts.init(document.getElementById("container"));
      window.addEventListener("resize", ()=>{
        this.myChart.resize();
      });
      this.initEcharts("china");
      // 点击触发
      this.myChart.on("click", param => {
        if (param.name in provinces) {
          // 处理省模块
          let names = param.name;
          for (let key in provinces) {
            if (names == key) {
              this.showProvince(provinces[key], key);
              break;
            }
          }
        } else if (param.name in cityMap) {
          // 处理市模块
          let names = param.name;
          for (let key in cityMap) {
            if (names == key) {
              this.showCitys(cityMap[key], key);
              break;
            }
          }
        }
      });
    },
    //展示对应的省
    showProvince(eName,param) {
      axios.get(`/map/province/${eName}.json`).then(res => {
        this.$echarts.registerMap(param, res.data);
        this.initEcharts(param);
      })
    },
    //展示对应市
    showCitys(cName, param) {
      console.log(cName, param)
      // 显示县级地图
      axios.get(`/map/city/${cName}.json`).then(res => {
        this.$echarts.registerMap(param, res.data);
        this.initEcharts(param);
      })
    },
    initEcharts(map) {
      let option = {
        geo: {
          map: map,
          roam: false,
          scaleLimit: {
            min: 1.2,
            max: 3
          },
          zoom: 1.2,
          //图形上的文本标签，可用于说明图形的一些数据信息
          label: {
            normal: {
              show: true,
              fontSize: "10",
              color: "rgba(0,0,0,0.7)"
            }
          },
          //地图区域的多边形 图形样式，有 normal 和 emphasis 两个状态
          itemStyle: {
            //normal 是图形在默认状态下的样式；
            normal: {
              borderColor: "rgba(0, 0, 0, 0.2)"
            },
            //emphasis 是图形在高亮状态下的样式，比如在鼠标悬浮或者图例联动高亮时。
            emphasis: {
              areaColor: "#F3B329",
              shadowOffsetX: 0,
              shadowOffsetY: 0,
              shadowBlur: 20,
              borderWidth: 0,
              shadowColor: "rgba(0, 0, 0, 0.5)"
            }
          }
        },
        series: [
          {
            name: "信息量",
            type: "map",
            //这里是'china',及因为js中注册的名字，如果是上海市，则该出需pName 指的是'shanghai'
            mapType: map,
            geoIndex: 0
            // data: dataList
          }
        ]
      };
      this.myChart.setOption(option);
    }
  }
};
</script>

<style scoped>
#container {
  width: 600px;
  height: 600px;
  margin: 0px auto 0;
}
.btn {
  position: absolute;
  right: 10%;
  z-index: 999;
}
</style>