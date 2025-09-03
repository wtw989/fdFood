<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch">
      <el-form-item label="分类名称" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="请输入分类名称"
          clearable
          class="w240"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="请选择状态"
          clearable
          class="w240"
        >
          <el-option
            v-for="dict in dict.type.sys_normal_disable"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
        >新增
        </el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="classifyList" @selection-change="handleSelectionChange">
      <el-table-column label="编号" align="center" width="80">
        <template slot-scope="scope">
          <span>{{ (queryParams.pageNum - 1) * queryParams.pageSize + scope.$index + 1 }}</span>
        </template>
      </el-table-column>
      <el-table-column label="分类名称" align="center" prop="name" :show-overflow-tooltip="true"/>
      <el-table-column label="产品数量" align="center" prop="productCount" width="100"/>
      <el-table-column label="状态" align="center" prop="status" width="100">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_normal_disable" :value="scope.row.status"/>
        </template>
      </el-table-column>
      <el-table-column label="排序" align="center" prop="orderNum" width="100"/>
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            :icon="scope.row.status === '0' ? 'el-icon-close' : 'el-icon-check'"
            @click="toggleStatus(scope.row)"
          >{{ scope.row.status === '0' ? '禁用' : '启用' }}
          </el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
          >修改
          </el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
          >删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改分类对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="分类名称" prop="name">
          <el-input v-model="form.name" placeholder="请输入分类名称"/>
        </el-form-item>
        <el-form-item label="排序" prop="orderNum">
          <el-input-number v-model="form.orderNum" controls-position="right" :min="1"/>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio
              v-for="dict in dict.type.sys_normal_disable"
              :key="dict.value"
              :label="dict.value"
            >{{ dict.label }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>

</template>

<script>
export default {
  name: "ProductClassify",
  dicts: ['sys_normal_disable'],
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 列表数据
      classifyList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        name: undefined,
        status: undefined
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        name: [
          {required: true, message: "分类名称不能为空", trigger: "blur"}
        ],
        orderNum: [
          {required: true, message: "排序不能为空", trigger: "blur"}
        ]
      }
    }
  },
  created() {
    this.getList()
  },
  methods: {
    /** 查询列表（此处使用临时数据以匹配页面样式） */
    getList() {
      this.loading = true
      // 模拟后端分页过滤
      const all = [
        {id: 1, name: '恒都牛肉', productCount: 1, status: '0', orderNum: 1, createTime: '2025-09-01 11:28:00'},
        {id: 2, name: '丰都麻辣鸽', productCount: 4, status: '0', orderNum: 2, createTime: '2025-09-01 11:28:00'},
        {id: 3, name: '香菇梆菜', productCount: 2, status: '0', orderNum: 3, createTime: '2025-09-01 11:28:00'}
      ].filter(item => {
        const matchName = !this.queryParams.name || item.name.includes(this.queryParams.name)
        const matchStatus = !this.queryParams.status || item.status === this.queryParams.status
        return matchName && matchStatus
      })
      this.total = all.length
      const start = (this.queryParams.pageNum - 1) * this.queryParams.pageSize
      const end = start + this.queryParams.pageSize
      this.classifyList = all.slice(start, end)
      this.loading = false
    },
    // 取消按钮
    cancel() {
      this.open = false
      this.reset()
    },
    // 表单重置
    reset() {
      this.form = {
        id: undefined,
        name: undefined,
        orderNum: 0,
        status: '0'
      }
      this.resetForm && this.resetForm("form")
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1
      this.getList()
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm && this.resetForm("queryForm")
      this.handleQuery()
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id)
      this.single = selection.length != 1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset()
      this.open = true
      this.title = "添加分类"
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset()
      this.form = {...row}
      this.open = true
      this.title = "修改分类"
    },
    /** 切换启用禁用 */
    toggleStatus(row) {
      row.status = row.status === '0' ? '1' : '0'
      this.$forceUpdate()
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (!valid) return
        if (this.form.id) {
          // 更新本地数据
          const index = this.classifyList.findIndex(i => i.id === this.form.id)
          if (index !== -1) this.$set(this.classifyList, index, {...this.classifyList[index], ...this.form})
        } else {
          // 本地新增（仅前端示例）
          const maxId = Math.max(0, ...this.classifyList.map(i => i.id || 0))
          const toAdd = {
            ...this.form,
            id: maxId + 1,
            productCount: 0,
            createTime: this.parseTime ? this.parseTime(new Date()) : new Date().toISOString()
          }
          this.classifyList.unshift(toAdd)
          this.total += 1
        }
        this.$modal && this.$modal.msgSuccess && this.$modal.msgSuccess("保存成功")
        this.open = false
        this.getList()
      })
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids
      this.$modal && this.$modal.confirm
        ? this.$modal.confirm('是否确认删除编号为"' + ids + '"的数据项？').then(() => {
          this.classifyList = this.classifyList.filter(item => item.id !== row.id)
          this.total = Math.max(0, this.total - 1)
          this.$modal.msgSuccess("删除成功")
        }).catch(() => {
        })
        : (this.classifyList = this.classifyList.filter(item => item.id !== row.id))
    }
  }
}
</script>


<style scoped lang="scss">
.app-container {
}

.w240 {
  width: 240px;
}
</style>


