<style lang="less">
@import url('../../assets/styles/variable.less');

.page-container { height: auto; padding: 20rpx 35rpx; background:#eee; overflow-x: hidden; overflow-y: scroll; -webkit-overflow-scrolling: touch;
  .page-title { height: 150rpx; padding-top: 20rpx; font-weight: bold; font-size: 46rpx; margin-bottom: 0rpx; padding-left: 2.5%; background: @wx-red; color: #fefefe; position: relative;
    .en { font-weight: normal; font-size: 28rpx; }
    .float-btn { width: 150rpx; height: 52rpx; line-height: 50rpx; text-align: center; position: absolute; top: 20rpx; right: 20rpx; font-weight: normal; font-size: 28rpx; background: rgba(0, 0, 0, .5); font-size: 26rpx; color: #fff; border-radius: 0; }
    .share-btn { top: 82rpx; }
  }
  .wiki-content { display: block; width: 100%; height: auto; margin-top: -26rpx; margin-bottom: 30rpx; overflow-x: hidden; }

  .scroll-to-top { height: 90rpx; width: 90rpx; position: fixed; bottom: 50rpx; right: 30rpx; background: rgba(0,0,0,.5);  border-radius: 50%;
    img { display: block; width: 60%; height: 60%; margin: 20rpx auto 0; }
  }
}
</style>

<template>
  <div class="page-container page-city-detail" v-show="wikiDataStatus == 'SUCCESS'" >
    <div class="page-title">
      <p>欢迎来到{{ cityInstance.name_zh }}</p>
      <p class="en">Welcome to {{ cityInstance.name_en }}</p>
      <button class="float-btn change-city-btn" @tap="goBackHome">切换城市</button>
      <button class="float-btn share-btn" open-type="share">分享好友</button>
    </div>

    <p style="font-size:22rpx;">以下内容来自维基导游(https://zh.wikivoyage.org)，仅供查阅参考</p>

    <rich-text class="wiki-content" v-html="cityWikiHtml"></rich-text>

    <p style="font-size:30rpx; color: #d84e43; margin-bottom: 20rpx;" >如对此词条感兴趣，可前往维基导游(https://zh.wikivoyage.org/{{ cityInstance.name_zh }})进行查看和编辑</p>

    <div class="scroll-to-top">
      <img :src="scrollToTopIcon" @tap="scrollToTop" />
    </div>
  </div>
</template>

<script>
import config from '@/config'

export default {
  name: 'CityWiki',
  props: [],
  mixins: [],
  data() {
    return {
      scrollToTopIcon: require('../../assets/images/gotop.png'),
      allCities: config.allCities || [],
      cityId: '',
      cityName: '',
      cityWikiHtml: '',
      wikiDataStatus: 'INIT'
    }
  },
  computed: {
    cityInstance() {
      let instance = {}
      this.allCities.forEach(c => {
        if (c.id == this.cityId && c.name_zh == this.cityName) {
          instance = c
        }
      })
      return instance
    }
  },

  onShareAppMessage (options) {
    var that = this
    return {
      title: `我在${this.cityName}等你，你呢？`,
      path: `pages/citywiki/main?id=${this.cityId}&name=${this.cityName}`
    }
  },
  // 获取url中的query对象，包含城市id及name
  onLoad(query) {
    console.log(query);
    this.cityId = query.id || ''
    this.cityName = query.name || ''
    this.wikiDataStatus = 'INIT'
  },

  mounted() {
    this.fetchWikiDataFromGitee()
  },
  methods: {
    goBackHome() {
      wx.reLaunch({
        url: "/pages/citylist/main"
      })
      // wx.navigateBack({
      //   delta: 2
      // })
    },
    fetchWikiDataFromGitee() {
      wx.showLoading({
        title: '数据获取中',
        mask: true
      })
      this.cityWikiHtml = ''
      this.wikiDataStatus = 'LOADING'
      const Gitee_URL = 'https://gitee.com/zhoujihao/public_data/raw/master/wikivoyage'
      const wikiNameMap = {
        '东京': '東京',
        '纽约': '紐約',
        '首尔': '首爾',
        '台北': '臺北'
      }
      let queryName = this.cityInstance.name_zh
      queryName = wikiNameMap[queryName] || queryName
      wx.request({
        url: `${Gitee_URL}/${queryName}.html`,
        method: 'GET',
        success: (res) => {
          if (res && res.data) {
            let htmlContent = res.data
            this.cityWikiHtml = htmlContent
            this.wikiDataStatus = 'SUCCESS'
            setTimeout(() => {
              wx.hideLoading()
            }, 100)
          } else {
            this.wikiDataStatus = 'ERROR'
            wx.showToast({
              title: '网络异常，请稍后再试',
              icon: 'none',
              duration: 2000
            })
          }
        },
        fail: () => {
          this.wikiDataStatus = 'ERROR'
          wx.showToast({
            title: '网络异常，请稍后再试',
            icon: 'none',
            duration: 2000
          })
        }
      })
    },
    //回到顶部
    scrollToTop() {  // 一键回到顶部
      if (wx.pageScrollTo) {
        wx.pageScrollTo({
          scrollTop: 0,
          duration: 300
        })
      } else {
        wx.showModal({
          title: '提示',
          content: '当前微信版本过低，无法使用该功能，请升级到最新微信版本后重试。'
        })
      }
    },
  },
  watch: {},
  components: {}
}
</script>
