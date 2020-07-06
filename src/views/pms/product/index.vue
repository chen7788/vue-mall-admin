<template>
  <div class="app-container">
    <el-card class="filter-container" shadow="never">
      <div>
        <i class="el-icon-search"/>
        <span>筛选搜索</span>
        <el-button style="float:right" type="primary" size="small" @click="handleSeachList()">查询结果</el-button>
        <el-button
          style="float:right;margin-right:15px"
          size="small"
          @click="handleResetSearch()"
        >重置</el-button>
      </div>
      <div style="margin-top:15px" class="form-container">
        <el-form :inline="true" :model="listQuery" size="small" label-width="140px">
          <el-form-item label="输入搜索：">
            <el-input v-model="listQuery.keyword" style="width:203px" placeholder="商品名称" />
          </el-form-item>
          <el-form-item label="商品货号：">
            <el-input v-model="listQuery.productSn" style="width: 203px" placeholder="商品货号"/>
          </el-form-item>
          <el-form-item label="商品分类：">
            <el-cascader v-model="selectProductCateValue" clearable :options="produtCateOptions" />
          </el-form-item>
          <el-form-item label="商品品牌：">
            <el-select v-model="listQuery.brandId" placeholder="请选择品牌" clearable>
              <el-option
                v-for="item in brandOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="上架状态：">
            <el-select v-model="listQuery.publishStatus" placeholder="全部" clearable>
              <el-option
                v-for="item in publishStatusOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="审核状态：">
            <el-select v-model="listQuery.verifyStatus" placeholder="全部" clearable>
              <el-option
                v-for="item in verifyStatusOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
        </el-form>
      </div>
    </el-card>
    <el-card class="operate-container" shadow="never">
      <i class="el-icon-tickets" />
      <span style="padding-left:10px">数据列表</span>
      <el-button class="btn-add" size="mini" style="float:right" @click="handleAddProduct()">添加</el-button>
    </el-card>
    <div class="table-container">
      <el-table
        ref="productTable"
        :data="list"
        style="width:100%"
        @selection-change="handleSelectionChange"
        v-loading="listLoading"
        border
      >
        <el-table-column type="selection" width="60" align="center"></el-table-column>
        <el-table-column label="编号" width="100" align="center">
          <template slot-scope="scope">{{scope.row.id}}</template>
        </el-table-column>
        <el-table-column label="商品图片" width="120" align="center">
          <template slot-scope="scope">
            <img style="height:80px" :src="scope.row.pic" >
          </template>
        </el-table-column>
        <el-table-column label="商品名称" align="center">
          <template slot-scope="scope">
            <p>{{scope.row.name}}</p>
            <p>品牌：{{scope.row.brandName}}</p>
          </template>
        </el-table-column>
        <el-table-column label="价格/货号" width="120" align="center">
          <template slot-scope="scope">
            <p>价格：￥{{scope.row.price}}</p>
            <p>货号：{{scope.row.productSn}}</p>
          </template>
        </el-table-column>
        <el-table-column label="标签" width="140" align="center">
          <template slot-scope="scope">
            <p>
              上架：
              <el-switch
                v-model="scope.row.publishStatus"
                :inactive-value="0"
                :active-value="1"
                @change="handlePublishStatusChange(scope,$index,scope.row)"
              />
            </p>
            <p>
              新品：
              <el-switch
                v-model="scope.row.publishStatus"
                :inactive-value="0"
                :active-value="1"
                @change="handlePublishStatusChange(scope,$index,scope.row)"
              />
            </p>
            <p>
              推荐：
              <el-switch
                v-model="scope.row.publishStatus"
                :inactive-value="0"
                :active-value="1"
                @change="handlePublishStatusChange(scope,$index,scope.row)"
              />
            </p>
          </template>
        </el-table-column>
        <el-table-column label="排序" width="100" align="center">
          <template slot-scope="scope">{{scope.row.sort}}</template>
        </el-table-column>
        <el-table-column label="SKU库存" width="100" align="center">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              circle
              @click="handleShowSkuEditDialog(scope.$index, scope.row)"
            />
          </template>
        </el-table-column>
        <el-table-column label="销量" width="100" align="center">
          <template slot-scope="scope">{{scope.row.sale}}</template>
        </el-table-column>
        <el-table-column label="审核状态" width="100" align="center">
          <template slot-scope="scope">
            <p>{{scope.row.verifyStatus | verifyStatusFilter}}</p>
            <p>
              <el-button type="text" @click="handleShowVerifyDetail(scope.$index, scope.row)">审核详情</el-button>
            </p>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="160" align="center">
          <template slot-scope="scope">
            <p>
              <el-button size="mini" @click="handleShowProduct(scope.$index, scope.row)">查看</el-button>
              <el-button size="mini" @click="handleUpdateProduct(scope.$index, scope.row)">编辑</el-button>
            </p>
            <p>
              <el-button size="mini" @click="handleShowLog(scope.$index, scope.row)">日志</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
            </p>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="batch-operate-container">
      <el-select v-model="operateType" size="small" placeholder="批量操作">
        <el-option
          v-for="item in operates"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        />
      </el-select>
      <el-button
        style="margin-left: 20px"
        class="search-button"
        type="primary"
        size="small"
        @click="handleBatchOperate()"
      >确定</el-button>
    </div>
    <div class="pagination-container">
      <el-pagination
        background
        layout="total, sizes,prev, pager, next,jumper"
        :page-size="listQuery.pageSize"
        :page-sizes="[5,10,15]"
        :total="total"
        :current-page.sync="listQuery.pageNum"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
    <el-dialog title="编辑货品信息" :visible.sync="editSkuInfo.dialogVisible" width="40%">
      <span>商品货号：</span>
      <span>{{ editSkuInfo.productSn }}</span>
      <el-input v-model="editSkuInfo.keyword" placeholder="按sku编号搜索" size="small" style="width:50%">
        <el-button slot="append" icon="el-icon-search" @click="handleSearchEditSku" />
      </el-input>
      <el-table style="width: 100%;margin-top: 20px" :data="editSkuInfo.stockList" border>
        <el-table-column label="SKU编号" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.skuCode" />
          </template>
        </el-table-column>
        <el-table-column
          v-for="(item,index) in editSkuInfo.productAttr"
          :key="item.id"
          :label="item.name"
          align="center"
        >
          <template slot-scope="scope">{{ getProductSkuSp(scope.row,index) }}</template>
        </el-table-column>
        <el-table-column label="销售价格" width="80" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.price"/>
          </template>
        </el-table-column>
        <el-table-column label="商品库存" width="80" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.stock"/>
          </template>
        </el-table-column>
        <el-table-column label="库存预警值" width="100" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.lowStock"/>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editSkuInfo.dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleEditSkuConfirm">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
const defaultListQuery = {
  keyword: null,
  pageNum: 1,
  pageSize: 5,
  publishStatus: null,
  verifyStatus: null,
  productSn: null,
  productCategoryId: null,
  brandId: null
}
export default {
  name: 'productList',
  data() {
    return {
      editSkuInfo: {
        dialogVisible: false,
        productId: null,
        productSn: "",
        productAttributeCategoryId: null,
        stockList: [],
        productAttr: [],
        keyword: null
      },
      operates: [
        {
          label: "商品上架",
          value: "publishOn"
        },
        {
          label: "商品下架",
          value: "publishOff"
        },
        {
          label: "设为推荐",
          value: "recommendOn"
        },
        {
          label: "取消推荐",
          value: "recommendOff"
        },
        {
          label: "设为新品",
          value: "newOn"
        },
        {
          label: "取消新品",
          value: "newOff"
        },
        {
          label: "转移到分类",
          value: "transferCategory"
        },
        {
          label: "移入回收站",
          value: "recycle"
        }
      ],
      operateType: null,
      listQuery: Object.assign({}, defaultListQuery),
      list: null,
      total: null,
      listLoading: true,
      selectProductCateValue: null,
      multipleSelection: [],
      productCateOptions: [],
      brandOptions: [],
      publishStatusOptions: [
        {
          value: 1,
          label: "上架"
        },
        {
          value: 0,
          label: "下架"
        }
      ],
      verifyStatusOptions: [
        {
          value: 1,
          label: "审核通过"
        },
        {
          value: 0,
          label: "未审核"
        }
      ]
    }
  }
};
</script>

<style lang="scss">
.form-container {
    padding-top: 10px;
    .el-select {
        width:203px
    }
    .el-cascader{
        width:203px
    }
}
</style>

<style scoped>
.el-icon-search {
  padding-right: 10px;
}
.operate-container{
    margin-top: 10px;
}
</style>
