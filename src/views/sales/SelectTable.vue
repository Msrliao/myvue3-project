<template>
  <div>
    <el-dialog
      v-model="dialogVisible"
      title="选择商品"
      width="800px"
      :before-close="handleClose"
    >
      <el-form ref="searchFormRef" :model="searchForm" size="small" class="mb-4">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-form-item label="商品编码" prop="code">
              <el-input v-model="searchForm.code" placeholder="请输入商品编码" />
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="商品名称" prop="name">
              <el-input v-model="searchForm.name" placeholder="请输入商品名称" />
            </el-form-item>
          </el-col>
          <el-col :span="8" class="flex items-center justify-end">
            <el-button type="primary" @click="handleSearch">查询</el-button>
            <el-button @click="handleReset">重置</el-button>
          </el-col>
        </el-row>
      </el-form>

      <el-table
        ref="oneTableRef"
        v-loading="loading"
        :data="oneTableData"
        style="width: 100%"
        @cell-dblclick="dblclick"
        @cell-click="cellClick"
        @current-change="handleCurrentChange"
      >
        <el-table-column type="selection" width="55" />
        <el-table-column prop="code" label="商品编码" width="120" />
        <el-table-column prop="name" label="商品名称" width="200" />
        <el-table-column prop="spec" label="规格型号" width="180" />
        <el-table-column prop="unit" label="单位" width="80" />
        <el-table-column prop="price" label="销售单价" width="100" />
        <el-table-column prop="stock" label="库存数量" width="100" />
      </el-table>

      <template #footer>
        <span class="dialog-footer">
          <el-button @click="handleClose">取消</el-button>
          <el-button type="primary" @click="handleConfirm">确定</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 编辑表格组件 -->
    <EdiTable
      ref="twoTableRef"
      :tableData="twoTableData"
      @edit="handleEdit"
      @clear="handleEdiTableClear"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { ElMessage, ElTable } from 'element-plus'
import EdiTable from './EdiTable.vue'

// 定义props
const props = defineProps<{
  visible: boolean
}>()

// 定义emits
const emit = defineEmits<{
  (e: 'update:visible', value: boolean): void
  (e: 'confirm', data: any[]): void
}>()

// 对话框可见性
const dialogVisible = computed({
  get: () => props.visible,
  set: (value) => emit('update:visible', value)
})

// 搜索表单
const searchFormRef = ref()
const searchForm = reactive({
  code: '',
  name: ''
})

// 表格引用
const oneTableRef = ref<InstanceType<typeof ElTable>>()
const twoTableRef = ref()

// 表格数据
const oneTableData = ref([
  {
    code: 'SP001',
    name: '笔记本电脑',
    spec: 'ThinkPad X1 Carbon',
    unit: '台',
    price: 9999,
    stock: 50
  },
  {
    code: 'SP002',
    name: '显示器',
    spec: 'Dell U2720Q',
    unit: '台',
    price: 3599,
    stock: 30
  },
  {
    code: 'SP003',
    name: '键盘',
    spec: '机械键盘青轴',
    unit: '个',
    price: 299,
    stock: 100
  }
])

// 编辑表格数据
let twoTableData = ref([])

// 加载状态
const loading = ref(false)

// 当前选中行
const currentRow = ref<any>(null)

// 查询数据
const handleSearch = () => {
  loading.value = true
  // 模拟API请求
  setTimeout(() => {
    // 这里可以根据searchForm进行筛选
    loading.value = false
  }, 500)
}

// 重置表单
const handleReset = () => {
  searchFormRef.value?.resetFields()
  handleSearch()
}

// 双击行添加到编辑表格
const dblclick = (row: any) => {
  // 检查是否已存在
  const exists = twoTableData.value.some((item: any) => item.code === row.code)
  if (exists) {
    ElMessage.warning('该商品已添加')
    return
  }

  const newItem = {
    XH: twoTableData.value.length + 1,
    code: row.code,
    name: row.name,
    spec: row.spec,
    unit: row.unit,
    price: row.price,
    quantity: 1,
    amount: row.price * 1
  }

  // 调用EdiTable组件的add方法
  twoTableRef.value.handleAdd(newItem)
}

// 单元格点击事件
const cellClick = (row: any) => {
  currentRow.value = row
}

// 当前行变化事件
const handleCurrentChange = (val: any) => {
  currentRow.value = val
}

// 处理编辑表格的编辑事件
const handleEdit = (data: any[]) => {
  twoTableData.value = data
}

// 处理编辑表格的清空事件
const handleEdiTableClear = () => {
  twoTableData.value = []
}

// 关闭对话框
const handleClose = () => {
  emit('update:visible', false)
}

// 确认选择
const handleConfirm = () => {
  emit('confirm', twoTableData.value)
  emit('update:visible', false)
}

// 组件挂载时获取数据
onMounted(() => {
  handleSearch()
})
</script>

<style scoped>
.flex {
  display: flex;
}
.items-center {
  align-items: center;
}
.justify-end {
  justify-content: flex-end;
}
</style>