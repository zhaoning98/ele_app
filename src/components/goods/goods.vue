<template>
	<div class="goods-wrapper">
	 <div class="menu-wrapper" ref="menuWrapper">
		<ul>
		  <li v-for="(item, index) in goods" class="menu-item" :class="{current: currentIndex===index}" v-on:click="selectMenu(index, $event)">
		  	<span class="text border-1p">
		  	  <span class="icon" :class="classMap[item.type]"></span>{{ item.name }}
		  	</span>
		  </li>
		</ul>
	 </div>
	 <div class="foods-wrapper" ref="foodsWrapper">
	   <ul>
	   	 <li v-for="item in goods" class="food-list food-list-hook">
	   	   <h1 class="title">{{ item.name }}</h1>
	   	   <ul>
	   	   	 <li v-for="food in item.foods" class="food-item border-1px">
	   	   	   <div class="icon">
	   	   	   	 <img width="57" height="57" :src="food.icon">
	   	   	   </div>
	   	   	   <div class="content">
	   	   	   	 <h2 class="name">{{ food.name }}</h2>
	   	   	   	 <p class="desc">{{ food.description }}</p>
	   	   	   	 <div class="extra">
	   	   	   	   <span class="count">月售{{ food.sellCount }}份</span><span>好评率{{ food.rating }}</span>
	   	   	   	 </div>
	   	   	   	 <div class="price">
	   	   	   	   <span class="now">￥{{ food.price }}</span><span class="old" v-show="food.oldPrice">￥{{ food.oldPrice }}</span>
	   	   	   	 </div>
	   	   	   	 <div class="cartcontrol-wrapper">
	   	   	   	   <v-cartcontrol @add="addFood" :food="food"></v-cartcontrol>
	   	   	   	 </div>
	   	   	   </div>
	   	   	 </li>
	   	   </ul>
	   	 </li>
	   </ul>
	 </div>
   <v-shopcart ref="shopcart" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice"></v-shopcart>
	</div>
</template>

<script >
  import BScroll from 'better-scroll';
  import shopcart from '../../components/shopcart/shopcart';
  import cartcontrol from '../../components/cartcontrol/cartcontrol';

  const ERR_OK = 0;
  
  export default {
    props: {
  	  seller: {
  	    type: Object
  	  }
    },
    data () {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      };
    },
    computed: {
  	  currentIndex() {
        for (let i = 0, len = this.listHeight.length; i < len; i++) {
          let height1 = this.listHeight[i],
              height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) { 
            return i;
          }
        }
  	    return 0;
  	  },
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created() {
  	  this.classMap = ['decrease', 'discount', 'special'  , 'invoice', 'guarantee'];

  	  this.$http.get('/api/goods')
          .then((response) => {
            response = response.data;
            if(response.errno === ERR_OK) {
              this.goods = response.data;

              // Vue.nextTick()在修改数据之后立即使用这个方法，获取更新后的 DOM。(异步更新队列)
              this.$nextTick(() => {
              	this._initScroll();
              	this._calculateHeight();
              });
            }
          });
    },
    methods: {
      addFood(target) { 
        this._drop(target);
      },
      _drop(target) {
        // 体验优化,异步执行下落动画 
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target);
        });
      },
      selectMenu(index, event) {
        if (!event._constructed) {
          //_constructed：better-scroll默认派发事件具有的属性、浏览器原生点击事件无该属性
          return;
        }
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el, 300);
        //派发滚动：scrollToElement(el, time, offsetX, offsetY, easing) 滚动到  某个元素，el（必填）表示 dom 元素，time 表示动画，offsetX 和 offsetY 表示坐标偏移量，easing 表示缓动函数
      },
      selectFood(food, event) {
        if (!event._constructed) {
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
      },
      /**
       * 初始化 bebetter-scroll
       **/
      _initScroll() {

      	this.menuScroll = new BScroll(this.$refs.menuWrapper, { 
      	  click: true //click: true 是否启用click事件
      	});

        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true,
          probeType: 3 //probeType: 1 滚动的时候会派发scroll事件，会截流。2滚动的时候实时派发scroll事件，不会截流。 3除了实时派发scroll事件，在swipe的情况下仍然能实时派发scroll事件
        });

        this.foodsScroll.on('scroll', (pos) => { 
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      }, 
      /**
       * 计算 右侧foodlist 各个模块的位置
       **/
      _calculateHeight() {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook'); 
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      }
    },
    components: {
      'v-cartcontrol': cartcontrol,
      'v-shopcart': shopcart
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin";

  .goods-wrapper
    display: flex
    position: absolute
    top: 174px
    bottom: 46px
    width: 100%
    overflow: hidden 
    .menu-wrapper 
  	  width: 80px
  	  background: #f3f5f7
  	  overflow-x: hidden
  	  overflow-y: hidden
  	  .menu-item
  	    display: table
  	    height: 54px
  	    width: 56px
  	    padding: 0 12px
  	    line-height: 14px
  	    &.current
  	      position: relative
  	      z-index: 10
  	      margin-top: -1px
  	      background: #fff
  	      font-weight: 700
  	      .text
  	        border-none()
        .icon
          display: none
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            display: inline-block
            bg-image('img/decrease_3')
          &.discount
            display: inline-block
            bg-image('img/discount_3')
          &.guarantee
            display: inline-block
            bg-image('img/guarantee_3')
          &.invoice
            display: inline-block
            bg-image('img/invoice_3')
          &.special
            display: inline-block
            bg-image('img/special_3')
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          border-1px(rgba(7, 17, 27, 0.1))
          font-size: 12px
  	.foods-wrapper
  	  flex: 1
  	  overflow-x: hidden
  	  overflow-y: hidden
  	  .title
  	    padding-left: 14px
  	    height: 26px
  	    line-height: 26px
  	    border-left: 2px solid #d9dde1
  	    font-size: 12px
  	    color: rgb(147, 153, 159)
  	    background: #f3f5f7
  	  .food-item
  	    display: flex
  	    margin: 18px
  	    padding-bottom: 18px
  	    border-1px(rgba(7, 17, 27, .1))
  	    &:last-child
  	      border-none()
  	      margin-bottom: 0
  	    .icon
  	      margin-right: 10px
        .content
          flex: 1
          .name
            margin: 2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color: rgb(7, 17, 27)
          .desc, .extra
            line-height: 10px
            font-size: 10px
            color: rgb(147, 153, 159)
          .desc
            line-height: 12px
            margin-bottom: 8px
          .extra
            .count
              margin-right: 12px
          .price
            font-weight: 700
            line-height: 24px
            .now
              margin-right: 8px
              font-size: 14px
              color: rgb(240, 20, 20)
            .old
              text-decoration: line-through
              font-size: 10px
              color: rgb(147, 153, 159)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 12px
</style>
