<template>
  <div id="stocks">
    <!-- 面包导航 -->
    <el-breadcrumb
      separator="/"
      style="padding-left: 10px; padding-bottom: 10px; font-size: 12px"
    >
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>物资管理</el-breadcrumb-item>
      <el-breadcrumb-item>出入库管理</el-breadcrumb-item>
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
              <el-form-item label="发放单号">
                <el-input
                  v-model="queryMap.outNum"
                  placeholder="发放单号"
                ></el-input>
              </el-form-item>
              <el-form-item label="发放类型">
                <el-select v-model="queryMap.type" placeholder="发放类型">
                  <el-option label="全部类型" value=""></el-option>
                  <el-option label="政府领取" value="0"></el-option>
                  <el-option label="医院领取" value="1"></el-option>
                  <el-option label="小区领取" value="2"></el-option>
                  <el-option label="个人领取" value="3"></el-option>
                  <el-option label="其他领取" value="4"></el-option>
                </el-select>
              </el-form-item>

              <el-form-item>
                <el-select v-model="queryMap.status" placeholder="请选择状态">
                  <el-option label="已发放" :value="0"></el-option>
                  <el-option label="回收站" :value="1"></el-option>
                  <el-option label="待审核" :value="2"></el-option>
                </el-select>
              </el-form-item>

              <el-form-item>
                <el-button type="primary" @click="search">查询</el-button>
                <el-button @click="reset">重置</el-button>
              </el-form-item>

              <el-form-item>
                <router-link to="/business/product/publish">
                  <el-button type="success"> 发放 </el-button>
                </router-link>
              </el-form-item>
            </el-form>
            <!--            数据表格-->
            <el-table
              size="mini"
              border
              :data="tableData"
              style="width: 100%; height: 450px"
            >
              <el-table-column label="#" prop="id" width="50"></el-table-column>
              <el-table-column
                prop="outNum"
                :show-overflow-tooltip="true"
                label="发放单号"
                width="180"
              ></el-table-column>
              <el-table-column prop="type" label="发放类型" width="100">
                <template slot-scope="scope">
                  <el-tag
                    effect="plain"
                    size="mini"
                    type="success"
                    v-if="scope.row.type === 0"
                    >政府领取</el-tag
                  >
                  <el-tag
                    effect="plain"
                    size="mini"
                    type="danger"
                    v-else-if="scope.row.type === 1"
                    >医院领取</el-tag
                  >
                  <el-tag
                    effect="plain"
                    size="mini"
                    type="warning"
                    v-else-if="scope.row.type === 2"
                    >小区领取</el-tag
                  >
                  <el-tag
                    effect="plain"
                    size="mini"
                    type="info"
                    v-else-if="scope.row.type === 3"
                    >个人领取</el-tag
                  >
                  <el-tag
                    effect="plain"
                    size="mini"
                    v-else-if="scope.row.type === 4"
                    >其他领取</el-tag
                  >
                </template>
              </el-table-column>
              <el-table-column prop="priority" label="紧急程度" width="180">
                <template slot-scope="scope">
                  <el-rate
                    :disabled="true"
                    v-model="scope.row.priority"
                    show-text
                    :texts="['不急', '常规', '紧急', '特急', '超急']"
                  >
                  </el-rate>
                </template>
              </el-table-column>
              <el-table-column
                prop="name"
                label="发放地点"
                width="150"
              ></el-table-column>
              <el-table-column
                prop="productNumber"
                label="总数"
                width="80"
              ></el-table-column>
              <el-table-column
                prop="phone"
                label="联系方式"
                width="120"
              ></el-table-column>
              <el-table-column prop="status" label="状态" width="100">
                <template slot-scope="scope">
                  <el-tag
                    size="mini"
                    type="danger"
                    effect="plain"
                    v-if="scope.row.status == 1"
                    >回收</el-tag
                  >
                  <el-tag
                    size="mini"
                    effect="plain"
                    v-if="scope.row.status == 0"
                    >已放</el-tag
                  >
                  <el-tag
                    size="mini"
                    effect="plain"
                    type="warning"
                    v-if="scope.row.status == 2"
                    >审核中</el-tag
                  >
                </template>
              </el-table-column>

              <el-table-column
                prop="operator"
                label="操作员"
                width="150"
              ></el-table-column>

              <el-table-column
                prop="createTime"
                label="发放时间"
                width="200px;"
              ></el-table-column>
              <el-table-column label="操作" fixed="right" width="200">
                <template slot-scope="scope">
                  <el-button
                    icon="el-icon-view"
                    type="text"
                    size="small"
                    @click="detail(scope.row.id)"
                  >
                    明细
                  </el-button>
                  <!--                        给操作员使用的按钮-->
                  <span v-if="scope.row.status == 0">
                    <el-popconfirm
                      @onConfirm="remove(scope.row.id)"
                      style="margin-left: 20px"
                      confirmButtonText="好的"
                      cancelButtonText="不用了"
                      icon="el-icon-info"
                      iconColor="red"
                      title="这是一段内容确定移入回收站吗？"
                    >
                      <el-button
                        type="text"
                        slot="reference"
                        size="mini"
                        icon="el-icon-s-operation"
                        >回收站</el-button
                      >
                    </el-popconfirm>
                  </span>
                  <!--   给操作员使用的按钮(回收站)-->
                  <span v-if="scope.row.status == 1">
                    <el-popconfirm
                      @onConfirm="back(scope.row.id)"
                      style="margin-left: 10px"
                      confirmButtonText="好的"
                      cancelButtonText="不用了"
                      icon="el-icon-info"
                      iconColor="green"
                      title="这是一段内容确定恢复数据吗？"
                    >
                      <el-button
                        type="text"
                        slot="reference"
                        size="mini"
                        icon="el-icon-s-operation"
                        >还原</el-button
                      >
                    </el-popconfirm>
                    <el-popconfirm
                      style="margin-left: 20px"
                      @onConfirm="del(scope.row.id)"
                      title="这是一段内容确定删除吗？"
                    >
                      <el-button
                        type="text"
                        slot="reference"
                        size="small"
                        icon="el-icon-delete"
                        >删除</el-button
                      >
                    </el-popconfirm>
                  </span>

                  <!--                        给审核员使用的按钮-->
                  <span v-if="scope.row.status === 2">
                    <el-popconfirm
                      style="margin-left: 20px"
                      @onConfirm="publish(scope.row.id)"
                      title="审核通过后库存将更新,是否通过"
                    >
                      <el-button
                        type="text"
                        slot="reference"
                        size="small"
                        icon="el-icon-refresh"
                        >通过</el-button
                      >
                    </el-popconfirm>
                    <el-popconfirm
                      style="margin-left: 20px"
                      @onConfirm="del(scope.row.id)"
                      title="这是一段内容确定删除吗？"
                    >
                      <el-button
                        type="text"
                        slot="reference"
                        size="small"
                        icon="el-icon-delete"
                        >删除</el-button
                      >
                    </el-popconfirm>
                  </span>
                </template>
              </el-table-column>
            </el-table>
            <!--                表格分页-->
            <el-pagination
              style="margin-top: 20px"
              background
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="queryMap.pageNum"
              :page-sizes="[10, 20, 30, 40]"
              :page-size="queryMap.pageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total"
            ></el-pagination>
            <!-- 发放明细 -->
            <el-dialog
              title="发放明细"
              :visible.sync="dialogVisible"
              @close="closeDetail"
              width="60%"
            >
              <!--                来源信息-->
              <el-card class="box-card" style="margin-bottom: 10px">
                <div class="text item">
                  <el-row :gutter="20">
                    <el-col :span="6">
                      <div class="grid-content bg-purple">
                        <span style="font-size: 11px; color: #303030"
                          >省区市:</span
                        >
                        &nbsp;<el-tag size="mini">{{
                          consumer.address
                        }}</el-tag>
                      </div>
                    </el-col>
                    <el-col :span="6">
                      <div class="grid-content bg-purple">
                        <span style="font-size: 11px; color: #303030"
                          >具体位置:</span
                        >
                        &nbsp;<el-tag size="mini">{{ consumer.name }}</el-tag>
                      </div>
                    </el-col>
                    <el-col :span="6">
                      <div class="grid-content bg-purple">
                        <span style="font-size: 11px; color: #303030"
                          >联系人 :</span
                        >
                        &nbsp;<el-tag size="mini">{{
                          consumer.contact
                        }}</el-tag>
                      </div>
                    </el-col>
                    <el-col :span="6">
                      <div class="grid-content bg-purple">
                        <span style="font-size: 11px; color: #303030"
                          >电话 : </span
                        >&nbsp;<el-tag size="mini">{{ consumer.phone }}</el-tag>
                      </div>
                    </el-col>
                  </el-row>
                </div>
              </el-card>
              <!--                步骤条-->
              <el-steps
                simple
                v-if="pStatus == 0"
                style="margin-left: 10px; margin-bottom: 5px"
                :space="200"
                :active="3"
                finish-status="success"
              >
                <el-step title="提交发放单"></el-step>
                <el-step title="审核发放单"></el-step>
                <el-step title="成功发放"></el-step>
              </el-steps>
              <el-steps
                simple
                v-if="pStatus == 2"
                style="margin-left: 10px; margin-bottom: 5px"
                :space="200"
                :active="2"
                finish-status="success"
              >
                <el-step title="提交发放单"></el-step>
                <el-step title="审核发放单"></el-step>
                <el-step title="成功发放"></el-step>
              </el-steps>
              <span>
                <template>
                  <el-table
                    height="260"
                    max-height="350"
                    border
                    :data="detailTable"
                    style="width: 100%"
                  >
                    <el-table-column prop="name" label="名称"></el-table-column>
                    <el-table-column
                      :show-overflow-tooltip="true"
                      prop="pnum"
                      label="商品编号"
                    ></el-table-column>
                    <el-table-column
                      prop="model"
                      label="规格"
                    ></el-table-column>
                    <el-table-column
                      prop="imageUrl"
                      label="图片"
                      align="center"
                      width="150px"
                      padding="0px"
                    >
                      <template slot-scope="scope">
                        <img
                          :src="
                            'https://www.zykhome.club/' + scope.row.imageUrl
                          "
                          alt
                          style="width: 30px; height: 30px"
                        />
                      </template>
                    </el-table-column>
                    <el-table-column
                      prop="count"
                      label="数量"
                    ></el-table-column>
                    <el-table-column prop="unit" label="单位"></el-table-column>
                  </el-table>
                  <!--              明细分页-->
                  <el-pagination
                    style="margin-top: 20px"
                    background
                    @current-change="handleDetailSizeChange"
                    :current-page="this.pageNum"
                    :page-size="3"
                    layout="prev, pager, next,total"
                    :total="this.detailTotal"
                  >
                  </el-pagination>
                </template>
              </span>
            </el-dialog>
          </el-card>
        </div>
      </el-col>
      <el-col :span="12">
        <div class="grid-content bg-purple-dark">
          <el-card class="box-card">
            <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
            <div id="tianxing" style="width: 650px; height: 350px"></div>
          </el-card>
          <el-card class="box-card" style="margin-top: 10px">
            <!-- 库存饼图 -->
            <div id="bingtu" style="width: 590px; height: 225px"></div>
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
      catetorys: [], //类别选择
      searchSelectProps: {
        expandTrigger: "hover",
        value: "id",
        label: "name",
        children: "children",
        checkStrictly: true,
      }, //级联搜索选择器配置项
      categorykeys: [],
      total: 0,
      tableData: [],
      queryMap: { pageSize: 9, pageNum: 1 },
      xData: [],
      yData: [],
      legendData: [], //饼图存放物资名称
      selected: {}, //存放选择的数据
      seriesData: [{ name: "", value: "" }], //饼图数据
      consumer: {},
      detailTotal: 0,
      detailTable: [],
      dialogVisible: false,
      pageNum: 1,
      total: 0,
      queryMap: { pageNum: 1, pageSize: 10, status: 0 },
      tableData: [],
      pStatus: "", //步骤flag
    };
  },
  methods: {
    /**
     * 搜索
     */
    search() {
      this.queryMap.pageNum = 1;
      this.getStockList();
    },
    /**
     * 加载库存信息
     */
    async getStockList() {
      const { data: res } = await this.$http.get(
        "business/product/findProductStocks",
        {
          params: this.queryMap,
        }
      );
      if (!res.success) {
        return this.$message.error("获取物资库存列表失败:" + res.data.errorMsg);
      } else {
        this.total = res.data.total;
        this.tableData = res.data.rows;
        this.xData = [];
        this.yData = [];
        this.selected = {};
        const $this = this;
        //构建表格条形统计图的数据
        this.tableData.forEach(function (e) {
          console.log(e);
          $this.xData.push(e.name);
          $this.yData.push(e.stock);
        });
        //重新绘制表格
        this.draw();
        //饼图
        this.findAllProductStocks();
      }
    },

    //改变页码
    handleSizeChange(newSize) {
      this.queryMap.pageSize = newSize;
      this.getStockList();
    },
    //翻页
    handleCurrentChange(current) {
      this.queryMap.pageNum = current;
      this.getStockList();
    },
    /**
     * 绘制条形统计图
     */
    draw() {
      var myChart = echarts.init(document.getElementById("tianxing"));
      // 指定图表的配置项和数据
      var option = {
        title: {
          text: "库存条形图",
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
          data: ["库存量"],
        },
        xAxis: {
          data: this.xData,
        },
        yAxis: {},
        series: [
          {
            name: "库存量",
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
                  //                             formatter: '{c}'
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
     * 绘制饼图
     */
    drawRound() {
      var myChart = echarts.init(document.getElementById("bingtu"));
      var option = {
        title: {
          text: "库存占比图",

          left: "left",
        },
        toolbox: {
          show: true,
          feature: {
            saveAsImage: {}, // 导出图片
          },
        },
        tooltip: {
          trigger: "item",
          formatter: "{a} <br/>{b} : {c} ({d}%)",
        },
        legend: {
          type: "scroll",
          orient: "vertical",
          right: 10,
          top: 20,
          bottom: 20,
          data: this.legendData,
          selected: this.selected,
        },
        series: [
          {
            name: "物资名称",
            type: "pie",
            radius: "55%",
            center: ["40%", "50%"],
            data: this.seriesData,
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: "rgba(0, 0, 0, 0.5)",
              },
            },
          },
        ],
      };
      myChart.setOption(option);
    },

    /**
     * 物资所有的库存信息
     */
    async findAllProductStocks() {
      const { data: res } = await this.$http.get(
        "business/product/findAllStocks",
        {
          params: this.queryMap,
        }
      );
      if (!res.success) {
        return this.$message.error("获取物资所有库存失败:" + res.data.errorMsg);
      } else {
        this.legendData = [];
        this.selected = {};
        this.seriesData = [{}];
        var $this = this;
        //构建饼图的数据对象
        res.data.forEach(function (e) {
          $this.legendData.push(e.name);
          $this.seriesData.push({ name: e.name, value: e.stock });
        });

        //重新绘制表格
        this.drawRound();
      }
    },
    /**
     * 分类搜索改变时
     */
    selectChange() {
      var str = "";
      for (var i = 0; i < this.categorykeys.length; i++) {
        str += this.categorykeys[i] + ",";
      }
      str = str.substring(0, str.length - 1);
      this.queryMap.categorys = str;
    },
    /**
     * 加载物资类别
     */
    async getCatetorys() {
      const { data: res } = await this.$http.get(
        "business/productCategory/categoryTree"
      );
      if (!res.success) {
        return this.$message.error("获取物资类别失败:" + res.data.errorMsg);
      } else {
        this.catetorys = res.data.rows;
      }
    },
    /**
     * 商品出库
     */
    reduce(pNum) {
      const { data: res } = this.$http.get(
        "/business/userProduct/reduce/" + pNum
      );
      this.search();
    },
    /**
     * 商品入库
     */
    add(pNum) {
      const { data: res } = this.$http.get("/business/userProduct/add/" + pNum);
      this.search();
    },
    /**
     * 加载表格数据
     */
    async loadTableData() {
      const { data: res } = await this.$http.get(
        "business/outStock/findOutStockList",
        {
          params: this.queryMap,
        }
      );
      if (!res.success) {
        return this.$message.error("获取列表失败:" + res.data.errorMsg);
      } else {
        this.total = res.data.total;
        this.tableData = res.data.rows;
      }
    },
    /**
     *物资发放审核
     */
    async publish(id) {
      const { data: res } = await this.$http.put(
        "business/outStock/publish/" + id
      );
      if (!res.success) {
        return this.$message.error("审核失败:" + res.data.errorMsg);
      } else {
        await this.loadTableData();
        return this.$message.success("发放已审核通过");
      }
    },

    /**
     * 关闭明细
     */
    closeDetail() {
      this.pageNum = 1;
    },
    /**
     *  改变页码
     */
    handleDetailSizeChange(newSize) {
      this.pageNum = newSize;
      this.detail(this.detailId);
    },

    /**
     * 查看发放单明细
     */
    async detail(id) {
      this.detailId = id;
      const { data: res } = await this.$http.get(
        "business/outStock/detail/" + id + "?pageNum=" + this.pageNum
      );
      if (!res.success) {
        this.$message.error("获取明细失败:" + res.data.errorMsg);
      } else {
        this.detailTable = res.data.itemVOS;
        this.detailTotal = res.data.total;
        this.consumer = res.data.consumerVO;
        this.pStatus = res.data.status;
        this.dialogVisible = true;
      }
    },

    /**
     * 从回收站恢复
     */
    async back(id) {
      const { data: res } = await this.$http.put(
        "business/outStock/back/" + id
      );
      if (!res.success) {
        return this.$message.error("从回收站恢复失败:" + res.data.errorMsg);
      } else {
        await this.loadTableData();
        return this.$message.success("从回收站中已恢复");
      }
    },

    /**
     * 移除回收站
     */
    async remove(id) {
      const { data: res } = await this.$http.put(
        "business/outStock/remove/" + id
      );
      if (!res.success) {
        return this.$message.error("移入回收站失败:" + res.data.errorMsg);
      } else {
        await this.loadTableData();
        return this.$message.success("移入回收站成功");
      }
    },

    /**
     * 重置查询表单
     */
    reset() {
      this.queryMap = { pageNum: 1, pageSize: 10, status: 0 };
    },
    /**
     * 改变页码
     */
    handleSizeChange(newSize) {
      this.queryMap.pageSize = newSize;
      this.loadTableData();
    },
    /**
     * 点击分页
     */
    handleCurrentChange(current) {
      this.queryMap.pageNum = current;
      this.loadTableData();
    },
    //搜索
    search() {
      this.queryMap.pageNum = 1;
      this.loadTableData();
    },
    /**删除明细
     */
    async del(id) {
      const { data: res } = await this.$http.get(
        "business/outStock/delete/" + id
      );
      if (!res.success) {
        return this.$message.error("删除失败:" + res.data.errorMsg);
      } else {
        await this.loadTableData();
        return this.$message.success("删除发放单记录成功");
      }
    },
  },
  created() {
    this.loadTableData();
    this.getStockList();
    this.getCatetorys();
    this.findAllProductStocks();
  },
  mounted: function () {
    this.draw();
    this.drawRound();
  },
};
</script>
<style>
.pagination {
  margin: 20px auto 0 100px;
}
</style>
