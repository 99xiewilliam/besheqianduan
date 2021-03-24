<template>
  <div class="app-container">
    <el-button style="margin-bottom:20px;" type="primary" icon="el-icon-document" @click="dialogFormVisible = true">
      添加标注任务
    </el-button>
    <el-dialog title="添加标注任务" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="项目名称">
          <el-input v-model="form.name" autocomplete="off" placeholder="请输入项目名称" />
          <el-input v-model="form.col_name" autocomplete="off" placeholder="请输入项目数据库名称" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="addItem()">确 定</el-button>
        <el-button @click="dialogFormVisible = false">取 消</el-button>
      </div>
    </el-dialog>
    <!-- 表格 -->
    <el-table ref="dragTable" v-loading="listLoading" :data="nowTableData" :row-key="getRowKey" fit style="width: 100%">
      <el-table-column label="项目名称">
        <template slot-scope="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>

      <el-table-column width="65" label="项目ID">
        <template slot-scope="{row}">
          <span>{{ row.num }}</span>
        </template>
      </el-table-column>

      <el-table-column class-name="status-col" label="状态" width="110">
        <template slot-scope="{row}">
          <el-tag :type="row.state | statusFilter">
            <span v-if="row.state == true">已标注</span>
            <span v-else>未标注</span>
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column width="110px" label="文档总数">
        <template slot-scope="{row}">
          <span>{{ row.sum_num }}</span>
        </template>
      </el-table-column>

      <el-table-column label="已标注文档总数" width="95">
        <template slot-scope="{row}">
          <span>{{ row.mark_num }}</span>
        </template>
      </el-table-column>

      <el-table-column label="最新更新时间" width="200px">
        <template slot-scope="{row}">
          <span>{{ row.time }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="操作" width="120">
        <template slot-scope="{row}">
          <span>
            <el-button type="success" style="width:100px;" @click="switchToPage(row.col_name)">操作</el-button>
          </span>
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
  </div>
</template>

<script>
import { fetchList } from '@/api/article'
import Sortable from 'sortablejs'
import axios from 'axios'

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
  // components: {
  //   AddItem
  // },
  data() {
    return {
      save: null,
      count: 1,
      list: [],
      list1: [],
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10
      },
      sortable: null,
      oldList: [],
      newList: [],
      dialogFormVisible: false,
      form: {
        name: '',
        col_name: ''
      },
      saveData: {
        name: '',
        num: 0,
        sum_num: 0,
        mark_num: 0,
        time: '',
        state: false,
        col_name: ''
      },
      pagesize: 1,
      currentPage: 1
    }
  },
  computed: {
    nowTableData() {
      return this.list.slice((this.currentPage - 1) * this.pagesize, this.currentPage * this.pagesize) || []
    }
  },
  watch: {
    '$route'(to, from) {
      this.getItems()
    }
  },
  created() {
    // this.getList()
    // this.getReference()
    this.getItems()
  },
  methods: {
    async getList() {
      this.listLoading = true
      const { data } = await fetchList(this.listQuery)
      console.log('1231312312312312')
      console.log(data)
      this.list = data.items
      this.listLoading = false
      this.oldList = this.list.map(v => v.id)
      this.newList = this.oldList.slice()
      this.$nextTick(() => {
        this.setSort()
      })
    },
    getRowKey(row) {
      return row.num
    },
    handleSizeChange: function(size) {
      this.pagesize = size
    },
    handleCurrentChange: function(currentPage) {
      this.currentPage = currentPage
    },
    getReference() {
      const url = 'http://localhost:10088/reference/Paper'
      this.listLoading = true
      axios.get(url)
        .then((response) => {
          const { data } = response
          console.log(data)
        })
    },
    // 获取后台数据
    getItems() {
      const url = 'http://localhost:10088/Item/getAll'
      this.listLoading = true
      axios.get(url)
        .then((response) => {
          const { data } = response
          console.log(data)
          this.list = data
          this.listLoading = false
          // this.oldList = this.list.map(v => v.id)
          // this.newList = this.oldList.slice()
          // this.$nextTick(() => {
          //   this.setSort()
          // })
        })
    },
    // add() {
    //   this.$layer.iframe({
    //     content: {
    //       content: addItem,
    //       parent: this,
    //       data: {
    //         info: { a: 1 }
    //       }
    //     },
    //     area: 'auto',
    //     title: '添加项目',
    //     shadeClose: false
    //   })
    // },
    // 添加文本类型
    addItem() {
      this.saveData.name = this.form.name
      this.saveData.col_name = this.form.col_name
      this.dialogFormVisible = false
      this.getTime()
      console.log(this.saveData.time)
      console.log(this.list[this.list.length - 1])
      this.saveData.num = this.list[this.list.length - 1].num + 1
      const url = 'http://localhost:10088/Item/addItem'
      axios.post(url, this.saveData).then((response) => {
        console.log(response)
        this.list.push(this.saveData)
      }).catch((error) => {
        console.log(error)
      })
    },
    // 转到下一层界面
    switchToPage(coll_name) {
      console.log(coll_name)
      this.$router.push({
        path: '/xieweihao/Document_information_extraction/' + coll_name,
        query: {
          name: coll_name
        }
      })
    },
    // 获取时间
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
      this.saveData.time = new Date().Format('yyyy-MM-dd hh:mm:ss')
      console.log(this.saveData.time)
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
</style>
