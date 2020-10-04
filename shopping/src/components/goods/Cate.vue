<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{path:'/home'}">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加分类区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table :data='catelist' class="TreeTable"
      :columns='columns' 
      :selection-type='false' 
      :expand-type='false'
      show-index
      index-text="#"
      border
      :show-row-hover='false'>
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" 
          v-if='scope.row.cat_deleted === false' 
          style="color:lightgreen"></i>
          <i class="el-icon-error"
          v-else style="color: red"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page.sync="querInfo.pagenum"
      :page-sizes="[3,5,10,15]"
      :page-size="querInfo.pagesize"
      layout="total,sizes,prev, pager,next,jumper"
      :total="total">
    </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
    title="添加分类"
    :visible.sync="addCateDialogVisible"
    width="50%">
      <el-form :model="addCateForm" 
      :rules="addcateFormRules" 
      ref="addCateFormRef" 
      label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCateDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //商品分类的数据列表，默认为空
      catelist: [],
      //商品分类的数据总数
      total: 0,
      //为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          //表示，当前列定义为模板列
          type: 'template',
          //表示当前这一列使用模板名称
          template: 'isok'
        },
        {
          label: '排序',
          //表示，当前列定义为模板列
          type: 'template',
          //表示当前这一列使用模板名称
          template: 'order'
        },
        {
          label: '操作',
          //表示，当前列定义为模板列
          type: 'template',
          //表示当前这一列使用模板名称
          template: 'opt'
        }],

      //控制添加分类对话框的显示与隐藏
      addCateDialogVisible:false,
      //添加分类的表单
      addCateForm: {
        //将要添加的分类的名称
        cat_name: '',
        //父级的分类ID
        cat_Pid: 0,
        //当前默认要添加的分类等级，默认为一级
        cat_level:0
      },
      //添加分类表单的验证
      addcateFormRules: {
        cat_name: [
          { required: true, messsge: '请输入分类名称',trigger: 'blur'}
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取商品分类数据
    async getCateList() {
      const{data: res} = await this.$http.get('categories',{ params:  this.querInfo})
      if(res.meta.status !== 200) return this.$messsge.error('获取商品分类失败')

      this.catelist = res.data.result
      //为总数据条数赋值
      this.total = res.data.total
    },

    //监听pagesize改变
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },

    //监听pagenum改变
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },

    showAddCateDialog() {
      this.addCateDialogVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.TreeTable{
  margin-top: 10px;
}
</style>