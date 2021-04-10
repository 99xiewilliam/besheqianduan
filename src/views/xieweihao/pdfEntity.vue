<template>
  <div class="components-container">
    <i
      class="el-icon-back"
      style="weight: 1000; cursor: pointer"
      @click="routeBack()"
    />
    <aside>
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="实体标注" name="first" />
        <el-tab-pane label="关系标注" name="second" />
      </el-tabs>
    </aside>

    <split-pane split="vertical" @resize="resize">
      <template slot="paneL">
        <el-card class="box-card">
          <div class="content-container">
            <iframe :src="frame_url" width="100%" height="100%" frameborder="0" id="myIframe">
            </iframe>
          </div>
        </el-card>
      </template>
      <template slot="paneR">
        <el-card class="box-card">
          <div class="action-area">
            <el-autocomplete
              v-model="state"
              :fetch-suggestions="querySearchAsync"
              placeholder="请输入内容"
              @select="handleSelect1"
            />
            <el-button class="right" @click="routeBack()">不合格</el-button>
            <el-button
              type="primary"
              class="right"
              @click="addFileMark()"
            >提交</el-button>
            <el-button class="right" @click="handleSkip()">下一篇</el-button>
          </div>
          <el-scrollbar
            style="height: 100%"
            wrap-style="overflow-x:hidden;"
          >
            <div class="entity-area">
              <li
                v-for="(label, index) in labels"
                :key="index"
                class="sublabelListArea"
              >
                <div class="labelBlock">
                  <div class="entity-bar" @click="handleFold(index)">
                    <div :class="`inline point dot-color-${label.color}`" />
                    <span class="labelname">{{ label.labelname }}</span>
                    <i
                      v-if="!noShowList[index]"
                      class="right inline el-icon-caret-bottom"
                    />
                    <i
                      v-if="noShowList[index]"
                      class="right inline el-icon-caret-left"
                    />
                    <!-- <el-checkbox
                      v-model="entitylist[index]"
                      class="right inline"
                      type="warning"
                      size="small"
                      label="连续标注模式"
                      border
                      @change="(value) => handleContinuousChange(value, index)"
                    /> -->
                    <el-button
                      class="right inline"
                      type="primary"
                      icon="el-icon-circle-plus-outline"
                      size="small"
                      @click="handleSingleAdd(index)"
                    >添加标注</el-button>
                  </div>

                  <div v-if="!noShowList[index]" class="entity-block-area">
                    <div margin-bottom="10px">{{ label.labeldescribe }}</div>
                    <li
                      v-for="(entity, entityIndex) in label.entitylist"
                      :key="entityIndex"
                    >
                      <div
                        class="test-entity-block"
                        
                      >
                        <span class="test-entity-text">{{
                          entity.text
                        }}</span>
                        <el-button
                          class="test-button"
                          icon="el-icon-delete"
                          circle
                          size="mini"
                          type="danger"
                          @click="removeEntity(index, entityIndex)"
                        />
                      </div>
                    </li>
                  </div>
                </div>
              </li>
            </div>
          </el-scrollbar>
        </el-card>
      </template>
    </split-pane>
  </div>
</template>

<script>
import splitPane from 'vue-splitpane'
import axios from 'axios'

export default {
  name: 'SplitpaneDemo',
  components: { splitPane },
  data() {
    return {
      list: this.$route.query.list,
      selectText: '',
      frame_url: '',
      pdf_url: '',
      pdf_obj: {
        'id': this.$route.query.id
      },
      id: this.$route.query.id,
      title: '',
      author: '',
      date: '',
      journal: '',
      src: '',
      summary: '',
      keywords: '',
      activeName: 'first',
      document_type: this.$route.query.document_type,
      labels: [

      ],
      labels1: [

      ],
      entitylist: [],
      // 文档部分
      targetText: ' ',
      targetTextArr: [],
      // 页面配置和状态部分
      curindex: 0, // 当前文档
      curMarkLabel: -1, // 当前使用的标签,值为-1代表当前未在标注实体
      continuousMark: false, // 连续标注模式
      noShowList: [],
      shows: [],
      state: '',
      results: [],
      time: '',
      fileMark: {
        document_id: '',
        document_type: this.$route.query.document_type,
        type: '',
        object_marks: [

        ],
        relation_marks: [
          {
            relaiton_id: '',
            start_type: '',
            relation_type: '',
            end_type: '',
            relations: [
              {
                start_object: '',
                end_object: '',
                advice: '',
                evi_level: '',
                evi_describe: '',
                reference: '',
                group: '',
                time: '',
                type: '',
                is_checked: false,
                is_passed: false,
                mark_user_id: '',
                mark_time: '',
                check_user_id: '',
                check_time: '',
                is_multiple_marked: false
              }
            ],
            is_checked: false,
            is_passed: false,
            mark_user: '',
            mark_time: '',
            check_user: '',
            check_time: ''
          }
        ]
      },
      fileMarkS: [],
      relationOptions: []
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
    //this.getData()
    this.getRelationChoice()
    this.switchPoint()
    this.$nextTick(() => {
      this.getAnnotate()
    })
  },
  created() {
    this.getData()
    this.getPdfData()
  },
  methods: {
    switchPoint() {
      let vm = this
      let iframe = document.getElementById("myIframe")
      let x1 = ''
      let x2 = ''
      let y1 = ''
      let y2 = ''
      iframe.onload = function () {
        iframe.contentDocument.addEventListener('mousedown', function (e) {
          x1 = e.pageX
          y1 = e.pageY
        }, true)

        iframe.contentDocument.addEventListener('mouseup', function (e) {
          x2 = e.pageX
          y2 = e.pageY
          if (x1 === x2 && y1 === y2) return;
          var choose = iframe.contentWindow.getSelection().toString()
          vm.selectText = choose
          console.log('selected words')
          console.log(vm.selectText)
        }, true)
      }
    },
    sendMessage() {
      let vm = this
      let iframe = document.getElementById('myIframe')
      iframe.contentWindow.postMessage(vm.selectText, "*")
    },
    getMessage() {
      let iframe = document.getElementById('myIframe')
      iframe.contentWindow.addEventListener('message', function (e) {
        console.log(e.data)
        iframe.contentWindow.PDFViewerApplication.findBar.findField.value = e.data
        iframe.contentWindow.PDFViewerApplication.findBar.highlightAll.checked = true
        iframe.contentWindow.PDFViewerApplication.findBar.dispatchEvent('highlightallchange')
      }, false)
    },
    // 获取后台文章的摘要等信息
    getPdfData() {
      const url = 'http://localhost:10088/Pdf/getPdfNew'
      axios({
        method: 'post',
        url: url,
        data: this.pdf_obj,
        responseType: 'blob'
      }).then(response => {
        console.log(response)
        this.pdf_url = window.URL.createObjectURL(response.data)
        this.frame_url = `./pdfjs-2/web/viewer.html?file=${this.pdf_url}`
        console.log(this.pdf_url)
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
    async getData() {
      // const url = 'http://localhost:10088/reference/getOnePaper'
      // await axios.post(url, this.id).then((response) => {
      //   console.log(response)
      //   this.summary = response.data.summary
      //   this.author = response.data.author
      //   this.date = response.data.date
      //   this.journal = response.data.journal
      //   this.keywords = response.data.keywords
      //   this.src = response.data.src
      //   this.title = response.data.title
      // })
      // this.targetText =
      //   '摘要：' +
      //   this.summary +
      //   '\n\n' +
      //   '关键字：' +
      //   this.keywords +
      //   '\n\n' +
      //   '作者：' +
      //   this.author +
      //   '\n\n' +
      //   '时间：' +
      //   this.date +
      //   '\n\n' +
      //   '期刊：' +
      //   this.journal +
      //   '\n\n' +
      //   '知识来源：' +
      //   this.src
      // document.getElementById('content-area').innerHTML = this.targetText
      // this.targetTextArr = this.targetText.split('')
      const url1 = 'http://localhost:10088/Entities/entity'
      await axios.get(url1).then((response) => {
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
    // 跳转到关系标注页面
    handleClick(tab, event) {
      console.log('到关系')
      console.log(this.document_type)
      if (tab.$options.propsData.label === '关系标注') {
        this.$router.push({
          path:
            '/xieweihao/Document_information_extraction/Paper/electronicDocument/pdfRelation',
          query: {
            id: this.id,
            list: this.list,
            document_type: this.document_type
          }
        })
      }
    },
    handleFold(index) {
      this.noShowList[index] = !this.noShowList[index]
      this.$forceUpdate()
    },
    // 将标注结果同步到实体背景色中
    syncResult() {
      for (var i in this.labels) {
        for (var j in this.labels[i].entitylist) {
          this.addColor(
            i,
            this.labels[i].entitylist[j].start,
            this.labels[i].entitylist[j].end
          )
        }
      }
    },
    // 获取标注结果
    getAnnotate() {
      // this.labels = JSON.parse(this.datalist[this.curindex].result);
      this.syncResult()
    },
    // 单个实体添加模式
    handleSingleAdd(index) {
      var self = this
      //var selection = window.getSelection()
      this.curMarkLabel = index
      this.continuousMark = false
      this.noShowList[index] = !this.noShowList[index] // Undo Extra Show Action
      console.log(this.noShowList[index])
      self.addEntity(self.curMarkLabel)
      // if (
      //   !selection.isCollapsed &&
      //   selection.focusNode.parentElement.className === 'content-area' &&
      //   selection.anchorNode.parentElement.className === 'content-area'
      // ) {
      //   // 进行标记
      //   var start = selection.focusOffset
      //   var end = selection.anchorOffset
      //   if (start > end) {
      //     var tmp = start
      //     start = end
      //     end = tmp
      //   }
      //   self.addEntity(self.curMarkLabel, start, end)
      //   window.getSelection().removeAllRanges()
      //   // 后处理
      //   if (!self.continuousMark) {
      //     self.curMarkLabel = -1
      //   }
      // } else {
      //   console.log('heme')
      // }
    },
    // 连续实体添加模式按钮点击处理
    handleContinuousChange(value, index) {
      if (value === true) {
        if (value === this.continuousMark && this.curMarkLabel !== index) {
          // 保持连续标注模式，当前标签更改
          for (var i in this.entitylist) {
            this.entitylist[i] = false
          }
          this.entitylist[index] = true
          this.curMarkLabel = index
        } else if (value !== this.continuousMark) {
          // 进入连续标注模式
          this.continuousMark = true
          this.curMarkLabel = index
        }
      } else {
        // 退出连续标注模式
        this.continuousMark = false
        this.curMarkLabel = -1
      }
    },
    // 高亮悬停实体 实体抽取和关系抽取可以复用
    // handleEntityMouseOver(entity) {
    //   var fixPos = document.documentElement.scrollTop
    //   document.getElementById('token_' + entity.start).scrollIntoView(false)
    //   document.documentElement.scrollTop = fixPos
    //   // Add highlight class to entity
    //   for (var i = entity.start; i < entity.end; i++) {
    //     document.getElementById('token_' + i).className =
    //       document.getElementById('token_' + i).className + 'highlight-color'
    //   }
    // },
    // // 取消高亮 实体抽取和关系抽取可以复用
    // handleEntityMouseOut(entity) {
    //   // Remove highlight class from entity
    //   for (var i = entity.start; i < entity.end; i++) {
    //     document.getElementById(
    //       'token_' + i
    //     ).className = document
    //       .getElementById('token_' + i)
    //       .className.replace('highlight-color', '')
    //   }
    // },
    // 向lables数组添加实体同时为实体添加背景色 实体抽取和关系抽取 可以部分复用的函数
    addEntity(labelIndex) {
      // Add Entity, Add Mark
      const text = this.selectText
      var node = {
        text: text
      }
      if (text !== '') {
        this.labels[labelIndex].entitylist.push(node)
        // this.sendMessage()
        // this.getMessage()
        this.selectText = ''
        this.$forceUpdate() // 嵌套数组更新没有监听到，强制更新数据
      }
      
      //this.addColor(labelIndex, start, end)
    },
    // 为实体添加背景色
    addColor(labelIndex, start, end) {
      for (var i = start; i < end; i++) {
        document.getElementById('token_' + i).className =
          document.getElementById('token_' + i).className +
          'dot-color-' +
          this.labels[labelIndex].color +
          ' '
      }
    },
    // 从labels中移除实体，去除背景色 不能复用的函数，但是实体抽取和关系抽取可以相互参考
    removeEntity(labelIndex, entityIndex) {
      // Remove Entity, Remove Mark
      // var start = this.labels[labelIndex].entitylist[entityIndex].start
      // var end = this.labels[labelIndex].entitylist[entityIndex].end
      // for (var i = start; i < end; i++) {
      //   var raw = document.getElementById('token_' + i).className
      //   var target = 'dot-color-' + this.labels[labelIndex].color + ' '
      //   // 移除实体颜色
      //   document.getElementById('token_' + i).className = raw.replace(
      //     target,
      //     ''
      //   )
      //   // 移除高亮
      //   document.getElementById(
      //     'token_' + i
      //   ).className = document
      //     .getElementById('token_' + i)
      //     .className.replace('highlight-color', '')
      // }
      // 从labels数组中移除entity
      this.labels[labelIndex].entitylist.splice(entityIndex, 1)
      this.$forceUpdate() // 嵌套数组更新没有监听到，强制更新数据
    },
    // 初始化labels 实体抽取和关系抽取可以复用的函数
    clearEntities() {
      for (var i in this.labels) {
        if (this.labels[i].entitylist.length !== 0) {
          for (var j in this.labels[i].entitylist) {
            console.log(this.labels[i].entitylist[j])
            var entity = this.labels[i].entitylist[j]
            var start = entity.start
            var end = entity.end
            for (var k = start; k < end; k++) {
              document.getElementById('token_' + k).className = ''
            }
          }
          this.labels[i].entitylist = []
        }
      }
    },
    // 处理光标选择事件 实体抽取和关系抽取可以复用
    handleSelect() {
      var self = this
      var selection = window.getSelection()
      if (self.curMarkLabel !== -1) {
        if (
          !selection.isCollapsed &&
          selection.focusNode.parentElement.className === 'content-area' &&
          selection.anchorNode.parentElement.className === 'content-area'
        ) {
          // 进行标记
          var start = selection.focusOffset
          var end = selection.anchorOffset
          if (start > end) {
            var tmp = start
            start = end
            end = tmp
          }
          self.addEntity(self.curMarkLabel, start, end)
          window.getSelection().removeAllRanges()
          // 后处理
          if (!self.continuousMark) {
            self.curMarkLabel = -1
          }
        }
      }
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
      this.time = new Date().Format('yyyy-MM-dd hh:mm:ss')
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
      const time = this.time
      const document_type = this.document_type
      axios.post(url, this.fileMark).then((response) => {
        console.log(response)
        console.log(response.data.msg)
        if (response.data.msg === '添加成功') {
          const url2 = 'http://localhost:10088/Item/updateTime'
          const obj = { time: time, name: document_type }
          axios.put(url2, obj).then((response) => {
            console.log(response)
          })
          this.$message({
            message: '恭喜你，添加成功',
            type: 'success'
          })
        }
      }).catch((error) => {
        console.log(error)
      })

      // const url1 = 'http://localhost:10088/reference/modify'
      // axios.put(url1, {
      //   id: this.id
      // }).then((response) => {
      //   console.log(response)
      // })
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
      this.$router.push({
        path: '/xieweihao/Document_information_extraction/Paper/electronicDocument',
        query: {
          document_type: this.document_type
        }
      })
    },
    saveInfo() {

    },
    handleSkip() {
      console.log('handleSkip')
      console.log(this.list)
      let judge = 0
      
      for (let i = 0; i < this.list.length; i++) {
        if (judge === 1) {
          this.pdf_obj.id = this.list[i].id.slice(19, this.list[i].id.length - 1)
          break
        }
        if (this.id === this.list[i].id.slice(19, this.list[i].id.length - 1)) {
          judge = 1
        }
      }
      console.log(this.id)
      this.id = this.pdf_obj.id
      console.log(this.pdf_obj.id)
      this.getPdfData()
      // this.$router.push({
      //   path: '/xieweihao/Document_information_extraction/Paper/ShowArticle',
      //   query: {
      //     id: this.id,
      //     list: this.list
      //   }
      // })
    },
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
