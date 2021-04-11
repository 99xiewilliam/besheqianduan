<template>
  <div class="app-container">
    <el-container>
      <el-header>
        手动添加图谱数据库
      </el-header>
      <el-container id="button_bg">
        <el-main id="button">
          <el-button>实体数据管理</el-button>
          <el-button @click="handleClick()">关系数据管理</el-button>
        </el-main>
      </el-container>
      <el-container class="mydiv1">
        <el-container>
          <div class="div1">当前实体类型:</div>
          <el-select v-model="value" placeholder="请选择" @change="handleChange2($event)">
            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-container>
        <el-container class="mydiv2">
          <el-button type="info" class="inRight">表格审核</el-button>
          <el-button type="success" class="inRight">图像显示</el-button>
        </el-container>
      </el-container>
      
      <div class="margin">
        <el-autocomplete
          v-model="state"
          :fetch-suggestions="querySearchAsync"
          placeholder="请输入内容"
          @select="handleSelect1"
        />
      </div>
      <el-table v-loading="listLoading" :data="nowTable" :row-key="getRowKey" border fit highlight-current-row style="width: 100%">
        <el-table-column align="center" label="实例ID" width="200px">
          <template slot-scope="{row}">
            <span>{{ row.document_id }}</span>
          </template>
        </el-table-column>

        <el-table-column width="300px" label="名称">
          <template slot-scope="{row}">
            <span>{{ row.name }}</span>
          </template>
        </el-table-column>

        <el-table-column width="500px" align="center" label="描述">
          <template slot-scope="{row}">
            <span>{{ row.description }}</span>
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

        <el-table-column align="center" label="关系操作" min-width="180">
          <template slot-scope="{row}">
            <el-button @click="js_method1(row)">合格</el-button>
            <el-button @click="js_method()">不合格</el-button>
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
      value: '',
      value1: '',
      state: '',
      dialogFormVisible: false,
      dialogFormVisible1: false,
      form: {
        name: '',
        type: '',
        iCD_11: '',
        description: '',
        object_type: '',
        document_id: ''
      },
      formLabelWidth: '120px',
      drawer: false,
      direction: 'rtl',
      pagesize: 1,
      currentPage: 1,
      relation_value: '',
      relation_options: [],
      innerDrawer: false,
      search_disease: [],
      tableData: [],
      tableData1: [],
      tableData2: [],
      currentRow: null,
      results: [],
      keep: {
        name: '',
        objID: ''
      },
      switchValue: true,
      switchVisible: false,
      level: '',
      describe: '',
      isgroup: [],
      advice: '',
      group: true,
      fileMark: {
        document_id: '',
        document_type: '',
        type: '',
        object_marks: [
        ],
        relation_marks: [
        ]
      },
      Entitylabels: [],
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
    //this.getList()
    this.getReference()
    this.getData()
    this.getRelationData()
    this.getEntityData()
  },
  methods: {
    async getList() {
      this.listLoading = true
      const { data } = await fetchList(this.listQuery)
      this.list = data.items
      console.log('9090909090')
      console.log(this.list)// test
      // this.total = data.total
      this.listLoading = false
      this.oldList = this.list.map(v => v.id)
      this.newList = this.oldList.slice()
      this.$nextTick(() => {
        //this.setSort()
      })
    },
    // 获取实体标注数据
    getReference() {
      const url = 'http://localhost:10088/FileMarks/getFileMark'
      this.listLoading = true
      axios.get(url)
        .then((response) => {
          // console.log(response)
          const { data } = response
          console.log(data)
          data.forEach((e) => {
            const document_id = e.document_id
            const document_type = e.document_type
            this.document_type = e.document_type
            e.object_marks.forEach((f) => {
              const object_type = f.object_type
              f.objects.forEach((g) => {
                const item = { document_id: document_id, document_type: document_type,
                  object_type: object_type, description: '', iCD_11: '', type: '', name: '', time: '' , is_checked: '', is_passed: ''}
                const obj = {}
                item.name = g.name
                item.iCD_11 = g.iCD_11
                item.description = g.description
                item.time = g.mark_time
                item.is_checked = g.is_checked
                item.is_passed = g.is_passed
                // obj.name = g.name
                // obj.objID = document_id
                // obj.object_type = object_type
                this.list.push(item)
                this.list1.push(item)
                console.log(this.list)
                // this.tableData.push(obj)
                // this.tableData1.push(obj)
                // this.tableData2.push(obj)
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
    // 获取后台实体类型数据
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
        // this.labels = data;
        console.log(this.options)
      })
    },
    getRowKey(row) {
      return row.name
    },
    js_method(row) {
      console.log(row)
      this.dialogFormVisible = true
      this.form.object_type = row.object_type
      this.form.name = row.name
      this.form.iCD_11 = row.iCD_11
      this.form.description = row.description
      this.form.document_id = row.document_id
    },
    js_method1(row) {
      this.drawer = true
      this.form.object_type = row.object_type
      this.form.name = row.name
      this.form.iCD_11 = row.iCD_11
      this.form.description = row.description
      this.form.document_id = row.document_id
    },
    // 搜索相关代码
    querySearchAsync(queryString, cb) {
      var search_data = this.list
      console.log(queryString)
      if (queryString === '') {
        this.list = this.list1
      }
      this.results = queryString
        ? search_data.filter(this.createStateFilter(queryString))
        : search_data
      console.log(this.results)

      const list = []
      for (const result of this.results) {
        list.push({ value: result.name })
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
    querySearchAsync1(queryString, cb) {
      var search_data = this.tableData
      console.log(queryString)
      if (queryString === '') {
        this.tableData = this.tableData1
      }
      this.results = queryString
        ? search_data.filter(this.createStateFilter(queryString))
        : search_data
      console.log(this.results)

      const list = []
      for (const result of this.results) {
        list.push({ value: result.name })
      }

      clearTimeout(this.timeout)
      this.timeout = setTimeout(() => {
        if (list.length !== this.tableData.length) {
          cb(list)
        } else {
          cb([])
        }
      }, 3000 * Math.random())
    },
    // 搜索相关代码
    handleSelect1(item) {
      console.log(item)
      this.list = this.results
      // this.nowTable;
    },
    // 搜索相关代码
    handleSelect(item) {
      console.log(this.results)
      this.tableData = this.results
      // this.nowTable;
    },
    handleClose(done) {
      this.$confirm('确认关闭？')
        .then(_ => {
          done()
          this.keep.name = ''
          this.keep.objID = ''
          this.value1 = ''
          this.relation_value = ''
          this.tableData = this.tableData2
          this.switchVisible = false
        })
        .catch(_ => {})
    },
    handleClose1(done) {
      this.$confirm('确认关闭？')
        .then(_ => {
          this.setCurrent()
          this.state = ''
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
        path: '/xieweihao/entityAudit/relationAudit',
        query: {
        }
      })
    },
    // 让Form显示
    addOneEntity() {
      this.dialogFormVisible1 = true
    },
    handleChange(e) {
      this.choice = e
      // const obj = {
      //   addedRelation: e
      // }
      // this.tableData.push(obj)
    },
    handleChange1(e) {
      console.log(e)
      this.tableData = this.tableData2
      this.tableData = this.tableData.filter((data) => {
        return data.object_type === e
      })
    },
    handleChange2(e) {
      console.log(e)
      this.list = this.list1
      this.list = this.list.filter((data) => {
        return data.object_type === e
      })
    },
    handleCurrentChange1(val) {
      console.log(val)
      if (val === null) {
        val = {}
      }
      this.currentRow = val
      if (Object.prototype.hasOwnProperty.call(val, 'name')) {
        this.keep.name = val.name
        console.log(this.keep.name)
      }
      if (Object.prototype.hasOwnProperty.call(val, 'objID')) {
        this.keep.objID = val.objID
      }
    },
    // 获取后台关系类型数据
    getRelationData() {
      const url = 'http://localhost:10088/Relations/relation'
      axios.get(url).then((response) => {
        const datas = response.data
        for (var data in datas) {
          const a = {
            value: '药品分类-适应症-疾病',
            label: '药品分类-适应症-疾病'
          }
          a.value = datas[data].object + '-' + datas[data].relation + '-' + datas[data].subject
          a.label = datas[data].object + '-' + datas[data].relation + '-' + datas[data].subject
          this.relation_options.push(a)
        }
      })
    },
    // 获取后台实体类型数据
    getEntityData() {
      const url = 'http://localhost:10088/Entities/entity'
      axios.get(url).then((response) => {
        const datas = response.data
        for (var data in datas) {
          const a = {
            color: 1,
            labelname: '藥品名',
            entitylist: []
          }
          a.color = parseInt(data)
          a.color = a.color + 1
          a.labelname = datas[data].name
          this.Entitylabels.push(a)
        }
      })
    },
    setCurrent(row) {
      this.$refs.singleTable.setCurrentRow(row)
      this.keep.name = ''
      this.keep.objID = ''
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
    // 让内部抽屉显示
    getInnerDrawer() {
      this.innerDrawer = true
      if (this.value1 !== '') {
        this.tableData = this.tableData.filter((data) => {
          return data.object_type === this.value1
        })
        console.log(this.tableData)
        this.tableData1 = this.tableData1.filter((data) => {
          return data.object_type === this.value1
        })
      }
    },
    confirmData() {
      this.innerDrawer = false
      this.switchVisible = true
    },
    // 提交实体数据
    submitEntityData() {
      this.dialogFormVisible = false
      for (let i = 0; i < this.relation_options.length; i++) {
        const arr = this.relation_options[i].label.split('-')
        const obj_rel = {
          relaiton_id: '',
          start_type: arr[0],
          relation_type: arr[1],
          end_type: arr[2],
          relations: [

          ],
          is_checked: false,
          is_passed: false,
          mark_user: '',
          mark_time: '',
          check_user: '',
          check_time: ''
        }
        this.fileMark.relation_marks.push(obj_rel)
      }
      for (let i = 0; i < this.Entitylabels.length; i++) {
        this.getTime()
        const obj_entity = {
          object_type: this.Entitylabels[i].labelname,
          objects: []
        }
        if (this.form.object_type === this.Entitylabels[i].labelname) {
          var o = {
            name: this.form.name,
            description: this.form.description,
            ICD_11: this.form.ICD_11,
            type: 'expertInput',
            is_checked: false,
            is_passed: false,
            mark_user_id: '',
            mark_time: this.time,
            check_user_id: '',
            check_time: this.time,
            multiple_marked: false
          }
          obj_entity.objects.push(o)
          this.fileMark.document_id = this.form.document_id
          this.fileMark.document_type = this.document_type
        }
        this.fileMark.object_marks.push(obj_entity)
      }
      const url = 'http://localhost:10088/ExpertInput/addExpertInput'
      axios.post(url, this.fileMark).then((response) => {
        if (response.data.msg === '添加成功') {
          this.fileMark = {
            document_id: '',
            document_type: '',
            type: '',
            object_marks: [

            ],
            relation_marks: [

            ]
          }
          this.$message({
            message: '恭喜你，添加成功',
            type: 'success'
          })
        } else {
          this.$message.error('添加失败')
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    // 提交关系数据
    submitRelationData() {
      for (let i = 0; i < this.Entitylabels.length; i++) {
        const obj_entity = {
          object_type: this.Entitylabels[i].labelname,
          objects: []
        }
        this.fileMark.object_marks.push(obj_entity)
      }
      for (let i = 0; i < this.relation_options.length; i++) {
        this.getTime()

        const arr = this.relation_options[i].label.split('-')
        const obj_rel = {
          relaiton_id: '',
          start_type: arr[0],
          relation_type: arr[1],
          end_type: arr[2],
          relations: [

          ],
          is_checked: false,
          is_passed: false,
          mark_user: '',
          mark_time: this.time,
          check_user: '',
          check_time: ''
        }
        if (this.relation_value === this.relation_options[i].label && this.keep.objID !== '') {
          const add_obj = {
            start_object: this.form.name,
            end_object: this.keep.name,
            advice: '',
            evi_level: '',
            evi_describe: '',
            reference: '',
            group: this.switchValue,
            time: this.time,
            type: 'expertInput',
            is_checked: false,
            is_passed: false,
            mark_user_id: '',
            mark_time: this.time,
            check_user_id: '',
            check_time: '',
            is_multiple_marked: false
          }
          obj_rel.relations.push(add_obj)

          this.fileMark.document_id = this.form.document_id + '&&' + this.keep.objID
          this.fileMark.document_type = this.document_type
        }

        this.fileMark.relation_marks.push(obj_rel)
      }
      const url = 'http://localhost:10088/ExpertInput/addExpertInput'
      axios.post(url, this.fileMark).then((response) => {
        if (response.data.msg === '添加成功') {
          this.fileMark = {
            document_id: '',
            document_type: '',
            type: '',
            object_marks: [

            ],
            relation_marks: [

            ]
          }
          this.$message({
            message: '恭喜你，添加成功',
            type: 'success'
          })
        } else {
          this.$message.error('添加失败')
        }
      }).catch((error) => {
        console.log(error)
      })
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

</style>
