<template>
  <div class="pos">
     <el-row>
       <el-col :span="7" class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border style="width:100%">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="count" label="数量" width="50"></el-table-column>
              <el-table-column prop="price" label="金额" width="70"></el-table-column>
              <el-table-column label="操作" width="100" fixed="right">
                <template scope="scope">
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="total" v-show="tableData.length>0">
              <small>总数量：</small>{{totalCount}}&nbsp;&nbsp;份<small>总计：</small>￥{{totalMoney}}元
            </div>
            <div class="div-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods">取消订单</el-button>
              <el-button type="success" @click="checkout">结账</el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">
            挂单
          </el-tab-pane>
          <el-tab-pane label="外卖">
            外卖
          </el-tab-pane>
        </el-tabs>
       </el-col>
       <el-col :span="17">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul>
              <li v-for="(goods,index) in oftenGoods" :key="index" @click="addOrderList(goods)">
                <span>{{goods.goodsName}}</span>
                <span class="o-price">￥{{goods.price}}</span>
              </li>
            </ul>
          </div>
        </div>
        <div class="goods-type">
          <el-tabs>
            <el-tab-pane label="汉堡">
              <ul class="cookList">
                <li v-for="(goods,index) in type0Goods" :key="index" @click="addOrderList(goods)">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="小食">
              <ul class="cookList">
                <li v-for="(goods,index) in type1Goods" :key="index" @click="addOrderList(goods)">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <ul class="cookList">
                <li v-for="(goods,index) in type2Goods" :key="index" @click="addOrderList(goods)">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <ul class="cookList">
                <li v-for="(goods,index) in type3Goods" :key="index" @click="addOrderList(goods)">
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
</template>

<script>
import axios from "axios"
export default {
  data () {
    return {
      totalCount:0,
      totalMoney:0,
      tableData:[],
      oftenGoods:[],
      type0Goods:[],
      type1Goods:[],
      type2Goods:[],
      type3Goods:[]
    }
  },
  created(){
    axios.get('http://jspang.com/DemoApi/oftenGoods.php').then(response=>{
      //console.log(response);
      this.oftenGoods=response.data;
    }).catch(error=>{
      alert("网络出错")
    });
    axios.get('http://jspang.com/DemoApi/typeGoods.php').then(response=>{
      //console.log(response);
      this.type0Goods=response.data[0];
      this.type1Goods=response.data[1];
      this.type2Goods=response.data[2];
      this.type3Goods=response.data[3];

    }).catch(error=>{
      alert("网络错误");
    })
  },
  mounted(){
    const orderHeight=document.body.clientHeight;
    //console.log(orderHeight);
    document.getElementById("order-list").style.height=orderHeight+"px";
  },
  methods:{
    //添加订单列表的方法
    addOrderList(goods){
      
      let isHave=false;
      //判断是否已存在订单
      for(let i=0;i<this.tableData.length;i++){
        if(this.tableData[i].goodsId==goods.goodsId){
          isHave=true;
        }
      }
      //根据isHave判断订单是否存在此商品
      if(isHave){
        let arr=this.tableData.filter(o=>o.goodsId==goods.goodsId);
        arr[0].count++;
      }else{
        let newGoods={goodsId:goods.goodsId,goodsName:goods.goodsName,price:goods.price,count:1};
        this.tableData.push(newGoods);
      }
      this.getAllMoney();
    },
    //删除所有订单
    delAllGoods(){
      this.tableData=[];
      this.totalCount=0;
      this.totalMoney=0;
    },
    //单品删除
    delSingleGoods(goods){
      this.tableData=this.tableData.filter(o=>o.goodsId!=goods.goodsId);
      this.getAllMoney();
    },
    //结账
    checkout(){
      if(this.totalCount!=0){
        this.tableData=[];
        this.totalCount=0;
        this.totalMoney=0;
        this.$message({
          message:"结账成功!!!",
          type:'success'
        })
      }else{
        this.$message.error("您没有选择商品!!无法结账!!!");
      }
    },
    //总计
    getAllMoney(){
      this.totalCount=0;
      this.totalMoney=0;
      //进行数量和金额汇总
      this.tableData.forEach(ele=>{
        this.totalCount+=ele.count;
        this.totalMoney+=ele.price*ele.count;
      })
    }
  }
}
</script>

<style scoped>
.pos{
  font-size: 12px;
}
.pos-order{
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
}
.div-btn{
  margin-top: 20px;
}
.title{
  padding: 10px;
  height: 20px;
  border-bottom: 1px solid #D3DCE6;
  background-color: #f9fafc;
  text-align: left
}
.often-goods-list{
  overflow: hidden;
  padding-bottom: 10px;
  background-color: #fff;
  border-bottom: 1px solid #C0CCDA;
}
.often-goods-list ul li{
   float: left;
   margin: 5px;
   padding: 10px;
   list-style: none; 
   border: 1px solid #e5e9f2;
   background-color: #fff;
   cursor: pointer
}
.o-price{
  color: #58b7ff;
}
.goods-type{
  clear: both;
}
.cookList li{
  float: left;
  list-style: none;
  margin: 2px;
  padding: 2px;
  width: 23%;
  height: auto;
  background-color: #fff;
  border: 1px solid #e5e9f2;
  cursor: pointer
}
.foodImg{
  width:40%;
}
.foodName{
  font-size: 18px;
  padding-left: 10px;
  color: brown;
}
.foodPrice{
  font-size: 16px;
  padding-top: 10px;
  padding-left: 10px;
}
.total{
  padding-right: 10px;
  text-align: right;
  height: 30px;
  line-height: 30px;
  border-bottom: 1px solid #d3d4d6;
}
</style>