<template>
  <div class="main">
    <top title="m-bookstore"></top>
    <div class="container" @scroll="containerScroll" v-el:container>
      <section class="main-card">
      <div class="search-box">输入书名/作者/关键字</div>
      <div class="slider-banner">
        <ul class="slider-wrap">
          <li class="item">
            <img src="http://image.read.duokan.com/mfsv2/download/fdsc3/p01qDSHWUGqf/DoSt6VIaNe0oC2.jpg"/>
          </li>
        </ul>
      </div>
      <ul class="nav-bar">
        <li class="item"><a class="free" v-link="{ path: '/free' }">免费</a></li>
        <li class="item"><a class="boy" v-link="{ path: '/boy' }">男生</a></li>
        <li class="item"><a class="girl" v-link="{ path: '/girl' }">女生</a></li>
        <li class="item"><a class="category" v-link="{ path: '/category' }">分类</a></li>
      </ul>
      </section><!-- 顶部轮播图 -->
      <section class="main-card" v-if="hot">
        <group-header :title="hot.title"></group-header>
        <book-table :books="hot.data"></book-table>
        <group-footer :text="hot.info.more_text"></group-footer>
      </section><!-- 本周最火 -->
      <section class="main-card" v-if="recommend">
        <header class="h5-header">
          <h2 class="title" v-text="recommend.title"></h2>
          <div class="recommend-tabs">
            <a class="tab" :class="{'on': recommend.type === 'boy'}" @click.prevent="switchType('boy')">男</a>
            <a class="tab" :class="{'on': recommend.type === 'girl'}" @click.prevent="switchType('girl')">女</a>
          </div>
        </header>
        <ul class="h5-book-list">
          <template v-for="book in recommendBooks">
            <h5-book v-if="$index === 0" :book="book"></h5-book>
            <li class="h5-book" v-else>
              <a class="info" v-link="{ name: 'book', params: { fiction_id: book.fiction_id}}">
                <span class="index" v-text="'0' + ($index+1)"></span>
                <span class="title" v-text="book.title"></span>
                <span class="author" v-text="book.authors"></span>
              </a>
            </li>
          </template>
        </ul>
        <footer class="h5-footer">
          <a class="next" @click.prevent="getNextBooks(recommend[recommend.type])">换一换</a>
          <a class="more" v-link="{ path: recommend.type === 'girl' ? '/girl' : '/boy' }" >查看全部</a>
        </footer>
      </section><!-- 重磅推荐 -->
      <section class="main-card" v-if="girl">
        <header class="h5-header">
          <h2 class="title" v-text="girl.title"></h2>
        </header>
        <ul class="h5-book-list">
          <h5-book v-for="book in girlBooks" :book="book"></h5-book>
        </ul>
        <footer class="h5-footer">
          <a class="next" @click.prevent="getNextBooks(girl)">换一换</a>
          <a class="more" v-link="{ path: '/girl' }" v-text="girl.info.more_text"></a>
        </footer>
      </section><!-- 女生最爱 -->
      <section class="main-card" v-if="boy">
        <header class="h5-header">
          <h2 class="title" v-text="boy.title"></h2>
        </header>
        <ul class="h5-book-list">
          <h5-book v-for="book in boyBooks" :book="book"></h5-book>
        </ul>
        <footer class="h5-footer">
          <a class="next" @click.prevent="getNextBooks(boy)">换一换</a>
          <a class="more" v-link="{ path: '/boy' }" v-text="boy.info.more_text"></a>
        </footer>
      </section><!-- 男生最爱 -->
      <section class="main-card" v-if="free">
        <group-header :title="free.title"></group-header>
        <book-table :books="free.data"></book-table>
        <group-footer :text="free.info.more_text"></group-footer>
      </section><!-- 限时免费 -->
      <section class="main-card" v-if="topic">
        <header class="h5-header">
          <h2 class="title" v-text="topic.title"></h2>
        </header>
        <ul class="banner-list">
          <li class="banner" v-for="banner in topic.data">
            <a>
              <img class="cover" :src="banner.ad_pic_url" :alt="banner.ad_name" />
            </a>
          </li>
        </ul>
        <footer class="h5-footer">
          <a class="more" v-text="topic.info.more_text"></a>
        </footer>
      </section><!-- 精选专题 -->
      <section class="main-card" v-if="waterfallBooks.length">
        <ul class="h5-book-list">
          <h5-book v-for="book in waterfallBooks" :book="book" track-by="$index"></h5-book>
        </ul>
      </section><!-- 瀑布流 -->
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import { getApiData, parseHiddenInfo } from 'common/js/utils.js'
  import Top from 'components/common/top/top'
  import GroupHeader from 'components/common/group-header/group-header'
  import BookTable from 'components/common/book-table/book-table'
  import GroupFooter from 'components/common/group-footer/group-footer'
  import H5Book from 'components/common/h5-book/h5-book'

  export default {
    components: {
      Top,
      GroupHeader,
      BookTable,
      GroupFooter,
      H5Book
    },
    computed: {
      recommendBooks () {
        let type = this.recommend[this.recommend.type]
        return type.data.filter((item, idx) => idx >= type.start && idx < type.start + 5)
      },
      girlBooks () {
        return this.girl.data.filter((item, idx) => idx >= this.girl.start && idx < this.girl.start + 5)
      },
      boyBooks () {
        return this.boy.data.filter((item, idx) => idx >= this.boy.start && idx < this.boy.start + 5)
      }
    },
    methods: {
      getNextBooks (obj) {
        obj.start = (obj.start + 5) % obj.data.length
      },
      switchType (type) {
        this.recommend.type = type
      },
      addWaterfallBooks () {
        let start = this.waterfallBooks.length
        getApiData(`/api/recommend?strat=${start}`, data => {
          if (start === this.waterfallBooks.length) {
            this.waterfallBooks.push(...data.items)
          }
        })
      },
      containerScroll () {
        if (!this.timer) {
          this.timer = setTimeout(() => {
            clearTimeout(this.timer)
            this.timer = null
            let el = this.$els.container
            if (el.scrollHeight - el.clientHeight - el.scrollTop < el.clientHeight * 0.5) {
              this.addWaterfallBooks()
            }
          }, 500)
        }
      }
    },
    data () {
      return {
        items: null,
        top: null,
        hot: null,
        recommend: null,
        girl: null,
        boy: null,
        free: null,
        topic: null,
        waterfallBooks: [],
        timer: null
      }
    },
    ready () {
      getApiData('/api/channel/418', data => {
        this.items = data
        ;['top', 'hot', 'recommend', 'girl', 'boy', 'free', 'topic'].map((attr, idx) => {
          let obj = {
            title: data.items[idx].ad_name,
            info: parseHiddenInfo(data.items[idx].hidden_info),
            data: data.items[idx].data.data
          }
          if (attr === 'boy' || attr === 'girl') {
            obj.start = 0
          } else if (attr === 'recommend') {
            obj.type = 'boy'
            obj.boy = {
              start: 0,
              data: data.items[idx].data.data.slice(0, 15)
            }
            obj.girl = {
              start: 0,
              data: data.items[idx].data.data.slice(15)
            }
            delete obj.data
          } else if (attr === 'free') {
            obj.data = data.items[idx].data.data.map(item => item.data)
          }
          this.$set(attr, obj)
        })
      })
    }
  }
</script>

<style lang="stylus" scoped>
  .search-box
    padding 1px 0 0 30px
    border 1px solid #f5f5f5
    border-radius 2px
    margin 10px 13px
    font-size 11px
    line-height 31px
    color rgba(0, 0, 0, .4)
    background url('search.png') no-repeat 8px center
    background-size 14px
  .slider-wrap
    width 100%
    .item
      img
        width 100%
  .nav-bar
    display flex
    width 100%
    .item
      flex 1
      font-size 12px
      line-height 3.4
      text-align center
    for pic in free boy girl category
      .{pic}
        display block
        padding-top 25px
        background url(s('%s.png', pic)) no-repeat center 10px
        background-size 22px
  .book-table
    padding 13px 14px 5px

  .h5-header
    position relative
    padding 15px 13px 14px 13px
    border-bottom 1px solid #f0f0f0
    font-size 13px
    line-height 13px
    .title
      font-weight bold
      color rgba(0, 0, 0, .9)
    .recommend-tabs
      position absolute
      top 15px
      right 13px
      color #999
      .tab
        padding 15px 8px 14px 8px
        &:first-child:after
          content ''
          position absolute
          display inline-block
          height 12px
          margin-left 8px
          border-right 1px solid #ccc
        &.on
          color #528ae8
  .h5-book-list
    padding 0 13px
  .h5-book
    padding 17px 0
    border-bottom 1px solid #f0f0f0
    &:last-child
      border-bottom none
    .index
      font-size 13px
      line-height 13px
      color rgba(237, 83, 15, .9)
    .title
      font-size 15px
      line-height 15px
      color rgba(0, 0, 0, .9)
    .author
      margin-left 13px
      font-size 12px
      line-height 12px
      color rgba(0, 0, 0, .4)
  .h5-footer
    border-top 1px solid #f0f0f0
    display flex
    .next
    .more
      flex 1
      padding 14px
      font-size 13px
      line-height 1.3em
      color rgba(0, 0, 0, .9)
      text-align center
    .more
      border-left 1px solid #f0f0f0
  .banner-list
    padding 13px
    font-size 0
    .banner
      display inline-block
      width 50%
      padding 0 5px
      box-sizing border-box
      .cover
        width 100%
        height 100%
</style>