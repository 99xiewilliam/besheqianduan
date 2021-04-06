/* eslint-disable */
<template>
  <div class="components-container">
    <div class="app-container">
      <i
        class="el-icon-back"
        style="weight: 1000; cursor: pointer"
        @click="routeBack()"
      />
      <!-- <span style="font-weight:bold; font-size:30px">{{ "标题：" }}</span> -->
    </div>
    <split-pane split="vertical" @resize="resize">
      <template slot="paneL">
        <el-card class="box-card">
          <el-scrollbar style="height:100%" wrap-style="overflow-x:hidden;">
            <div class="content-container">
              <!-- <el-scrollbar style="height:100%" wrap-style="overflow-x:hidden;"> -->
              <button
                @click="logContent"
              >
                log content
              </button>
              <div class="pdf">
                <p class="arrow">
                  <span class="turn" :class="{grey: pdfCurrentPage==1}" @click="changePdfPage(0)">Preview</span>
                  {{ pdfCurrentPage }} / {{ pageCount }}
                  <span class="turn" :class="{grey: pdfCurrentPage==pageCount}" @click="changePdfPage(1)">Next</span>
                </p>
                <pdf
                  ref="pdf"
                  :src="pdf_url"
                  :page="pdfCurrentPage"
                  @num-pages="pageCount=$event"
                  @page-loaded="pdfCurrentPage=$event"
                  @loaded="loadPdfHandler"
                />
              </div>
            <!-- </el-scrollbar> -->
            </div>
          </el-scrollbar>
        </el-card>
      </template>
      <template slot="paneR">
        <el-card class="box-card">
          <div class="action-area">
            <el-button class="right" @click="handleSkip">不合格</el-button>
            <el-button
              type="primary"
              class="right"
              @click="addOcrData()"
            >提交</el-button>
          </div>
          <el-input
            v-model="textarea1"
            type="textarea"
            autosize
            placeholder="请输入标题"
          />
          <el-input
            v-model="targetText"
            type="textarea"
            :autosize="{ minRows: 10, maxRows: 60}"
            placeholder="请输入内容"
            style="margin-top: 20px;"
          />
        </el-card>
      </template>
    </split-pane>
  </div>
</template>

<script>
/* eslint-disable */
import splitPane from 'vue-splitpane'
import axios from 'axios'
import pdf from 'vue-pdf'
export default {
  name: 'SplitpaneDemo',
  components: { splitPane, pdf },
  data() {
    return {
      pdf_url: '',
      pdf_obj: {
        'id': this.$route.query.id
      },
      pdfCurrentPage: 0,
      pageCount: 0,
      textarea1: '',
      fileList: [],
      saveData: {},
      id: this.$route.query.id,
      activeName: 'first',
      labels: [

      ],
      labels1: [

      ],
      entitylist: [],
      // 文档部分
      targetText: '',
      targetTextArr: [],
      // 页面配置和状态部分
      curindex: 0, // 当前文档
      curMarkLabel: -1, // 当前使用的标签,值为-1代表当前未在标注实体
      continuousMark: false, // 连续标注模式
      noShowList: [],
      shows: [],
      state: '',
      results: [],
      time: ''
    }
  },
  // watch: {
  //   $route(to, from) {
  //     this.getData();
  //   },
  // },
  computed: {

  },
  mounted() {
    // this.getData()
    // this.getRelationChoice()
    // this.$nextTick(() => {
    //   this.getAnnotate()
    // })
  },
  created() {
    // this.getData()
    this.getPdfData()
  },
  methods: {
    getPdfData() {
      const url = 'http://localhost:10088/Pdf/getPdf'
      // axios.get(url).then((response) => {
      //   console.log(response)
      //   console.log(typeof response.data[0].base64)
      //   let blob = new Blob([response.data[0].base64], {
      //       type: 'text/plain'
      //   });
      //   this.pdf_url = URL.createObjectURL(blob);
      // console.log(imgUrl)
      // })
      // axios({
      //   method: 'GET',
      //   url: 'http://localhost:10088/Pdf/getPdf',
      //   params: {
      //     id: '606ae579cdb2ca093653a88d'
      //   },
      // headers: {
      //   'Content-Type': 'application/vnd.openxmlformats- officedocument.spreadsheetml.sheet'
      // },
      //   responseType: 'blob'
      // }).then(response => {
      //   this.pdf_url = this.getObjectURL(response.data)
      //   this.pdfCurrentPage = 1
      // })
      // axios.get(url, {
      //   params: {
      //     id: '606ae579cdb2ca093653a88d'
      //   },
      //   paramsSerializer: function(params) {
      //     return qs.stringify(params, { arrayFormat: 'brackets' })
      //   },
      //   headers: {
      //     'Content-Type': 'application/vnd.openxmlformats- officedocument.spreadsheetml.sheet'
      //   },
      //   responseType: 'blob'
      // }).then(response => {
      //   this.pdf_url = this.getObjectURL(response.data)
      //   this.pdfCurrentPage = 1
      // })
      //this.pdf_obj.id = this.id
      axios({
        method: 'post',
        url: url,
        data: this.pdf_obj,
        responseType: 'blob'
      }).then(response => {
        console.log(response)
        this.pdf_url = this.getObjectURL(response.data)
      })
    },
    // 处理文件流
    getObjectURL(file) {
      let url = null
      if (window.createObjectURL !== undefined) { // basic
        url = window.createObjectURL(file)
      } else if (window.webkitURL !== undefined) { // webkit or chrome
        try {
          url = window.webkitURL.createObjectURL(file)
        } catch (error) {
          console.log(error)
        }
      } else if (window.URL !== undefined) { // mozilla(firefox)
        try {
          url = window.URL.createObjectURL(file)
        } catch (error) {
          console.log(error)
        }
      }
      return url
    },
    changePdfPage (val) {
      // console.log(val)
      if (val === 0 && this.pdfCurrentPage > 1) {
        this.pdfCurrentPage--
        // console.log(this.currentPage)
      }
      if (val === 1 && this.pdfCurrentPage < this.pageCount) {
        this.pdfCurrentPage++
        // console.log(this.currentPage)
      }
    },
    // pdf加载时
    loadPdfHandler (e) {
      this.pdfCurrentPage = 1 // 加载的时候先加载第一页
      console.log(this.pageCount)
    },
    logContent() {
      let win = this
			this.$refs.pdf.pdf.forEachPage(function(page) {
				return page.getTextContent()
				.then(function(content) {
					var text = content.items.map(item => item.str)
          win.targetText = win.targetText + text
					console.log(text)
				})
			});
		},
    handleRemove(file, fileList) {
        console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    handleExceed(files, fileList) {
      this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`);
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${ file.name }？`);
    },
    beforeUpload(file) {
      console.log(file)
      let imgUrl = URL.createObjectURL(file);
      console.log(imgUrl)
      this.src = imgUrl
      let types = ['image/jepg', 'image/png', 'image/gif', 'image/bmp']
      const isImage = types.includes(file.type)
      const isLtSize = file.size / 1024 / 1024 < 5;
      if (!isImage) {
        this.$message.error('上传图片只能是 JPG、GIF、BMP、PNG 格式!')
        return false
      }

      if (!isLtSize) {
        this.$message.error('上传图片大小不能超过 5MB!')
        return false
      }

      return true
    },
    handleSuccess(res, file) {
      console.log(res)
      this.targetText = res
    },
    addOcrData() {
      this.saveData.content = this.targetText
      this.saveData.title = this.textarea1
      this.getTime()
      const url = 'http://localhost:10088/OcrData/insertData'
      axios.post(url, this.saveData).then((response) => {
        console.log(response)
        if (response.data.code === 0) {
          this.$message({
            message: 'add success',
            type: 'success'
          })
        }
      }).catch((error) => {
        console.log(error)
      })

    },
    // 获取后台文章的摘要等信息
    getData() {
      this.targetText =
        '摘要：' +
        this.summary +
        '\n\n' +
        '关键字：' +
        this.keywords +
        '\n\n' +
        '作者：' +
        this.author +
        '\n\n' +
        '时间：' +
        this.date +
        '\n\n' +
        '期刊：' +
        this.journal +
        '\n\n' +
        '知识来源：' +
        this.src
      document.getElementById('content-area').innerHTML = this.targetText
      this.targetTextArr = this.targetText.split('')
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
          this.labels.push(a)
        }
        // this.labels = data;
        this.labels1 = this.labels
        for (var i = 0; i < this.labels.length; i++) {
          this.shows.push(true)
        }
      })
    },
    resize() {
      console.log('resize')
    },
    // 搜索相关代码
    querySearchAsync(queryString, cb) {
      this.labels = this.labels1
      var search_data = this.labels

      if (queryString === '') {
        for (var i = 0; i < this.labels.length; i++) {
          this.shows[i] = true
        }
        this.labels = this.labels1
      }
      this.results = queryString
        ? search_data.filter(this.createStateFilter(queryString))
        : search_data

      const list = []
      for (const result of this.results) {
        list.push({ value: result.labelname })
      }

      clearTimeout(this.timeout)
      this.timeout = setTimeout(() => {
        if (list.length !== this.labels.length) {
          cb(list)
        } else {
          cb([])
        }
      }, 1000 * Math.random())
    },
    // 搜索相关代码
    handleSelect1(item) {
      var lit = []
      if (item !== '') {
        for (var i = 0; i < this.labels.length; i++) {
          if (item.value === this.labels[i].labelname) {
            this.shows[i] = true
            lit.push(this.labels[i])
          } else {
            this.shows[i] = false
          }
        }
        this.labels = lit
      }
    },
    // 搜索相关代码
    createStateFilter(queryString) {
      return (state) => {
        console.log(
          state.labelname.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        )

        return (
          state.labelname.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        )
      }
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
      this.saveData.time = new Date().Format('yyyy-MM-dd hh:mm:ss')
    },
    // 添加标注
    addFileMark() {
      this.fileMark.document_id = this.id
      this.fileMark.document_type = this.$route.query.document_type
      this.fileMark.object_marks = []
      this.fileMark.relation_marks = []
      this.getTime()
      for (var i = 0; i < this.labels.length; i++) {
        var objs = []
        for (var j = 0; j < this.labels[i].entitylist.length; j++) {
          var o = {
            name: this.labels[i].entitylist[j].text,
            description: '',
            ICD_11: '',
            type: '',
            is_checked: false,
            is_passed: false,
            mark_user_id: 1,
            mark_time: this.time,
            check_user_id: '',
            check_time: this.time,
            multiple_marked: false
          }
          objs.push(o)
        }
        var obj = {
          object_type: this.labels[i].labelname,
          objects: objs
        }
        this.fileMark.object_marks.push(obj)
      }

      for (let i = 0; i < this.relationOptions.length; i++) {
        const arr = this.relationOptions[i].label.split('-')
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
        this.fileMark.relation_marks.push(obj_rel)
      }

      const url = 'http://localhost:10088/FileMarks/addFileMark'
      axios.post(url, this.fileMark).then((response) => {
        console.log(response)
        console.log(response.data.msg)
        if (response.data.msg === '添加成功') {
          this.$message({
            message: '恭喜你，添加成功',
            type: 'success'
          })
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    // 选择关系
    getRelationChoice() {
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
          this.relationOptions.push(a)
        }
      })
    },
    routeBack() {
      this.$router.back(-1)
      // this.$router.push({
      //   path: '/xieweihao/Document_information_extraction/' + this.$route.query.document_type,
      //   query: {
      //     name: this.$route.query.document_type
      //   }
      // })
    },
    saveInfo() {

    },
    handleSkip() {},
    handleSubmit() {}
  }
}
</script>

<style  scoped>
.components-container {
  position: relative;
  height: 100vh;
}

.left-container {
  background-color: #f38181;
  height: 100%;
}

.right-container {
  background-color: #fce38a;
  height: 200px;
}

.top-container {
  background-color: #fce38a;
  width: 100%;
  height: 100%;
}

.bottom-container {
  width: 100%;
  background-color: #95e1d3;
  height: 100%;
}
.text {
  font-size: 14px;
}

.item {
  padding: 18px 0;
}

.box-card {
  width: 100%;
  height: 100%;
}

.bg-purple {
  background: #d3dce6;
}

.bg-purple-light {
  background: #e5e9f2;
}

.grid-content {
  border-radius: 4px;
  min-height: 60px;
}

.sublabelListArea {
  list-style-type: none;
  white-space: nowrap;
  margin-top: 10px;
  padding: 10px;
  overflow: hidden;
}

.labelBlock {
  border: 1px solid #d7d8d9;
  background-color: #f5f5f6;
}

.inline {
  display: inline-block;
}

.point {
  width: 10px;
  height: 10px;
  border-radius: 5px;
}

.hideOverFlow {
  overflow: hidden;
}

.dot-color-1 {
  background: #f06292;
}

.dot-color-2 {
  background: #ba68c8;
}

.dot-color-3 {
  background: #9575cd;
}

.dot-color-4 {
  background: #7986cb;
}

.dot-color-5 {
  background: #64b5f6;
}

.dot-color-6 {
  background: #4fc3f7;
}

.dot-color-7 {
  background: #4dd0e1;
}

.dot-color-8 {
  background: #4db6ac;
}

.dot-color-9 {
  background: #81c784;
}

.dot-color-10 {
  background: #aed581;
}

.dot-color-11 {
  background: #dce775;
}

.dot-color-12 {
  background: #fff176;
}

.sublabelcheckbox {
  float: left;
  margin-top: 10px;
}

.entity-area {
  overflow: hidden;
}

.entity-area {
  height: 500px;
  overflow-y: auto;
}

.entity-bar {
  border: 1px solid #ebecec;
  cursor: pointer;
  overflow: hidden;
  padding: 10px;
}

.el-table--enable-row-hover .el-table__body tr:hover > td {
  background-color: transparent !important;
}

.entity-block {
  border: 1px solid #ebecec;
  background-color: white;
  width: 100%;
  padding: 10px;
  margin-top: 10px;
}
.entity-text {
  background-color: white;
  width: calc(100% - 50px);
  overflow: hidden;
  white-space: nowrap;
  -o-text-overflow: ellipsis;
  text-overflow: ellipsis;
}
.test-entity-block {
  border: 1px solid #ebecec;
  background-color: #f5f5f6;
  width: 100%;
  padding: 10px;
  margin-top: 10px;
}
.test-entity-text {
  width: calc(100% - 50px);
  display: inline-block;
  overflow: hidden;
  white-space: nowrap;
  -o-text-overflow: ellipsis;
  text-overflow: ellipsis;
}
.test-button {
  float: right;
  display: inline-block;
  vertical-align: center;
  vertical-align: middle;
}

.entity-block-area {
  padding: 10px;
  background-color: white;
}
.highlight-color {
  background-color: #ffff00;
}

.right {
  float: right;
  margin-right: 5px;
}

.transparent {
  background-color: transparent;
}

.mark-area {
  position: absolute;
  z-index: 1;
  line-height: 3em;
  white-space: pre-wrap;
  opacity: 0.5;
  font-family: "\5FAE\8F6F\96C5\9ED1", "Avenir", Helvetica, Arial, sans-serif;
  font-size: 14px;
  margin: 0;
  background-color: transparent;
}
.content-area {
  white-space: pre-wrap;
  font-family: "\5FAE\8F6F\96C5\9ED1", "Avenir", Helvetica, Arial, sans-serif;
  font-size: 14px;
  position: absolute;
  line-height: 3em;
  z-index: 2;
  background-color: transparent;
  margin: 0;
}

.content-container {
  position: relative;
  height: 50em;
  overflow-y: auto;
}

.content-display-area {
  width: 50%;
  padding: 10px;
  border: 1px solid #ebecec;
  margin-bottom: 14px;
  line-height: 25px;
  display: inline-block;
  height: 100%;
}

.title {
  font-size: 20px;
  margin-bottom: 14px;
}
.action-area {
  height: 50px;
}
</style>
