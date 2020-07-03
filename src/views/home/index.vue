<template>
  <div class="home-container">
    <div class="address-layout">
      <el-row :gutter="20">
        <el-col v-for=" i in [0,1,2]" :key="i.label" :span="7">
          <div class="out-border">
            <div class="layout-title">后台项目</div>
            <div class="color-main address-content">
              <a href="https://github.com/macrozheng/mal">mall</a>
            </div>
          </div>
        </el-col>
        <el-card class="mine-layout">
          <div style="text-align:center">
            <img
              width="150px"
              height="150px"
              src="http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg"
            >
          </div>
          <div style="text-algin:center">mall全套学习教程连载中</div>
          <div style="text-align: center;margin-top: 5px">
            <span class="color-main">关注公号</span>，第一时间获取。
          </div>
        </el-card>
      </el-row>
    </div>
    <div class="total-layout">
      <el-row :gutter="20">
        <el-col v-for=" i in [0,1,2]" :key="i.label" :span="6">
          <div class="total-frame">
            <img :src="img_home_order" class="total-icon">
            <div class="total-title">今日订单总数</div>
            <div class="total-value">200</div>
          </div>
        </el-col>
      </el-row>
    </div>
    <div class="un-handle-layout">
      <div class="layout-title">待处理事务</div>
      <div v-for=" i in [0,1,2]" :key="i.label" class="un-handle-content">
        <el-row :gutter="20">
          <el-col v-for=" j in [0,1,2]" :key="j.label" :span="8">
            <div class="un-handle-item">
              <span class="font-medium">待付款订单</span>
              <span style="float:right" class="color-danger">(10)</span>
            </div>
          </el-col>
        </el-row>
      </div>
    </div>
    <div class="overview-layout">
      <el-row :gutter="20">
        <el-col v-for="i in [0,1]" :key="i.label" :span="12">
          <div class="out-border">
            <div class="layout-title">商品总览</div>
            <div style="padding:40px">
              <el-row>
                <el-col
                  v-for="k in [0,1,2,3]"
                  :key="k.label"
                  :span="6"
                  class="color-danger overview-item-value"
                >100</el-col>
              </el-row>
              <el-row>
                <el-col
                  v-for="l in [0,1,2,3]"
                  :key="l.label"
                  :span="6"
                  class="overview-item-title"
                >100</el-col>
              </el-row>
            </div>
          </div>
        </el-col>
      </el-row>
    </div>
    <div class="statistics-layout">
      <div class="layout-title">订单统计</div>
      <el-row>
        <el-col :span="4">
          <div style="padding:20px">
            <div v-for="i in [0,1,2,3]" :key="i.label">
              <div style="color: #909399;font-size: 14px">本月订单总数</div>
              <div style="color: #606266;font-size: 24px;padding: 10px 0">10000</div>
              <div>
                <span class="color-success" style="font-size: 14px">+10%</span>
                <span style="color: #C0C4CC;font-size: 14px">同比上月</span>
              </div>
            </div>
          </div>
        </el-col>
        <el-col :span="20">
          <div style="padding:10px;border-left:1px solid #DCDFE6">
            <el-date-picker
              v-model="orderCountDate"
              style="float:right;z-index:1"
              size="small"
              type="daterange"
              align="right"
              unlink-panels
              range-separator="至"
              start-placeholde="开始日期"
              end-placeholde="结束日期"
              @change="handleDateChange"
               :picker-options="pickerOptions"
            />
            <div>
              <ve-line
                :data="chartData"
                :legend-visible="false"
                :loading="loading"
                :data-empty="dataEmpty"
                :settings="chartSettings"
              />
            </div>
          </div>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
import img_home_order from '@/assets/images/home_order.png'
import { str2Date } from '@/utils/date'

const DATA_FROM_BACKEND = {
  columns: ['date', 'orderCount', 'orderAmount'],
  rows: [
    { date: '2018-11-01', orderCount: 10, orderAmount: 1093 },
    { date: '2018-11-02', orderCount: 20, orderAmount: 2230 },
    { date: '2018-11-03', orderCount: 33, orderAmount: 3623 },
    { date: '2018-11-04', orderCount: 50, orderAmount: 6423 },
    { date: '2018-11-05', orderCount: 80, orderAmount: 8492 },
    { date: '2018-11-06', orderCount: 60, orderAmount: 6293 },
    { date: '2018-11-07', orderCount: 20, orderAmount: 2293 },
    { date: '2018-11-08', orderCount: 60, orderAmount: 6293 },
    { date: '2018-11-09', orderCount: 50, orderAmount: 5293 },
    { date: '2018-11-10', orderCount: 30, orderAmount: 3293 },
    { date: '2018-11-11', orderCount: 20, orderAmount: 2293 },
    { date: '2018-11-12', orderCount: 80, orderAmount: 8293 },
    { date: '2018-11-13', orderCount: 100, orderAmount: 10293 },
    { date: '2018-11-14', orderCount: 10, orderAmount: 1293 },
    { date: '2018-11-15', orderCount: 40, orderAmount: 4293 }
  ]
};

export default {
  data() {
    return {
      img_home_order,
      pickerOptions: {
        shortcuts: [
          {
            text: '最近一周',
            onClick(picker) {
              const end = new Date()
              const start = new Date()
              start.setFullYear(2018)
              start.setMonth(10)
              start.setDate(1)
              end.setTime(start.getTime() + 3600 * 1000 * 24 * 7)
              picker.$emit('pick', [start, end])
            }
          },
          {
            text: '最近一月',
            onClick(picker) {
              const end = new Date()
              const start = new Date()
              start.setFullYear(2018)
              start.setMonth(10)
              start.setDate(1)
              end.setTime(start.getTime() + 3600 * 1000 * 24 * 30)
              picker.$emit('pick', [start, end])
            }
          }
        ]
      },
      orderCountDate: '',
      chartSettings: {
        xAxisType: 'time',
        area: true,
        axisSite: { right: ['orderAmount'] },
        labelMap: { orderCount: '订单数量', orderAmount: '订单金额' }
      },
      chartData: {
        columns: [],
        rows: []
      },
      loading: false,
      dataEmpty: false
    }
  },
  created() {
    this.initOrderCountDate()
    this.getData()
  },
  methods: {
    handleDateChange() {
      this.getData()
    },
    initOrderCountDate() {
      const start = new Date()
      start.setFullYear(2018)
      start.setMonth(10)
      start.setDate(1)
      const end = new Date()
      end.setTime(start.getTime() + 1000 * 60 * 60 * 24 * 7)
      this.orderCountDate = [start, end]
    },
    getData() {
      setTimeout(() => {
        this.chartData = {
          columns: ['date', 'orderCount', 'orderAmount'],
          rows: []
        }
        for (let i = 0; i < DATA_FROM_BACKEND.rows.length; i++) {
          const item = DATA_FROM_BACKEND.rows[i]
          const currDate = str2Date(item.date)
          const start = this.orderCountDate[0]
          const end = this.orderCountDate[1]
          if (
            currDate.getTime() >= start.getTime() &&
            currDate.getTime() <= end.getTime()
          ) {
            this.chartData.rows.push(item)
          }
        }
        this.dataEmpty = false
        this.loading = false
      }, 1000)
    }
  }
}
</script>

<style scoped>
.home-container {
  margin-top: 40px;
  margin-left: 120px;
  margin-right: 120px;
  margin-bottom: 20px
}
.out-border {
  border: 1px solid #dcdfe6;
}
.layout-title {
  padding: 15px 20px;
  font-weight: bold;
  background: #f2f6fc;
  color: #606266;
}
.address-content {
  padding: 20px;
  font-size: 18px;
  color: #409eff;
}
.total-frame {
  border: 1px solid #dcdfe6;
  padding: 20px;
  margin-top: 20px;
  height: 100px;
}
.total-icon {
  color: #409eff;
  width: 60px;
  height: 60px;
}
.total-title {
  position: relative;
  font-size: 16px;
  color: #909399;
  left: 70px;
  top: -60px;
}
.total-value {
  position: relative;
  font-size: 18px;
  color: #606266;
  left: 70px;
  top: -45px;
}
.mine-layout {
  position: absolute;
  right: 10px;
  top: 1px;
  width: 250px;
  height: 235px;
  z-index: 1;
}
.un-handle-layout {
  margin-top: 20px;
  border: 1px solid #dcdfe6;
}
.un-handle-content {
  padding: 20px 40px;
}
.un-handle-item {
  border-bottom: 1px solid #ebeef5;
  padding: 10px;
}
.overview-layout {
  margin-top: 20px;
}
.overview-item-value {
  font-size: 24px;
  text-align: center;
}
.overview-item-title {
  margin-top: 10px;
  text-align: center;
}
.color-danger {
  color: chocolate;
}
.statistics-layout {
    margin-top: 20px;
    border: 1px solid #DCDFE6;
  }
</style>
