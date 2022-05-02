<template>
  <div class="app-container">
    <!-- 头部区域 -->
    <Headr></Headr>
    <Goods
      v-for="item in list"
      :key="item.id"
      :id="item.id"
      :title="item.goods_name"
      :pic="item.goods_img"
      :price="item.goods_price"
      :state="item.goods_state"
      :count="item.goods_count"
      @state-change="getNewState"
    ></Goods>
    <Footer
      :lis="fullState"
      @full-change="getFullstate"
      :amout="amt"
      :all="total"
    ></Footer>
  </div>
</template>

<script>
// 导入组件
import axios from "axios";
import Headr from "@/components/Header/Header.vue";
import Goods from "@/components/Goods/Goods.vue";
import Footer from "@/components/Footer/Footer.vue";
import bus from "@/components/eventBus.js";

export default {
  data() {
    return {
      // 用来存储购物车的列表数据
      list: [],
    };
  },
  components: {
    Headr,
    Goods,
    Footer,
  },
  methods: {
    async initCartList() {
      // 调用axios
      const { data: res } = await axios.get("https://www.escook.cn/api/cart");
      // 只要请求回来的数据，在页面渲染期间要用到，必须转存到data
      if (res.status === 200) {
        this.list = res.list;
      }
    },
    // 接收子组件传递过来的数据
    getNewState(e) {
      // console.log(val);
      this.list.some((item) => {
        if (item.id == e.id) {
          item.goods_state = e.value;
          //终止后续循环
          return true;
        }
      });
    },
    getFullstate(val) {
      // console.log(e);
      this.list.forEach((item) => (item.goods_state = val));
    },
  },
  created() {
    // 调用请求数据方法
    this.initCartList();
    bus.$on("share", (val) => {
      this.list.some((item) => {
        if (item.id === val.id) {
          item.goods_count = val.value;
          return true;
        }
      });
    });
  },
  //计算属性
  computed: {
    // 动态计算出全选的状态
    fullState() {
      return this.list.every((item) => item.goods_state);
    },
    // 已勾选商品的总价格
    amt() {
      //1.先过滤
      // 2.在累加
      return this.list
        .filter((item) => item.goods_state)
        .reduce(
          (total, item) => (total += item.goods_price * item.goods_count),
          0
        );
    },
    total() {
      return this.list
        .filter((item) => item.goods_state)
        .reduce((t, item) => (t += item.goods_count), 0);
    },
  },
};
</script>

<style lang="less" scoped>
.app-container {
  padding-top: 45px;
  padding-bottom: 50px;
}
</style>
