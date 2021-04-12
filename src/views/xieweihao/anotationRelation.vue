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

    <h1>{{ "标题：" + title }}</h1>
    <split-pane split="vertical" @resize="resize">
      <template slot="paneL">
        <el-card class="box-card">
          <div class="content-container">
            <xmp id="content-area" class="content-area" @mouseup="handleSelect">
              <el-scrollbar style="height:100%" wrap-style="overflow-x:hidden;">
              {{ targetText }}
              </el-scrollbar>
            </xmp>
            <span id="mark-area" class="mark-area transparent">
              <el-scrollbar
                style="height: 100%"
                wrap-style="overflow-x:hidden;"
              >
                <span
                  v-for="(char, index) in targetTextArr"
                  :key="index"
                  class="transparent"
                >
                  <span :id="`token_${index}`">{{ char }}</span>
                </span>
              </el-scrollbar>
            </span>
          </div>
        </el-card>
      </template>
      <template slot="paneR">
        <el-card class="box-card" style="text-align: center">
          <el-container>
            <el-header v-show="showDot">
              <h2>添加关系</h2>
              <hr>
            </el-header>
            <!-- <div v-show="tableShow">
              <el-table
                :data="tableData"
                style="width: 100%"
                max-height="250"
              >

                <el-table-column
                  prop="addedRelation"
                  label="已添加关系"
                  width="120"
                />
                <el-table-column
                  label="是否为关系组"
                  width="300"
                >
                  <template slot-scope="scope">
                    <el-switch
                      v-model="isgroup[scope.$index]"
                      active-text="是"
                      inactive-text="否"
                    />
                  </template>
                </el-table-column>
                <el-table-column
                  fixed="right"
                  label="操作"
                  width="120"
                >
                  <template slot-scope="scope">
                    <el-button
                      type="text"
                      size="small"
                      @click="deleteRow(scope.$index)"
                    >
                      移除
                    </el-button>
                  </template>
                </el-table-column>
              </el-table>
            </div> -->
            <el-main>
              <i
                v-show="showDot"
                class="el-icon-circle-plus"
                style="font-size: 40px; color: blue"
                @click="getDialogForm()"
              />
              <div v-show="showCard">
                <el-card class="box-card">
                  <div slot="header" class="clearfix">
                    <span>添加关系</span>
                  </div>
                  <div class="text item">
                    <el-select v-model="form.region" placeholder="请选择活动区域" @change="handleChange($event)">
                      <el-option
                        v-for="item in options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value"
                      />
                    </el-select>
                  </div>
                  <div class="text item">
                    <el-button @click="showCardDisaper()">取 消</el-button>
                    <el-button
                      type="primary"
                      @click="dealWithRelation()"
                    >确 定</el-button>
                  </div>
                </el-card>
              </div>
              <div v-show="showCard2">
                <el-card class="box-card">
                  <div class="">
                    <el-scrollbar style="height:100%">
                      <div style="width:700px;height:700px;">
                        <div slot="header" class="clearfix">
                          <span>添加关系</span>
                        </div>
                        <div class="text item">
                          <el-select v-model="form.region" placeholder="请选择活动区域" @change="handleChange($event)">
                            <el-option
                              v-for="item in options1"
                              :key="item.value"
                              :label="item.label"
                              :value="item.value"
                            />
                          </el-select>
                        </div>
                        <div class="text item">
                          <span>起始实体：</span>
                          <li
                            v-for="(label, index) in labels"
                            :key="index"
                            class="sublabelListArea"
                          >
                            <div class="labelBlock">
                              <div class="entity-bar" @click="handleFold(index)">
                                <i
                                  v-if="!noShowList[index]"
                                  class="right inline el-icon-caret-bottom"
                                />
                                <i
                                  v-if="noShowList[index]"
                                  class="right inline el-icon-caret-left"
                                />
                                <el-button
                                  class="inline"
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
                                    @mouseover="handleEntityMouseOver(entity)"
                                    @mouseout="handleEntityMouseOut(entity)"
                                  >
                                    <span class="test-entity-text">{{
                                      targetText.substr(
                                        entity.start,
                                        entity.end - entity.start
                                      )
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
                        <div class="text item">
                          <span>目标实体：</span>
                          <li
                            v-for="(label, index) in label1"
                            :key="index"
                            class="sublabelListArea"
                          >
                            <div class="labelBlock">
                              <div class="entity-bar" @click="handleFold(index)">
                                <i
                                  v-if="!noShowList[index]"
                                  class="right inline el-icon-caret-bottom"
                                />
                                <i
                                  v-if="noShowList[index]"
                                  class="right inline el-icon-caret-left"
                                />
                                <el-button
                                  class="inline"
                                  type="primary"
                                  icon="el-icon-circle-plus-outline"
                                  size="small"
                                  @click="handleSingleAdd1(index)"
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
                                    @mouseover="handleEntityMouseOver(entity)"
                                    @mouseout="handleEntityMouseOut(entity)"
                                  >
                                    <span class="test-entity-text">{{
                                      targetText.substr(
                                        entity.start,
                                        entity.end - entity.start
                                      )
                                    }}</span>
                                    <el-button
                                      class="test-button"
                                      icon="el-icon-delete"
                                      circle
                                      size="mini"
                                      type="danger"
                                      @click="removeEntity1(index, entityIndex)"
                                    />
                                  </div>
                                </li>
                              </div>
                            </div>
                          </li>
                        </div>
                        <div class="text item">
                          <span>建议：</span>
                          <el-select v-model="form.region" placeholder="请选择建议" @change="handleChange($event)">
                            <el-option
                              v-for="item in options3"
                              :key="item.value"
                              :label="item.label"
                              :value="item.value"
                            />
                          </el-select>
                        </div>
                        <div class="text item">
                          <span>是否为关系组：</span>
                          <el-switch
                            v-model="group"
                            active-text="是"
                            inactive-text="否"
                          />
                        </div>
                        <div class="text item">
                          <span>证据级别：</span>
                          <el-input v-model="level" placeholder="请输入证据级别" />
                        </div>
                        <div class="text item">
                          <span>临床依据：</span>
                          <el-input v-model="describe" placeholder="请输入临床依据" />
                        </div>
                        <div class="text item">
                          <el-button @click="showCard2Disapeer">取 消</el-button>
                          <el-button
                            type="primary"
                            @click="checkRelation()"
                          >确 定</el-button>
                        </div>
                      </div>
                    </el-scrollbar>
                  </div>
                </el-card>
              </div>
            </el-main>
          </el-container>
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
      judge: 0,
      id: this.$route.query.id,
      title: '',
      author: '',
      date: '',
      journal: '',
      src: '',
      summary: '',
      keywords: '',
      activeName: 'second',
      labels: [
        {
          color: 1,
          labelname: '起始实体',
          entitylist: []
        }
      ],
      label1: [
        {
          color: 1,
          labelname: '目标实体',
          entitylist: []
        }
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
      dialogFormVisible: false,
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
      options: [
      ],
      options1: [],
      choice: '',
      dialogFormVisible1: false,
      options3: [],
      close: false,
      showCard: false,
      showDot: true,
      showCard2: false,
      time: '',
      tableData: [],
      tableShow: false,
      level: '',
      describe: '',
      isgroup: [],
      advice: '',
      group: true,
      fileMark: {
        document_id: this.$route.query.id,
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
      Entitylabels: [],
      document_type: this.$route.query.document_type
    }
  },
  mounted() {
    // this.getData()
    this.getEntityChioce()
    this.$nextTick(() => {
      this.getAnnotate()
    })
  },
  created() {
    this.getData()
  },
  methods: {
    // 获取后台文章摘要
    async getData() {
      const url = 'http://localhost:10088/reference/getOne' + this.$route.query.document_type
      await axios.post(url, this.id).then((response) => {
        console.log(response)
        this.summary = response.data.summary
        this.author = response.data.author
        this.date = response.data.date
        this.journal = response.data.journal
        this.keywords = response.data.keywords
        this.src = response.data.src
        this.title = response.data.title
      })
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

      const url1 = 'http://localhost:10088/Relations/relation'
      axios.get(url1).then((response) => {
        const datas = response.data
        for (var data in datas) {
          const a = {
            value: '药品分类-适应症-疾病',
            label: '药品分类-适应症-疾病'
          }
          a.value = datas[data].object + '-' + datas[data].relation + '-' + datas[data].subject
          a.label = datas[data].object + '-' + datas[data].relation + '-' + datas[data].subject
          this.options.push(a)
        }
      })
    },
    resize() {
      console.log('resize')
    },
    // handleClick(tab, event) {
    //   console.log(tab, event)
    // },
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
    // 起始单个实体添加模式
    handleSingleAdd(index) {
      this.judge = 0
      var self = this
      var selection = window.getSelection()
      this.curMarkLabel = index
      this.continuousMark = false
      this.noShowList[index] = !this.noShowList[index] // Undo Extra Show Action
      console.log(this.noShowList[index])
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
      } else {
        console.log('itheima')
        console.log(!selection.isCollapsed)
      }
    },
    // 目标单个实体添加模式
    handleSingleAdd1(index) {
      console.log('target12121212')
      this.judge = 1
      var self = this
      var selection = window.getSelection()
      this.curMarkLabel = index
      this.continuousMark = false
      this.noShowList[index] = !this.noShowList[index] // Undo Extra Show Action
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
        self.addEntity1(self.curMarkLabel, start, end)
        window.getSelection().removeAllRanges()
        // 后处理
        if (!self.continuousMark) {
          self.curMarkLabel = -1
        }
      } else {
        console.log('itheima')
        console.log(!selection.isCollapsed)
      }
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
    handleEntityMouseOver(entity) {
      var fixPos = document.documentElement.scrollTop
      document.getElementById('token_' + entity.start).scrollIntoView(false)
      document.documentElement.scrollTop = fixPos
      // Add highlight class to entity
      for (var i = entity.start; i < entity.end; i++) {
        document.getElementById('token_' + i).className =
          document.getElementById('token_' + i).className + 'highlight-color'
      }
    },
    // 取消高亮 实体抽取和关系抽取可以复用
    handleEntityMouseOut(entity) {
      // Remove highlight class from entity
      for (var i = entity.start; i < entity.end; i++) {
        document.getElementById(
          'token_' + i
        ).className = document
          .getElementById('token_' + i)
          .className.replace('highlight-color', '')
      }
    },
    // 起始向lables数组添加实体同时为实体添加背景色 实体抽取和关系抽取 可以部分复用的函数
    addEntity(labelIndex, start, end) {
      // Add Entity, Add Mark
      var node = {
        start: start,
        end: end,
        text: this.targetText.substr(start, end - start)
      }
      this.labels[labelIndex].entitylist.push(node)
      this.$forceUpdate() // 嵌套数组更新没有监听到，强制更新数据
      this.addColor(labelIndex, start, end)
    },
    // 目标向lables数组添加实体同时为实体添加背景色 实体抽取和关系抽取 可以部分复用的函数
    addEntity1(labelIndex, start, end) {
      // Add Entity, Add Mark
      var node = {
        start: start,
        end: end,
        text: this.targetText.substr(start, end - start)
      }
      this.label1[labelIndex].entitylist.push(node)
      this.$forceUpdate() // 嵌套数组更新没有监听到，强制更新数据
      this.addColor(labelIndex, start, end)
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
      console.log(labelIndex)
      console.log(entityIndex)
      console.log(this.labels[labelIndex].entitylist[entityIndex])
      var start = this.labels[labelIndex].entitylist[entityIndex].start
      var end = this.labels[labelIndex].entitylist[entityIndex].end
      for (var i = start; i < end; i++) {
        var raw = document.getElementById('token_' + i).className
        var target = 'dot-color-' + this.labels[labelIndex].color + ' '
        // 移除实体颜色
        document.getElementById('token_' + i).className = raw.replace(
          target,
          ''
        )
        // 移除高亮
        document.getElementById(
          'token_' + i
        ).className = document
          .getElementById('token_' + i)
          .className.replace('highlight-color', '')
      }
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
    // 移除实体
    removeEntity1(labelIndex, entityIndex) {
      // Remove Entity, Remove Mark
      var start = this.label1[labelIndex].entitylist[entityIndex].start
      var end = this.label1[labelIndex].entitylist[entityIndex].end
      for (var i = start; i < end; i++) {
        var raw = document.getElementById('token_' + i).className
        var target = 'dot-color-' + this.label1[labelIndex].color + ' '
        // 移除实体颜色
        document.getElementById('token_' + i).className = raw.replace(
          target,
          ''
        )
        // 移除高亮
        document.getElementById(
          'token_' + i
        ).className = document
          .getElementById('token_' + i)
          .className.replace('highlight-color', '')
      }
      // 从labels数组中移除entity
      this.label1[labelIndex].entitylist.splice(entityIndex, 1)
      this.$forceUpdate() // 嵌套数组更新没有监听到，强制更新数据
    },
    // 初始化labels 实体抽取和关系抽取可以复用的函数
    clearEntities1() {
      for (var i in this.label1) {
        if (this.label1[i].entitylist.length !== 0) {
          for (var j in this.label1[i].entitylist) {
            var entity = this.label1[i].entitylist[j]
            var start = entity.start
            var end = entity.end
            for (var k = start; k < end; k++) {
              document.getElementById('token_' + k).className = ''
            }
          }
          this.label1[i].entitylist = []
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
          if (self.judge === 0) {
            self.addEntity(self.curMarkLabel, start, end)
          } else if (self.judge === 1) {
            self.addEntity1(self.curMarkLabel, start, end)
          }

          window.getSelection().removeAllRanges()
          // 后处理
          if (!self.continuousMark) {
            self.curMarkLabel = -1
          }
        }
      }
    },
    handleSkip() {},
    handleSubmit() {},
    // handleSelect(key, keyPath) {
    //   console.log(key, keyPath)
    // },
    // 让card出现
    getDialogForm() {
      // this.dialogFormVisible = true;
      this.showCard = true
      this.showDot = false
      this.tableShow = false
      this.describe = ''
      this.level = ''
      const length = this.labels[0].entitylist.length
      const length1 = this.label1[0].entitylist.length
      for (let j = length - 1; j >= 0; j--) {
        this.removeEntity(0, j)
      }

      for (let i = length1 - 1; i >= 0; i--) {
        this.removeEntity1(0, i)
      }
    },
    showCardDisaper() {
      this.showCard = false
      this.showDot = true
    },
    showCard2Disapeer() {
      this.showCard = true
      this.showCard2 = false
      this.level = ''
      this.describe = ''

      const length = this.labels[0].entitylist.length
      const length1 = this.label1[0].entitylist.length

      for (let j = length - 1; j >= 0; j--) {
        this.removeEntity(0, j)
      }

      for (let i = length1 - 1; i >= 0; i--) {
        this.removeEntity1(0, i)
      }
    },
    handleChange(e) {
      this.choice = e
      const obj = {
        addedRelation: e
      }
      this.tableData.push(obj)
    },

    handleChange1(e) {

    },
    dealWithRelation() {
      this.options1 = []
      this.options1.push(this.choice)
      this.showCard = false
      this.showCard2 = true
    },
    handleClick(tab, event) {
      if (tab.$options.propsData.label === '实体标注') {
        this.$router.push({
          path: '/xieweihao/Document_information_extraction/Paper/ShowArticle',
          query: {
            id: this.id,
            list: this.$route.query.list,
            document_type: this.document_type
          }
        })
      }
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
      this.time = new Date().Format('yyyy-MM-dd hh:mm:ss')
      console.log(this.time)
    },
    deleteRow(index) {
      this.tableData.splice(index, 1)
      if (this.tableData.length === 0) {
        this.tableShow = false
      }
    },
    // 让table显现出来让card消失
    checkRelation() {
      this.tableShow = true
      this.showCard2 = false
      this.showDot = true
      this.fileMark.relation_marks = []
      this.fileMark.object_marks = []
      this.getTime()
      console.log(this.Entitylabels)
      for (let i = 0; i < this.Entitylabels.length; i++) {
        const obj_entity = {
          object_type: this.Entitylabels[i].labelname,
          objects: []
        }
        this.fileMark.object_marks.push(obj_entity)
      }
      const arr = this.choice.split('-')
      console.log(this.options)
      for (let i = 0; i < this.options.length; i++) {
        const arr2 = this.options[i].label.split('-')
        const obj_rel = {
          relaiton_id: '',
          start_type: arr2[0],
          relation_type: arr2[1],
          end_type: arr2[2],
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
      const obj = {
        start_object: this.labels[0].entitylist[0].text,
        end_object: this.label1[0].entitylist[0].text,
        advice: this.advice,
        evi_level: this.level,
        evi_describe: this.describe,
        reference: '',
        group: this.group,
        time: this.time,
        type: this.choice,
        is_checked: false,
        is_passed: false,
        mark_user_id: '',
        mark_time: this.time,
        check_user_id: '',
        check_time: '',
        is_multiple_marked: false
      }

      for (let i = 0; i < this.fileMark.relation_marks.length; i++) {
        if (this.fileMark.relation_marks[i].start_type === arr[0] && this.fileMark.relation_marks[i].relation_type === arr[1] && this.fileMark.relation_marks[i].end_type === arr[2]) {
          this.fileMark.relation_marks[i].relations.push(obj)
          break
        }
      }

      this.isgroup.push(this.group)
      const url = 'http://localhost:10088/FileMarks/addFileMark'
      const time = this.time
      const document_type = this.document_type
      axios.post(url, this.fileMark).then((response) => {
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

      const url1 = 'http://localhost:10088/reference/modify'
      axios.put(url1, {
        id: this.id,
        category: this.$route.query.document_type
      }).then((response) => {
        console.log(response)
      })
    },
    // 获取后台实体名
    getEntityChioce() {
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
    routeBack() {
      console.log('跳转标记')
      console.log(this.document_type)
      this.$router.push({
        path: '/xieweihao/Document_information_extraction/Paper/ShowArticle',
        query: {
          id: this.id,
          list: this.$route.query.list,
          document_type: this.document_type
        }
      })
    }
  }
}
</script>

<style  scoped>
.components-container {
  position: relative;
  height: 1000px;
  /* height: 100%; */

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
  height: 50em;
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
</style>
