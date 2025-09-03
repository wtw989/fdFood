<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch">
      <el-form-item label="排序号" prop="orderNum">
        <el-input
          v-model="queryParams.orderNum"
          placeholder="请输入排序号"
          clearable
          class="w240"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择状态" clearable class="w200">
          <el-option v-for="dict in dict.type.sys_normal_disable" :key="dict.value" :label="dict.label"
                     :value="dict.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="站点名称" prop="siteName">
        <el-input
          v-model="queryParams.siteName"
          placeholder="请输入站点名称"
          clearable
          class="w300"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button type="primary" plain icon="el-icon-plus" size="mini" @click="handleAdd">新增</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="bannerList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center"/>
      <el-table-column label="编号" align="center" width="80">
        <template slot-scope="scope">
          <span>{{ (queryParams.pageNum - 1) * queryParams.pageSize + scope.$index + 1 }}</span>
        </template>
      </el-table-column>
      <el-table-column label="图片" align="center" width="160">
        <template slot-scope="scope">
          <el-image :src="scope.row.image" fit="cover" class="banner-thumb"/>
        </template>
      </el-table-column>
      <el-table-column label="状态" align="center" width="120">
        <template slot-scope="scope">
          <span :style="{ color: scope.row.status === '0' ? '#67C23A' : '#F56C6C' }">{{
              scope.row.status === '0' ? '已上架' : '已下架'
            }}</span>
        </template>
      </el-table-column>
      <el-table-column label="排序号" prop="orderNum" align="center" width="100"/>
      <el-table-column label="配置站点" prop="siteUrl" width="220"/>
      <el-table-column label="站点名称" prop="siteName" width="200"/>
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width" width="200">
        <template slot-scope="scope">
          <el-button size="mini" type="text" icon="el-icon-delete" @click="handleDelete(scope.row)">删除</el-button>
          <el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)">修改</el-button>
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

    <!-- 添加/编辑 -->
    <el-dialog :title="title" :visible.sync="open" width="600px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="90px">
        <el-form-item label="banner图片" prop="image">
          <image-upload v-model="form.image" :limit="1" :fileSize="5" :isShowTip="false"/>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio v-for="dict in dict.type.sys_normal_disable" :key="dict.value" :label="dict.value">{{
                dict.label
              }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="排序号" prop="orderNum">
          <el-input-number v-model="form.orderNum" controls-position="right" :min="0"/>
        </el-form-item>
        <el-form-item label="站点名称" prop="siteName">
          <el-input v-model="form.siteName" placeholder="请输入站点名称"/>
        </el-form-item>
        <el-form-item label="配置站点" prop="siteUrl">
          <el-input v-model="form.siteUrl" placeholder="请输入链接地址"/>
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
import ImageUpload from '@/components/ImageUpload/index.vue'

export default {
  name: "BannerManage",
  dicts: ['sys_normal_disable'],
  components: {ImageUpload},
  data() {
    return {
      loading: true,
      ids: [],
      single: true,
      multiple: true,
      showSearch: true,
      total: 0,
      bannerList: [],
      title: "",
      open: false,
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        orderNum: undefined,
        status: undefined,
        siteName: undefined
      },
      form: {},
      rules: {
        image: [{required: true, message: '请上传banner图片', trigger: 'change'}],
        siteName: [{required: true, message: '站点名称不能为空', trigger: 'blur'}],
        siteUrl: [{required: true, message: '站点地址不能为空', trigger: 'blur'}],
        orderNum: [{required: true, message: '排序号不能为空', trigger: 'change'}]
      }
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.loading = true
      const img1 = require('@/assets/images/pay.png')
      const all = [
        {
          id: 1,
          image: img1,
          status: '0',
          orderNum: 1,
          siteUrl: 'www.hrcon.com',
          siteName: '恒都牛肉官网',
          createTime: '2025-08-01 11:28:00'
        },
        {
          id: 2,
          image: img1,
          status: '0',
          orderNum: 2,
          siteUrl: 'www.hrcon.com',
          siteName: '源小幺丰都麻辣鸡官网',
          createTime: '2024-08-01 11:28:00'
        },
        {
          id: 3,
          image: img1,
          status: '1',
          orderNum: 3,
          siteUrl: 'www.hrcon.com',
          siteName: '抓贼记丰都麻辣鸡',
          createTime: '2024-02-01 11:28:00'
        }
      ].filter(item => {
        const matchOrder = !this.queryParams.orderNum || String(item.orderNum).includes(String(this.queryParams.orderNum))
        const matchStatus = !this.queryParams.status || item.status === this.queryParams.status
        const matchName = !this.queryParams.siteName || item.siteName.includes(this.queryParams.siteName)
        return matchOrder && matchStatus && matchName
      })
      this.total = all.length
      const start = (this.queryParams.pageNum - 1) * this.queryParams.pageSize
      const end = start + this.queryParams.pageSize
      this.bannerList = all.slice(start, end)
      this.loading = false
    },
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id)
      this.single = selection.length != 1
      this.multiple = !selection.length
    },
    handleQuery() {
      this.queryParams.pageNum = 1
      this.getList()
    },
    resetQuery() {
      this.resetForm && this.resetForm('queryForm')
      this.handleQuery()
    },
    handleAdd() {
      this.reset()
      this.open = true
      this.title = '新增Banner'
    },
    handleUpdate(row) {
      this.reset()
      this.form = {...row}
      this.open = true
      this.title = '修改Banner'
    },
    handleDelete(row) {
      const ids = row.id || this.ids
      this.$modal && this.$modal.confirm
        ? this.$modal.confirm('是否确认删除编号为"' + ids + '"的数据项？').then(() => {
          this.bannerList = this.bannerList.filter(item => item.id !== row.id)
          this.total = Math.max(0, this.total - 1)
          this.$modal.msgSuccess('删除成功')
        }).catch(() => {
        })
        : (this.bannerList = this.bannerList.filter(item => item.id !== row.id))
    },
    cancel() {
      this.open = false
      this.reset()
    },
    reset() {
      this.form = {
        id: undefined,
        image: '',
        status: '0',
        orderNum: 0,
        siteUrl: undefined,
        siteName: undefined
      }
      this.resetForm && this.resetForm('form')
    },
    submitForm() {
      this.$refs['form'].validate(valid => {
        if (!valid) return
        // 取第一个图片地址
        const imageUrl = Array.isArray(this.form.image)
          ? (this.form.image[0] || '')
          : (typeof this.form.image === 'string' ? (this.form.image.split(',')[0] || '') : '')
        if (imageUrl) {
          this.form.image = imageUrl
        }
        if (this.form.id) {
          const index = this.bannerList.findIndex(i => i.id === this.form.id)
          if (index !== -1) this.$set(this.bannerList, index, {...this.bannerList[index], ...this.form})
        } else {
          const maxId = Math.max(0, ...this.bannerList.map(i => i.id || 0))
          const fallbackImg = require('@/assets/images/pay.png')
          const toAdd = {
            ...this.form,
            id: maxId + 1,
            image: this.form.image || fallbackImg,
            createTime: this.parseTime ? this.parseTime(new Date()) : new Date().toISOString()
          }
          this.bannerList.unshift(toAdd)
          this.total += 1
        }
        this.$modal && this.$modal.msgSuccess && this.$modal.msgSuccess('保存成功')
        this.open = false
        this.getList()
      })
    }
  }
}
</script>


<style scoped lang="scss">
.w240 {
  width: 240px;
}

.w200 {
  width: 200px;
}

.w300 {
  width: 300px;
}

.banner-thumb {
  width: 120px;
  height: 90px;
}
</style>
