<template>
  <div>
    <!--    面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

<!--    卡片视图区域-->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
<!--      表格-->
      <tree-table :data="catelist"
                  :columns="columns"
                  :selection-type="false"
                  :expand-type="false"
                  show-index
                  index-text="#"
                  border
                  :show-row-hover="false"
                  class="treeTable">
<!--        是否有效-->
        <template slot="isok" slot-scope="scope">
          <i style="color: lightgreen" v-if="scope.row.cat_deleted === false" class="el-icon-success"></i>
          <i style="color: red" v-else class="el-icon-error"></i>
        </template>
<!--        排序-->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
<!--        操作-->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showeditCateDialog(scope.row)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCate(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>
<!--      分页区域-->
      <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="querInfo.pagenum"
          :page-sizes="[3, 5, 10, 15]"
          :page-size="querInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
      </el-pagination>
    </el-card>
<!--    添加分类的对话框-->
    <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%"
        @close="addCateDialogClosed">
<!--      添加分类的表单-->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父类分级:">
<!--          options指定数据源-->
<!--          v-model必须绑定数组-->
          <el-cascader
              v-model=" selectedKeys"
              :options="parentCateList"
              :props="cascaderProps"
              @change="parentCateChange"
              clearable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
    </el-dialog>
<!--    编辑页面-->
    <el-dialog title="修改分类" :visible.sync="editCateDialogVisible" width="50%" @close="editCateDialogClosed">
      <el-form :model="editCate" :rules="editCateRules" ref="editCateRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCate.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCateInfo">确 定</el-button>
      </span>
    </el-dialog>
<!--  删除页面-->

  </div>
</template>

<script>
export default {
  name: "Cate",
//  商品分类的列表
  data() {
    return {
      //查询条件
      querInfo:{
        type:3,
        pagenum: 1,
        pagesize: 5
      },
      //商品分类的数据列表
      catelist:[],
      //总数据条数
      total:0,
      //为table指定列的定义
      columns:[
          {
            label:'分类名称',
            prop:'cat_name'
          },
          {
            label:'是否有效',
            //表示将当前列定义为模板列
            type:'template',
            //表示当前这一列使用的模板名称
            template:'isok'
          },
          {
            label:'排序',
            //表示将当前列定义为模板列
            type:'template',
            //表示当前这一列使用的模板名称
            template:'order'
          },
          {
            label:'操作',
            //表示将当前列定义为模板列
            type:'template',
            //表示当前这一列使用的模板名称
            template:'opt'
          }
      ],
      //控制添加分类对话框的显示与隐藏
      addCateDialogVisible:false,
      //添加分类的表单数据对象
      addCateForm:{
        //将要添加的分类名称
        cat_name:'',
        //父级分类的id
        cat_pid: 0,
        //当前分类的层级 分类等级默认为1级分类
        cat_level: 0
      },
      //添加分类表单的验证规则对象
      addCateFormRules:{
        cat_name:[
            {required:true,message:'请输入分类名称',trigger:'blur'}
        ]
      },
      //父级分类的列表
      parentCateList:[],
      cascaderProps:{
        expandTrigger: 'hover',
        value:'cat_id',
        label:'cat_name',
        children:'children',
        checkStrictly:true
      },
      //选中的父级分类id数组
      selectedKeys:[],
      //编辑页面显示与隐藏
      editCateDialogVisible:false,
      //查询到的角色信息对象
      editCate:{},
      // 添加分类表单的验证规则对象
      editCateRules: {
        cat_name: [{ required: true, message: '请输入要修改的信息', trigger: 'blur' }]
      },
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取商品分类数据
    async getCateList() {
      const{data:res} = await this.$http.get('categories',{params:this.querInfo})
      if(res.meta.status !==200) {
        return this.$message.error('获取商品分类失败')
      }
      //把数据列表赋值给catelist
      this.catelist = res.data.result
      //为总数据条数赋值
      this.total = res.data.total
    },
    //监听pagesize改变的事件
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    //监听pagenum的改变
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    //点击按钮展示添加分类的对话框
    showAddCateDialog() {
      //先获取父级分类的列表然后再显示对话框
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    //获取父级分类的数据列表
    async getParentCateList() {
      const {data:res} = await this.$http.get('categories',{params:{type:2}})
      if(res.meta.status !==200 ) {
        return this.$message.error('获取父级分类数据失败！')
      }
      this.parentCateList = res.data
    },
    //选择项发生变化触发该函数
    parentCateChange() {
      //如果selectedKeys数组中的length大于0，证明选中了父级分类，否则，说明没有选中任何父级分类
      if(this.selectedKeys.length > 0) {
        //父级分类的id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length-1]
        //为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      }
      else {
        //父级分类的id
        this.addCateForm.cat_pid = 0
        //为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      }
    },
    //点击按钮添加新的分类
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if(!valid) return
        const{data:res} = await this.$http.post('categories',this.addCateForm)
        if(res.meta.status !==201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    //监听对话框的关闭事件
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    //修改分类信息并提交
    async editCateInfo() {
      const { data: res } = await this.$http.put('categories/' + this.editCate.cat_id, { cat_name: this.editCate.cat_name })
      if (res.meta.status !== 200) {
        return this.$message.error('更新分类数据失败!')
      }
      this.$message.success('更新分类数据成功!')
      this.getCateList()
      this.editCateDialogVisible = false
    },
    //打开编辑页面时获取数据
    async showeditCateDialog(cateInfo) {
      this.editCateId = cateInfo.cat_id
      const { data: res } = await this.$http.get('categories/' + cateInfo.cat_id)
      this.editCate = res.data
      console.log(this.editCate)
      // console.log(res.data)
      this.editCateDialogVisible = true
    },
    //监听编辑对话框关闭事件
    editCateDialogClosed() {
      this.$refs.editCateRef.resetFields()
    },
    //删除事件获取数据
    async removeCate(id) {
      const confirRustle = await this.$confirm('此操作将永久删除该文件, 是否继续?', '删除分类', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirRustle !== 'confirm') {
        return this.$message.info('已取消删除操作!')
      }
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('分类删除失败!')
      }
      this.$message.success('该分类已经成功删除!')
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader{
  width: 100%;
}
</style>