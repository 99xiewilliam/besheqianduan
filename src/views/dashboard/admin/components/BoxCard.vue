<template>
  <el-card class="box-card-component" style="margin-left:8px;">
    <div slot="header" class="box-card-header">
      <img src="./covid-5247929_640.png">
    </div>
    <div style="position:relative;">
      <!-- <pan-thumb :image="avatar" class="panThumb" /> -->
      <mallki class-name="mallki-text" text="各种类已标注情况" />
      <!-- <el-scrollbar style="height:100%" wrap-style="overflow-x:hidden;"> -->
        <div style="padding-top:10px;" class="progress-item" v-for="(item, index) in options" :key="index">
          <span>{{item.name}}</span>
          <el-progress :percentage="item.percentage" />
        </div>
      <!-- </el-scrollbar> -->
      <!-- <div class="progress-item">
        <span>JavaScript</span>
        <el-progress :percentage="18" />
      </div>
      <div class="progress-item">
        <span>CSS</span>
        <el-progress :percentage="12" />
      </div> -->
      <div class="progress-item" style="padding-top:30px;">
        <span>已标注总情况</span>
        <el-progress :percentage="66" />
      </div>
    </div>
  </el-card>
</template>

<script>
import { mapGetters } from 'vuex'
import PanThumb from '@/components/PanThumb'
import Mallki from '@/components/TextHoverEffect/Mallki'

export default {
  components: { PanThumb, Mallki },

  filters: {
    statusFilter(status) {
      const statusMap = {
        success: 'success',
        pending: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      statisticsData: {
        article_count: 1024,
        pageviews_count: 1024
      },
      options:[
        {
          name: '文献',
          percentage: 40
        },
        {
          name: '医学书籍',
          percentage: 40
        },
        {
          name: '指南',
          percentage: 40
        },
        {
          name: '药品说明书',
          percentage: 40
        },
        {
          name: '诊断报告',
          percentage: 40
        },
        {
          name: '文本',
          percentage: 20
        }
      ]
    }
  },
  computed: {
    ...mapGetters([
      'name',
      'avatar',
      'roles'
    ])
  }
}
</script>

<style lang="scss" >
.box-card-component{
  .el-card__header {
    padding: 0px!important;
  }
}
</style>
<style lang="scss" scoped>
.box-card-component {
  .box-card-header {
    position: relative;
    height: 220px;
    img {
      width: 100%;
      height: 100%;
      transition: all 0.2s linear;
      &:hover {
        transform: scale(1.1, 1.1);
        filter: contrast(130%);
      }
    }
  }
  .mallki-text {
    position: absolute;
    top: 0px;
    right: 0px;
    font-size: 20px;
    font-weight: bold;
  }
  .panThumb {
    z-index: 100;
    height: 70px!important;
    width: 70px!important;
    position: absolute!important;
    top: -45px;
    left: 0px;
    border: 5px solid #ffffff;
    background-color: #fff;
    margin: auto;
    box-shadow: none!important;
    ::v-deep .pan-info {
      box-shadow: none!important;
    }
  }
  .progress-item {
    margin-bottom: 10px;
    font-size: 14px;
  }
  @media only screen and (max-width: 1510px){
    .mallki-text{
      display: none;
    }
  }
}
</style>
