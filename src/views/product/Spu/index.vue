<template lang="">
    <div>
        <el-card style="margin:20px 0px">
          <CategorySelect 
          @getCategoryId="getCategoryId" 
          :show="scene != 0"
          ></CategorySelect>
        </el-card>
        <div v-show="scene == 0">
        <el-card>
            <div v-show="scene == 0">
              <el-button 
            type="primary" 
            icon="el-icon-plus" :disabled="!category3Id" @click="addSpu">
            添加SPU</el-button>
          <el-table style="width:100%" border :data="records">
                <el-table-column
                    type="index"
                    label="序号"
                    width="80"
                    align="center">
                </el-table-column>
                <el-table-column
                    prop="spuName"
                    label="SPU名称"
                    width="width">
                </el-table-column>   
                <el-table-column
                    prop="description"
                    label="SPU描述"
                    width="width">
                </el-table-column> 
                <el-table-column
                    prop="prop"
                    label="操作"
                    width="width">
                    <template slot-scope="{row,$index}">
                      <HintButton
                        type="success" 
                        icon="el-icon-plus" 
                        size="mini"
                        title="添加sku"
                        @click="addSku(row)" 
                        ></HintButton>
                      <HintButton 
                        type="warning" 
                        icon="el-icon-edit" 
                        size="mini"
                        title="修改spu"
                        @click="updateSpu(row)"
                        ></HintButton>
                        <HintButton 
                         type="info" 
                        icon="el-icon-info" 
                        size="mini"
                        title="查看当前spu全部sku列表"
                        @click="handler(row)" 
                      ></HintButton>
                      <el-popconfirm
                title="这是一段内容确定删除吗？"
                @onConfirm="deleteSpu(row)"
              >
                <hint-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  title="删除spu"
                  slot="reference"
                ></hint-button>
              </el-popconfirm>
                    </template>
                </el-table-column>     
          </el-table>
          <el-pagination
              style="text-align: center"
              :current-page="page"
              :page-sizes="[3, 5, 10]"
              :page-size="limit"
              layout="prev, pager, next, jumper,->, sizes,total"
              @current-change="getSpuList"
              @size-change="handleSizeChange"
              :total="total">
        </el-pagination>
            </div>
        <SpuForm v-show="scene == 1" @changeScene="changeScene" ref="spu"></SpuForm>
        <SkuForm v-show="scene == 2" @changeScenes="changeScenes" ref="sku" ></SkuForm>
        </el-card>
        <el-dialog 
        :title="`${spu.spuName}的sku列表`" 
        :visible.sync="dialogTableVisible"
        :before-close="close">
              <el-table :data="skuList" style="width:100%" border v-loading="loading">
                <el-table-column prop="skuName" label="名称" width="width"></el-table-column>
                <el-table-column prop="price" label="价格" width="width"></el-table-column>
                <el-table-column prop="weight" label="重量" width="width"></el-table-column>
                <el-table-column label="默认图片" width="width">
                  <template slot-scope="{row,$index}">
                     <img :src="row.skuDefaultImg" alt="" style="width:100px;height:100px;">
                  </template>
                </el-table-column>
              </el-table>
        </el-dialog>        
        </div>
    </div>
</template>
<script>
import SpuForm from './SpuForm';
import SkuForm from './SkuForm';
export default {
  name: 'Spu',
  data(){
    return {
      category1Id:'',
      category2Id:'',
      category3Id:'',
      show:true,
      page:1,
      limit:3,
      records:[],
      total:0,
      scene:0,
      isShowTable:true,
      spu:{},
      skuList:[],
      dialogTableVisible: false,
      loading:true,
      attrInfo: {
        attrName: "", //属性名
        attrValueList: [
          //属性值，因为属性值可以有多个因此用数组，每一个属性值都是一个对象需要attrId，valueName
        ],
        categoryId: 0, //三级分类的id
        categoryLevel: 3, //因为服务器也需要区分几级id
      },
    }
  },
  methods: {
    handleCurrentChange(page) {
        this.page = page;
        this.getSpuList();
    },
    getCategoryId({categoryId,level}){
        if (level == 1) {
            this.category1Id = categoryId;
            this.category2Id='';
            this.category3Id='';
        } else if (level == 2) {
            this.category2Id = categoryId;
            this.category3Id='';
        } else {
            this.category3Id = categoryId;
            this.getSpuList();
        }
    },
    async getSpuList(pages = 1){
      this.page = pages; 
      const {page,limit,category3Id} = this;
      let result = await this.$API.spu.reqSpuList(page,limit,category3Id);
      if(result.code == 200) {
        this.total = result.data.total;
        this.records = result.data.records;
      }
    },
    handleSizeChange(limit){
      this.limit = limit;
      this.getSpuList();
    },
    addSpu(){
      this.scene = 1;
      this.$refs.spu.addSpuData(this.category3Id);
    },
    updateSpu(row){
      this.scene = 1;
      this.$refs.spu.initSpuData(row);
    },
    changeScene({ scene, flag }) {
      //flag这个形参为了区分保存按钮是添加还是修改
      //切换结构（场景）
      this.scene = scene;
      //子组件通知父组件切换场景，需要再次获取SPU列表的数据进行展示
      if (flag == "修改") {
        this.getSpuList(this.page);
      } else {
        this.getSpuList();
      }
    },
    changeScenes(scene) {
      this.scene = scene;
    },
    async deleteSpu(row) {
      let result = await this.$API.spu.reqDeleteSpu(row.id);
      if (result.code == 200) {
        this.$message({ type: "success", message: "删除成功" });
        //代表SPU个数大于1删除的时候停留在当前页，如果SPU个数小于1 回到上一页
        this.getSpuList(this.records.length > 1 ? this.page : this.page - 1);
      }
    },
    addSku(row) {
      //切换场景为2
      this.scene = 2;
      //父组件调用子组件的方法，让子组件发请求------三个请求
      this.$refs.sku.getData(this.category1Id, this.category2Id, row);
    },
    async handler(spu) {
      //点击这个按钮的时候，对话框可见的
      this.dialogTableVisible = true;
      //保存spu信息
      this.spu = spu;
      //获取sku列表的数据进行展示
      let result = await this.$API.spu.reqSkuList(spu.id);
      if (result.code == 200) {
        this.skuList = result.data;
        //loading隐藏
        this.loading = false;
      }
    },
    //关闭对话框的回调
    close(done){
      //loading属性再次变为真
      this.loading = true;
      //清除sku列表的数据
      this.skuList = [];
      //管理对话框
      done();
    },
    cancel(){
      this.$emit('changeScenes',0);
      Object.assign(this._data,this.$options.data())
    }
  }, 
  components:{
    SpuForm,
    SkuForm
  }
}
</script>
<style lang=""> 
</style>
