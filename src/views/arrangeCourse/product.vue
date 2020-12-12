<template>
  <div class="cark1">
    <div style="margin-bottom: 16px">
      <!-- 清空选中的 -->
      <a-button type="primary" :disabled="!hasSelected" :loading="loading" @click="deleteBatch"> 批量删除 </a-button>
      <a-button type="primary" style="margin-left: 20px" @click="DrawerFun('add')">
        <a-icon type="plus-circle" />
        新增产品
      </a-button>

    </div>
    <!--  表格的内容 -->
    <a-spin :spinning="spinning">
      <a-table
        :columns="columns"
        :pagination="false"
        :data-source="data"
        rowKey="id"
        bordered
        :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
      >
        <!--操作页面 -->
        <span slot="action" slot-scope="text, record">
          <div style="display: flex">
            <a-button type="primary" size="small" @click="DrawerFun('update', record)">
              <a-icon type="edit" /> 修改
            </a-button>
            <a-divider type="vertical" />
            <a-button type="danger" size="small" @click="deleteItem(record)"> 删除 </a-button>
          </div>
        </span>
      </a-table>
    </a-spin>
    <!-- 分页 -->
    <a-pagination :default-current="1" v-model="pageNo" :total="total" show-quick-jumper @change="reloadAll(pageNo)"  show-size-changer
      @showSizeChange="changePageSize" />
    <!-- 弹出框 -->
    <a-drawer
      :title="product == undefined ? '产品录入' : '修改'"
      :closable="false"
      :visible="showDrawer"
      @close="showDrawer = false"
      width="500px"
    >
      <sidebar> 录入</sidebar>
      <div style="margin: 15px">
        <table class="add">
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">产品名称: </span>
            </td>
            <td>
              <a-select style="width: 230px; margin-left: 15px" v-model="product" placeholder="请选择产品">
                <a-select-option v-for="item in productList" :key="item" :value="item"> {{ item }}</a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">学期段 :</span>
            </td>
            <td>
              <a-select
                style="width: 230px; margin-left: 15px"
                @change="changeClassTime"
                v-model="classTime"
                placeholder="请选择学期段"
                mode="multiple"
              >
                <a-select-option v-for="item in semesterList" :key="item" :value="item"> {{ item }}</a-select-option>
              </a-select>
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">单价 :</span>
            </td>
            <td>
              <a-input-number
                placeholder="请输入单价"
                :formatter="(value) => `¥ ${value}`.replace(/\B(?=(\d{3})+(?!\d))/g, ',')"
                style="width: 160px; margin-left: 15px"
                v-model="price"
                :parser="(value) => value.replace(/\¥\s?|(,*)/g, '')"
              />
              <a-select style="width: 65px; margin-left: 5px" v-model="unit" default-value="次">
                <a-select-option :value="'次'"> 次</a-select-option>
                <a-select-option :value="'月'"> 月</a-select-option>
              </a-select>
            </td>
          </tr>
          <tr v-if="unit=='月'">
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">折算率 :</span>
            </td>
            <td>
              <a-input-number
                placeholder="请输入折算率"
                style="width: 160px; margin-left: 15px"
                v-model="conversionRate"
                :parser="(value) => value.replace(/\¥\s?|(,*)/g, '')"
              />
            </td>
          </tr>
        </table>
        <div style="margin-left: -10px">
          <sidebar>
            优惠设置
            <a-button @click="addDiscounts" size="small" style="margin-left: 8px" type="primary">添加优惠</a-button>
          </sidebar>
        </div>
        <table class="add" v-for="(item, i) of discountsList" :key="i">
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">达到:</span>
            </td>
            <td>
              <a-input-number
                style="width: 230px; margin-left: 40px"
                placeholder="请输入优惠价格"
                :formatter="(value) => `¥ ${value}`.replace(/\B(?=(\d{3})+(?!\d))/g, ',')"
                type="number"
                :parser="(value) => value.replace(/\¥\s?|(,*)/g, '')"
                v-model="item.sum"
              />
            </td>
            <td>
              <a-button style="margin-left: 10px" type="danger" size="small" @click="discountsList.splice(index, 1)">
                删除
              </a-button>
            </td>
          </tr>
          <tr>
            <td>
              <img src="../../assets/星号.png" width="10" height="10" />
              <span style="margin-left: 5px">减免:</span>
            </td>
            <td>
              <a-input-number
                style="width: 230px; margin-left: 40px"
                :formatter="(value) => `¥ ${value}`.replace(/\B(?=(\d{3})+(?!\d))/g, ',')"
                v-model="item.reduce"
                :parser="(value) => value.replace(/\¥\s?|(,*)/g, '')"
              />
            </td>
          </tr>
        </table>
      </div>
      <div style="text-align: right; margin-top: 15px"><a-button @click="submit" type="primary"> 确定</a-button></div>
    </a-drawer>
  </div>
</template>
<script>
const columns = [
  {
    title: '产品',
    dataIndex: 'productName',
  },
  {
    title: '学期段',
    dataIndex: 'semester',
  },
  {
    title: '单价',
    dataIndex: 'price',
    customRender: (text, record) => {
      return record.price + '/' + record.unit
    },
  },
  {
    title: '优惠',
    dataIndex: 'reducePrice',
    customRender: (text, record) => {
      let re = ''
      for (let item of JSON.parse(record.reducePrice)) {
        re += '满' + item.sum + '减' + item.reduce + '' + ' '
      }
      return re
    },
  },

  {
    title: '操作',
    dataIndex: 'action',
    scopedSlots: { customRender: 'action' },
  },
]
import moment from 'moment'
import Sidebar from '../../components/sidebar.vue'

export default {
  components: { Sidebar },
  data() {
    return {
      id: '',
      data: [],
      selectedRowKeys: [], // 选中的对象
      loading: false,
      columns, //表头的内容
      showDrawer: false,
      name: '',
      // 基本信息
      classTime: [], //学期段
      semesterList: [],
      price: 0, //单价
      productList: [],
      product: '', //产品名称
      unit: '', //单位
      showdetails: false,
      statusBtn: '', //按钮的状态
      //  删除 loading
      removeLoading: false,
      //   优惠
      discountsList: [
        {
          sum: '', //优惠价格
          reduce: '', //生成优惠价格
        },
      ],
      conversionRate:0,//折算率
      pageNo: 1,
      total: 0,
      spinning: false,
      pageSize :10,
    }
  },
  computed: {
    //清空
    hasSelected() {
      return this.selectedRowKeys.length > 0
    },
  },
  methods: {
     // 页码数量
    changePageSize(current, size) {
      this.pageSize = size
      this.reloadAll(current)
    },
    //   添加优惠
    addDiscounts() {
      let obj = {
        sum: '', //优惠价格
        reduce: '', //生成优惠价格
      }
      this.discountsList.push(obj)
    },
    // 删除目分数情况
    deleteItem(obj) {
      let $this = this
      this.$confirm({
        title: '确认删除?',
        onOk() {
          $this.axios.post('pc/product/saveOrUpdateProduct', { id: obj.id, delFlag: 1 }).then((res) => {
            if (res.success && res.result) {
              $this.$message.info(res.message)
              $this.reloadAll(1)
            } else {
              $this.$message.error('删除失败')
            }
          })
        },
      })
    },
    //学期段
    changeClassTime(value) {
      this.classTime = value
    },
    //修改和新增弹窗按钮
    DrawerFun(type, obj) {
      this.loadAddInfo()
      this.showDrawer = true
      this.statusBtn = type
      this.id = type == 'add' ? '' : obj.id
      this.product = type == 'add' ? '' : obj.productName
      this.price = type == 'add' ? '' : obj.price
      this.conversionRate = type == 'add' ? '' : obj.conversionRate
      this.unit = type == 'add' ? '次' : obj.unit
      this.classTime = type == 'add' ? [] : obj.semester.split(',')
      this.discountsList = type == 'add' ? [] : JSON.parse(obj.reducePrice)
      // 禁用弹出框的内容
      this.showdetails = type != 'add'
    },
    deleteBatch() {
      let $this = this
      $this.loading = true
      let ids = []
      for (let item of $this.selectedRowKeys) {
        let obj = {
          id: item,
          delFlag: 1,
        }
        ids.push(obj)
      }
      this.$confirm({
        title: '确认删除?',
        onOk() {
          $this.axios.post('pc/product/updateBatch', ids).then((res) => {
            if (res.success && res.result) {
              $this.$message.info(res.message)
              $this.reloadAll(1)
            } else {
              $this.$message.error('删除失败')
            }
            $this.loading = false
          })
        },
        onCancel() {
          $this.loading = false
        },
      })
    },
    //保存
    submit() {
      if (
        this.product == '' ||
        this.classTime == undefined ||
        this.price == '' ||
        this.unit == '' ||
        this.discountsList == undefined
      ) {
        this.$message.error('以上内容不能为空')
        return
      }
      this.showDrawer = false
      let data = {
        productName: this.product,
        semester: this.classTime.join(','),
        price: this.price,
        unit: this.unit,
        conversionRate: this.conversionRate,
        reducePrice: JSON.stringify(this.discountsList),
      }
      if (this.id != '' && this.statusBtn == 'update') {
        data.id = this.id
      }
      this.axios.post('pc/product/saveOrUpdateProduct', data).then((res) => {
        if (res.success && res.result) {
          this.reloadAll(1)
          this.$message.info(res.message)
        } else {
          this.$message.error('保存失败')
        }
      })
    },
    //  全选的对象
    onSelectChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    loadAddInfo() {
      this.productList = []
      this.axios.post('vx/coursePublish/findCategoryByName', { name: '产品' }).then((res) => {
        for (let category of res.result) {
          this.productList.push(category.title)
        }
      })
      if (this.semesterList.length == 0) {
        this.axios.post('vx/coursePublish/findCategoryByName', { name: '学期段' }).then((res) => {
          for (let category of res.result) {
            this.semesterList.push(category.title)
          }
        })
      }
    },
    reloadAll(current) {
      let data = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
      }
      this.spinning = true
      this.axios.post('pc/product/listByPage', data).then((res) => {
        if (res.success) {
           if(res.result.records.length!=0){
                this.data = res.result.records
                this.total = res.result.total
            }
        }
        this.spinning = false
      })
    },
  },
  mounted() {
    this.reloadAll(1)
  },
}
</script>
<style scoped>
.cark1 {
  background-color: #fff;
  border-radius: 10px;
  padding: 10px 10px;
}

.flex_center {
  display: flex;
  align-items: center;
  margin-top: 10px;
}

::v-deep .ant-table-thead > tr > th {
  color: rgba(0, 0, 0, 0.57);
  font-weight: 550;
}

.add tr {
  height: 60px;
}

::v-deep .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
</style>
<style>
</style>

