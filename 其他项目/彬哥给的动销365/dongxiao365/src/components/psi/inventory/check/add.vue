<template>
  <div>
    <breadcrumb/>
    <h3>基本信息：</h3>
    <!------------第一部分-------------->
    <el-form :inline="true" :model="order" ref="ruleForm" :rules="rules">
      <el-form-item label="仓库：">
        <com-warehouse v-model="order.warehouseId"></com-warehouse>
      </el-form-item>
      <el-form-item label="类型：" prop="type">
        <el-select v-model="order.type" placeholder="类型">
          <el-option
            v-for="item in type"
            :key="item.value"
            :label="item.name"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="盘点日期：" prop="checkDate">
        <el-date-picker
          v-model="order.checkDate"
          type="date"
          clearable
          placeholder="选择日期时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="盘点人：">
        <com-chose-user v-model="order.operatorId"></com-chose-user>
      </el-form-item>
    </el-form>
    <hr>
    <h3>商品信息：</h3>
    <!-------------------按钮--------------->
    <div class="tool-box" v-if="this.order.warehouseId">
      <el-button @click="chooseCommodity" type="warning" icon="el-icon-plus">选择商品</el-button>&nbsp;&nbsp;&nbsp;
      <el-checkbox v-model="checkedAll" @change="getCheckedAll">盘点仓库全部商品</el-checkbox>
    </div>
    <div class="tool-box" v-else>
      <div class="el-upload__tip">提示：选择仓库以后才可以添加商品</div>
    </div>
    <!--表格-->
    <el-row>
      <el-form :model="dynamicValidateForm" ref="dynamicValidateForm" label-width="0px" class="table-dynamic-form">
        <el-table
          :data="dynamicValidateForm.commodityList"
          stripe
          border
          style="width: 100%">
          <el-table-column label="选择" width="60px">
            <template slot-scope="scope">
              <div class="cell" v-text="scope.$index+1"></div>
            </template>
          </el-table-column>
          <el-table-column prop="itemName" label="商品名称"></el-table-column>
          <el-table-column prop="specification" label="规格属性"></el-table-column>
          <el-table-column prop="inventoryCount" label="库存数量" align="right">
            <template slot-scope="scope">
              <el-tooltip effect="dark" :content="'1'+scope.row.auxiliaryUnitName+'='+scope.row.unitConvertRatio+''+scope.row.basicUnitName" placement="top-start" v-if="scope.row.auxiliaryUnitName">
                <div>{{scope.row,scope.row.inventoryCount|unit}}</div>
              </el-tooltip>
              <div v-else>
                {{scope.row,scope.row.inventoryCount|unit}}
              </div>
            </template>
          </el-table-column>
          <el-table-column prop="checkCount" label="盘点数量">
            <template slot-scope="scope">
              <el-row :gutter="10">
                <el-col :span="12">
                  <el-form-item
                    :prop="'commodityList.' + scope.$index + '.checkCount'"
                    :rules="[{ required: true, message: '请输入盘点数量', trigger: 'blur' }]">
                    <el-input-number v-model.number="scope.row.checkCount" :min="0" controls-position="right" class="input-full"></el-input-number>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item
                    :prop="'commodityList.' + scope.$index + '.itemUnitId'"
                    :rules="[{ required: true, message: '请选择单位', trigger: 'blur' }]">
                    <com-item-unit v-model="scope.row.itemUnitId" :itemId="scope.row.itemId" class="input-full"></com-item-unit>
                  </el-form-item>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <el-table-column prop="diffCount" label="差异数量" align="right">
            <template slot-scope="scope">
              {{scope.row|diffUnit}}
            </template>
          </el-table-column>
          <el-table-column fixed="right" label="操作" width="80" align="center">
            <template slot-scope="scope">
              <el-button type="button" class="text-danger" @click="deleteCommodity(scope.$index)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form>
    </el-row>
    <!--底部-->
    <h3>备注：</h3>
    <el-row>
      <el-input
        type="textarea"
        :rows="6"
        placeholder="请输入备注,长度在500字以内"
        v-model="order.remark"
        :maxlength="500">
      </el-input>
    </el-row>
    <br/>
    <div>
      <el-button @click="$router.back()">返回</el-button>
      <el-button type="primary" @click="onSubmit('ruleForm')">保存</el-button>
    </div>
    <!------------弹窗部分---------->
    <el-dialog title="选择商品" :visible.sync="dialog.chooseCommodityVisible" width="800px">
      <div>
        <el-row>
          <el-col :span="6">
            <el-input placeholder="请输入商品类型" v-model="filterText" suffix-icon="el-icon-search" clearable></el-input>
            <!---------------商品组织结构---------------->
            <br/>
            <br/>
            <el-tree v-loading="treeLoading"
                     :data="commodityTree"
                     :props="defaultProps"
                     :highlight-current="true"
                     node-key="id"
                     @node-click="handleNodeClick"
                     default-expand-all
                     ref="tree"
                     :filter-node-method="filterNode">
            </el-tree>
          </el-col>
          <el-col :span="18">
            <!--------------------商品列表展示----------------------->
            <!--头部按钮-->
            <div class="text-right">
              <el-form :inline="true" class="form-compact" @submit.native.prevent>
                <el-form-item>
                  <el-search v-model="page.keyword" @click="loadTableData" placeholder="请输入商品名称或商品编码" @keyup.enter.native="loadTableData"></el-search>
                </el-form-item>
              </el-form>
            </div>
            <br/>
            <el-table
              v-loading="tableLoading"
              :data="tableData"
              class="table-compact"
              stripe
              row-key="id"
              width="100%"
              height="500"
              ref="multipleTable"
              @selection-change="handleTableSelectionChange"
              border>
              <el-table-column type="selection" width="55" reserve-selection></el-table-column>
              <el-table-column sortable prop="itemCode" width="114" label="编码"></el-table-column>
              <el-table-column sortable prop="itemName" width="250" label="商品名称"></el-table-column>
              <el-table-column sortable prop="inventoryCount" width="150" label="当前库存"></el-table-column>
            </el-table>
          </el-col>
        </el-row>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialog.chooseCommodityVisible = false">取消</el-button>
        <el-button type="primary" @click="shureForChooseItem">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import ComChoseUser from '../../../lib/use-selector'
  import ComWarehouse from '../../../lib/warehouse'
  import ComItemUnit from '../../../lib/item-unit'
  import ItemUtil from '../../../../util/itemUtil'

  export default {
    components: {
      ComChoseUser, ComItemUnit, ComWarehouse
    },
    data() {
      return {
        checkedAll: false,
        type: [
          {'value': 0, 'name': '部分盘点'},
          {'value': 1, 'name': '整仓盘点'},
          {'value': 2, 'name': '抽样盘点'},
          {'value': 3, 'name': '其他'}
        ],
        warehouse: [],
        units: [],
        order: {warehouseId: 0, checkDate: new Date()},
        dynamicValidateForm: {
          commodityList: []
        },
        dialog: {  // 弹窗
          chooseCommodityVisible: false,
          chooseUserVisible: false// 选择人员
        },
        index: null, // 当前表单选择商品的行的下标
        products: [],
        state: "",
        indexForCodeSearch: 0,
        defaultProps: {// 组织架构基础数据类型
          children: 'children',
          label: 'name',
          value: 'id'
        },
        filterText: '', // 商品组织结构筛选
        treeLoading: false,
        commodityTree: [{'name': '全部商品', 'id': 0, 'children': []}], // 商品组织结构
        page: {
          categoryId: null,
          keyword: ''
        },
        tableLoading: false,
        tableData: [],
        chosenList: [], // 商品表单列表选中集合
        rules: {
          checkDate: [
            {required: true, message: '请选择盘点日期', trigger: 'blur'}
          ]
        }
      }
    },
    mounted() {
      this.loadCommodityTree();
    },
    methods: {
      shureForChooseItem() {
        this.dynamicValidateForm.commodityList = JSON.parse(JSON.stringify(this.chosenList));
        this.dialog.chooseCommodityVisible = false;
      },
      deleteCommodity(index) {
        this.dynamicValidateForm.commodityList.splice(index, 1);
      },
      chooseCommodity() {
        this.dialog.chooseCommodityVisible = true;
        this.page.categoryId = null;
        if (this.$refs.tree) { this.$refs.tree.setCurrentKey(0); }
        this.loadTableData();
      },
      /* 加商品织架构*/
      loadCommodityTree() {
        this.treeLoading = true;
        this.$http.get('/psi/item/type/list', {}, r => {
          this.commodityTree[0].children = r.data;
          this.treeLoading = false;
        });
      },
      // 是否盘点仓库全部商品
      getCheckedAll(val) {
        if (val === true) {
          this.dynamicValidateForm.commodityList = this.tableData;
        } else {
          this.dynamicValidateForm.commodityList = [];
        }
      },
      handleRowKey(row) {
        return row.id;
      },
      handleNodeClick(data) {
        this.page.categoryId = data.id;
        this.loadTableData();
      },
      /* 商品组织架构的过滤*/
      filterNode(value, data) {
        if (!value) { return true; }
        return data.name.indexOf(value) !== -1;
      },
      loadTableData() {
        if (this.order.warehouseId > 0) {
          this.tableLoading = true;
          this.page.warehouseId = this.order.warehouseId;
          this.$http.get('/psi/stock/list/all/simple', this.page, r => {
            this.tableData = r.data;
            this.tableLoading = false;
            if (this.$refs.multipleTable) { this.$refs.multipleTable.doLayout(); }
          });
        }
      },
      onSubmit(formName) {
        if (!this.dynamicValidateForm.commodityList || this.dynamicValidateForm.commodityList.length === 0) {
          this.$message({
            message: '请选择需要盘点的商品',
            type: 'error'
          });
          return;
        }
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.submit();
          }
        });
      },
      submit() {
        const params = {
          warehouseId: this.order.warehouseId,
          type: this.order.type,
          operatorId: this.order.operatorId,
          operatorName: this.order.operatorName,
          checkDate: this.order.checkDate,
          remark: this.order.remark,
          detailsRequestList: [],
          result: 1
        };
        for (let i = 0; i < this.dynamicValidateForm.commodityList.length; i++) {
          const obj = this.dynamicValidateForm.commodityList[i];
          const item = {};
          const count = ItemUtil.toBase(obj, obj.itemUnitId, obj.checkCount);
          const inventoryCount = ItemUtil.toBase(obj, obj.itemUnitId, obj.inventoryCount);
          item.itemId = obj.itemId;
          item.receiptCount = inventoryCount;
          item.checkCount = count;
          item.diffCount = inventoryCount - count;
          if (item.diffCount !== 0) {
            params.result = -1;
          }
          params.detailsRequestList.push(item)
        }
        this.$http.post('psi/item/stock/check/add', params, r => {
          this.$message({
            type: "success",
            message: "添加成功"
          });
          this.$router.back();
        })
      },
      handleTableSelectionChange(val) {
        this.chosenList = val;
      },
      // 表格选中状态
      toggleSelection(rows) {
        if (rows) {
          rows.forEach(row => {
            this.$refs.multipleTable.toggleRowSelection(row);
          });
        } else {
          this.$refs.multipleTable.clearSelection();
        }
      }
    },
    watch: {
      /* 商品组织架构的过滤*/
      filterText(val) {
        this.$refs.tree.filter(val);
      },
      // 当仓库每次变化时都要重置列表
      'order.warehouseId': function (val) {
        this.dynamicValidateForm.commodityList = [];
        if (val) { this.loadTableData(); }
      }
    },
    filters: {
      unit(item, val) {
        return ItemUtil.format(item, val);
      },
      diffUnit(item) {
        const unitId = item.itemUnitId;
        const inventoryCount = item.inventoryCount;
        let checkCount = item.checkCount;
        if (!checkCount) { checkCount = 0; }
        const auxiliaryUnitId = item.auxiliaryUnitId;
        if (unitId === auxiliaryUnitId) {
          const unitConvertRatio = item.unitConvertRatio;
          const val = inventoryCount - checkCount * unitConvertRatio;
          return ItemUtil.format(item, val);
        } else {
          return ItemUtil.format(item, inventoryCount - checkCount);
        }
      }
    }
  }
</script>

<style scoped>
  .form_content {
    color: #17c295;
  }

  .stamp {
    font-size: 18px;
    position: absolute;
    right: 10px;
    top: -20px;
    z-index: 100;
    transform: rotate(-30deg);
  }

  .stamp_yes {
    padding: 6px 12px;
    color: green;
    border: double 3px;
  }

  .stamp_no {
    padding: 6px 12px;
    color: red;
    border: double 3px;
  }

  .stamp_not_excute {
    padding: 6px 12px;
    color: #F39C12;
    border: double 3px;
  }

  .title {
    font-size: 20px;
    font-weight: bold;
  }
</style>
