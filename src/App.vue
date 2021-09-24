<template>
  <div class="app-container">
    <!-- 头部 -->
    <Header></Header>
    <!-- 循环渲染每个商品 -->
    <Goods v-for="item in list" 
           :key="item.id" 
           :id="item.id"
           :title="item.goods_name"
           :pic="item.goods_img"
           :price="item.goods_price"
           :state="item.goods_state"
           :count="item.goods_count"
           @state-change="getNewState">
    </Goods>
    <!-- 底部区域 -->
    <Footer :isfull="fullState"
            :amount="amt"
            :all="total"
            @full-change="getFullState">
    </Footer>
  </div>
</template>

<script>
//导入需要的组件
import Header from './components/Header/Header.vue'
import Goods from './components/Goods/Goods.vue'
import Footer from './components/Footer/Footer.vue'
import bus from './components/eventBus.js'

//导入axios请求库
import axios from 'axios'

export default {
  data(){
    return{
      //用来存储购物车列表数据，默认空数组
      list:[]
    }
  },
  computed:{
    //动态计算出全选的状态是true还是false
    fullState(){
      return this.list.every(item => item.goods_state)
    },
    //已勾选商品的总价格
    amt(){
      //先filter过滤再reduce添加
      return this.list
      .filter(item => item.goods_state)
      .reduce((total, item) => (total += item.goods_price * item.goods_count),0)
    },
    //已勾选商品的总数量
    total(){
      return this.list
      .filter(item => item.goods_state)
      .reduce((t, item) => (t += item.goods_count),0)
    }
  },
  created(){
    //调用请求数据得方法
    this.initCarList()
    bus.$on('share',val=>{
      this.list.some(item =>{
        if(item.id === val.id){
          item.goods_count = val.value
          return true
        }
      })
    })
  },
  methods:{
    async initCarList(){
      //调用axios的get方法，请求列表数据
     const{data: res} = await axios.get('https://www.escook.cn/api/cart')
     //只要请求回来的数据，在页面渲染期间要用到，则必须转存到data里
     if(res.status === 200){
       this.list = res.list
     }
    },
    //接受子组件传递过来的数据
    getNewState(val){
      console.log(val)
      this.list.some(item =>{
        if(item.id === val.id){
          item.goods_state = val.value
          return true
        }
      })
    },
    //获取子组件传递过来的全选按钮状态
    getFullState(val){
      console.log(val)
      this.list.forEach(item => (item.goods_state = val))
    },
  },
  components:{
    Header,
    Goods,
    Footer
  }
}
</script>

<style lang="less" scoped>
.app-container {
  padding-top: 45px;
  padding-bottom: 50px;
}
</style>
