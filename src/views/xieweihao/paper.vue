<template>
  <div class="app-container">
    <!-- Note that row-key is necessary to get a correct row order. -->
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
    <el-button
      style="margin-bottom: 20px; margin-left: 20px;"
      type="primary"
      @click="goToPicture()"
    >
      图片读取文本
    </el-button>
    <el-table
      ref="dragTable"
      v-loading="listLoading"
      :data="nowTable"
      :row-key="getRowKey"
      border
      fit
      highlight-current-row
      style="width: 100%"
    >
      <el-table-column align="center" label="序号" width="65">
        <template slot-scope="{ row }">
          <span>{{ row.id.counter }}</span>
        </template>
      </el-table-column>

      <el-table-column width="180px" align="center" label="出版时间">
        <template slot-scope="{ row }">
          <!-- <span>{{ row.timestamp | parseTime('{y}-{m}-{d} {h}:{i}') }}</span> -->
          <span>{{ row.date }}</span>
        </template>
      </el-table-column>

      <el-table-column min-width="300px" label="题目">
        <template slot-scope="{ row }">
          <span>{{ row.title }}</span>
        </template>
      </el-table-column>

      <el-table-column width="110px" align="center" label="作者">
        <template slot-scope="{ row }">
          <span>{{ row.author }}</span>
        </template>
      </el-table-column>

      <el-table-column class-name="status-col" label="标注情况" width="110">
        <template slot-scope="{ row }">
          <el-tag :type="row.is_marked | statusFilter">
            <span v-if="row.is_marked == true">已标注</span>
            <span v-else>未标注</span>
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column align="center" label="操作" width="120">
        <template slot-scope="{ row }">
          <span>
            <el-button
              type="success"
              style="width: 100px"
              @click="nextPage(row)"
            >开始操作</el-button>
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
  data() {
    return {
      count: 1,
      name: this.$route.query.name,
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
      pagesize: 1,
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
    this.getReference()
  },
  methods: {
    async getList() {
      this.listLoading = true
      const { data } = await fetchList(this.listQuery)
      console.log('909090909090')
      console.log(this.list)
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
    // 获取后台文章的文章列表
    getReference() {
      const url = 'http://localhost:10088/reference/' + this.name
      this.listLoading = true
      axios.get(url).then((response) => {
        console.log(response)
        const { data } = response
        this.list = data
        this.list1 = this.list

        console.log(888888) // test
        console.log(this.list) // test
        console.log(this.list[0].id)
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
      console.log(this.name)
      // const url = 'http://localhost:10088/reference/get'
      // const query = {
      //   category: 'search',
      //   content: obj.title
      // }
      // axios.post(url, query).then((response) => {
      //   console.log(response)
      // })
      this.$router.push({
        path: '/xieweihao/Document_information_extraction/Paper/ShowArticle',
        query: {
          id: obj.id,
          title: obj.title,
          author: obj.author,
          date: obj.date,
          journal: obj.journal,
          src: obj.src,
          summary: obj.summary,
          keywords: obj.keywords,
          document_type: this.name
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
    // 和搜索相关的代码
    createStateFilter(queryString) {
      return (state) => {
        console.log(
          state.title.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        )

        return (
          state.title.toLowerCase().indexOf(queryString.toLowerCase()) === 0
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
