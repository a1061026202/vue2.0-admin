<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-alert title="注意：只允许为第三级分类设置相关参数"
                type="warning"
                :closable="false"
                show-icon>
      </el-alert>

      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类: </span>
          <el-cascader v-model="selectedKeys"
                       :options="catelist"
                       :props="cascaderProps"
                       @change="parentCateChanged"
                       clearable></el-cascader>
        </el-col>
      </el-row>

      <!-- tab 页签区 -->
      <el-tabs v-model="activeName"
               @tab-click="handleTabClick">
        <!-- 添加动态参数面板 -->
        <el-tab-pane label="动态参数"
                     name="many">
          <el-button type="primary"
                     size="mini"
                     :disabled="isBtnDisabled"
                     @click="addDialogVisible = true">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData"
                    border
                    stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <!--eslint-disable-next-line vue/no-unused-vars-->
              <template slot-scope="scope">
                <el-tag v-for="(item, i) in scope.row.attr_vals"
                        :key="i"
                        closable
                        @close="handleClosed(i,scope.row)">{{ item }}</el-tag>
                <el-input v-if="scope.row.inputVisible"
                          class="input-new-tag"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="small"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>

            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <!--eslint-disable-next-line vue/no-unused-vars-->
              <template slot-scope="scope">
                <el-button type="primary"
                           size="mini"
                           icon="el-icon-edit"
                           @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger"
                           size="mini"
                           icon="el-icon-delete"
                           @click="removeAttr(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的参数 -->
        <el-tab-pane label="静态属性"
                     name="only">
          <el-button type="primary"
                     size="mini"
                     :disabled="isBtnDisabled"
                     @click="addDialogVisible = true">添加属性</el-button>
          <el-table :data="onlyTableData"
                    border
                    stripe>
            <el-table-column type="expand">
              <!--eslint-disable-next-line vue/no-unused-vars-->
              <template slot-scope="scope">
                <el-tag v-for="(item, i) in scope.row.attr_vals"
                        :key="i"
                        closable
                        @close="handleClosed(i,scope.row)">{{ item }}</el-tag>
                <el-input v-if="scope.row.inputVisible"
                          class="input-new-tag"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="mini"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>

            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <!--eslint-disable-next-line vue/no-unused-vars-->
              <template slot-scope="scope">
                <el-button type="primary"
                           size="mini"
                           icon="el-icon-edit"
                           @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger"
                           size="mini"
                           icon="el-icon-delete"
                           @click="removeAttr(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加对话框 -->
    <el-dialog :title="'添加' + tittleText"
               :visible.sync="addDialogVisible"
               width="50%"
               @close="addDialogClosed">
      <!-- 添加参数 -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="100px">
        <el-form-item :label="tittleText + ':'"
                      prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addDialogVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="addParams">确定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑对话框 -->
    <el-dialog :title="'修改' + tittleText"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="editDialogClosed">
      <!-- 添加参数 -->
      <el-form :model="editForm"
               :rules="addFormRules"
               ref="editFormRef"
               label-width="100px">
        <el-form-item :label="tittleText + ':'"
                      prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="editParams">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 商品分类列表
      catelist: [],
      selectedKeys: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 被激活页签的名称
      activeName: 'many',
      // 动态参数数据
      manyTableData: [],
      // 静态参数数据
      onlyTableData: [],
      // 对话框
      addDialogVisible: false,
      // 添加参数表单数据对象
      addForm: {},
      // 添加表单的验证规则对象
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 控制修改对话框的显示和隐藏
      editDialogVisible: false,
      // 编辑表单数据
      editForm: {}
    }
  },
  created () {
    this.getCateList()
  },
  computed: {
    // 如果按钮需要禁用则返回true
    isBtnDisabled () {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类的id
    cateId () {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    // 判断添加动态/静态
    tittleText () {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    }
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取数据失败')
      this.catelist = res.data
    },
    // 级联选择框选中的项变化会触发
    parentCateChanged () {
      this.getParamsData()
    },
    // Tab 页签点击事件的处理函数
    handleTabClick () {
      this.getParamsData()
    },
    // 获取参数的列表数据
    async getParamsData () {
      // 证明选择的三级分类
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // 根据所选的分类id，和当前所处的面板获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) return this.$message.error('获取数据失败')

      res.data.forEach((item) => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })

      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 监听对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮添加参数
    addParams () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          { attr_name: this.addForm.attr_name, attr_sel: this.activeName }
        )

        if (res.meta.status !== 201) return this.$message.error('添加参数失败')
        this.$message.success('添加参数成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 点击按钮展示修改对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${id}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) return this.$message.error('获取参数失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听修改对话框关闭
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 点击确定更新编辑对话框
    editParams () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return this.$message.error('属性填写有无,请重新填写')

        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.editForm.attr_sel
          }
        )

        if (res.meta.status !== 200) return this.$message.error('修改失败')

        this.$message.success('修改成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 删除
    async removeAttr (id) {
      const confirmResult = await this.$confirm('是否删除该参数', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)
      if (confirmResult === 'confirm') {
        const { data: res } = await this.$http.delete(
          `categories/${this.cateId}/attributes/${id}`
        )
        if (res.meta.status !== 200) return this.$message.error('删除失败')
        this.$message.success('删除成功')
        this.getParamsData()
      }
    },
    // 文本框的失去焦点或按下回车都会触发
    handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // 如果没有return ，则需要做后续内容
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false

      // 需要发起请求保存数据
      this.saveAttrvals(row)
    },
    // 点击按钮显示文本输入框
    showInput (row) {
      row.inputVisible = true
      // 获取文本框的焦点
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 将attr_vals 的操作，保存到数据库
    async saveAttrvals (row) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')

        })
      if (res.meta.status !== 200) return this.$message.error('操作失败')
      this.$message.success('操作成功')
    },
    // 删除蓝色标签
    handleClosed (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrvals(row)
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
