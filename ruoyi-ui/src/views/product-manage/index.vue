<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch">
      <el-form-item label="产品名称" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="请输入产品名称"
          clearable
          class="w260"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="布局展示" prop="layout">
        <el-select v-model="queryParams.layout" placeholder="请选择布局展示" clearable class="w200">
          <el-option v-for="opt in layoutOptions" :key="opt" :label="opt" :value="opt"/>
        </el-select>
      </el-form-item>
      <el-form-item label="产品分类" prop="category">
        <el-select v-model="queryParams.category" placeholder="请选择产品分类" clearable class="w200">
          <el-option v-for="c in categoryOptions" :key="c" :label="c" :value="c"/>
        </el-select>
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择状态" clearable class="w200">
          <el-option v-for="dict in dict.type.sys_normal_disable" :key="dict.value" :label="dict.label"
                     :value="dict.value"/>
        </el-select>
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
      <el-col :span="1.5">
        <el-button type="success" plain icon="el-icon-upload2" size="mini" @click="handleImport">导入</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="productList" @selection-change="handleSelectionChange">
      <el-table-column label="编号" align="center" width="80">
        <template slot-scope="scope">
          <span>{{ (queryParams.pageNum - 1) * queryParams.pageSize + scope.$index + 1 }}</span>
        </template>
      </el-table-column>
      <el-table-column label="图片" align="center" width="110">
        <template slot-scope="scope">
          <el-image :src="scope.row.image" fit="cover" class="product-thumb"/>
        </template>
      </el-table-column>
      <el-table-column label="产品名称" prop="name" :show-overflow-tooltip="true" width="220"/>
      <el-table-column label="产品分类" prop="category" width="120"/>
      <el-table-column label="生效链接" width="240">
        <template slot-scope="scope">
          <el-tag size="mini" type="danger" class="mr4">{{ scope.row.tag }}</el-tag>
          <span>{{ scope.row.link }}</span>
        </template>
      </el-table-column>
      <el-table-column label="其他平台" width="160">
        <template slot-scope="scope">
          <el-tag v-for="p in scope.row.platforms" :key="p" size="mini" class="mr4">{{ p }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="总点击数" prop="clicks" align="center" width="100">
        <template slot-scope="scope">
          <el-link type="primary" :underline="false" @click="openClicksDialog(scope.row)">{{ scope.row.clicks }}</el-link>
        </template>
      </el-table-column>
      <el-table-column label="置顶" align="center" width="100">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.weightOn" @change="onWeightChange(scope.row)"/>
        </template>
      </el-table-column>
      <el-table-column label="布局展示" prop="layout" align="center" width="100"/>
      <el-table-column label="状态" align="center" width="90">
        <template slot-scope="scope">
          <span :style="{ color: scope.row.status === '0' ? '#67C23A' : '#F56C6C' }">{{
              scope.row.status === '0' ? '上架' : '下架'
            }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)">编辑</el-button>
          <el-button size="mini" type="text" icon="el-icon-delete" @click="handleDelete(scope.row)">删除</el-button>
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

    <!-- 添加/编辑产品对话框（完整） -->
    <el-dialog :title="title" :visible.sync="open" width="820px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="100px">
        <el-row :gutter="16">
          <el-col :span="14">
            <el-form-item label="产品名称" prop="name">
              <el-input v-model="form.name" placeholder="请输入产品名称"/>
            </el-form-item>
          </el-col>
          <el-col :span="10">
            <el-form-item label="产品分类" prop="category">
              <el-select v-model="form.category" placeholder="请选择产品分类" filterable>
                <el-option v-for="c in categoryOptions" :key="c" :label="c" :value="c"/>
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="16">
          <el-col :span="14">
            <el-form-item label="产品分类" prop="category">
              <el-select v-model="form.category" placeholder="请选择产品分类" filterable>
                <el-option v-for="c in categoryOptions" :key="c" :label="c" :value="c"/>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="10" class="flex-center">
            <el-form-item label="产品置顶" prop="pinned">
              <el-switch v-model="form.pinned"/>
              <div class="tip-text">置顶的产品将显示在列表最前面</div>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="16" type="flex" align="middle">
          <el-col :span="14">
            <el-form-item label="前端布局展示" prop="layout">
              <el-radio-group v-model="form.layout">
                <el-radio :label="'2*1'">2*1布局</el-radio>
                <el-radio :label="'1*1'">1*1布局</el-radio>
              </el-radio-group>
              <div class="tip-text">2x1：独占一行显示两张图片；1x1：占一行一个位置显示一张图片</div>
            </el-form-item>
          </el-col>
          <el-col :span="10">
            <div class="mb6">布局预览：</div>
            <div v-if="form.layout==='2*1'" class="layout-preview layout-2x1">
              <div class="images">
                <div class="preview-item">图1</div>
                <div class="preview-item">图2</div>
              </div>
              <div class="preview-title">产品名称</div>
            </div>
            <div v-else class="layout-preview layout-1x1">
              <div class="preview-1x1">图</div>
              <div class="preview-title">产品名称</div>
            </div>
          </el-col>
        </el-row>

        <el-row :gutter="16">
          <el-col :span="14">
            <el-form-item label="产品图片" prop="image">
              <el-upload
                class="avatar-uploader"
                action="#"
                :show-file-list="false"
                :http-request="handleImageUpload"
                :before-upload="beforeImageUpload"
                accept="image/png,image/jpg,image/jpeg"
              >
                <img v-if="form.image" :src="form.image" class="avatar"/>
                <div v-else class="avatar-placeholder">
                  <i class="el-icon-plus f20"/>
                </div>
              </el-upload>
              <div class="tip-text">请上传图片大小为约400*400px的png,jpg,jpeg文件，大小在20M以内</div>
            </el-form-item>
          </el-col>

        </el-row>

        <el-divider>第三方平台配置</el-divider>
        <div v-for="(cfg, idx) in form.platformConfigs" :key="idx" class="mb12">
          <el-row :gutter="12" type="flex" align="middle">
            <el-col :span="6">
              <el-select v-model="cfg.platform" placeholder="请选择平台">
                <el-option v-for="p in platformOptions" :key="p" :label="p" :value="p"/>
              </el-select>
            </el-col>
            <el-col :span="10">
              <el-input v-model="cfg.link" placeholder="请输入链接地址"/>
            </el-col>
            <el-col :span="4">
              <div class="row-center">
                <span class="mr8">启用:</span>
                <el-switch v-model="cfg.enabled"/>
              </div>
            </el-col>
            <el-col :span="4" class="text-right">
              <el-button type="text" @click="removePlatformRow(idx)" icon="el-icon-delete"/>
            </el-col>
          </el-row>
        </div>
        <el-button type="primary" icon="el-icon-plus" plain @click="addPlatformRow" class="w100">添加平台链接
        </el-button>

        <el-form-item label="状态" prop="status" class="mt16">
          <el-radio-group v-model="form.status">
            <el-radio v-for="dict in dict.type.sys_normal_disable" :key="dict.value" :label="dict.value">{{
                dict.label
              }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

    <!-- 点击数详情对话框 -->
    <el-dialog :title="'统计详情'" :visible.sync="clicksDialogVisible" width="720px" append-to-body>
      <div class="stat-header">
        <div class="stat-title">{{ clicksProduct.name }}</div>
        <div class="stat-total">总点击数：<span class="stat-total-num">{{ clicksTotal }}</span></div>
        <div class="stat-filters">
          <el-date-picker
            v-model="clicksDateStart"
            type="date"
            size="small"
            value-format="yyyy-MM-dd"
            placeholder="开始日期"
            class="mr8"
          />
          <span class="mr8">至</span>
          <el-date-picker
            v-model="clicksDateEnd"
            type="date"
            size="small"
            value-format="yyyy-MM-dd"
            placeholder="结束日期"
            class="mr8"
          />
          <el-button size="small" @click="resetClicksFilters">重 置</el-button>
        </div>
      </div>

      <el-table :data="clicksDisplayList" stripe size="mini" :default-sort="{prop: 'date', order: 'descending'}">
        <el-table-column label="日期" prop="date" width="200" sortable/>
        <el-table-column label="点击数" prop="count" align="left" sortable>
          <template slot-scope="scope">
            <el-link type="primary" :underline="false">{{ scope.row.count }}</el-link>
          </template>
        </el-table-column>
      </el-table>

      <div slot="footer" class="dialog-footer">
        <el-button @click="clicksDialogVisible = false">关 闭</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "ProductManage",
  dicts: ['sys_normal_disable'],
  data() {
    return {
      loading: true,
      ids: [],
      single: true,
      multiple: true,
      showSearch: true,
      total: 0,
      productList: [],
      title: "",
      open: false,
      layoutOptions: ['1*1', '2*1'],
      categoryOptions: ['恒都牛肉', '丰都麻辣鸡', '香菇梆菜'],
      platformOptions: ['京东', '抖音', '小红书', '小程序'],
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        name: undefined,
        layout: undefined,
        category: undefined,
        status: undefined
      },
      form: {},
      rules: {
        name: [{required: true, message: '产品名称不能为空', trigger: 'blur'}],
        category: [{required: true, message: '产品分类不能为空', trigger: 'change'}],
        image: [{required: true, message: '产品图片不能为空', trigger: 'change'}],
      },
      // 点击数弹窗相关
      clicksDialogVisible: false,
      clicksDetailList: [],
      clicksProduct: {},
      clicksTotal: 0,
      clicksDateStart: undefined,
      clicksDateEnd: undefined
    }
  },
  computed: {
    clicksDisplayList() {
      // 过滤并按日期倒序
      const start = this.clicksDateStart
      const end = this.clicksDateEnd
      const inRange = (d) => {
        if (!start && !end) return true
        if (start && d < start) return false
        if (end && d > end) return false
        return true
      }
      return [...(this.clicksDetailList || [])]
        .filter(i => inRange(i.date))
        .sort((a, b) => (a.date < b.date ? 1 : a.date > b.date ? -1 : 0))
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.loading = true
      const img = require('@/assets/images/test.png')
      const all = [
        {
          id: 1,
          image: img,
          name: '恒都京东直营 旗舰店',
          category: '恒都牛肉',
          tag: '淘宝',
          link: 'https://taobao.com/apple',
          platforms: ['京东', '抖音'],
          clicks: 241,
          weightOn: true,
          layout: '2*1',
          status: '0'
        },
        {
          id: 2,
          image: img,
          name: '源小幺丰都麻辣鸡 工厂总店',
          category: '丰都麻辣鸡',
          tag: '京东',
          link: 'https://jd.com/apple',
          platforms: ['小红书', '抖音'],
          clicks: 741,
          weightOn: true,
          layout: '1*1',
          status: '0'
        },
        {
          id: 3,
          image: img,
          name: '抓贼记 丰都麻辣鸡',
          category: '丰都麻辣鸡',
          tag: '抖音',
          link: 'https://v.douyin.com',
          platforms: ['小红书', '京东'],
          clicks: 141,
          weightOn: false,
          layout: '1*1',
          status: '1'
        },
        {
          id: 4,
          image: img,
          name: '香菇梆菜',
          category: '香菇梆菜',
          tag: '小程序',
          link: '#小程序://8VzDvInn8',
          platforms: ['小红书', '京东'],
          clicks: 441,
          weightOn: false,
          layout: '2*1',
          status: '0'
        }
      ].filter(item => {
        const matchName = !this.queryParams.name || item.name.includes(this.queryParams.name)
        const matchLayout = !this.queryParams.layout || item.layout === this.queryParams.layout
        const matchCat = !this.queryParams.category || item.category === this.queryParams.category
        const matchStatus = !this.queryParams.status || item.status === this.queryParams.status
        return matchName && matchLayout && matchCat && matchStatus
      })
      this.total = all.length
      const start = (this.queryParams.pageNum - 1) * this.queryParams.pageSize
      const end = start + this.queryParams.pageSize
      this.productList = all.slice(start, end)
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
      this.title = '新增产品'
    },
    handleUpdate(row) {
      this.reset()
      this.form = {...row}
      this.open = true
      this.title = '编辑产品'
    },
    handleDelete(row) {
      const ids = row.id || this.ids
      this.$modal && this.$modal.confirm
        ? this.$modal.confirm('是否确认删除编号为"' + ids + '"的数据项？').then(() => {
          this.productList = this.productList.filter(item => item.id !== row.id)
          this.total = Math.max(0, this.total - 1)
          this.$modal.msgSuccess('删除成功')
        }).catch(() => {
        })
        : (this.productList = this.productList.filter(item => item.id !== row.id))
    },
    handleImport() {
      this.$message && this.$message.info('导入功能待接入接口')
    },
    onWeightChange(row) {
      this.$message && this.$message.success('置顶已' + (row.weightOn ? '开启' : '关闭'))
    },
    addPlatformRow() {
      if (!this.form.platformConfigs) this.$set(this.form, 'platformConfigs', [])
      this.form.platformConfigs.push({platform: undefined, link: undefined, enabled: true})
    },
    removePlatformRow(index) {
      this.form.platformConfigs.splice(index, 1)
    },
    beforeImageUpload(file) {
      const isImage = ['image/png', 'image/jpeg', 'image/jpg'].includes(file.type)
      const isLt20M = file.size / 1024 / 1024 < 20
      if (!isImage) this.$message && this.$message.error('仅支持png/jpg/jpeg 格式')
      if (!isLt20M) this.$message && this.$message.error('图片大小需小于20MB')
      return isImage && isLt20M
    },
    handleImageUpload(req) {
      // 前端演示：本地读取为预览
      const reader = new FileReader()
      reader.onload = () => {
        this.$set(this.form, 'image', reader.result)
      }
      reader.readAsDataURL(req.file)
      // 模拟成功回调
      typeof req.onSuccess === 'function' && req.onSuccess({}, req.file)
    },
    cancel() {
      this.open = false
      this.reset()
    },
    reset() {
      this.form = {
        id: undefined,
        image: undefined,
        name: undefined,
        category: undefined,
        link: undefined,
        platforms: [],
        platformConfigs: [
          {platform: undefined, link: undefined, enabled: true},
          {platform: undefined, link: undefined, enabled: false}
        ],
        clicks: 0,
        weightOn: false,
        pinned: false,
        layout: '2*1',
        status: '0'
      }
      this.resetForm && this.resetForm('form')
    },
    submitForm() {
      this.$refs['form'].validate(valid => {
        if (!valid) return
        // 由第三方配置派生展示字段
        const enabledPlatforms = (this.form.platformConfigs || []).filter(i => i.enabled && i.platform)
        this.form.platforms = enabledPlatforms.map(i => i.platform)
        const firstEnabled = enabledPlatforms.find(i => i.link)
        if (!this.form.link && firstEnabled) this.form.link = firstEnabled.link
        if (this.form.id) {
          const index = this.productList.findIndex(i => i.id === this.form.id)
          if (index !== -1) this.$set(this.productList, index, {...this.productList[index], ...this.form})
        } else {
          const maxId = Math.max(0, ...this.productList.map(i => i.id || 0))
          const toAdd = {...this.form, id: maxId + 1, image: require('@/assets/images/pay.png'), clicks: 0}
          this.productList.unshift(toAdd)
          this.total += 1
        }
        this.$modal && this.$modal.msgSuccess && this.$modal.msgSuccess('保存成功')
        this.open = false
        this.getList()
      })
    },
    openClicksDialog(row) {
      this.clicksProduct = row
      this.clicksDateStart = undefined
      this.clicksDateEnd = undefined
      const today = new Date()
      const list = []
      let sum = 0
      for (let i = 6; i >= 0; i--) {
        const d = new Date(today)
        d.setDate(today.getDate() - i)
        const dateStr = `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`
        // 简单按总量分布生成演示数据
        const base = Math.max(1, Math.round(row.clicks / 10))
        const fluctuation = Math.floor(Math.random() * base)
        const count = Math.max(0, base + (i % 2 === 0 ? fluctuation : -Math.floor(fluctuation / 2)))
        list.push({ date: dateStr, count })
        sum += count
      }
      // 归一化到接近总点击（演示）
      if (sum > 0 && row.clicks) {
        const factor = row.clicks / sum
        let adjustedSum = 0
        for (let j = 0; j < list.length; j++) {
          list[j].count = Math.max(0, Math.round(list[j].count * factor))
          adjustedSum += list[j].count
        }
        // 调整差值到最后一天
        const diff = (row.clicks || 0) - adjustedSum
        list[list.length - 1].count += diff
      }
      this.clicksDetailList = list
      this.clicksTotal = list.reduce((acc, cur) => acc + cur.count, 0)
      this.clicksDialogVisible = true
    },
    resetClicksFilters() {
      this.clicksDateStart = undefined
      this.clicksDateEnd = undefined
    }
  }
}
</script>


<style scoped lang="scss">
.w260 {
  width: 260px;
}

.w200 {
  width: 200px;
}

.w100 {
  width: 100%;
}

.mr4 {
  margin-right: 4px;
}

.mr8 {
  margin-right: 8px;
}

.mb6 {
  margin-bottom: 6px;
}

.mb12 {
  margin-bottom: 12px;
}

.mt6 {
  margin-top: 6px;
}

.mt16 {
  margin-top: 16px;
}

.text-right {
  text-align: right;
}

.flex-center {
  display: flex;
  align-items: center;
}

.row-center {
  display: flex;
  align-items: center;
}

.product-thumb {
  width: 80px;
  height: 80px;
}

.layout-preview {
  border: 1px dashed #ddd;
  padding: 8px;
  border-radius: 6px;
  width: 260px;
}

.layout-2x1 {
  .images {
    display: flex;
    gap: 8px;
  }

  .preview-item {
    flex: 1;
    height: 60px;
    background: #f5f7fa;
    border: 1px solid #ebeef5;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.layout-1x1 {
  .preview-1x1 {
    height: 60px;
    background: #f5f7fa;
    border: 1px solid #ebeef5;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.preview-title {
  text-align: center;
  color: #909399;
  margin-top: 6px;
}

.tip-text {
  color: #999;
  font-size: 12px;
}

.avatar {
  width: 120px;
  height: 120px;
  border-radius: 6px;
  object-fit: cover;
}

.avatar-placeholder {
  width: 120px;
  height: 120px;
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #909399;
}

.f20 {
  font-size: 20px;
}

.el-link {
  font-weight: 600;
}

.stat-header {
  margin-bottom: 12px;
}

.stat-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 6px;
}

.stat-total {
  color: #606266;
  margin-bottom: 8px;
}

.stat-total-num {
  color: #409EFF;
  font-weight: 700;
}

.stat-filters {
  display: flex;
  align-items: center;
}
</style>
