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
      <van-row v-for="item in tableData" :key="item.contentId">
        <div class="box" @click="showTask(item)">
          <div style="display: flex;justify-content: space-between;margin: 10px 0;">
            <div>
              <div>{{item.taskList.length}}条评论</div>
              <div style="color: red;">{{item.evaluateStatus === 1 ? '已点评' : ''}}</div>
            </div>
            <div>{{dateStr(item.createdTime)}} </div>
          </div>
          <div>
            {{item.simpleContent.text}}
          </div>
          <div>
            <span v-for="(i, index) in item.simpleContent.images" :key="i.src">
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
      <div v-if="!taskList.length" style="text-align: center;margin: 10px;">暂无评论</div>
      <div v-for="task in taskList" :key="task.id">
        <div class="box" style="width: calc(100% - 20px);">
        <div style="display: flex;align-items: center;">
          <div>
            <img :src="task.userInfo.avatar" style="width: 60px;height: 60px;margin: 5px;">
          </div>
          <div>
            <div style="color: red;">{{task.evaluateStatus === 1 ? '点评' : ''}}</div>
            <div>{{task.userInfo.nickName}}<span v-show="task.parentUserInfo" style="margin: 0 10px;">回复</span>{{task.parentUserInfo ? `${task.parentUserInfo.nickName}` : ''}}</div>
            <div>{{dateStr(task.createdTime)}}</div>
          </div>
        </div>
        <div>
          {{ task.content.text }}
        </div>
      </div>
      </div>
    </van-popup>
    <van-popup v-model:show="loading" style="width: 100vw;height: 100vh;max-width: 100vw;">
      <van-loading type="spinner" color="#1989fa" style="top: 50%; left: 50%; transform: translate(-50%, -50%); width: 100px; text-align: center;" />
    </van-popup>
  </div>
</template>

<script>
import axios from 'axios'
import { showToast } from 'vant'
import 'vant/es/toast/style'

const columns = [
  { id: 'a0521', name: '一隻柳【0521】', value: '0521'},
  { id: 'a1250', name: '这是我家阿刃1250', value: '1250'},
  { id: 'a1136', name: '南风1136', value: '1136'},
  { id: 'a1087', name: '风起 1087', value: '1087'},
  { id: 'a1077', name: '曹1077', value: '1077'},
  { id: 'a1140', name: '知临1140', value: '1140'},
  { id: 'a1067', name: '子静 1067', value: '1067'},
  { id: 'a0999', name: '小影子【999】', value: '0999'},
  { id: 'a0785', name: '正己化人0785', value: '0785'},
  { id: 'a0115', name: '筱杭（0115）', value: '0115'},
  { id: 'a0983', name: '悟空0983', value: '0983'},
  { id: 'a0724', name: '知追0724', value: '0724'},
  { id: 'a0225', name: 'L&LH0225', value: '0225'},
  { id: 'a0801', name: 'Qin韵【0801】', value: '0801'},
  { id: 'a0779', name: '2毛毛毛【0779】', value: '0779'},
  { id: 'a0579', name: '风中吃西瓜579', value: '0579'},
  { id: 'a0503', name: '讲文明院长【0503】', value: '0503'},
  { id: 'a0893', name: '大风机关 【0893】', value: '0893'},
  { id: 'a0585', name: '乏善可陈0585', value: '0585'},
  { id: 'a0696', name: '张小旭【0696】', value: '0696'},
  { id: 'acqx', name: '陈秋杏', value: 'cqx'},
  { id: 'a0138', name: '饮川【0138】', value: '0138'},
  { id: 'a0485', name: '路十一0485', value: '0485'},
  { id: 'a0464', name: '太阳当空赵【0464】', value: '0464'},
  { id: 'a0264', name: '0264小毛', value: '0264'},
  { id: 'a0439', name: '谢喻生【439】', value: '0439'},
  { id: 'a0530', name: '東南隅【0530】', value: '0530'},
  { id: 'a0578', name: '小王【0578】', value: '0578'},
  { id: 'alywz', name: '乐也屋子', value: 'lywz'},
  { id: 'a0830', name: '兔斯基爱写字0830', value: '0830'},
  { id: 'axxx', name: '小星星', value: 'xxx'},
  { id: 'a0087', name: '而归【0087】', value: '0087'},
  { id: 'a0855', name: 'WJW【0855】', value: '0855'},
  { id: 'a0069', name: '今天航海【0069】', value: '0069'},
]

const imgRpl = (src) => {
  return false 
    ? `${src.replace('https://cc.hjfile.cn/cc', '').replace('https://i2n.hjfile.cn', '').replace('https://thirdwx.qlogo.cn/mmopen/vi_32/POgEwh4mIHO4nibH0KlMECNjjGxQUq24ZEaGT4poC6icRiccVGKSyXwibcPq4BWmiaIGuG1icwxaQX6grC9VemZoJ8rg/132', '/u/132.jpg')}`
    : src
}

export default {
  data() {
    return { 
      allData: {},
      tableData: [],
      allTableData: [],
      headInfo: {},
      show: false,
      index: 0,
      images: [],
      pageConfig: {
        page: 1,
        pageSize: 5
      },
      columns,
      customFieldName: {
        text: 'name',
        value: 'value'
      },
      showPicker: false,
      showTaskPicker: false,
      taskList: [],
      loading: false
    }
  },
  computed: {
    dateStr() {
      return function(date) {
        return date.replace('T', ' ').replace('+0800', '')
      }
    }
  },
  async created() {
    await this.getData('0521')
  },
  methods: {
    async getData(id) {
      this.loading = true
      await axios.get(`/data/${id}.json`).then(async res => {
        this.allData = res.data.data
        this.headInfo = this.allData.dakaUserInfo
        this.allTableData = this.allData.items
        this.headInfo.avatar = imgRpl(this.headInfo.avatar)
        this.allTableData.forEach(i => {
          i.simpleContent = JSON.parse(i.simpleContent)
          i.simpleContent.images.forEach(j => {
            j.src = imgRpl(j.src)
          })
          i.taskList = []
          i.evaluateStatus = 0
        })
        this.pageConfig.page = 1
        this.pageConfig.pageSize = 5
        this.tableData = this.allTableData.slice((this.pageConfig.page - 1)*this.pageConfig.pageSize, this.pageConfig.pageSize)
        await this.getCurrData()
      }).finally(() => {
        this.loading = false
      })
    },
    async getCurrData() {
        for (let item of this.tableData) {
          const {contentId, taskId} = item
          await axios.get(`/task/${contentId}_${taskId}.json`).then(res => {
            item.taskList = res.data.data.items
            item.taskList.forEach(i => {
              i.content = JSON.parse(i.content)
              i.userInfo.avatar = imgRpl(i.userInfo.avatar)
            })
            if (item.taskList.some(x => x.evaluateStatus === 1)) {
              item.evaluateStatus = 1
            }
          })
        }
    },
    onChange(index) {
      this.index = index;
    },
    showTask({taskList}) {
      this.taskList = taskList
      if (this.taskList.length) {
        this.showTaskPicker = true
      } else {
        showToast('暂无评论')
      }
      // this.loading = true
      // axios.get(`/task/${contentId}_${taskId}.json`).then(res => {
      //   this.taskList = res.data.data.items
      //   this.taskList.forEach(i => {
      //     i.content = JSON.parse(i.content)
      //     i.userInfo.avatar = imgRpl(i.userInfo.avatar)
      //   })
      //   this.showTaskPicker = true
      // }).finally(() => {
      //   this.loading = false
      // })
    },
    showImagePreview(images, index) {
      this.index = index
      this.images = images.map(i => i.src)
      this.show = true
    },
    async pageChange(e) {
      this.pageChange.page = e
      this.tableData = this.allTableData.slice((this.pageConfig.page - 1)*this.pageConfig.pageSize, this.pageConfig.page*this.pageConfig.pageSize)
      await this.getCurrData()
    },
    onConfirm ({ selectedOptions }) {
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
