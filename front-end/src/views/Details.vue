<!--
 * @Description: 商品详情页面组件
 -->
<template>
  <div id="details">
    <!-- 头部 -->
    <div class="page-header">
      <div class="title">
        <p>{{ productDetails.product_name }}</p>
        <div class="list">
          <ul>
            <li>
              <router-link to>Description</router-link>
            </li>
            <li>
              <router-link to>Parameter</router-link>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <!-- 头部END -->

    <!-- 主要内容 -->
    <div class="main">
      <!-- 左侧商品图 -->
      <div class="block">
        <img style="height:560px;" :src="productPicture" />
      </div>
      <!-- 左侧商品轮播图END -->

      <!-- 右侧内容区 -->
      <div class="content">
        <h1 class="name">{{ productDetails.product_name }}</h1>
        <p class="intro">{{ productDetails.product_intro }}</p>
        <div class="price">
          <span>{{ productDetails.product_selling_price }}RMB</span>
          <span
            v-show="
              productDetails.product_price !=
                productDetails.product_selling_price
            "
            class="del"
            >{{ productDetails.product_price }}RMB</span
          >
        </div>
        <div class="pro-list">
          <span class="pro-name">{{ productDetails.product_name }}</span>
          <span class="pro-price">
            <span>{{ productDetails.product_selling_price }}RMB</span>
            <span
              v-show="
                productDetails.product_price !=
                  productDetails.product_selling_price
              "
              class="pro-del"
              >{{ productDetails.product_price }}RMB</span
            >
          </span>
          <p class="price-sum">
            Subtotal: {{ productDetails.product_selling_price }}RMB
          </p>
        </div>
        <!-- 内容区底部按钮 -->
        <div class="button">
          <el-button class="shop-cart" :disabled="dis" @click="addShoppingCart"
            >Add to cart</el-button
          >
          <el-button class="like" @click="addCollect">Wishlist</el-button>
        </div>
        <!-- 内容区底部按钮END -->
        <div class="pro-policy">
          <ul>
            <li><i class="el-icon-circle-check"></i> Base Direct Delivery</li>
            <li><i class="el-icon-circle-check"></i> After Sale Service</li>
            <li>
              <i class="el-icon-circle-check"></i> Online Customer Service
            </li>
          </ul>
        </div>
      </div>
      <!-- 右侧内容区END -->
    </div>
    <!-- 主要内容END -->
  </div>
</template>
<script>
import { mapActions } from "vuex";
export default {
  data() {
    return {
      dis: false, // 控制“加入购物车按钮是否可用”
      productID: "", // 商品id
      productDetails: "", // 商品详细信息
      productPicture: "", // 商品图片
    };
  },
  // 通过路由获取商品id
  activated() {
    if (this.$route.query.productID != undefined) {
      this.productID = this.$route.query.productID;
    }
  },
  watch: {
    // 监听商品id的变化，请求后端获取商品数据
    productID: function(val) {
      this.getDetails(val);
      this.getDetailsPicture(val);
    },
  },
  methods: {
    ...mapActions(["unshiftShoppingCart", "addShoppingCartNum","setUser", "setShowLogin"]),
    // 获取商品详细信息
    getDetails(val) {
      this.$axios({
        method: "post",
        url: "http://localhost:80/back-end/product.php?action=getProductById",
        data: {
          product_id: val,
        },
        transformRequest: [
          function(data) {
            // 将{username:111,password:111} 转成 username=111&password=111
            var ret = "";
            for (var it in data) {
              // 如果要发送中文 编码
              ret +=
                encodeURIComponent(it) +
                "=" +
                encodeURIComponent(data[it]) +
                "&";
            }
            return ret.substring(0, ret.length - 1);
          },
        ],
      })
        .then((res) => {
          console.log(res.data);
          this.productDetails = res.data.product_info;
          this.productPicture = res.data.product_info.product_picture;
          alert("!");
        })
        .catch((err) => {
          return Promise.reject(err);
        });
    },

    // 加入购物车
    addShoppingCart() {
      // 判断是否登录,没有登录则显示登录组件
      if (!this.$store.state.islogin) {
        this.$store.dispatch("setShowLogin", true);
        return;
      }

      this.$axios({
        method: "post",
        url: "http://localhost:80/back-end/shoppingChart.php?action=addChart",
        data: {
          user_id: this.$store.state.user_id,
          product_id: this.productID,
          num: 1,
        },
        transformRequest: [
          function(data) {
            // 将{username:111,password:111} 转成 username=111&password=111
            var ret = "";
            for (var it in data) {
              // 如果要发送中文 编码
              ret +=
                encodeURIComponent(it) +
                "=" +
                encodeURIComponent(data[it]) +
                "&";
            }
            return ret.substring(0, ret.length - 1);
          },
        ],
      })
        .then((res) => {
          this.notifySucceed("success!");
          console.log(res.data);
        })
        .catch((err) => {
          return Promise.reject(err);
        });
    },
    addCollect() {
      // 判断是否登录,没有登录则显示登录组件
      if (!this.$store.state.islogin) {
        this.$store.dispatch("setShowLogin", true);
        return;
      }
console.log(this.$store.state.user_id)
      this.$axios({
        method: "post",
        url: "http://localhost:80/back-end/collect.php?action=addCollect",
        data: {
          user_id: this.$store.state.user_id,
          product_id: this.productID,
        },
        transformRequest: [
          function(data) {
            // 将{username:111,password:111} 转成 username=111&password=111
            var ret = "";
            for (var it in data) {
              // 如果要发送中文 编码
              ret +=
                encodeURIComponent(it) +
                "=" +
                encodeURIComponent(data[it]) +
                "&";
            }
            return ret.substring(0, ret.length - 1);
          },
        ],
      })
        .then((res) => {
            this.notifySucceed(res.data.msg);
        })
        .catch((err) => {
          return Promise.reject(err);
        });
    },
  },
};
</script>
<style>
/* 头部CSS */
#details .page-header {
  height: 64px;
  margin-top: -20px;
  z-index: 4;
  background: #fff;
  border-bottom: 1px solid #e0e0e0;
  -webkit-box-shadow: 0px 5px 5px rgba(0, 0, 0, 0.07);
  box-shadow: 0px 5px 5px rgba(0, 0, 0, 0.07);
}
#details .page-header .title {
  width: 1225px;
  height: 64px;
  line-height: 64px;
  font-size: 18px;
  font-weight: 400;
  color: #212121;
  margin: 0 auto;
}
#details .page-header .title p {
  float: left;
}
#details .page-header .title .list {
  height: 64px;
  float: right;
}
#details .page-header .title .list li {
  float: left;
  margin-left: 20px;
}
#details .page-header .title .list li a {
  font-size: 14px;
  color: #616161;
}
#details .page-header .title .list li a:hover {
  font-size: 14px;
  color: #ec9d8f;
}
/* 头部CSS END */

/* 主要内容CSS */
#details .main {
  width: 1225px;
  height: 560px;
  padding-top: 30px;
  margin: 0 auto;
}
#details .main .block {
  float: left;
  width: 560px;
  height: 560px;
}
#details .el-carousel .el-carousel__indicator .el-carousel__button {
  background-color: rgba(163, 163, 163, 0.8);
}
#details .main .content {
  float: left;
  margin-left: 25px;
  width: 640px;
}
#details .main .content .name {
  height: 30px;
  line-height: 30px;
  font-size: 24px;
  font-weight: normal;
  color: #212121;
}
#details .main .content .intro {
  color: #b0b0b0;
  padding-top: 10px;
}
#details .main .content .store {
  color: #ec9d8f;
  padding-top: 10px;
}
#details .main .content .price {
  display: block;
  font-size: 18px;
  color: #ec9d8f;
  border-bottom: 1px solid #e0e0e0;
  padding: 25px 0 25px;
}
#details .main .content .price .del {
  font-size: 14px;
  margin-left: 10px;
  color: #b0b0b0;
  text-decoration: line-through;
}
#details .main .content .pro-list {
  background: #f9f9fa;
  padding: 30px 60px;
  margin: 50px 0 50px;
}
#details .main .content .pro-list span {
  line-height: 30px;
  color: #616161;
}
#details .main .content .pro-list .pro-price {
  float: right;
}
#details .main .content .pro-list .pro-price .pro-del {
  margin-left: 10px;
  text-decoration: line-through;
}
#details .main .content .pro-list .price-sum {
  color: #ec9d8f;
  font-size: 24px;
  padding-top: 20px;
}
#details .main .content .button {
  height: 55px;
  margin: 10px 0 20px 0;
}
#details .main .content .button .el-button {
  float: left;
  height: 55px;
  font-size: 16px;
  color: #fff;
  border: none;
  text-align: center;
}
#details .main .content .button .shop-cart {
  width: 340px;
  background-color: #ec9d8f;
}
#details .main .content .button .shop-cart:hover {
  background-color: #f25807;
}

#details .main .content .button .like {
  width: 260px;
  margin-left: 40px;
  background-color: #b0b0b0;
}
#details .main .content .button .like:hover {
  background-color: #757575;
}
#details .main .content .pro-policy li {
  float: left;
  margin-right: 20px;
  color: #b0b0b0;
}
/* 主要内容CSS END */
</style>
