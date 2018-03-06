<template>
  <div>
    <div class="goods">
      <div class="menu-wrapper"
           ref="menuWrapper">
        <ul>
          <!-- :class="{'current':currentIndex===index}" 当满足条件时：配置current类名 -->
          <li v-for='(item,index) in goods'
              :key='index'
              class="menu-item"
              :class="{'current':currentIndex===index}"
              @click="selectMenu(index,$event)"
              ref='menuList'>
            <span class="text border-1px">
              <span v-show='item.type>0'
                    class="icon"
                    :class="classMap[item.type]"></span>{{item.name}}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper"
           ref="foodsWrapper">
        <ul>
          <li v-for="(item,index) in goods"
              class="food-list"
              ref="foodList"
              :key=index>
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li v-for="food in item.foods"
                  class="food-item border-1px"
                  :key='food.id'>
                <div class="icon">
                  <img width='57'
                       height="57"
                       :src="food.icon">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span>
                    <span>好评率{{food.rating}}%</span>
                    <div class="price">
                      <span class="now">￥{{food.price}}</span>
                      <span class="old"
                            v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                    </div>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    </div>
  </div>
</template>

<script type='text/ecmascript6'>
import BScroll from 'better-scroll';
import shopcart from 'components/shopcart/shopcart';

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
      scrollY: []
    };
  },
  computed: {
    // 映射右侧滚动的height 对应着左面滚动条的栏目。
    currentIndex () {
      for (let i = 0; i < this.listHeight.length; i++) {
        // console.log(this.listHeight.length);
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i + 1];
        /*
        * !height2 是表示没有下一个高度 也就是最后一栏时
        * scrollY > height1 && scrollY < height2 表示落在一个中间某一个栏中
        */
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i;
        }
      }
      return 0;
    }
  },
  created () {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    this.$http.get('/api/goods').then(response => {
      response = response.body;
      if (response.errno === ERR_OK) {
        this.goods = response.data;
        // console.log(this.goods)
        /*
        * `Vue.nextTick(callback)`，当数据发生变化，更新后执行回调.
        * `Vue.$nextTick(callback)`，当dom发生变化，更新后执行的回调.
        */
        this.$nextTick(() => {
          this._initScroll();
          this._calculateHeight();
        });
      }
    });
  },
  methods: {
    /*
    * new Bscroll(dom, json对象)
    */
    selectMenu (index, event) {
      // 取消浏览器pc界面点击事件，只留下betterScroll触发的点击事件，修复了pc界面点击一下触发两次click事件。
      if (!event._constructed) {
        return;
      }
      let foodList = this.$refs.foodList;
      let el = foodList[index];
      // betterScroll接口   dom 滚动时间300毫秒
      this.foodsScroll.scrollToElement(el, 300);
    },
    _initScroll () {
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      });
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3
      });
      // 绑定scroll事件，可以实时获取y的值。 参数pos
      this.foodsScroll.on('scroll', pos => {
        // console.log(pos);
        /**
         * 打印数据如下：pos就是每次滑动的坐标{x,y}两个参数.
         * {x: 0, y: -3.7559814453125}
         * {x: 0, y: -7.717987060546875}
         * {x: 0, y: -11.67999267578125}
         * {x: 0, y: -15.641998291015625}
         * {x: 0, y: -20.31298828125}
         * {x: 0, y: -26.59698486328125}
         * {x: 0, y: -32.378997802734375}
         * {x: 0, y: -37.048980712890625}
         * {x: 0, y: -40.4019775390625}
         */
        if (pos.y <= 0) {
          this.scrollY = Math.abs(Math.round(pos.y));
        }
      });
    },
    _calculateHeight () {
      let foodList = this.$refs.foodList;
      let height = 0;
      // 把第一次height记录下来，也就是第一个foodList（热销榜）添加道数组里
      this.listHeight.push(height);
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i];
        // 把height累加，依次添加道数组上
        height += item.clientHeight;
        this.listHeight.push(height);
      }
    }
  },
  components: {
    shopcart
  }
};
</script>

<style lang='stylus' scoped>
@import '../../common/stylus/mixin'

.goods
  display flex
  position absolute
  top 174px
  bottom 46px
  width 100%
  overflow hidden

  .menu-wrapper
    flex 0 0 80px
    width 80px
    background-color #f3f5f7

    .menu-item
      display table
      height 54px
      width 56px
      padding 0 12px

      &.current
        position relative
        margin-top -1px
        z-index 10
        background #ffffff
        font-weight 700

        .text
          border-none()

      .icon
        display inline-block
        vertical-align top
        width 12px
        height 12px
        margin-right 2px
        background-size 12px 12px
        background-repeat no-repeat

        &.decrease
          bg-image('decrease_3')

        &.discount
          bg-image('discount_3')

        &.guarantee
          bg-image('guarantee_3')

        &.invoice
          bg-image('invoice_3')

        &.special
          bg-image('special_3')

      .text
        display table-cell
        width 56px
        vertical-align middle
        border-1px(rgba(7, 17, 27, 0.1))
        font-size 12px

  .foods-wrapper
    flex 1

    .title
      padding-left 14px
      height 26px
      line-height 26px
      border-left 2px solid #d9dde1
      font-size 12px
      color rgb(147, 153, 159)
      background #f3f5f7

    .food-item
      display flex
      margin 18px
      padding-bottom 18px
      border-1px(rgba(7, 17, 27, 0.1))

      &:last-child
        margin-bottom 0
        border-none()

      .icon
        flex 0 0 57px
        margin-right 10px

      .content
        flex 1

        .name
          margin 2px 0 8px 0
          height 14px
          line-height 14px
          font-size 14px
          color rgb(7, 17, 27)

        .desc, .extra
          line-height 10px
          font-size 10px
          color rgb(147, 153, 159)

        .desc
          line-height 12px
          margin-bottom 8px

        .extra
          .count
            margin-right 12px

        .price
          font-weight 700
          line-height 24px

          .now
            margin-right 8px
            font-size 14px
            color rgb(240, 20, 20)

          .old
            text-decoration line-through
            font-size 10px
            color rgb(147, 153, 159)
</style>
