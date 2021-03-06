<template>
  <div class="app-container">
    <!-- 顶部通知栏 -->
    <div v-if="!paintingInfo.status" class="phone-notice">
      <van-notice-bar
        :color="Color.colorbrand"
        :speed="50"
        background="rgba(244,205,205,1)"
        left-icon="volume-o"
        :scrollable="true"
      >
        美团下单享受优惠！详情联系我们。
      </van-notice-bar>
    </div>

    <!-- 轮播图 -->
    <carousel v-scroll-reveal.scaleUp="{ scale: 0.15 }" :list="carouselList" />

    <!-- 招生详情 -->
    <!-- <recruitment v-scroll-reveal.scaleUp="{ scale: 0.15 }" /> -->

    <!-- 介绍 -->
    <paintIntroduce v-scroll-reveal.smooth="{ easing: 'ease-in' }" />

    <!-- 动态 -->
    <paintAffaris
      v-scroll-reveal.smooth="{ easing: 'ease-in' }"
      :article-list="articleList"
    />
    <!-- 画室环境 -->
    <paintingEnvironment
      v-scroll-reveal.smooth="{ easing: 'ease-in' }"
      :list="environmentList"
    />

    <!-- 师资力量 -->
    <teachers
      v-scroll-reveal.smooth="{ easing: 'ease-in' }"
      :list="teacherList"
    />

    <!-- 学生作品 -->
    <worksCarousel
      v-scroll-reveal.smooth="{ easing: 'ease-in' }"
      :student-works-list="studentWorksList"
    />

    <!-- 优势 -->
    <advantage v-scroll-reveal.smooth="{ easing: 'ease-in' }" />

    <!-- 底部菜单栏 -->
    <div v-if="!paintingInfo.status">
      <message-board v-scroll-reveal.smooth="{ easing: 'ease-in' }" />
    </div>

    <!-- 底部footer -->
    <bottom-footer :painting-info="paintingInfo"></bottom-footer>
  </div>
</template>
<script>
import { mapMutations } from 'vuex'
import carousel from '../components/works/carousel'
import advantage from '../components/works/advantage'
import teachers from '../components/works/teachers'
import paintIntroduce from '../components/index/paintIntroduce'
import paintAffaris from '../components/index/paintAffaris'
import worksCarousel from '../components/index/worksCarousel'
import paintingEnvironment from '../components/index/paintingEnvironment'
import BottomFooter from '../components/common/bottomFooter'
import { Color } from '../config/color'
import { Api } from '../api/index'
import MessageBoard from '@/components/common/messageBoard'
export default {
  head: {
    title: '品贤画室 | For Art,For U'
  },
  components: {
    MessageBoard,
    carousel,
    paintIntroduce,
    advantage,
    paintAffaris,
    paintingEnvironment,
    teachers,
    worksCarousel,
    BottomFooter
  },
  data() {
    return {
      Color
    }
  },
  /**
   * 获取服务端渲染数据
   */
  async asyncData({ $axios }) {
    // 判断是否在服务端

    // 如果是在首次加载首页，则从服务器读取数据。

    // 获取画室信息
    if (process.server) {
      const paintingInfo = await $axios({
        method: 'post',
        url: Api.getPaintingInfo
      })
      // 获取前台轮播图
      const bannerList = await $axios.get(Api.courselBannerList)
      // 获取首页画室动态列表
      const studentWorksList = await $axios({
        method: 'get',
        url: Api.getStudentWorksList,
        data: { start: 0, end: 30 }
      })
      // 获取首页文章列表
      const articleList = await $axios({
        method: 'post',
        url: Api.getArticleList,
        data: { pageSize: 6, pageNum: 1 }
      })
      // 获取画室环境列表
      const environmentList = await $axios({
        method: 'get',
        url: Api.environmentList
      })
      // 获取教师列表
      const teacherList = await $axios({ method: 'get', url: Api.teacherList })

      return {
        carouselList: bannerList.data.data,
        paintingInfo: paintingInfo.data.data[0],
        articleList: articleList.data.data,
        studentWorksList: studentWorksList.data.data,
        environmentList: environmentList.data.data,
        teacherList: teacherList.data.data
      }
    }
    // 如果是在客户端刷新首页，则从缓存里读取数据。
    if (process.client) {
      const teacherList = JSON.parse(localStorage.getItem('teacherList'))
      const carouselList = JSON.parse(localStorage.getItem('bannerList'))
      const studentWorksList = JSON.parse(
        localStorage.getItem('studentWorksList')
      )
      const articleList = JSON.parse(localStorage.getItem('articleList'))
      const environmentList = JSON.parse(
        localStorage.getItem('environmentList')
      )
      const paintingInfo = JSON.parse(localStorage.getItem('paintingInfo'))
      return {
        carouselList,
        paintingInfo,
        articleList,
        studentWorksList,
        environmentList,
        teacherList
      }
    }
  },
  created() {
    if (process.client) {
      localStorage.setItem('paintingInfo', JSON.stringify(this.paintingInfo))
      this.setPaintingInfo(this.paintingInfo)
      const token = localStorage.getItem('token')
      if (token) {
        const user = JSON.parse(localStorage.getItem('user'))
        this.changeUser(user)
        this.changeIsLogin(true)
      } else {
        this.changeUser(null)
        this.changeIsLogin(false)
      }
    }
  },
  mounted() {
    // 不论是从哪里获取的数据，都再次存一遍。
    if (this.teacherList) {
      localStorage.setItem('teacherList', JSON.stringify(this.teacherList))
      localStorage.setItem('bannerList', JSON.stringify(this.carouselList))
      localStorage.setItem(
        'studentWorksList',
        JSON.stringify(this.studentWorksList)
      )
      localStorage.setItem('articleList', JSON.stringify(this.articleList))
      localStorage.setItem(
        'environmentList',
        JSON.stringify(this.environmentList)
      )
      localStorage.setItem('paintingInfo', JSON.stringify(this.paintingInfo))
    }
    this.changeTabbarShow(true)
    this.changTopbarShow(true)
  },
  methods: {
    ...mapMutations([
      'setPaintingInfo',
      'changeTabbarShow',
      'changTopbarShow',
      'changeUser',
      'changeIsLogin'
    ])
  }
}
</script>

<style lang="less" scoped>
.app-container {
  background: #fff;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}

@media screen and(min-width: 720px) {
  .phone-notice {
    display: none;
  }
}
</style>
