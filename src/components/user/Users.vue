/<template>
  <div>
      <!--面包屑-->
      <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>库存模块</el-breadcrumb-item>
        <el-breadcrumb-item>仓库管理</el-breadcrumb-item>
      </el-breadcrumb>
      <!--卡片Main-->
      <el-card>
        <el-row :gutter="20">
            <el-col :span="8">
              <el-input placeholder="请输入内容" v-model="queryInfo.ckName" clearable @clear="getUserList">
                <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
              </el-input>
            </el-col>
            <el-col :span="4">
                <el-button type="primary" @click="addDialogVisible = true">添加仓库</el-button>
            </el-col>
        </el-row>
        <!--表格-->
        <el-table :data="userlist" border stripe style="margin-top: 20px">
          <el-table-column label="仓库id" prop="id" v-if="false"></el-table-column>
          <el-table-column width="70%" type="index" label="序号"></el-table-column>
          <el-table-column label="仓库编号" prop="ckCode"></el-table-column>
          <el-table-column label="仓库名称" prop="ckName"></el-table-column>
          <el-table-column label="仓库负责人" prop="ckFzr"></el-table-column>
          <el-table-column label="负责人电话" prop="ckPhone"></el-table-column>
          <el-table-column label="仓库地址" prop="ckAdress"></el-table-column>
          <el-table-column label="状态">
            <template slot-scope="scope">
              <el-switch v-model="scope.row.status" @change="userStateChanged(scope.row)"></el-switch>
            </template>
          </el-table-column>
          <el-table-column label="操作">
              <template slot-scope="scope">
                <div style="display: none"> {{scope.row}} </div>
              <!--编辑按钮-->
              <el-tooltip effect="dark" content="编辑" placement="top">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDiaLog(scope.row.id)"></el-button>
              </el-tooltip>
              <!--删除按钮-->
              <el-tooltip effect="dark" content="删除" placement="top">
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.ckCode)"></el-button>
              </el-tooltip>
              <!--分配角色按钮-->
              <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
        <!--分页-->
        <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pageNum"
        :page-sizes="[3, 5, 10, 100]"
        :page-size="queryInfo.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
        </el-pagination>
      </el-card>
      <!--添加用户弹窗-->
      <el-dialog
      title="添加仓库"
      :visible.sync="addDialogVisible"
      width="30%" @close="addDialogClosed">
      <!--主体区域-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
      <el-form-item label="仓库编号" prop="ckCode" label-width="100px">
      <el-input v-model="addForm.ckCode"></el-input>
      </el-form-item>
      <el-form-item label="仓库名称" prop="ckName" label-width="100px">
      <el-input v-model="addForm.ckName"></el-input>
      </el-form-item>
      <el-form-item label="父级仓库" prop="ckParent" label-width="100px">
      <el-input v-model="addForm.ckParent"></el-input>
      </el-form-item>
      </el-form>
      <!--底部区域-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
      </el-dialog>
      <!--修改用户对话框-->
      <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="30%" @close="editDialogClosed">
      <!--主体区域-->
      <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="70px">
      <el-form-item label="仓库编号" label-width="100px" prop="ckCode">
      <el-input v-model="editForm.ckCode" disabled></el-input>
      </el-form-item>
      <el-form-item label="仓库名称" label-width="100px" prop="ckName">
      <el-input v-model="editForm.ckName"></el-input>
      </el-form-item>
      <el-form-item label="父级仓库" label-width="100px" prop="ckParent">
      <el-input v-model="editForm.ckParent"></el-input>
      </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <el-button @click="editDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
      </el-dialog>

  </div>
</template>

<script>
export default {
    data() {
      // var checkMobile = (rule, value, cb) => {
      //   const regMobile = /^(0|86|17951)?(13[0-9]|15[012345678]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      //   if (regMobile.test(value)) {
      //     return cb()
      //   }
      //   cb(new Error('请输入合法的手机号'))
      // }
        return {
            queryInfo: {
                ckCode: '',
                ckName: '',
                pageNum: 1,
                pageSize: 5
            },
            editInfo: {
              id: ''
            },
            userlist: [],
            total: 0,
            editForm: {
              ckName: '',
              ckCode: '',
              ckParent: ''
            },
            delete: {
              ckCode: ''
            },
            addDialogVisible: false,
            editDialogVisible: false,
            addForm: {
              ckName: '',
              ckCode: '',
              ckParent: ''
            },
            addFormRules: {
              ckName: [
                { required: true, message: '请输入仓库名称', trigger: 'blur' },
                { min: 2, max: 10, message: '仓库名称长度在2-10个字符之间', trigger: 'blur' }
              ],
              ckCode: [
                { required: true, message: '请输入仓库编号', trigger: 'blur' },
                { min: 2, max: 10, message: '仓库编号长度在2-10个字符之间', trigger: 'blur' }
              ],
              ckParent: [
                { required: true, message: '请输入父级仓库编号', trigger: 'blur' },
                { min: 1, max: 10, message: '父级仓库编号长度在1-10个字符之间', trigger: 'blur' }
              ]
              // mobile: [
              //   { required: true, message: '请输入父级仓库编号', trigger: 'blur' },
              //   { validator: checkMobile, trigger: 'blur' }
              // ]
            }
        }
    },
    created() {
        this.getUserList()
    },
    methods: {
        async getUserList() {
            const { data: res } = await this.$http.post('zl/ListOfCk', this.queryInfo);
              if (res.code !== 200) {
                  return this.$message.error(res.msg);
              }
            this.userlist = res.data.list
            this.total = res.data.total
        },
        handleSizeChange(newSize) {
          console.log(newSize)
          this.queryInfo.pageSize = newSize
          this.getUserList()
        },
        handleCurrentChange(newPage) {
          console.log(newPage)
          this.queryInfo.pageNum = newPage
          this.getUserList()
        },
        addDialogClosed() {
          console.log(this.$refs)
          this.$refs.addFormRef.resetFields()
        },
        addUser() {
          this.$refs.addFormRef.validate(async valid => {
            console.log(valid)
            if (!valid) return
            const { data: res } = await this.$http.post('zl/addCk', this.addForm)
            if (res.code !== 200) {
              return this.$message.error(res.msg)
            }
            this.$message.success(res.msg)
            this.addDialogVisible = false
            this.getUserList()
          })
        },
        async showEditDiaLog(id) {
          console.log(this.$refs)
          console.log(id)
          this.editInfo.id = id
          const { data: res } = await this.$http.post('zl/CkxxById', this.editInfo)
          if (res.code !== 200) {
            return this.$message.error(res.msg)
          }
          this.editForm = res.data
          this.editDialogVisible = true
        },
        editDialogClosed() {
          this.$refs.editFormRef.resetFields()
        },
        editUserInfo() {
          this.$refs.editFormRef.validate(async valid => {
            if (!valid) return
            const { data: res } = await this.$http.post('zl/updateCk', this.editForm)
            if (res.code !== 200) {
              return this.$message.error(res.msg)
            }
            this.editDialogVisible = false
            this.getUserList()
            this.$message.success(res.msg)
          })
        },
        async removeUserById(ckCode) {
          this.delete.ckCode = ckCode
          const confirmResult = await this.$confirm('此操作将永久删除该仓库, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        }
        const { data: res } = await this.$http.post('/zl/deleteCk', this.delete)
        if (res.code !== 200) {
          return this.$message.error(res.msg)
        }
         this.$message.success(res.msg)
         this.getUserList()
        }
        // async userStateChanged(userinfo) {
        //   console.log(userinfo)
        //   const { data: res } = await this.$http.put('zl/ListOfCk');
        //   if (res.code !== 200) {
        //     userinfo.status = !userinfo.statusre
        //     return this.$message.error('更新失败！')
        //   }
        //   this.$message.success('更新成功！')
        // }
    }
}
</script>

<style>

</style>
