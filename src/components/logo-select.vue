<template>
  <div>
    <div class="main-logo">
      <div class="selected-title" @click="toggleFlag">
        <i :class="'selected-icon icon iconfont ' + selectedSearch.icon"></i>
        <span style="font-size:38px">{{selectedSearch.name}}</span>
      </div>
      <span class="logoList-arrow" @click="toggleFlag"></span>
      <!-- tranition是vue的过渡效果 -->
      <transition name="logofade">
        <div v-show="logoListFlag" class="container">
          <div v-for="group in searchConf" class="c-row">
            <div class="label">{{group.label}}</div>
            <ul class="logoList">
              <li v-for="(item) in group.items" class="logoItem" @mouseover="logoListHover(item)" v-bind:key='item.item'
                :class="{selectback:item.item==logoHover}" @click="logoSelected(item)">
                <i v-if="item.icon && item.icon!=''" :class="'search-icon icon iconfont ' + item.icon"></i>
                <span style="font-size:17px">{{item.name}}</span>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <div class="search-input">
      <input type="text" v-model="keyword" @keyup="get($event)" @keydown.enter="search()" @keydown.down="selectDown()"
        @keydown.up.prevent="selectUp()">
      <!-- 这是一个小叉叉，点击它可清除输入框内容 -->
      <span class="search-reset" @click="clearInput()">&times;</span>
      <button class="search-btn" @click="search()">搜一下</button>
      <div class="search-select">
        <!-- transition-group也是vue2.0中的新特性,tag='ul'表示用ul包裹v-for出来的li -->
        <transition-group name="itemfade" tag="ul" mode="out-in" v-cloak>
          <li v-for="(value,index) in myData" :class="{selectback:index==now}"
            class="search-select-option search-select-list" @mouseover="selectHover(index)" @click="selectClick(index)"
            :key="value">
            {{value}}
          </li>
        </transition-group>
      </div>
    </div>
  </div>

</template>

<script>
  // export defult是es6中的方法，将整个花括号对象暴露出去;
  export default {
    data: function () {
      return {
        logoHover: '',
        logoListFlag: false,
        mouseLeaveFlag: false,
        myData: [], //用来接收ajax得到的数据
        keyword: '', //v-model绑定的输入框的value
        now: -1,
        searchConf: [{
          label: "常用",
          items: [{
            item: "baidu",
            icon: 'icon-baidu-fill',
            name: '百度',
            searchSrc: 'https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd='
          }, {
            item: "360",
            name: '360搜索',
            icon: 'icon--o',
            searchSrc: 'https://www.so.com/s?ie=utf-8&q='
          }, {
            item: "google",
            name: '谷歌',
            icon: 'icon-google-line',
            searchSrc: 'https://www.google.com/search?q='
          }, {
            item: "bing",
            name: '必应',
            icon: 'icon-Bing',
            searchSrc: 'https://cn.bing.com/search?q='
          }, {
            item: "sougou",
            name: 'sougou搜狗',
            icon: '',
            searchSrc: 'https://www.sogou.com/web?query='
          }]
        }, {
          label: "商城",
          items: [{
            item: "淘宝",
            name: '淘宝',
            icon: 'icon-taobao-line',
            searchSrc: 'https://s.taobao.com/search?q='
          }, {
            item: "京东",
            name: '京东',
            icon: 'icon-zhifu-jingdong',
            searchSrc: 'https://search.jd.com/Search?keyword='
          }, {
            item: "duoduo",
            name: '拼多多',
            icon: 'icon-pintuan',
            searchSrc: 'https://youhui.pinduoduo.com/search/landing?keyword='
          }]
        }, {
          label: "社交",
          items: [{
            item: "zhihu",
            name: '知乎',
            icon: 'icon-zhihu-fill',
            searchSrc: 'http://www.zhihu.com/search?q='
          }, {
            item: "weibo",
            name: '微博',
            icon: 'icon-weibo-fill',
            searchSrc: 'https://s.weibo.com/weibo?q='
          }, {
            item: "tencentVideo",
            name: '腾讯视频',
            icon: 'icon-tengxunshipin-',
            searchSrc: 'http://v.qq.com/x/search?opensearch=1&q='
          }, {
            item: "douban",
            name: '豆瓣',
            icon: 'icon-douban-line',
            searchSrc: 'https://www.douban.com/search?q='
          }, {
            item: "BiliBili",
            name: 'B站',
            icon: 'icon-douban',
            searchSrc: 'https://search.bilibili.com/all?keyword='
          }, {
            item: "GitHub",
            name: 'GitHub',
            icon: 'icon-github-fill',
            searchSrc: 'https://github.com/search?q='
          }]
        }],
        selectedSearch: {
          item: "baidu",
          icon: 'icon-baidu-fill',
          name: '百度',
          searchSrc: 'https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd='
        }
      }
    },
    methods: {
      toggleFlag: function () {
        this.logoListFlag = !this.logoListFlag;
        this.mouseLeaveFlag = true;
      },
      logoListHover: function (item) {
        this.logoHover = item.item;
      },
      logoSelected: function (item) {
        this.selectedSearch = item;
        this.logoListFlag = false;
        // 触发父组件的自定义事件，向父组件传参数,selectNow是选择了哪个搜索引擎的索引，父组件得到了之后就可以指定搜索时跳转到哪个搜索引擎
        this.$emit('getindex', this.selectedNow);
      },
      mouseLeaveList: function () {
        //鼠标离开列表事件
        this.mouseLeaveFlag = false;
        this.logoListFlag = false;
      },
      // &event是实参，表示event对象
      get: function (ev) {
        // 如果按得键是上或者下，就不进行ajax
        if (ev.keyCode == 38 || ev.keyCode == 40) {
          return;
        }

        this.$http.jsonp('https://sug.so.360.cn/suggest?word=' + this.keyword + '&encodein=utf-8&encodeout=utf-8')
          .then(function (res) {
            this.myData = res.data.s;
          });
      },
      selectDown: function () {
        this.now++;
        //到达最后一个时，再按下就回到第一个
        if (this.now == this.myData.length) {
          this.now = 0;
        }
        this.keyword = this.myData[this.now];
      },
      selectUp: function () {
        this.now--;
        //同上  
        if (this.now == -1) {
          this.now = this.myData.length - 1;
        }
        this.keyword = this.myData[this.now];
      },
      search: function () {
        //打开对应的搜索界面
        window.open(this.selectedSearch.searchSrc + this.keyword);
      },
      selectHover: function (index) {
        this.now = index
      },
      selectClick: function (index) {
        this.keyword = this.myData[index];
        this.search();
      },
      clearInput: function () {
        this.keyword = '';
        this.$http.jsonp('https://sug.so.360.cn/suggest?word=' + this.keyword + '&encodein=utf-8&encodeout=utf-8')
          .then(function (res) {
            this.myData = res.data.s;
          });
      },
      getIndex: function (index) {
        this.searchIndex = index;
      }
    }
  }

</script>

<style scoped type="text/css">
  .search-icon {
    font-size: 16px !important
  }

  ul {
    list-style: none;
    padding: 0;
    margin: 0
  }

  .c-row {
    float: left;
    width: 100%;
    padding: 5px 0;
  }

  .label {
    float: left;
    height: 40px;
    width: 70px;
    line-height: 40px;
    font-size: 18px;
    background: #eaeff3;
    margin: 0 5px;
  }

  .main-logo {
    width: 600px;
    height: 140px;
    position: relative;
  }

  .main-logo img {
    display: block;
    margin: 0 auto;
    user-select: none;
    cursor: pointer;
  }

  .container {
    width: 600px;
    height: 300px;
    position: absolute;
    background: aliceblue;
    z-index: 999;
  }

  .logoList-arrow {
    position: absolute;
    width: 0;
    height: 0;
    border: 8px solid;
    border-color: #000 transparent transparent transparent;
    right: 100px;
    top: 66px;
    cursor: pointer
  }

  .logoList {
    float: left;
    position: relative;
    width: 510px;
    background: aliceblue;
  }

  .logoList li {
    width: 120px;
    height: 40px;
    background-color: #FEFEFE;
    line-height: 40px;
    float: left;
    margin: 0 3px 2px 2px
  }

  .logoList li img {
    width: 100%;
    margin-top: 10px;
  }

  .logofade-enter-active,
  .logofade-leave-active {
    transition: all .5s;
  }

  .logofade-enter,
  .logofade-leave-active {
    opacity: 0;
    transform: translateY(20px);
  }

  .selectback {
    background-color: #eee !important;
    cursor: pointer
  }

  .selected-title {
    font-size: 40px;
    line-height: 140px;
  }

  .selected-icon {
    font-size: 40px;
  }

  .search-input {
    height: 45px;
    width: 600px;
    margin: 0 auto;
    margin-top: 10px;
    position: relative;
  }

  .search-input input {
    border: 1px solid #e4e4e4;
    box-sizing: border-box;
    width: 500px;
    height: 45px;
    font-size: 18px;
    float: left;
    padding-left: 10px;
    padding-right: 10px;
    overflow: hidden;
  }

  .search-btn {
    height: 45px;
    width: 100px;
    border: 1px solid mediumseagreen;
    background-color: mediumseagreen;
    color: white;
    font-size: 16px;
    font-weight: bold;
    float: left;
  }

  .search-btn {
    cursor: pointer
  }

  .search-select {
    position: absolute;
    top: 45px;
    width: 500px;
    box-sizing: border-box;
    z-index: 999;
  }

  .search-select li {
    border: 1px solid #d4d4d4;
    ;
    border-top: none;
    border-bottom: none;
    background-color: #fff;
    width: 100%
  }

  .search-select-option {
    box-sizing: border-box;
    padding: 7px 10px;
  }

  .selectback {
    background-color: #eee !important;
    cursor: pointer
  }

  input::-ms-clear {
    display: none
  }

  .search-reset {
    width: 21px;
    height: 21px;
    position: absolute;
    display: block;
    line-height: 21px;
    text-align: center;
    cursor: pointer;
    font-size: 20px;
    right: 110px;
    top: 12px
  }

  .search-select-list {
    transition: all 0.5s
  }

  .itemfade-enter,
  .itemfade-leave-active {
    opacity: 0;
  }

  .itemfade-leave-active {
    position: absolute;
  }

  .selectback {
    background-color: #eee !important;
    cursor: pointer
  }

  .search-select ul {
    margin: 0;
    text-align: left;
  }

</style>
