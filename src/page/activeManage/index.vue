<template>
  <div class="allActive">
    <el-row class="head">
      <el-col :span="18" style="padding:20px 0;background: #33CCCC;">
        <el-col :span="8" @click.native="showTable('init')">
          {{ totalActiveNum }}<br>
          <span>数据总数</span>
        </el-col>

        <el-col :span="8" style="border: none;">
          {{ auditNum }}<br>
          <span>待下载</span>
        </el-col>
        <el-col :span="8" @click.native="showTable('state')">
        {{ stateData }}<br>
        <span>状态异常数据</span>
        </el-col>
      </el-col>

      <el-col :span="5" @click.native="showTable('rstate')" :offset="1" style="background:  #3091F2;padding:40px 0;">
        {{ rstateData }}<br>
        <span>入库异常数据</span>
      </el-col>
    </el-row>

    <el-card class="box-card">
      <h3>一级数据列表</h3>
      <el-row type="flex" align="middle" :gutter="20" style="padding:20px 0;">
        <el-col :span="5" style="width: 160px;text-align: center;">
          已选择{{ activeNum }}条数据
        </el-col>
        <el-col :span="5" style="width: 140px;">
        <!--  <el-select v-model="currentType" placeholder="请选择活动分类">
            <el-option
              v-for="type in types"
              :value="type">
            </el-option>
          </el-select>  -->
        </el-col>
        <el-col :span="13">
          <!--<el-button :plain="true" type="info">设置活动分类</el-button>-->
          <el-button :plain="true" @click.native="handleRemove" type="danger">删除</el-button>
          <el-button :plain="true" @click.native="handleMoveToTop" type="info">置顶</el-button>
          <el-button :plain="true" @click.native="handleCopy" type="info">复制</el-button>
        </el-col>
      </el-row>

      <el-table :data="tableData"  style="width: 100%"
        border
        stripe
        max-height="550"
                @cell-click="handleSelect" @selection-change="selectionchange"
                >

        <el-table-column type="selection" width="50" ></el-table-column>
        <el-table-column property="satelliteID" label="卫星" ></el-table-column>
        <el-table-column property="sensorID" label="传感器"></el-table-column>
        <el-table-column property="productID" label="产品序列号" ></el-table-column>
        <el-table-column property="orderid" label="订单号"></el-table-column>
        <el-table-column property="produceTime" label="订单时间" width="200" ></el-table-column>
        <el-table-column property="state" label="处理"></el-table-column>
        <el-table-column property="rstate" label="入库"></el-table-column>
      <!--  <el-table-column property="sceneID" label="景序列号"></el-table-column>
        <el-table-column property="productLevel" label="产品级别"></el-table-column>  -->
        <el-table-column inline-template label="操作" align="center" property="id">
          <el-button type="text" size="mini" @click.native="">查看详情</el-button>
        </el-table-column>

      <!--   <el-table-column property="rspath" label="产品名" width="300"></el-table-column>
         <el-table-column property="gfpath" label="存放路径" width="300"></el-table-column> -->
      </el-table>

      <el-row type="flex" justify="end" style="padding:20px 0; ">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="page"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="pageSize"
          layout="sizes, prev, pager, next"
          :total="total">
        </el-pagination>
      </el-row>
    </el-card>


  </div>
</template>
<script>
  import $ from 'jquery'
  import axios from 'axios'

  export default {
    name: 'allActive',
    data: function () {
      return {
        page: 1,
        pageSize: 20,
        total: 200,

        totalActiveNum: 3,
        auditNum: 15,
        stateData:0,
        rstateData:0,

        stateTable:'init',
        activeNum: 0,
        totalSignUp: 204,
        currentType: '全部',
        types: ['全部','测试活动','免费活动','收费活动'],
        selectItems:[],
        tableData: [],
      }
    },
    computed:{

    },
    created: function(){
      // 组件创建完后获取数据，
      // 此时 data 已经被 observed 了
      this.showTable('init');
    },
    methods: {
      handleSelect: function (row, column, cell, event) {
       if (column.label == '操作') {
         this.$router.push('/activeManage/detail/page1');
        } else if(column.type == 'selection'){
            row.$info = !row.$selected;
       }else{
            row.$selected = !row.$selected;
            row.$info = row.$selected;
       }
      },
      selectionchange: function (val) {
        var arr = [];
        val.forEach(function (item) {
            arr.push(item.id);
        });
        this.selectItems = arr;
        this.activeNum = this.selectItems.length;
      },
      handleRemove:function(){
        var tableData = this.tableData;
        this.selectItems.forEach(function (id) {
          tableData.forEach(function (data) {
              if(id == data.id){
                tableData.splice(tableData.indexOf(data),1)
              }
          })
        });
        this.selectItems = [];
      },
      //请求数据
      axiosreq:function(params,show){
        if(!params || $.isEmptyObject(params)){
          return;
        }
        var _this = this;   //很重要！！
        var url="http://127.0.0.1:7777/rsdatas";
        //清除 空参数
        for(var key in params){if(!params[key]) delete params[key];}
        axios.get(url, {
          params: params
        }).then(function (res) {
          //绑定表格 分页数据
          var data = res.data;
          _this.tableData = data.data;

          _this.page = params.page;
          _this.pageSize = params.pageSize;
          _this.total = data.total;

          console.log(show);
          if(show){
            _this.totalActiveNum = data.total;
            _this.auditNum =  data.pendingDownload;
            _this.stateData =  data.stateData;
            _this.rstateData =  data.rstateData;
          }
          // console.log( _this.tableData, _this.totalActiveNum, _this.page, _this.pageSize, _this.total)
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      //分页
      handleSizeChange:function(val) {
        this.pagesize = val;
        // console.log(this.page);
        this.chooseTable(1,val,'produceTime',this.stateTable,false);
      },
      handleCurrentChange:function(val) {
        this.page = val;
        // console.log(val);
        this.chooseTable(val,this.pagesize,'produceTime',this.stateTable,false);
      },

      showTable:function(state){
        this.stateTable = state;
        //是否刷新展示数据
        var show = state == 'init' ? true : false;
        this.chooseTable(1,20,'produceTime',state,show);
      },
      /**
       * 返回资源数据
       * @param page 分页页码 默认为1
       * @param pageSize 当前页展示多少条数据 默认为20
       * @param sort 排序 默认以 produceTime 倒序
       * @param state table展示状态 默认为 init 初始化加载全部数据 ，state 加载 state = 3 的数据，rstate 加载 rstate = 101 的数据
       * @param show 展示数据是否刷新 默认为 true 刷新展示数据
       */
      chooseTable:function(page,pageSize,sort,state,show){
        //添加 分页、 参数、排序
        var param = {};
        param.page = page||1;
        param.pageSize = pageSize||20;
        param.sort = sort||'produceTime';

        console.log(show);
        if(state == 'state'){
          param.state = 3;
        }else if(state == 'rstate'){
          param.rstate = 101;
        }
        this.axiosreq(param,show);
      }
    }
  }
</script>
<style>
  .allActive > .head > .el-col > .el-col {
    padding: 20px 0;
    border-right: solid 1px #fff;
  }

  .allActive .head {
    text-align: center;
    color: #fff;
    font-size: 30px;
    margin-bottom: 50px;
  }

  .allActive .head span {
    font-size: 16px;
  }
</style>
