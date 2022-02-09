<template>
  <!-- 商品列表组件 -->
  <div class="shop">
    <div class="shop-list" ref="shopListBox" v-show="isShow">
      <div class="goods" id="goods">
        <ul class="goods-list fix" 
              ref="goodsList"
              @touchstart="touchStart($event)"
              @touchmove="touchMove($event)"
              @touchend="touchEnd($event)">
          <li v-for="(item, index) in myData" :key="index">        
            <a class="goods-a" :title="item.GoodsName" href="javascript:;" @click="gotoDetail(item.GoodsId)" target="_blank">
              <div class="goods-main">          
                <img class="lazy" :src="item.ImgUrl ? item.ImgUrl : require('@/assets/img/lazy_default.png')" :onerror="errorImg">
                <span class="icon new">新品</span>
              </div>
              <div class="goods-desc">
                    <h3>
                      <span>{{item.ly == 1 ? '天猫' : '淘宝'}}</span><em> | {{item.GoodsName}}</em>
                    </h3>            
                    <div class="list-price">                
                      <span class="price-new"><i>￥</i>{{item.LastPrice}}</span><em class="del">￥{{item.GoodsPrice}}</em>
                      <span class="only-time">{{item.AllDayCount}}人在抢</span>            
                    </div>
                </div>        
            </a>        
            <a rel="nofollow" :title="item.GoodsName" href="javascript:;" class="coupon-con" target="_blank" @click="gotoCoupon(item.GoodsId, item.ActLink)">
                <p>领券</p>
                <div class="quan-money">省<span>{{item.ActMoney}}</span>元</div>
                <div class="coupon-tit">去领券</div>
            </a>    
          </li>
        </ul>
        
        <div class="load-more" v-show="downFlag" ref="loadmore">
            <div v-show="haveMore" class="load-container load">
              <div class="loader">Loading...</div>
              <div class="loader-tit">正在加载...</div>
            </div>
            <div v-show="!haveMore" class="loadingText">没有更多了</div>
        </div>

      </div>
    </div>

      <loading :isLoading="isLoading"></loading>

      <!-- 请求数据错误展示占位图 -->
      <sold-out :isOut="isOut">
        <span>{{outText}}</span>
      </sold-out>

  </div>
</template>

<script>
import Export from '@/assets/js/export'
import { getShopList } from '@/assets/js/api'
import loading from './loading'
import soldOut from './soldOut'
import { url } from '@/assets/js/url'  
import '@/mock'

export default {
  components: {
    loading, 
    soldOut
  },
  data () {
    return {
      myData: [],
      isInitData: true,
      isLoading: false,
      isShow: false,
      isOut: false,
      outText: '',
      errorImg: "this.src='" + require('@/assets/img/lazy_default.png') + "'",

      //上拉加载更多
      pageNum: 20, //每页显示数量
      pageNo: 1, //页码
      downFlag: false,
      scrollHt: 0, //滚动距离
      maxScrollHt: '', //最大滚动相差值
      startPosition: '', //初始位置
      endPosition: '', //结束位置
      baseHt: 20, //加载基准
      moveY: 0,
      moreHt: 0, //上拉加载高度
      haveMore: true, //有更多数据
      times: 0, //调用次数
    }
  },
  props: {
    isSearchInitData: Boolean
  },
  watch:{
      //'$route':'getListData'
      // 对路由变化作出响应...
  },
  methods: {
    touchStart (e) {
      let touch = e.touches[0];
      this.startPosition = {
        x:touch.pageX,
        y:touch.pageY
      }

      if (this.$refs.goodsList) {
        let loadmoreHt = parseFloat(getComputedStyle(this.$refs.loadmore).height);
        if (isNaN(loadmoreHt)) {
          loadmoreHt = 0;
        }

        this.contentHt = parseFloat(getComputedStyle(this.$refs.goodsList).height) + loadmoreHt;

      } else {
        this.contentHt = 0;
      }
      this.boxHt = parseFloat(getComputedStyle(this.$refs.shopListBox).height);
      this.moreHt = parseFloat(getComputedStyle(this.$el.querySelector('.load-more')).height);

      this.maxScrollHt = this.contentHt - this.boxHt;
    },
    touchMove (e) {
      let touch = e.touches[0];
      let more = this.$el.querySelector('.load-more').style.display;
      this.endPosition = {
        x:touch.pageX,
        y:touch.pageY
      }
      this.moveY = this.endPosition.y - this.startPosition.y;
      if (this.moveY < 0) { //上拉加载
        if (this.scrollHt >= (this.maxScrollHt - this.baseHt)) {
          this.downFlag = true;
          if (this.myData.length >= 10) {
            this.times++;
            if (this.times == 1) {
              this.pageNo++;
              this.isInitData = false;
              this.getListData();
            }
          } else {
            this.haveMore = false;
          }
          
        }
      } else if (this.moveY > 0) {
        if (this.haveMore) {
          this.downFlag = false;
        }
      }
    },
    touchEnd(e){
      
    },
    gotoDetail (id) {
      window.location.href = `..${url}/detail.html?id=${id}`;
    },
    gotoCoupon (id, url) {
      let ua = window.navigator.userAgent.toLowerCase();
      if (ua.match(/MicroMessenger/i) == 'micromessenger') {
        window.location.href = `..${url}/coupon.html?id=${id}`;
      } else {
        window.location.href = url;
      }
    },
    getListData () {
      let me = this;

      if (me.isSearchInitData) {
        me.pageNo = 1;
        me.isInitData = true;
      }

      if (me.isInitData) {
        me.isLoading = true;
        me.isOut = false;
      }


      getShopList(
        me.pageNo, 
        me.pageNum, 
        null,
        Export.getUrlParam().category, 
        Export.getUrlParam().keyword
      )
      .then(res => {
        console.log('shopList===', res)
          if (me.isInitData) {
            me.myData = [];

            if (res.error == 0 && typeof(res.data) != undefined) {
              me.isLoading = false;

              if (res.data) {
                me.isShow = true;
                // me.myData = res.data;

                if (res.data.length >= 10) {  
                  me.downFlag = false;
                  me.haveMore = true;
                  me.times = 0;
                }
              } else {
                me.isShow = false;
                me.isOut = true;
                me.outText = '~_~抱歉，木有找到相关商品信息！';
              }
            } else {
              me.isLoading = false;
              me.isShow = false;
              me.isOut = true;
              me.outText = '请求数据失败';
            }
          } else {
            if (res.error == 0 && typeof(res.data) != undefined) {
              me.isLoading = false;

              if (res.data) {
                setTimeout(() => {
                  // me.myData.push(...res.data);
                  
                  me.downFlag = false;
                  me.haveMore = true;

                  setTimeout(() => {
                    me.times = 0;
                  }, 1000);
                }, 1000);
              } else {
                me.haveMore = false;
              }
            } else {
              me.isLoading = false;
              me.downFlag = false;
              me.haveMore = true;
              me.times = 0;
            }
          }
          me.myData.push({
            GoodsName:"狗狗",
            ImgUrl:"https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.zcool.cn%2Fcommunity%2F014caa554541210000019ae9f0567c.jpg&refer=http%3A%2F%2Fimg.zcool.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1646977633&t=b89ad786b02cd1c780afa994c38b3a22",
            LastPrice: 99,
            GoodsPrice: 188,
            AllDayCount: 10,
            ActMoney: 20
            },
            {
            GoodsName:"猫猫",
            ImgUrl:"https://img1.baidu.com/it/u=1159982953,1956383570&fm=253&fmt=auto&app=138&f=JPEG?w=500&h=500",
            LastPrice: 188,
            GoodsPrice: 288,
            AllDayCount: 10,
            ActMoney: 30
            },
            {
            GoodsName:"🐠🐠",
            ImgUrl:"https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fhbimg.b0.upaiyun.com%2Fcffdcb76f4f7d4c08847864c5615be56108b36f03e0c8-uuUWpQ_fw658&refer=http%3A%2F%2Fhbimg.b0.upaiyun.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1646978000&t=1dc0090b008357361e0176861d2b9065",
            LastPrice: 19,
            GoodsPrice: 39,
            AllDayCount: 55,
            ActMoney: 4
            },
            {
            GoodsName:"鸟鸟",
            ImgUrl:"https://img0.baidu.com/it/u=878445410,2434516592&fm=253&fmt=auto&app=138&f=PNG?w=600&h=400",
            LastPrice: 79,
            GoodsPrice: 129,
            AllDayCount: 89,
            ActMoney: 9
            },)
      })
      .catch(err => {
        me.isLoading = false;
        me.isShow = false;
        me.isOut = true;
        me.outText = '网络请求异常，请稍后再试';
      })

    }
  },
  created () {
    Export.rem();
    this.getListData();
  },
  mounted () {
    
  }
}

</script>

<style scoped>
.list-summary{
  padding: .2rem 0;
  font-size: .24rem;
  border-bottom: 1px solid #eee;
  border-top: 1px solid #eee;
  background: #fff;
}
.list-summary span{
  height: .4rem;
  line-height: .4rem;
  padding-left: .2rem;
  border-left: 2px solid #eb3e42;
  display: block;
  color: #000;
  font-size: .28rem;
}

.goods {
    width: 100%;
    height: 100%;
}
.goods-list {
    width: 100%;
    background: #fff;
    padding-bottom: 2%;
}
.goods-list li{
  position: relative;
  float: left;
  width: 47%;
  margin: 2% 0 0 2%;
  background: #fff;
}

.goods-list .goods-a {
    display: block;
    position: relative;
}
.goods-list li img {
    width: 100%;
    height: 3.52rem;
    position: relative;
    display: inherit;
}
 .goods-main {
    width: 100%;
    height: 3.52rem;
    background: #eee;
    /*background: url(../assets/img/lazy_default.png) no-repeat center;
    background-size: 100%;*/ 
} 
.goods-main .icon{
  display: block;
  width: .56rem;
  height: .32rem;
  line-height: .32rem;
  text-align: center;
  position: absolute;
  top: 0;
  left: 0;
  color: #fff;
  font-size: .24rem;
  letter-spacing: -1px;
}
.goods-main .icon.new{
  background: #66cc99;
}
.goods-list li h3{
  width: 100%;
    font-size: .24rem;
    font-weight: 300;
    color: #808080;
    height: .46rem;
    line-height: .46rem;
    text-indent: .1rem;
    overflow: hidden;
    word-wrap: break-word;
    word-break: break-all;
    white-space: nowrap;
    text-overflow: ellipsis;
    -o-text-overflow: ellipsis;
}
.list-price {
    width: 100%;
    line-height: 0;
    position: relative;
    bottom: 0;
    padding: 0 0 .6rem;
    background: #fff;
}
.price-new {
    font-size: .32rem;
    color: #F8285C;
}
.del {
    color: #b2b2b2;
    text-decoration: line-through;
    margin-left: .04rem;
    font-size: .24rem;
}

.only-time {
    display: block;
    position: absolute;
    top: .15rem;
    right: .05rem; 
    height: .28rem;
    line-height: .28rem;
    color: #b2b2b2;
    font-size: .2rem;
    text-align: right;
}
.coupon-con {
  display: block;
  position: absolute;
  right: 0;
  top: 0;
  width: 1.5rem;
  text-align: center;
  background: #ff4c66;
}
.coupon-con p {
  font-size: .20rem;
  color: #fff;
  padding: .05rem 0;
}
.coupon-con .coupon-tit {
  font-size: .24rem;
  color: #ff4c66;
  background: #fcff00;
  padding: .05rem 0;
}
.goods-list .quan-money {
  font-size: .24rem;
  padding-bottom: .05rem;
  color: #fff;
  text-align: center;
}
.goods-list .quan-money span {
  font-size: .32rem;
  padding: 0 .05rem;
}

.load-more {
  width: auto;
  height: auto;
  display: flex;
  text-align: center;
  align-items: center;
  justify-content: center;
  padding: 0 0 .3rem 0;
}
/* .loadingImg {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: auto;
  font-size: .3rem;
  color: #545454;
}
.loadingImg > img{
  height: .5rem;
  width: .5rem;
  margin-right: .3rem;
} */
.loadingText{
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: auto;
  font-size: .3rem;
}

/* loading动画 */
.load-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: auto;
}
.load .loader {
  font-size: 0;
  text-indent: -9999rem;
  border-top: .05rem solid rgba(0, 0, 0, .1);
  border-right: .05rem solid rgba(0, 0, 0, .1);
  border-bottom: .05rem solid rgba(0, 0, 0, .1);
  border-left: .05rem solid #fc0786;
  -webkit-animation: load 1.1s infinite linear;
  animation: load 1.1s infinite linear;
}
.loader-tit {
  font-size: .3rem;
  color: #545454;
  margin-left: .3rem;
}
.load .loader,
.load .loader:after {
  border-radius: 50%;
  width: .4rem;
  height: .4rem;
}
@-webkit-keyframes load {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
@keyframes load {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
</style>
