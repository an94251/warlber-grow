<template>
  <div>
    <breadcrumb/>
    <!--头部-->
    <search-bar type="stock_report" @onSearch="handleFilter"/>
    <!--表格-->
    <div>
      <el-table
        :data="tableData"
        v-loading="loading"
        class="table-compact"
        ref="multipleTable"
        stripe
        border
        style="width: 100%">
        <el-table-column sortable prop="updateTime" label="上报日期" width="100">
          <template slot-scope="scope">
            {{scope.row.updateTime|date}}
          </template>
        </el-table-column>
        <el-table-column sortable prop="customerName" label="客户名称"></el-table-column>
        <!--<el-table-column sortable prop="customerLabel" label="客户类型">-->
        <!--<template slot-scope="scope">-->
        <!--<span v-for="item in scope.row.labelList">【{{item.labelName}}】</span>-->
        <!--</template>-->
        <!--</el-table-column>-->
        <el-table-column sortable prop="manager" label="客户经理" width="100"></el-table-column>
        <el-table-column sortable prop="itemCode" label="商品编码" width="100"></el-table-column>
        <el-table-column sortable prop="itemName" label="商品名称"></el-table-column>
        <el-table-column sortable prop="specification" label="规格属性"></el-table-column>
        <el-table-column sortable prop="itemCategory" label="商品类型"></el-table-column>
        <el-table-column sortable prop="itemBrand" label="品牌"></el-table-column>
        <el-table-column sortable prop="salesmanName" label="上报人" width="100"></el-table-column>
        <!--<el-table-column sortable  label="操作">-->
        <!--<template slot-scope="scope">-->
        <!--<el-button type="text" @click="">待定</el-button>-->
        <!--</template>-->
        <!--</el-table-column>-->
      </el-table>
      <el-pagination
        background
        @current-change="handlePaginationChange"
        @size-change="handleSizeChange"
        :current-page="page.pageNum"
        :page-size="page.pageSize"
        layout="total, sizes,prev, pager, next, jumper"
        :total="page.total">
      </el-pagination>
    </div>
  </div>
</template>

<script>
  import SearchBar from '../search-bars'

  export default {
    components: {
      SearchBar
    },
    data() {
      return {
        tableData: [],
        page: {
          pageNum: 1,
          pageSize: 20,
          sortName: '',
          sortOrder: '',
          total: 0,
          keyword: '',
          orderId: ''
        },
        searchBoxVisible: true,
        messageQuestion: "这是铺货上报明细,在这里你可以客户的商品的铺货情况",
        loading: true
      }
    },
    created() {
      this.init(this.$route.query);
      this.loadTableData();
    },
    methods: {
      init(val) {
        if (val && val.id) {
          this.page.orerId = val.id;
        }
      },
      /* 加载表格数据*/
      loadTableData() {
        this.loading = true;
        this.$http.get('psi/distrib/details/list', this.page, r => {
          this.tableData = r.data.list;
          this.page.total = r.data.total;
          this.loading = false;
        });
      },
      // 分页
      handlePaginationChange(pageNum) {
        this.page.pageNum = pageNum;
        this.loadTableData();
      },
      // 分页改变事件
      handleSizeChange(val) {
        this.page.pageSize = val;
        this.loadTableData();
      },
      // 筛选
      handleFilter(val) {
        this.page = this.$util.extend(this.page, val);
        this.loadTableData();
      },
      handleClearFilter() {
        this.page.startDate = null;
        this.page.endDate = null;
        this.loadTableData();
      }
    },
    watch: {},
    computed: {}
  }
</script>

<style scoped>

</style>
