<template>
  <div class="app-container">
    <el-page-header @back="goBack" />
    <el-autocomplete
      v-model="state"
      :fetch-suggestions="querySearchAsync"
      placeholder="请输入内容"
      @select="handleSelect"
    />
    <!-- <el-button
      style="margin-bottom: 20px; margin-left: 20px"
      type="primary"
    >
      搜索
    </el-button> -->
    <!-- <div class="pdf">
      <p class="arrow">
        <span @click="changePdfPage(0)" class="turn" :class="{grey: currentPage==1}">Preview</span>
        {{pdfCurrentPage}} / {{pageCount}}
        <span @click="changePdfPage(1)" class="turn" :class="{grey: currentPage==pageCount}">Next</span>
      </p>
      <pdf ref="pdf"
      :src="pdf_url"
      :page="pdfCurrentPage"
      @num-pages="pageCount=$event"
      @page-loaded="pdfCurrentPage=$event"
      @loaded="loadPdfHandler"/>
    </div> -->

    <el-table
      ref="dragTable"
      v-loading="listLoading"
      :data="nowTable"
      :row-key="getRowKey"
      border
      fit
      highlight-current-row
      style="width: 100%"
      :row-class-name="rowIndex"
    >
      <el-table-column align="center" label="序号" width="65">
        <template slot-scope="{ row }">
          <span>{{ computeTableIndex(row) }}</span>
        </template>
      </el-table-column>

      <el-table-column width="110px" align="center" label="id">
        <template slot-scope="{ row }">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>

      <el-table-column min-width="300px" label="题目">
        <template slot-scope="{ row }">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>

      <!-- <el-table-column class-name="status-col" label="预览情况" width="110">
        <template slot-scope="{ row }">
          <el-tag :type="row.is_marked | statusFilter">
            <span v-if="row.is_marked == true">已预览</span>
            <span v-else>未预览</span>
          </el-tag>
        </template>
      </el-table-column> -->

      <el-table-column align="center" label="操作" width="120">
        <template slot-scope="{ row }">
          <span>
            <el-button
              type="success"
              style="width: 100px"
              @click="nextPage(row)"
            >预览</el-button>
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
      :page-sizes="[5, 10, 15, 20]"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<script>
import { fetchList } from '@/api/article'
import Sortable from 'sortablejs'
import axios from 'axios'
// import pdf from 'vue-pdf'

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
  components: {
    // pdf
  },
  data() {
    return {
      // pdf_url: '',
      // pdf_obj: {
      //   'id': ''
      // },
      // pdfCurrentPage: 0,
      // pageCount: 0,
      document_type: this.$route.query.document_type,
      count: 1,
      list: [],
      list1: [],
      total: null,
      listLoading: true,
      listQuery: {
        page: 10,
        limit: 20
      },
      sortable: null,
      oldList: [],
      newList: [],
      pagesize: 5,
      currentPage: 1,
      state: '',
      timeout: null,
      results: []
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

  watch: {
    $route(to, from) {
      this.getReference()
    }
  },
  created() {
    // this.getList()
    // this.getReference()
    // this.getPdfData()
    this.getReference()
  },
  methods: {
    async getList() {
      this.listLoading = true
      const { data } = await fetchList(this.listQuery)
      this.list = data.items
      this.list1 = this.list
      this.listLoading = false
      this.oldList = this.list.map((v) => v.id)
      this.newList = this.oldList.slice()
      this.$nextTick(() => {
        this.setSort()
      })
    },
    getRowKey(row) {
      return row.num
    },
    rowIndex({ row, rowIndex }) {
      row.rowIndex = rowIndex
    },
    computeTableIndex(row) {
      return (this.currentPage - 1) * this.pagesize + row.rowIndex + 1
    },
    goBack() {
      this.$router.back(-1)
    },
    // getPdfData() {
    //   const url = 'http://localhost:10088/Pdf/getPdf'
    //   // axios.get(url).then((response) => {
    //   //   console.log(response)
    //   //   console.log(typeof response.data[0].base64)
    //   //   let blob = new Blob([response.data[0].base64], {
    //   //       type: 'text/plain'
    //   //   });
    //   //   this.pdf_url = URL.createObjectURL(blob);
    //   // console.log(imgUrl)
    //   // })
    //   // axios({
    //   //   method: 'GET',
    //   //   url: 'http://localhost:10088/Pdf/getPdf',
    //   //   params: {
    //   //     id: '606ae579cdb2ca093653a88d'
    //   //   },
    //   // headers: {
    //   //   'Content-Type': 'application/vnd.openxmlformats- officedocument.spreadsheetml.sheet'
    //   // },
    //   //   responseType: 'blob'
    //   // }).then(response => {
    //   //   this.pdf_url = this.getObjectURL(response.data)
    //   //   this.pdfCurrentPage = 1
    //   // })
    //   // axios.get(url, {
    //   //   params: {
    //   //     id: '606ae579cdb2ca093653a88d'
    //   //   },
    //   //   paramsSerializer: function(params) {
    //   //     return qs.stringify(params, { arrayFormat: 'brackets' })
    //   //   },
    //   //   headers: {
    //   //     'Content-Type': 'application/vnd.openxmlformats- officedocument.spreadsheetml.sheet'
    //   //   },
    //   //   responseType: 'blob'
    //   // }).then(response => {
    //   //   this.pdf_url = this.getObjectURL(response.data)
    //   //   this.pdfCurrentPage = 1
    //   // })
    //   this.pdf_obj.id = '606ae579cdb2ca093653a88d'
    //   axios({
    //     method: 'post',
    //     url: url,
    //     data: this.pdf_obj,
    //     responseType: 'blob'
    //   }).then(response => {
    //     console.log(response)
    //     this.pdf_url = this.getObjectURL(response.data)
    //   })
    // },
    // // 处理文件流
    // getObjectURL(file) {
    //   let url = null
    //   if (window.createObjectURL !== undefined) { // basic
    //     url = window.createObjectURL(file)
    //   } else if (window.webkitURL !== undefined) { // webkit or chrome
    //     try {
    //       url = window.webkitURL.createObjectURL(file)
    //     } catch (error) {
    //       console.log(error)
    //     }
    //   } else if (window.URL !== undefined) { // mozilla(firefox)
    //     try {
    //       url = window.URL.createObjectURL(file)
    //     } catch (error) {
    //       console.log(error)
    //     }
    //   }
    //   return url
    // },
    // changePdfPage (val) {
    //   // console.log(val)
    //   if (val === 0 && this.pdfCurrentPage > 1) {
    //     this.pdfCurrentPage--
    //     // console.log(this.currentPage)
    //   }
    //   if (val === 1 && this.pdfCurrentPage < this.pageCount) {
    //     this.pdfCurrentPage++
    //     // console.log(this.currentPage)
    //   }
    // },
    // // pdf加载时
    // loadPdfHandler (e) {
    //   this.pdfCurrentPage = 1 // 加载的时候先加载第一页
    //   console.log(this.pageCount)
    // },
    // 获取后台文章的文章列表
    getReference() {
      const url = 'http://localhost:10088/Pdf/getPdfInfo'
      this.listLoading = true
      axios.get(url).then((response) => {
        const { data } = response
        this.list = data
        this.list1 = this.list
        this.listLoading = false
        this.oldList = this.list.map((v) => v.id)
        this.newList = this.oldList.slice()
        this.$nextTick(() => {
          this.setSort()
        })
      })
    },
    // 进入下一层页面
    nextPage(obj) {
      console.log(obj)
      let id = obj.id
      console.log(id.slice(19, id.length - 1))
      id = id.slice(19, id.length - 1)
      // const url = 'http://localhost:10088/reference/get'
      // const query = {
      //   category: 'search',
      //   content: obj.title
      // }
      // axios.post(url, query).then((response) => {
      //   console.log(response)
      // })

      this.$router.push({
        path: '/xieweihao/Document_information_extraction/Paper/electronicDocument/pdfEntity',
        query: {
          id: id,
          document_type: this.document_type
        }
      })
    },
    // 和搜索相关的代码
    handleSizeChange: function(size) {
      this.pagesize = size
    },
    // 和搜索相关的代码
    handleCurrentChange: function(currentPage) {
      this.currentPage = currentPage
    },
    // 和搜索相关的代码
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
    // 和搜索相关的代码
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
    // 和搜索相关的代码
    handleSelect(item) {
      console.log(item)
      this.list = this.results
      this.nowTable
    },
    // 跳转去pictureToWord页面
    goToPicture() {
      this.$router.push({
        path: '/xieweihao/Document_information_extraction/Paper/pictureToWord',
        query: {

        }
      })
    },
    goToElectronicDocument() {
      this.$router.push({
        path: '/xieweihao/Document_information_extraction/Paper/electronicDocument'
      })
    },
    loadAll() {
      return []
    },
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll(
        '.el-table__body-wrapper > table > tbody'
      )[0]
      this.sortable = Sortable.create(el, {
        ghostClass: 'sortable-ghost', // Class name for the drop placeholder,
        setData: function(dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData('Text', '')
        },
        onEnd: (evt) => {
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
.sortable-ghost {
  opacity: 0.8;
  color: #fff !important;
  background: #42b983 !important;
}
</style>

<style scoped>
.icon-star {
  margin-right: 2px;
}
.drag-handler {
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.show-d {
  margin-top: 15px;
}
</style>
