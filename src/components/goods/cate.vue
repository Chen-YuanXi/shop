<template>
    <div>
        <!-- 面包屑区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>商品管理</el-breadcrumb-item>
          <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- 表格 -->
            <tree-table class="treeTable" :data="catelist" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border :show-row-hover="false">
                <!-- 是否有效 -->
                <template slot="isok" slot-scope="scope">
                  <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen;"></i>
                  <i class="el-icon-error" v-else style="color: red;"></i>
                </template>
                <!-- 排序 -->
                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag size="mini" v-else-if="scope.row.cat_level === 1" type="success">二级</el-tag>
                    <el-tag size="mini" v-else type="warning">三级</el-tag>
                </template>
                <!-- 操作 -->
                <template slot="opt" slot-scope="scope">
                  <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>  
                  <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>  
                </template>
            </tree-table>
            <!-- 分页 -->
            <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="queryInfo.pageNum"
              :page-sizes="[3, 5, 10, 15]"
              :page-size="queryInfo.pageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加分类的对话框 -->
        <el-dialog
          title="添加分类"
          :visible.sync="addCatDialogVisible"
          width="50%"
          @close="addCatDialogClose">
          <el-form ref="addCateFormRef" :rules="addCateFormRules" :model="addCateForm" label-width="80px">
            <el-form-item label="分类名称" prop="cat_name">
              <el-input v-model="addCateForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类">
              <el-cascader
                expand-trigger="hover"
                :options="parentCateList"
                :props="cascaderProps"
                v-model="selectedKeys"
                @change="parentCateChanged"
                clearable
                change-on-select>
              </el-cascader>
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
    data(){
        return{
            // 查询条件
            queryInfo:{
                type:3,
                pagenum:1,
                pagesize:5
            },
            catelist:[],
            total:0,
            // 为table指定列的定义
            columns:[
                {
                    label:'分类名称',
                    prop:'cat_name'
                },
                {
                    label:'是否有效',
                    // 表示将当前定义为模板列
                    type:'template',
                    // 表示当前这一列使用模板名称
                    template:'isok'
                },
                {
                    label:'排序',
                    // 表示将当前定义为模板列
                    type:'template',
                    // 表示当前这一列使用模板名称
                    template:'order'
                },
                {
                    label:'操作',
                    // 表示将当前定义为模板列
                    type:'template',
                    // 表示当前这一列使用模板名称
                    template:'opt'
                }
            ],
            // 控制添加分类对话框
            addCatDialogVisible:false,
            addCateForm:{
                cat_name:'',
                cat_pid:0,
                cat_level:0
            },
            addCateFormRules: {
              cat_name: [
                { required: true, message: '请输入分类名称', trigger: 'blur' }
              ]
            },
            parentCateList:[],
            cascaderProps:{
                value:'cat_id',
                label:'cat_name',
                children:'children'
            },
            selectedKeys:[]
        }
    },
    created(){
        this.getCateList()
    },
    methods:{
        // 获取商品分类数据
        async getCateList(){
            const {data:res} = await this.$http.get('categories',{params:this.queryInfo})
            if(res.meta.status !== 200){
                return this.$message.error('获取商品分类数据失败')
            }
            this.catelist = res.data.result
            this.total = res.data.total
        },
        handleSizeChange(newSize){
            this.queryInfo.pagesize = newSize
            this.getCateList()
        },
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage
            this.getCateList()
        },
        showAddCateDialog(){
            this.getParentCateList()
            this.addCatDialogVisible = true
        },
        async getParentCateList(){
            const {data:res} = await this.$http.get('categories',{params:{type:2}})
            if(res.meta.status !== 200){
                return this.$message('获取分类数据列表失败')
            }
            this.parentCateList = res.data
            console.log(this.parentCateList);
            
        },
        parentCateChanged(){
            if(this.selectedKeys.length >0){
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length-1]
                this.addCateForm.cat_level = this.selectedKeys.length
                return
            }else {
                 this.addCateForm.cat_pid = 0
                 this.addCateForm.cat_level = 0
            }

        },
        addCate(){
            this.$refs.addCateFormRef.validate(async valid=>{
                if(!valid) return
                const {data:res} = await this.$http.post('categories',this.addCateForm)
                if(res.meta.status !== 201){
                    // console.log(this.$message.error);  
                    return this.$message.error('添加失败');
                }
                this.$message.success('添加成功')
                this.getCateList();
                this.addCatDialogVisible = false;
            })         
        },
        addCatDialogClose(){
            this.$refs.addCateFormRef.resetFields()
            this.selectedKeys= []
            this.addCateForm.cat_pid = 0
            this.addCateForm.cat_level = 0
        }
    }
}
</script>

<style lang="less" scoped>
.treeTable{
    margin-top: 15px
}
.el-cascader{
    width: 100%;
}
</style>

