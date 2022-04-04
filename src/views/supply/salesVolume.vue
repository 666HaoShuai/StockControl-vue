<template>
  <div id="stocks">
    <!-- 面包导航 -->
    <el-breadcrumb
      separator="/"
      style="padding-left: 10px; padding-bottom: 10px; font-size: 12px"
    >
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>物资管理</el-breadcrumb-item>
      <el-breadcrumb-item>销量分析</el-breadcrumb-item>
    </el-breadcrumb>
    <el-row :gutter="20">
      <el-col :span="12">
        <div class="outStocks">
          <!-- 卡片部分-->
          <el-card>
            <!--搜索部分-->
            <el-form
              size="mini"
              :inline="true"
              :model="queryMap"
              class="demo-form-inline"
            >
              <el-form-item label="物品名称">
                <el-input
                  v-model="queryMap.pname"
                  placeholder="支持模糊查询"
                ></el-input>
              </el-form-item>

              <el-form-item>
                <el-button type="primary" @click="search">查询</el-button>
                <el-button @click="reset">重置</el-button>
              </el-form-item>
            </el-form>
            <!--            数据表格-->
            <el-table
              size="mini"
              border
              :data="tableData"
              style="width: 100%; height: 100%"
            >
              <el-table-column
                label="#销量排名"
                type="index"
                width="80"
              ></el-table-column>
              <el-table-column
                prop="pname"
                label="物品名称"
                width="120"
              ></el-table-column>
              <el-table-column
                prop="ownStock"
                label="我的库存"
                width="80"
              ></el-table-column>
              <el-table-column prop="recommendedScore" label="推荐等级" width="200">
                <template slot-scope="scope">
                  <el-rate
                    :disabled="true"
                    v-model="scope.row.recommendedScore"
                    show-text
                    :texts="['不推荐','适量','常规','推荐','超级推荐']"
                  >
                  </el-rate>
                </template>
              </el-table-column>
              <el-table-column
                prop="pnum"
                label="编号"
              ></el-table-column>
            </el-table>
          </el-card>
        </div>
      </el-col>
      <el-col :span="12">
        <div class="grid-content bg-purple-dark">
          <el-card class="box-card">
            <!-- 广告位 -->
            <el-carousel height="180px" width="350px">
              <el-carousel-item v-for="item in imageList" :key="item">
                <img height="100%" width="100%" :src="item.src" />
              </el-carousel-item>
            </el-carousel>
          </el-card>
          <el-card class="box-card">
            <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
            <div id="tianxing" style="width: 650px; height: 350px"></div>
          </el-card>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import echarts from "echarts";
export default {
  data() {
    return {
      tableData: [],
      xData: [],
      yData: [],
      queryMap: { pname: ''},
      imageList: [
        {
          src: require("@/assets/image1.jpg"),
        },
        {
          src: require("@/assets/广告位.png"),
        },
        {
          src: require("@/assets/广告位.png"),
        },
      ]
    };
  },
  methods: {
    /**
     * 搜索
     */
    search() {
      this.getStockList();
    },
    /**
     * 加载推荐信息
     */
    async getStockList() {
      const { data: res } = await this.$http.get(
        "/business/userProduct/querySales",
        {
          params: this.queryMap,
        }
      );
      if (!res.success) {
        return this.$message.error("获取推荐列表失败:" + res.data.errorMsg);
      } else {
        this.tableData = res.data;
        this.xData = [];
        this.yData = [];
        const $this = this;
        //构建表格条形统计图的数据
        this.tableData.forEach(function (e) {
          console.log(e);
          $this.xData.push(e.pname);
          $this.yData.push(e.psum);
        });
        //重新绘制表格
        this.draw();
      }
    },
    /**
     * 绘制条形统计图
     */
    draw() {
      var myChart = echarts.init(document.getElementById("tianxing"));
      // 指定图表的配置项和数据
      var option = {
        title: {
          text: "销量分析图",
        },
        toolbox: {
          show: true,
          feature: {
            dataZoom: {
              yAxisIndex: "none",
            },
            dataView: { readOnly: false }, //  缩放
            magicType: { type: ["bar"] }, ///　　折线  直方图切换
            restore: {}, // 重置
            saveAsImage: {}, // 导出图片
          },
        },
        tooltip: {},
        legend: {
          data: ["销量"],
        },
        xAxis: {
          data: this.xData,
        },
        yAxis: {},
        series: [
          {
            name: "销量",
            type: "bar",
            showBackground: true,
            data: this.yData,
            itemStyle: {
              normal: {
                //好，这里就是重头戏了，定义一个list，然后根据所以取得不同的值，这样就实现了，
                color: function (params) {
                  // build a color map as your need.
                  var colorList = [
                    "#0780cf ",
                    "#fa6d1d ",
                    "#ac2026 ",
                    "#701866 ",
                    "#d22e8d ",
                    "#FE8463",
                    "#a195c5 ",
                    "#FAD860",
                    "#F3A43B",
                    "#60C0DD",
                    "#D7504B",
                    "#a195c5  ",
                    "#F4E001",
                    "#F0805A",
                    "#63b2ee",
                  ];
                  return colorList[params.dataIndex];
                }, //以下为是否显示，显示位置和显示格式的设置了
                label: {
                  show: true,
                  position: "top",
                  //                           formatter: '{c}'
                  formatter: "{b}\n{c}",
                },
              },
            },
          },
        ],
      };

      // 使用刚指定的配置项和数据显示图表。
      myChart.setOption(option);
    },
    /**
     * 重置查询表单
     */
    reset() {
      this.queryMap = { pname: '' };
    }
  },
  created() {
    this.getStockList();
  },
  mounted: function () {
    this.draw();
  },
};
</script>
<style>
.pagination {
  margin: 20px auto 0 100px;
}
</style>
