<template>
  <div class="container">
    <div class="desc">{{ descText }}</div>
    <div ref="echart" class="echart" @click="updateDraw" />
    <pre>{{ currentData }}</pre>
  </div>
</template>

<script>
export default {
  name: "index",
  data() {
    return {
      echart: null,
      enableDraw: false,
      limitMax: 130000000,
      limitMin: 40000000,
      options: {
        xAxis: {
          name: "年 (民國)",
          type: "category",
          data: [
            "99",
            "100",
            "101",
            "102",
            "103",
            "104",
            "105",
            "106",
            "107",
            "108",
            "109"
          ]
        },
        yAxis: {
          name: "美元",
          type: "value",
          min: 40000000,
          max: 130000000,
          axisLabel: {
            formatter: value => {
              return `$${this.formatCurrency(value)}`;
            }
          }
        },
        series: [
          {
            data: [50000000, 55000000, 58000000, 62000000, 68000000],
            type: "line",
            lineStyle: {
              color: "#FFB91C",
              width: 3
            },
            // smooth: true, // 折線條拐點角度平滑化
            itemStyle: {
              borderType: "solid",
              color: "#FFB91C",
              borderColor: "#FFB91C",
              borderWidth: 2
            }
          }
        ]
      }
    };
  },
  computed: {
    descText() {
      return this.enableDraw ? "點擊結束繪製" : "點擊開始繪製";
    },
    currentData() {
      return this.options.xAxis.data.map((item, index) => {
        return {
          year: item,
          value: this.options.series[0].data[index] || 0
        };
      });
    }
  },
  methods: {
    initChart() {
      this.echart = this.$echarts.init(this.$refs.echart);
      this.echart.setOption(this.options);
      this.listenersMousemove();
    },
    listenersMousemove() {
      this.echart.getZr().on("mousemove", this.handleEchartMousemove);
      // this.echart
      //   .getZr()
      //   .on("mousemove", this.debounce(this.handleEchartMousemove));
    },
    handleEchartMousemove(params) {
      if (!this.enableDraw) return;

      const pointInPixel = [params.offsetX, params.offsetY];
      const pointInGrid = this.echart.convertFromPixel(
        { seriesIndex: 0 },
        pointInPixel
      );

      // x 軸 (年)
      const xValue = pointInGrid[0];

      // y 軸 (美元)
      const yValue = parseInt(pointInGrid[1]);
      this.updateChart(xValue, yValue);
    },
    updateChart(xValue, yValue) {
      const dataMaxIndex = this.options.series[0].data.length - 1;

      if (xValue < dataMaxIndex) {
        this.options.series[0].data.splice(5, dataMaxIndex + 1);
      } else if (xValue < dataMaxIndex) {
        // 刪除大於的部分
        this.options.series[0].data.splice(xValue, dataMaxIndex);
      } else if (xValue === dataMaxIndex) {
        // 直接替換最後面的資料
        this.options.series[0].data[dataMaxIndex] = yValue;
      } else {
        this.options.series[0].data.push(yValue);
      }

      this.echart.setOption(this.options);
    },
    updateDraw() {
      this.enableDraw = !this.enableDraw;
    },
    formatCurrency: function(n) {
      return n.toFixed(0).replace(/./g, function(c, i, a) {
        return i && c !== "." && (a.length - i) % 3 === 0 ? "," + c : c;
      });
    },
    debounce(fn, ms = 30) {
      let timer;
      return function(args) {
        clearTimeout(timer);
        timer = setTimeout(() => fn(args), ms);
      };
    }
  },
  mounted() {
    this.initChart();
  }
};
</script>

<style scoped>
.container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-flow: column nowrap;
  width: 100%;
}

.desc {
  position: absolute;
  top: 15%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 16px;
  font-size: 24px;
  opacity: 0.3;
  border-radius: 12px;
  border: dashed 3px gray;
}

.echart {
  width: 100%;
  max-width: 1140px;
  height: 500px;
}
</style>
