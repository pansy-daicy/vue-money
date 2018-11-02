<template>
    <div class="pos">
        <div>
            <el-row>
                <el-col :span="7" class="pos-order" id="order-list">
                    <el-tabs>
                        <el-tab-pane label="点餐">
                            <el-table :data="tableData" border style="width:100%">
                                <el-table-column prop="goodsName" label="商品"></el-table-column>
                                <el-table-column prop="count" label="量"></el-table-column>
                                <el-table-column prop="price" label="金额"></el-table-column>
                                <el-table-column label="操作" width="100" fixed="right">
                                    <template scope="scope">
                                        <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                                        <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                                    </template>
                                </el-table-column>
                            </el-table>
                            <div class="totalDiv">
                                <small>数量：{{totalCount}}</small>
                                <strong></strong>&nbsp;&nbsp;&nbsp;&nbsp;
                                <small>统计：{{totalMoney}}</small>
                                <strong></strong>元
                            </div>
                            <div class="order-btn">
                                <el-button type="warning">挂单</el-button>
                                <el-button type="danger" @click="delAllGoods()">删除</el-button>
                                <el-button type="success" @click="checkout()">结账</el-button>
                            </div>
                        </el-tab-pane>
                        <el-tab-pane label="挂单"></el-tab-pane>
                        <el-tab-pane label="外卖"></el-tab-pane>
                    </el-tabs>
                </el-col>
                <!-- 商品展示 -->
                <el-col :span="17">
                    <div class="often-goods">
                        <div class="title">常用商品</div>
                        <div class="often-goods-list">
                            <ul>
                                <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                                    <span>{{goods.goodsName}}</span>
                                    <span class="o-price">￥{{goods.price}}元</span>
                                </li>
                            </ul>
                        </div>
                    </div>
                    <div class="goods-type">
                        <el-tabs>
                            <el-tab-pane label="汉堡">
                                <ul class='cookList'>
                                    <li v-for="goods in type0Goods" @click="addOrderList(goods)">
                                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                        <span class="foodName">{{goods.goodsName}}</span>
                                        <span class="foodPrice">￥{{goods.price}}元</span>
                                    </li>
                                </ul>
                            </el-tab-pane>
                            <el-tab-pane label="小食">
                                <ul class='cookList'>
                                    <li v-for="goods in type1Goods" @click="addOrderList(goods)">
                                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                        <span class="foodName">{{goods.goodsName}}</span>
                                        <span class="foodPrice">￥{{goods.price}}元</span>
                                    </li>
                                </ul>
                            </el-tab-pane>
                            <el-tab-pane label="饮料">
                                <ul class='cookList'>
                                    <li v-for="goods in type2Goods" @click="addOrderList(goods)">
                                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                        <span class="foodName">{{goods.goodsName}}</span>
                                        <span class="foodPrice">￥{{goods.price}}元</span>
                                    </li>
                                </ul>
                            </el-tab-pane>
                            <el-tab-pane label="套餐">
                                <ul class='cookList'>
                                    <li v-for="goods in type3Goods" @click="addOrderList(goods)">
                                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                        <span class="foodName">{{goods.goodsName}}</span>
                                        <span class="foodPrice">￥{{goods.price}}元</span>
                                    </li>
                                </ul>
                            </el-tab-pane>
                        </el-tabs>
                    </div>
                </el-col>
            </el-row>
        </div>
    </div>
</template>
<script>
import axios from 'axios'
export default {
  name: "Pos",
  data() {
    return {
      tableData:[],
      oftenGoods:[],
      type0Goods:[],
      type1Goods:[],
      type2Goods:[],
      type3Goods:[],
      totalMoney: 0, //订单总价格
      totalCount: 0  //订单商品总数量
    };
  },
 mounted() {
    this.$ajax({
      method: "get",
      url: "../static/oftenGoods.json"
    })
      .then(response => {
        let _data = response.data;
       
        this.oftenGoods= _data.oftenGoods;
          console.log(this.oftenGoods);
      })
      .catch(function(err) {
        console.log(err);
      });
      this.$ajax({
      method: "get",
      url: "../static/typeGoods.json"
    })
      .then(response => {
        let _data = response.data;
        this.type0Goods= _data.type0Goods;
        this.type1Goods= _data.type1Goods;
        this.type2Goods= _data.type2Goods;
        this.type3Goods= _data.type3Goods;
         
      })
      .catch(function(err) {
        console.log(err);
      });
  },
  methods:{
    //   添加订单列表的方法
    addOrderList(goods){
        this.totalCount = 0;
        this.totalMoney = 0;
        let isHave = false;
        for(let i = 0; i < this.tableData.length; i++){
            if(this.tableData[i].goodsId == goods.goodsId){
                isHave = true;
            }
        }
        if(isHave) {
            let arr =  this.tableData.filter(o => o.goodsId == goods.goodsId);
            arr[0].count++;
        }else {
            let newGoods = {goodsId:goods.goodsId, goodsName: goods.goodsName, price: goods.price, count: 1}
             this.tableData.push(newGoods);
        }
         this.getAllMoney();
    },
    // 删除单个商品
    delSingleGoods(goods) {
        this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId);
         this.getAllMoney();
    },
     //删除所有商品
        delAllGoods() {
            this.tableData = [];
            this.totalCount = 0;
            this.totalMoney = 0;
        },
    // 汇总数量和金额
    getAllMoney(){
        this.totalCount = 0;
        this.totalMoney = 0;
        if(this.tableData) {
            this.tableData.forEach((element) => {
                 this.totalCount += element.count;
                  this.totalMoney = this.totalMoney + (element.price * element.count);
            })
        }
    },
    // 结账方法模拟
    checkout(){
        if(this.totalCount !=0){
            this.tableData = [];
                this.totalCount = 0;
                this.totalMoney = 0;
                this.$message({
                    message:'结账成功，感谢你又为店里出了一份力!',
                    type:'success'
                })
        }else{
             this.$message.error('不能空结。老板了解你急切的心情！');
        }
    }
  }
};
</script>
<style scoped>
.pos {
  font-size: 12px;
}
.pos-order {
  background-color: #fff;
  border-right: 1px solid #c0ccda;
  padding-left: 10px;
  padding-right: 10px;
}
.order-btn {
  margin-top: 10px;
  text-align: center;
}
.title {
  height: 20px;
  border-bottom: 1px solid #d3dce6;
  background-color: #f9fafc;
  padding: 10px;
}
.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #e5e9f2;
  padding: 10px;
  margin: 5px;
  background-color: #fff;
}
.goods-type {
  clear: both;
  padding: 10px;
}
.o-price {
  color: #58b7ff;
}
.often-goods-list {
    border-bottom: 1px solid #C0CCDA;
    height: auto;
    overflow: hidden;
    padding-bottom: 10px;
    background-color: #F9FAFC;
}
.cookList li {
    list-style:none;
    width:23%;
    border:1px solid #E5E9F2;
    height: auto;
    overflow: hidden;
    background-color: #fff;
    padding: 2px;
    float: left;
    margin: 2px;
    cursor: pointer;
}
.cookList li span {

    display: block;
    float: left;
}

.foodImg {
    width: 40%;
}

.foodName {
    font-size: 18px;
    padding-left: 10px;
    color: brown;
}
.foodPrice {
    font-size: 16px;
    padding-left: 10px;
    padding-top: 10px;
}

.totalDiv {
    height: auot;
    overflow: hidden;
    text-align: right;
    font-size: 16px;
    background-color: #fff;
    border-bottom: 1px solid #E5E9F2;
    padding: 10px;
}

</style>

