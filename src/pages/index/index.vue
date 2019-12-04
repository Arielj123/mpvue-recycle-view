<template>
  <scroll-view
    scroll-y
    class="main-box"
    :style="{height:systemHeight+'px'}"
    @scrolltolower="handleToBottom"
    @scroll="onScroll"
    lower-threshold="300rpx"
  >
    <!-- 全部内容盒子高度 -->
    <div class="scroll-box" :style="{height:allBoxHeight+'px'}">
      <!-- 可视区域盒子 -->
      <div class="scroll-main" :style="{top:top+'px'}">
        <card v-for="(item,i) in forEachArr" :key="i" :text="item" />
      </div>
    </div>
  </scroll-view>
</template>

<script>
import card from "@/components/card";
export default {
  data() {
    return {
      forEachArr: [], // 显示数据
      allDataArr: [], // 全部数据
      allBoxHeight: 0, // 全部内容盒子高度
      top: 0, // 可视区域盒子距离顶部距离
      queryData: {
        platform: "ZPo4MV4TqsLfAHkist6wQai7S8tzDVmz",
        page: 1,
        pagenum: 10 // 每页条数
      },
      showLen: 3, // 显示数据页数
      currentPage: 1, // 当前滑动到的页数
      itemHeight: 200, // 以各手机屏幕最低高度
      systemHeight: 0, // 设备屏幕高度
      isdisabaled: false
    };
  },

  watch: {
    currentPage(newVal, oldVal) {
      console.log("newVal, oldVal", newVal, oldVal);
      if (newVal < oldVal) {
        this.computedShowData();
      } else {
        if (newVal <= this.queryData.page) {
          this.computedShowData();
        }
      }
    }
  },

  methods: {
    // 请求数据
    queryList() {
      if (!this.isdisabaled) {
        wx.showLoading({ title: "加载中..." });
        this.isdisabaled = true;
        wx.request({
          url: "https://w.taopaitang.com/api/discover",
          method: "GET",
          data: this.queryData,
          header: {
            "content-type": "application/json" // 默认值
          },
          dataType: "json",
          success: res => {
            wx.hideLoading();
            if (res.data.code !== 200) {
              wx.showToast({
                title: res.data.message,
                icon: "none",
                duration: 2000,
                mask: true
              });
            } else {
              let data = res.data.data.items;
              if (this.queryData.page === 1) {
                this.allDataArr = data;
              } else {
                this.allDataArr = this.allDataArr.concat(data);
              }
              this.computedShowData();
              this.isdisabaled = false;
            }
          }
        });
      }
    },
    // 触底 ***必须异步
    async handleToBottom() {
      this.queryData.page += 1;
      await this.queryList();
    },
    // 滚动条
    onScroll(val) {
      let currentHeight = val.target.scrollTop + this.systemHeight;
      // 计算每一页数据的高度
      let onePageHeight = this.itemHeight * this.queryData.pagenum;
      // 现在显示的数据的页数
      if (currentHeight < onePageHeight) {
        this.currentPage = 1;
      } else {
        this.currentPage = Math.ceil(currentHeight / onePageHeight);
      }
      console.log(
        "currentPage",
        this.currentPage,
        currentHeight,
        onePageHeight
      );
    },
    // 计算显示数据及显示盒子距顶部距离
    computedShowData() {
      this.allBoxHeight = this.allDataArr.length * this.itemHeight;
      let {
        queryData: { page, pagenum },
        showLen,
        currentPage
      } = this;
      if (currentPage < showLen) {
        this.top = 0;
        this.forEachArr = this.allDataArr.slice(0, showLen * pagenum);
      } else {
        // 加一是为了将显示内容区域显示在屏幕中，否则无法触发触底事件
        this.top = (currentPage + 1 - showLen) * pagenum * this.itemHeight;
        this.forEachArr = this.allDataArr.slice(
          (currentPage - 2) * pagenum,
          currentPage * pagenum
        );
      }
      console.log("this.forEachArr", this.forEachArr);
    }
  },

  onLoad() {
    this.queryList();
    wx.getSystemInfo({
      success: res => {
        this.systemHeight = res.windowHeight;
      }
    });
  },

  components: {
    card
  }
};
</script>

<style scoped>
.main-box .scroll-box {
  position: relative;
  width: 100%;
}
.main-box .scroll-box .scroll-main {
  position: absolute;
  left: 0;
  width: 100%;
}
</style>
