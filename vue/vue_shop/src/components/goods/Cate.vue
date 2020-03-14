<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-check" v-if="scope.row.cat_deleted === false" style="color:lightgreen;"></i>
          <i class="el-icon-close" v-else style="color:red;"></i>
        </template>
        <!-- 分级 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" v-else-if="scope.row.cat_level === 1" type="success">二级</el-tag>
          <el-tag size="mini" v-else type="warning">三级</el-tag>
        </template>
        <!-- 排序 -->
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
          <el-button type="primary" icon="el-icon-delete" size="mini"></el-button>
        </template>
      </tree-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCatDialogVisible" width="50%" @close="addCatDialogClosed">
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称: " prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类: ">
            <!-- options指定数据源 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{ expandTrigger: 'hover' ,value : 'cat_id',label :'cat_name',children:'children',checkStrictly:'true'}"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCatDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //商品分类的数据列表，默认为空
      catelist: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      columns: [
        {
          label: "分类名称",
          prop: "cat_name"
        },
        {
          label: "是否有效",
          //表示当前列定义为模板列
          type: "template",
          // 表示当前这一列使用的模板名称
          template: "isok"
        },
        {
          label: "排序",
          //表示当前列定义为模板列
          type: "template",
          // 表示当前这一列使用的模板名称
          template: "order"
        },
        {
          label: "操作",
          //表示当前列定义为模板列
          type: "template",
          // 表示当前这一列使用的模板名称
          template: "opt"
        }
      ],
      //控制添加分类对话框显示与隐藏
      addCatDialogVisible: false,
      //添加分类的表单数据对象
      addCateForm: {
        //将要添加的分类的名称
        cat_name: "",
        //父级分类的ID
        cat_pid: 0,
        //分类添加的等级，默认一级
        cat_level: 0
      },
      //添加分类表单的 验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" }
        ]
      },
      //父级分类的列表
      parentCateList: [],
      //选中父级分类的ID数组
      selectedKeys:[]
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo
      });
      if (res.meta.status !== 200)
        return this.$message.error("获取商品分类失败");
      //把数据列表赋值给cateList
      //   this.$message.success("获取商品分类成功");
      // console.log(res.data);
      this.catelist = res.data.result;
      this.total = res.data.total;
    },
    //监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    //监听pagenum的改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    //点击按钮，显示添加分类的对话框
    showAddCateDialog() {
      //先获取父级分类的列表
      this.getParentCateList();
      this.addCatDialogVisible = true;
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 }
      });
      if (res.meta.status !== 200)
        return this.$message.error("获取父级分类失败");
      this.parentCateList = res.data;
    },
    parentCateChanged(){
        //如果selectedKeys数组中的length大于0，证明选中的父级分类,反之，就说明没有选中任何父级分类
        console.log(this.selectedKeys);
        if(this.selectedKeys.length > 0){
            // 父级分类ID
            this.addCateForm.cat_pid =this.selectedKeys[this.selectedKeys.length -1];
            // 为当前分类的等级赋值
            this.addCateForm.cat_level = this.selectedKeys.length;
            return;
        }
        else{
             // 父级分类ID
            this.addCateForm.cat_pid = 0;
            // 为当前分类的等级赋值
            this.addCateForm.cat_level = 0;
        }
    },
    addCate(){
        this.$refs.addCateFormRef.validate(async valid=>{
            if(!valid) return
            const{data:res}= await this.$http.post('categories',this.addCateForm);
            if(res.meta.status !== 201)
                return this.$message.error('添加分类失败')
            this.$message.success('添加分类成功');
            this.getCateList();
            this.addCatDialogVisible = false;
        })
    },
    //监听对话框的关闭事件，重置表单数据
    addCatDialogClosed(){
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = [];
        this.addCateForm.cat_level = 0;
        this.addCateForm.cat_pid = 0;
    }
  }
};
</script>

<style lang="less" scoped>
.el-button {
  margin-bottom: 10px;
}
.el-cascader{
    width: 100%;
}
</style>