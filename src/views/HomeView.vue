<template>
  <div class="HomeView">
    <div class="box" style="width: calc(100% - 20px);" @click="showPicker = true">
      <div style="display: flex;align-items: center;">
        <div>
          <img :src="headInfo.avatar" style="width: 60px;height: 60px;margin: 5px;">
        </div>
        <div>{{headInfo.nickName}}</div>
      </div>
      
    </div>
    <div>
      <van-row v-for="item in tableData">
        <div class="box" @click="showTask(item)">
          <div>{{item.createdTime}}</div>
          <div>
            {{item.simpleContent.text}}
          </div>
          <div>
            <span v-for="(i, index) in item.simpleContent.images">
              <img :src="i.src" style="width: 100px;height: 100px;margin: 5px;" @click.stop="showImagePreview(item.simpleContent.images, index)">
            </span>
          </div>
        </div>
      </van-row>
    </div>
    <van-image-preview v-model:show="show" :images="images" :startPosition="index" @change="onChange">
      <template v-slot:index>第{{ index + 1 }}页</template>
    </van-image-preview>
    <van-pagination v-model="pageConfig.page" :total-items="allData.total" :items-per-page="pageConfig.pageSize" @change="pageChange" />
    <van-popup v-model:show="showPicker" round position="bottom">
      <van-picker :columns="columns" :columns-field-names="customFieldName" @confirm="onConfirm" @cancel="showPicker = false"/>
    </van-popup>
    <van-popup v-model:show="showTaskPicker" round position="bottom" :style="{ height: '80%' }">
      <div v-for="task in taskList">
        <div class="box" style="width: calc(100% - 20px);">
        <div style="display: flex;align-items: center;">
          <div>
            <img :src="task.userInfo.avatar" style="width: 60px;height: 60px;margin: 5px;">
          </div>
          <div>
            <div>{{task.userInfo.nickName}}{{task.parentUserInfo ? `回复${task.parentUserInfo.nickName}` : ''}}</div>
            <div>{{task.createdTime}}</div>
          </div>
        </div>
        <div>
          {{ task.content.text }}
        </div>
      </div>
      </div>
    </van-popup>
  </div>
</template>

<script>
import axios from 'axios'

// import a0779 from '../data/0779.json'
// import a0264 from '../data/0264.json'
// import a1077 from '../data/1077.json'
// import acqx from '../data/cqx.json'
// import a0893 from '../data/0893.json'
// import a0530 from '../data/0530.json'
// import a0087 from '../data/0087.json'
// import a1087 from '../data/1087.json'
// import a0579 from '../data/0579.json'
// import a0503 from '../data/0503.json'
// import a0069 from '../data/0069.json'
// import alywz from '../data/lywz.json'
// import a0485 from '../data/0485.json'
// import a1136 from '../data/1136.json'
// import a0464 from '../data/0464.json'
// import a0830 from '../data/0830.json'
// import a0983 from '../data/0983.json'
// import a0578 from '../data/0578.json'
// import axxx from '../data/xxx.json'
// import a0999 from '../data/0999.json'
// import a0115 from '../data/0115.json'
// import a0439 from '../data/0439.json'
// import a0521 from '../data/0521.json'
// import a0138 from '../data/0138.json'
// import a0696 from '../data/0696.json'
// import a1250 from '../data/1250.json'
// import a1067 from '../data/1067.json'

const columns = [
  { id: 'a0779', name: '2毛毛毛【0779】', value: '0779'},
  { id: 'a0264', name: '0264小毛', value: '0264'},
  { id: 'a1077', name: '曹1077', value: '1077'},
  { id: 'acqx', name: '陈秋杏', value: 'cqx'},
  { id: 'a0893', name: '大风机关 【0893】', value: '0893'},
  { id: 'a0530', name: '東南隅【0530】', value: '0530'},
  { id: 'a0087', name: '而归【0087】', value: '0087'},
  { id: 'a1087', name: '风起 1087', value: '1087'},
  { id: 'a0579', name: '风中吃西瓜579', value: '0579'},
  { id: 'a0503', name: '讲文明院长【0503】', value: '0503'},
  { id: 'a0069', name: '今天航海【0069】', value: '0069'},
  { id: 'alywz', name: '乐也屋子', value: 'lywz'},
  { id: 'a0485', name: '路十一0485', value: '0485'},
  { id: 'a1136', name: '南风1136', value: '1136'},
  { id: 'a0464', name: '太阳当空赵【0464】', value: '0464'},
  { id: 'a0830', name: '兔斯基爱写字0830', value: '0830'},
  { id: 'a0983', name: '悟空0983', value: '0983'},
  { id: 'a0578', name: '小王【0578】', value: '0578'},
  { id: 'axxx', name: '小星星', value: 'xxx'},
  { id: 'a0999', name: '小影子【999】', value: '0999'},
  { id: 'a0115', name: '筱杭（0115）', value: '0115'},
  { id: 'a0439', name: '谢喻生【439】', value: '0439'},
  { id: 'a0521', name: '一隻柳【0521】', value: '0521'},
  { id: 'a0138', name: '饮川【0138】', value: '0138'},
  { id: 'a0696', name: '张小旭【0696】', value: '0696'},
  { id: 'a1250', name: '这是我家阿刃1250', value: '1250'},
  { id: 'a1067', name: '子静 1067', value: '1067'},
]

export default {
  data() {
    return { 
      // originData: {
      //   a0779,
      //   a0264,
      //   a1077,
      //   acqx,
      //   a0893,
      //   a0530,
      //   a0087,
      //   a1087,
      //   a0579,
      //   a0503,
      //   a0069,
      //   alywz,
      //   a0485,
      //   a1136,
      //   a0464,
      //   a0830,
      //   a0983,
      //   a0578,
      //   axxx,
      //   a0999,
      //   a0115,
      //   a0439,
      //   a0521,
      //   a0138,
      //   a0696,
      //   a1250,
      //   a1067,
      // },
      allData: {},
      tableData: [],
      allTableData: [],
      headInfo: {},
      show: false,
      index: 0,
      images: [],
      pageConfig: {
        page: 1,
        pageSize: 10
      },
      columns,
      customFieldName: {
        text: 'name',
        value: 'value'
      },
      showPicker: false,
      showTaskPicker: false,
      taskList: []
    }
  },
  created() {
    this.getData('0069')
  },
  methods: {
    getData(id) {
      axios.get(`/data/${id}.json`).then(res => {
        console.log('getData', res);
        this.allData = res.data.data
        this.headInfo = this.allData.dakaUserInfo
        this.allTableData = this.allData.items
        this.allTableData.forEach(i => {
          i.simpleContent = JSON.parse(i.simpleContent)
        })
        this.pageConfig.page = 1
        this.pageConfig.pageSize = 10
        this.tableData = this.allTableData.slice((this.pageConfig.page - 1)*this.pageConfig.pageSize, this.pageConfig.pageSize)
      })
    },
    onChange(index) {
      console.log('index ---',index);
      this.index = index;
    },
    showTask({contentId, taskId}) {
      axios.get(`/task/${contentId}_${taskId}.json`).then(res => {
        console.log(res);
        this.taskList = res.data.data.items
        this.taskList.forEach(i => {
          i.content = JSON.parse(i.content)
        })
        this.showTaskPicker = true
      })
    },
    showImagePreview(images, index) {
      this.index = index
      this.images = images.map(i => i.src)
      this.show = true
    },
    pageChange(e) {
      this.pageChange.page = e
      console.log('this.pageChange.page', this.pageChange.page);
      this.tableData = this.allTableData.slice((this.pageConfig.page - 1)*this.pageConfig.pageSize, this.pageConfig.page*this.pageConfig.pageSize)
    },
    onConfirm ({ selectedOptions }) {
      console.log('selectedOptions', selectedOptions);
      this.getData(selectedOptions[0].value)
      this.showPicker = false;
    }
  }
}
</script>

<style>
.HomeView {padding: 10px;}
.box { border: 1px solid rgba(0, 0, 0, 0.1); margin: 10px; padding: 10px; border-radius: 10px; width: 100%; box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1) }
</style>
