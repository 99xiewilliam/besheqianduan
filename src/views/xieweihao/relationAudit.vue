<template>
  <div class="app-container">
    <el-container>
      <el-header>
        审查关系标注数据
      </el-header>
      <el-container id="button_bg">
        <el-main id="button">
          <el-button @click="handleClick()">实体数据审核</el-button>
          <el-button>关系数据审核</el-button>
        </el-main>
      </el-container>
      <el-container class="mydiv1">
        <el-container>
          <div class="div1">当前关系类型:</div>
          <el-select v-model="value_option2" placeholder="请选择活动区域" @change="handleChange($event)">
            <el-option
              v-for="item in options2"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </el-container>
        <el-container class="mydiv2">
          <el-button type="info" class="inRight" @click="getTable()">表格审核</el-button>
          <el-button type="success" class="inRight" @click="getChart()">图像显示</el-button>
        </el-container>
      </el-container>
      <!-- <el-dialog title="修改关系" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="" :label-width="formLabelWidth">
            <el-select v-model="value_option1" placeholder="请选择活动区域" @change="handleChange($event)">
              <el-option
                v-for="item in options1"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="起始实体：" :label-width="formLabelWidth">
            <el-select v-model="value" placeholder="请选择">
              <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="目标实体：" :label-width="formLabelWidth">
            <el-select v-model="value" placeholder="请选择">
              <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>

        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="danger" @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="success" @click="dialogFormVisible = false">确 定</el-button>
        </div>
      </el-dialog>

      <el-dialog title="添加关系" :visible.sync="dialogFormVisible1">
        <el-form :model="form">
          <el-form-item label="" :label-width="formLabelWidth">
            <el-select v-model="value" placeholder="请选择">
              <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="起始实体：" :label-width="formLabelWidth">
            <el-input placeholder="请输入疾病" size="mini" />
          </el-form-item>
          <el-form-item label="目标实体：" :label-width="formLabelWidth">
            <el-input placeholder="请输入疾病" size="mini" />
          </el-form-item>
          <el-form-item label="建议：" :label-width="formLabelWidth">
            <el-select v-model="value" placeholder="请选择">
              <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="证据级别：" :label-width="formLabelWidth">
            <el-input placeholder="请输入证据级别" size="mini" />
          </el-form-item>
          <el-form-item label="临床依据：" :label-width="formLabelWidth">
            <el-input placeholder="请输入临床依据" size="mini" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="danger" @click="dialogFormVisible1 = false">取 消</el-button>
          <el-button type="success" @click="dialogFormVisible1 = false">确 定</el-button>
        </div>
      </el-dialog>
      <el-drawer
        title="新建关系"
        :visible.sync="drawer"
        :direction="direction"
        :before-close="handleClose"
      >
        <div>
          <span>当前选中的起点实体：</span>
          <span>实体类型</span>
          <span />
        </div>
        <div>
          <span>实体ID</span>
          <span />
        </div>
        <div>
          <span>属性摘要</span>
          <span />
        </div>
        <el-divider />
        <div>
          <span>终点实体配置：</span>
          <span>类型</span>
          <span>
            <el-select v-model="value" placeholder="请选择">
              <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </span>
        </div>
        <div>
          <span>
            实体ID
          </span>
          <span />
        </div>
        <div>
          <span>
            属性摘要
          </span>
          <span />
        </div>
        <el-divider />
        <div>
          <span>
            实体关系配置
          </span>
          <el-select v-model="value" placeholder="请选择">
            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </div>
        <div style="text-align: center;height:60px;">
          <div style="text-align: center;height:40px;">
            <el-button type="primary" class="margin-top:20px;">
              提交
            </el-button>
          </div>
        </div>

      </el-drawer> -->
      <div class="margin">
        <el-autocomplete
          v-model="state"
          :fetch-suggestions="querySearchAsync"
          placeholder="请输入内容"
          @select="handleSelect"
        />
      </div>
      <div class="divMain" id="main" v-show="showChart">
        <!-- <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">
          <div id="main" class="width: 100%; height: 100%">

          </div>
        </el-row> -->
      </div>
      <el-table v-loading="listLoading" :data="nowTable" :row-key="getRowKey" border fit highlight-current-row style="width: 100%" v-show="showTable">
        <el-table-column align="center" label="序号" width="65">
          <template slot-scope="{row}">
            <span>{{ row.id }}</span>
          </template>
        </el-table-column>

        <el-table-column min-width="240px" label="关系描述">
          <template slot-scope="{row}">
            <span>{{ row.evi_describe }}</span>
          </template>
        </el-table-column>

        <el-table-column width="150px" align="center" label="起点实例ID">
          <template slot-scope="{row}">
            <span>{{ row.document_id }}</span>
          </template>
        </el-table-column>

        <el-table-column class-name="status-col" label="起点实例摘要" width="80px">
          <template slot-scope="{row}">
            <span>{{ row.start_object }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="终点实例ID" width="180">
          <template slot-scope="{row}">
            <span>{{ row.document_id }}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" label="终点实例摘要" width="180">
          <template slot-scope="{row}">
            <span>{{ row.end_object }}</span>
          </template>
        </el-table-column>
        <el-table-column class-name="status-col" label="审核情况" width="110">
          <template slot-scope="{ row }">
            <el-tag :type="row.is_passed | statusFilter">
              <span v-if="row.is_passed == true">通过</span>
              <span v-else>不通过</span>
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column align="center" label="审核" min-width="180">
          <template slot-scope="{row}">
            <el-button @click="changeStatus(row, 1)">合格</el-button>
            <el-button @click="changeStatus(row, 0)">不合格</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        background
        layout="total, sizes, prev, pager, next, jumper"
        :total="list.length"
        :page-size="pagesize"
        :current-page="currentPage"
        :page-sizes="[1, 2, 5, 10]"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-container>
  </div>
</template>

<script>
import { fetchList } from '@/api/article'
import Sortable from 'sortablejs'
import axios from 'axios'
import * as echarts from 'echarts';

export default {
  name: 'DragTable',
  filters: {
    statusFilter(status) {
      const statusMap = {
        true: 'success',
        false: 'info'
        // false: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      showTable: true,
      showChart: false,
      count: 1,
      list: [],
      list1: [],
      total: '',
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10
      },
      sortable: null,
      oldList: [],
      newList: [],
      options: [],
      options1: [],
      options2: [],
      value_option1: '',
      value_option2: '',
      value: '',
      state: '',
      dialogFormVisible: false,
      dialogFormVisible1: false,
      form: {
        name: '',
        region: '',
        date1: '',
        date2: '',
        delivery: false,
        type: [],
        resource: '',
        desc: ''
      },
      formLabelWidth: '120px',
      drawer: false,
      direction: 'rtl',
      pagesize: 1,
      currentPage: 1,
      time: '',
      document_type: ''
    }
  },
  computed: {
    nowTable() {
      return (
        this.list.slice(
          (this.currentPage - 1) * this.pagesize,
          this.currentPage * this.pagesize
        ) || []
      )
    }
  },
  created() {
    // this.getList()
    this.getReference()
    this.getData()
    this.getData2()
  },
  methods: {
    async getList() {
      this.listLoading = true
      const { data } = await fetchList(this.listQuery)
      this.list = data.items
      console.log(this.list)// test
      // this.total = data.total
      this.listLoading = false
      this.oldList = this.list.map(v => v.id)
      this.newList = this.oldList.slice()
      this.$nextTick(() => {
        //this.setSort()
      })
    },
    getChart() {
      this.showTable = false
      this.showChart = true

      let chartDom = document.getElementById('main')
      let myChart = echarts.init(chartDom, 'dark')
      let option
      //myChart.showLoading();
      let data = []
      let category = []
      
      for (let i = 0; i < this.options2.length; i++) {
        data.push(this.options2[i].label)
        let obj_cate = {}
        obj_cate.keyword = {}
        obj_cate.name = this.options2[i].label
        category.push(obj_cate)
      }
      let nodes = []
      for(let i = 0; i < this.list.length; i++) {
        let obj_node = {}
        obj_node.name = this.list[i].start_object
        obj_node.value = 10
        let type = this.list[i].start_type + '-' + this.list[i].relation_type + '-' + this.list[i].end_type
        obj_node.category = this.getIndex(type, data)
        nodes.push(obj_node)
        let obj_node1 = {}
        obj_node1.name = this.list[i].end_object
        obj_node1.value = 10
        obj_node1.category = this.getIndex(type, data)
        nodes.push(obj_node1)
      }
      let links = []
      for(let i = 0; i + 1 < nodes.length; i = i + 2) {
        let obj_link = {}
        obj_link.source = i
        obj_link.target = i + 1
        links.push(obj_link)
      }
      option = {
          legend: {
              data: data
          },
          series: [{
              type: 'graph',
              layout: 'force',
              animation: false,
              label: {
                  position: 'right',
                  formatter: '{b}'
              },
              draggable: true,
              data: nodes.map(function (node, idx) {
                  node.id = idx;
                  return node;
              }),
              categories: category,
              force: {
                  edgeLength: 5,
                  repulsion: 20,
                  gravity: 0.2
              },
              edges: links
          }]
      };

      myChart.setOption(option);
    },
    getIndex(type, category) {
      for (let i = 0; i < category.length; i++) {
        if (type === category[i]) {
          return i
        }
      }
    }, 
    getTable() {
      this.showChart = false
      this.showTable = true
    },
    // 获取标注数据
    getReference() {
      const url = 'http://localhost:10088/FileMarks/getFileMark'
      this.listLoading = true
      axios.get(url)
        .then((response) => {
          const { data } = response
          let count = 0
          data.forEach((e) => {
            const document_id = e.document_id
            const document_type = e.document_type
            this.document_type = e.document_type
            e.relation_marks.forEach((f) => {
              const start_type = f.start_type
              const relation_type = f.relation_type
              const end_type = f.end_type
              f.relations.forEach((g) => {
                count = count + 1
                const item = { document_id: document_id, document_type: document_type, id: count,
                  start_type: start_type, relation_type: relation_type, end_type: end_type,
                  start_object: g.start_object, end_object: g.end_object, advice: g.advice,
                  evi_level: g.evi_level, evi_describe: g.evi_describe, group: g.group, is_checked: g.is_checked, is_passed: g.is_passed}

                this.list.push(item)
                this.list1.push(item)
              })
            })
          })
          this.listLoading = false
          this.oldList = this.list.map(v => v.id)
          this.newList = this.oldList.slice()
          this.$nextTick(() => {
            //this.setSort()
          })
        })
    },
    // 获取后台实体数据并转成当前耶页面需要的形式
    getData() {
      const url = 'http://localhost:10088/Entities/entity'
      axios.get(url).then((response) => {
        console.log(response.data)
        const datas = response.data
        for (var data in datas) {
          const a = {
            value: '',
            label: ''
          }
          a.value = datas[data].name
          a.label = datas[data].name
          this.options.push(a)
        }
        console.log(this.options)
      })
    },
    // 获取关系数据类型
    getData2() {
      const url = 'http://localhost:10088/Relations/relation'
      axios.get(url).then((response) => {
        console.log(response.data)
        const datas = response.data
        for (var data in datas) {
          const a = {
            value: '药品分类-适应症-疾病',
            label: '药品分类-适应症-疾病'
          }
          a.value = datas[data].object + '-' + datas[data].relation + '-' + datas[data].subject
          a.label = datas[data].object + '-' + datas[data].relation + '-' + datas[data].subject
          this.options1.push(a)
          this.options2.push(a)
        }
      })
    },
    getRowKey(row) {
      return row.id.counter
    },
    js_method() {
      this.dialogFormVisible = true
    },
    js_method1() {
      this.drawer = true
    },
    // 搜索相关代码
    querySearchAsync(queryString, cb) {
      var search_data = this.list
      console.log(queryString)
      this.results = queryString
        ? search_data.filter(this.createStateFilter(queryString))
        : search_data
      console.log(this.results)

      const list = []
      for (const result of this.results) {
        list.push({ value: result.title })
      }

      clearTimeout(this.timeout)
      this.timeout = setTimeout(() => {
        if (list.length !== this.list.length) {
          cb(list)
        } else {
          cb([])
        }
      }, 3000 * Math.random())
    },
    // 搜索相关代码
    createStateFilter(queryString) {
      return (state) => {
        console.log(
          state.name.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        )

        return (
          state.name.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        )
      }
    },
    // 搜索相关代码
    handleSelect(item) {
      console.log(item)
      // this.list = this.results;
      // this.nowTable;
    },
    // 搜索相关代码
    handleChange(e) {
      console.log(e)
      this.list = this.list1
      this.list = this.list.filter((data) => {
        return data.start_type + '-' + data.relation_type + '-' + data.end_type === e
      })
    },
    handleClose(done) {
      this.$confirm('确认关闭？')
        .then(_ => {
          done()
        })
        .catch(_ => {})
    },
    handleSizeChange: function(size) {
      this.pagesize = size
    },
    handleCurrentChange: function(currentPage) {
      this.currentPage = currentPage
    },
    handleClick() {
      this.$router.push({
        path: '/xieweihao/entityAudit',
        query: {
        }
      })
    },
    // 让form表单显现出来
    addOneEntity() {
      this.dialogFormVisible1 = true
    },
    // 获取当前时间
    getTime() {
      /*eslint no-extend-native: ["error", { "exceptions": ["Date"] }]*/
      Date.prototype.Format = function(fmt) {
        var o = {
          'M+': this.getMonth() + 1, // 月份
          'd+': this.getDate(), // 日
          'h+': this.getHours(), // 小时
          'm+': this.getMinutes(), // 分
          's+': this.getSeconds(), // 秒
          'q+': Math.floor((this.getMonth() + 3) / 3), // 季度
          'S': this.getMilliseconds() // 毫秒
        }
        if (/(y+)/.test(fmt)) {
          fmt = fmt.replace(RegExp.$1, (this.getFullYear() + '').substr(4 - RegExp.$1.length))
        }
        for (var k in o) {
          if (new RegExp('(' + k + ')').test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? (o[k]) : (('00' + o[k]).substr(('' + o[k]).length)))
        }
        return fmt
      }
      this.time = new Date().Format('yyyy-MM-dd hh:mm:ss')
      console.log(this.time)
    },
    changeStatus(obj, judge) {
      let relation = {}
      relation.document_id = obj.document_id
      relation.start_type = obj.start_type
      relation.relation_type = obj.relation_type
      relation.end_type = obj.end_type
      relation.start_object = obj.start_object
      relation.end_object = obj.end_object
      if (judge === 1) {
        relation.status = true
      }else {
        relation.status = false
      }
      const url = 'http://localhost:10088/FileMarks/updateRelationStatus'
      axios.put(url, relation).then((response) => {
        console.log(response)
        this.list = []
        this.getReference()
        this.nowTable
        //this.getReference()
        //window.reload()
        //this.$router.go(0)
      })
    },
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll('.el-table__body-wrapper > table > tbody')[0]
      this.sortable = Sortable.create(el, {
        ghostClass: 'sortable-ghost', // Class name for the drop placeholder,
        setData: function(dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData('Text', '')
        },
        onEnd: evt => {
          const targetRow = this.list.splice(evt.oldIndex, 1)[0]
          this.list.splice(evt.newIndex, 0, targetRow)

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0]
          this.newList.splice(evt.newIndex, 0, tempIndex)
        }
      })
    }
  }
}
</script>

<style>
.sortable-ghost{
  opacity: .8;
  color: #fff!important;
  background: #42b983!important;
}
</style>

<style scoped>
.icon-star{
  margin-right:2px;
}
.drag-handler{
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.show-d{
  margin-top: 15px;
}
.el-header, .el-footer {
    background-color: #B3C0D1;
    color: #333;
    text-align: center;
    line-height: 60px;
    font-weight: bold;
    font-size: 30px;
  }
.mydiv1 {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
.mydiv2 {
  display: flex;
  flex-direction: row;
}
.inRight {
  margin-left: auto;
}
#button {
  text-align: center;
  margin: auto;
}
#button_bg {
  background-color: #B3C0D1;
}

.div1 {
  width: 150px;
  line-height: 40px;
  height: 40px;
}

.margin {
  margin-top: 10px;
}

.divMain {
  width: 1850px;
  line-height: 40px;
  height: 800px;
  position: relative;
}
</style>
