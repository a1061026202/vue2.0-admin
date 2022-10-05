<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="addRightDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolelist"
                border
                stripe>
        <el-table-column type="expand">
          <!--eslint-disable-next-line vue/no-unused-vars-->
          <template v-slot="scope">
            <!--eslint-disable-next-line vue/valid-v-for-->
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
                    v-for="(item1, i1) in scope.row.children"
                    :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable
                        @close="removeRightById(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限和三级权限 -->
              <!-- 通过for循环,嵌套渲染二级权限 -->
              <el-col :span="19">
                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                        v-for="(item2, i2) in item1.children"
                        :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success"
                            closable
                            @close="removeRightById(scope.row, item2.id)">{{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="13">
                    <el-tag closable
                            @close="removeRightById(scope.row, item3.id)"
                            type="warning"
                            v-for="item3 in item2.children"
                            :key="item3.id">{{ item3.authName }}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称"
                         prop="roleName"></el-table-column>
        <el-table-column label="角色描述"
                         prop="roleDesc"></el-table-column>
        <el-table-column label="操作"
                         width="300px">
          <!--eslint-disable-next-line vue/no-unused-vars-->
          <template slot-scope="scope">
            <el-tooltip effect="dark"
                        content="编辑"
                        placement="top"
                        :enterable="false">
              <el-button type="primary"
                         icon="el-icon-edit"
                         size="mini"
                         @click="editRightDialog(scope.row.id)"></el-button>
            </el-tooltip>
            <el-tooltip effect="dark"
                        content="删除"
                        placement="top"
                        :enterable="false">
              <el-button type="danger"
                         icon="el-icon-delete"
                         size="mini"
                         @click="removeUser(scope.row.id)"></el-button>
            </el-tooltip>
            <el-tooltip effect="dark"
                        content="分配权限"
                        placement="top"
                        :enterable="false">
              <el-button type="warning"
                         icon="el-icon-setting"
                         size="mini"
                         @click="showSetRightDialog(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限"
               :visible.sync="setRightDialogVisible"
               width="30%"
               @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightslist"
               :props="treeProps"
               show-checkbox
               node-key="id"
               default-expand-all
               :default-checked-keys="defKeys"
               ref="treeRef"></el-tree>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色"
               :visible.sync="addRightDialogVisible"
               width="50%"
               @close="addFormClosed">
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addRuleFormRef"
               label-width="120px">
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>

        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <!-- 底部区域 -->
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addRightDialogVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="addUser">确定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色 -->
    <el-dialog title="添加角色"
               :visible.sync="editRightDialogVisible"
               width="50%"
               @close="editFormClosed">
      <el-form :model="editRightForm"
               :rules="addFormRules"
               ref="editRuleFormRef"
               label-width="120px">
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="editRightForm.roleName"></el-input>
        </el-form-item>

        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input v-model="editRightForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <!-- 底部区域 -->
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editRightDialogVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="editUpdateUser">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: '',
  data () {
    return {
      rolelist: [],
      // 控制分配权限对话框
      setRightDialogVisible: false,
      // 所有权限的数据
      rightslist: [],
      // 树形控件属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的数组
      defKeys: [],
      roleId: '',

      // 添加角色
      addRightDialogVisible: false,
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在2~15字符之间', trigger: 'blur' }
        ],
        roleDesc: [
          { message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      // 编辑角色对话框
      editRightDialogVisible: false,
      editRightForm: {}
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
    },
    // 根据id删除三级标签
    async removeRightById (role, rightId) {
      const confirmresult = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)

      if (confirmresult !== 'confirm') return this.$message.info('已取消了删除')
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      this.$message.success(res.meta.msg)

      //   this.getRolesList()
      // 删除后页面不刷新还是最新数据
      role.children = res.data
    },
    // 展示分配权限对话框
    async showSetRightDialog (role) {
      // 获取对话框id
      this.roleId = role.id
      // 获取所有权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }
      // console.log(res.data)
      // 获取到的权限数据保存到data中
      this.rightslist = res.data

      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式获取角色下三级权限的id并保存在数组中
    getLeafKeys (node, arr) {
      // 如果当前node节点不包含children属性,则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach((item) => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框关闭事件
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) return this.$message.error('修改失败')

      this.$message.success('新分配成功')
      this.setRightDialogVisible = false
      this.getRolesList()
    },
    // 判断添加用户对话框关闭清空
    addFormClosed () {
      this.$refs.addRuleFormRef.resetFields()
    },
    // 添加用户
    addUser () {
      this.$refs.addRuleFormRef.validate(async valid => {
        if (!valid) {
          this.$message.error('数据为填写完整')
          return
        }

        // 校验成功
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) return this.$message.error('添加角色失败')
        this.$message.success(res.meta.msg)
        this.addRightDialogVisible = false
        this.getRolesList()
      })
    },
    // 删除分配角色
    async removeUser (id) {
      const confirmResult = await this.$confirm(
        '是否删除该角色',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      if (confirmResult === 'confirm') {
        const { data: res } = await this.$http.delete(`roles/${id}`)
        if (res.meta.status !== 200) return this.$message.error('删除角色失败')
        this.$message.success('删除角色成功')
        this.getRolesList()
      }
    },
    // 编辑角色
    async editRightDialog (id) {
      const { data: res } = await this.$http.get(`roles/${id}`)
      if (res.meta.status !== 200) return this.$message.error('获取数据失败')
      this.editRightForm = res.data
      this.editRightDialogVisible = true
    },
    editFormClosed () {
      this.$refs.editRuleFormRef.resetFields()
    },
    // 更新编辑
    editUpdateUser () {
      this.$refs.editRuleFormRef.validate(async (valid) => {
        if (!valid) return this.$message.error('填写信息错误')
        const { data: res } = await this.$http.put(`roles/${this.editRightForm.roleId}`,
          { roleName: this.editRightForm.roleName, roleDesc: this.editRightForm.roleDesc })

        if (res.meta.status !== 200) return this.$message.error('更新数据失败')

        this.$message.success('更新用户信息成功')
        this.editRightDialogVisible = false
        this.getRolesList()
      })
    }
  },
  mounted () { }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
