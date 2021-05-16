// 1.不把长列表数据全部展示在页面上
// 2.截取一部分数据用来填充屏幕的容器区域
// 3.长列表数据不可视部分使用空白占位符
// 4.监听滚动事件根据滚动位置动态改变可视列表
// 5.监听滚事件根据滚动位置动态改变空白填充区域 

<template>
  <div class="news_box">
    <!-- passive修饰符确保滚动行为总是生效 -->
    <div class="scroll_c" ref="scrollCon" @scroll.passive="handleScroll">
      <div :style="blankFillStyle">
        <div class="new" v-for="(item, index) in showDataList" :key="index">
          <div>{{ item.from }}</div>
          <div>{{ "id:" + item.id }}</div>
          <div>{{ item.date }}</div>
        </div>
        <div v-if="status">
          <h1>
            {{ msg }}
          </h1>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * 1.计算滚动容器最大列表容积
 * 2.监听滚动事件
 *
 *
 *
 * */
import axios from "axios";
export default {
  data() {
    return {
      status: false,
      msg: "正在加载",
      allDataList: [],
      oneNewH: 100, // 记录单挑数据的高度
      conSize: 0, // 记录容器的最大容积数
      sIndex: 0, // 滚动时，记录滚动可视区域最上面一个可视item索引
      isScroll: true, // 记录滚动的有效状态
    };
  },
  computed: {
    // 滚动容器的最后一个可视元素的索引
    endIndex() {
      let endIndex = this.sIndex + this.conSize*2;
      if (!this.allDataList[endIndex]) {
        endIndex = this.allDataList.length - 1;
      }
      return endIndex;
    },
    // 定义一个待显示的数组列表
    showDataList() {
      let statrIndex = 0
      if (this.sIndex <= this.conSize) {
          statrIndex = 0
      } else {
          statrIndex = this.sIndex - this.conSize
      }
      return this.allDataList.slice(statrIndex, this.endIndex);
    },
    // // 上空白的高度
    // topBlankFill() {
    //   return this.sIndex * this.oneNewH;
    // },
    // // 下空白的高度
    // bottomBlankFill() {
    //   return (this.allDataList.length - this.endIndex) * this.oneNewH;
    // },
    // 上下空白填充高度
    blankFillStyle() {
      let statrIndex = 0
      if (this.sIndex <= this.conSize) {
          statrIndex = 0
      } else {
          statrIndex = this.sIndex - this.conSize
      }
      return {
        paddingTop: statrIndex * this.oneNewH + "px",
        paddingBottom:
          (this.allDataList.length - this.endIndex) * this.oneNewH + "px",
      };
    },
  },
  methods: {
    // 滚动行为
    handleScroll() {
      if (this.isScroll) { // 限制滚动事件节流
        this.isScroll = false;
        let timer = setTimeout(() => {
            this.isScroll = true;
            clearTimeout(timer)
        }, 30)
        this.setDataStart()
      }
    },
    async setDataStart () {
        // 记录滚动可视区域最上面一个的item索引
        let currentIndex = ~~(this.$refs.scrollCon.scrollTop / this.oneNewH);
        if (this.sIndex === currentIndex) return; // 还没有滚动到下一个的时候不拉取数据
        this.sIndex = ~~(this.$refs.scrollCon.scrollTop / this.oneNewH);
        // 代表滚动到底部
        if (this.sIndex + this.conSize > this.allDataList.length - 1 && !this.status) {
          let newList = await this.getList(20);
          this.allDataList = [...this.allDataList, ...newList];
          console.log("棍帝");
        }
    },
    // 计算容器的最大容积
    getConSize() {
      // ~~（8.9）直接取整
      // 容器最大容纳条目的数量为（容器的最大容积数）：容器的高度取整 / 每个条目的高度 + 2 
      this.conSize = ~~(this.$refs.scrollCon.offsetHeight / this.oneNewH) + 2;
    },
    getList(num) {
      this.status = true;
      return axios.get("http://localhost:8888/data?num=" + num).then((res) => {
        //   console.log(res);
        this.status = false;
        return res.data.list;
      });
    },
  },
  mounted() {
    this.getConSize();
    window.onresize = this.getConSize;
    window.orientationchange = this.getConSize;
  },
  async created() {
    let newList = await this.getList(20);
    this.allDataList = newList;
  },
};
</script>

<style>
.news_box {
  text-align: center;
  height: 100%;
  width: 100%;
}
.scroll_c {
  width: 100%;
  height: 100%;
  overflow: auto;
}
.new {
  height: 99px;
  border-bottom: 1px solid red;
}
</style>