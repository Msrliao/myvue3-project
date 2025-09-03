<template>
  <div class="edi-table-container">
    <el-table
      v-loading="loading"
      :data="oneTableData.value"
      border
      style="width: 100%"
      @cell-click="handleCellClick"
    >
      <el-table-column type="index" label="序号" width="50" />
      <el-table-column prop="XH" label="XH" width="80" />
      <el-table-column prop="code" label="商品编码" width="120" />
      <el-table-column prop="name" label="商品名称" width="200" />
      <el-table-column prop="spec" label="规格型号" width="180" />
      <el-table-column prop="unit" label="单位" width="80" />
      <el-table-column prop="price" label="单价" width="100" />
      <el-table-column prop="quantity" label="数量" width="100">
        <template #default="scope">
          <el-input
            v-model="oneTableData.value[scope.$index].quantity"
            @change="handleQuantityChange(scope.$index)"
            type="number"
            min="1"
          />
        </template>
      </el-table-column>
      <el-table-column prop="amount" label="金额" width="120" />
      <el-table-column label="操作" width="100" fixed="right">
        <template #default="scope">
          <el-button
            type="danger"
            text
            @click="handleDelete(scope.$index)"
            size="small"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="edi-table-footer" v-if="oneTableData.value.length > 0">
      <el-button type="primary" @click="clearData">清空</el-button>
      <el-button @click="handleAdd">添加行</el-button>
      <div class="total-info">
        总计：<span class="total-amount">{{ totalAmount }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { ElMessage } from 'element-plus'

// 定义props
const props = defineProps<{
  tableData: any[]
}>()

// 定义emits
const emit = defineEmits<{
  (e: 'edit', data: any[]): void
  (e: 'clear', data: any[]): void
}>()

// 表格数据，使用ref创建以保持响应性
const oneTableData = ref(props.tableData || [])

// 加载状态
const loading = ref(false)

// 监听外部tableData变化
watch(() => props.tableData, (newVal) => {
  oneTableData.value = [...newVal]
}, { deep: true })

// 监听内部数据变化，同步到父组件
watch(() => oneTableData.value, (newVal) => {
  emit('edit', [...newVal])
}, { deep: true })

// 计算总金额
const totalAmount = computed(() => {
  return oneTableData.value.reduce((sum, item) => sum + (item.amount || 0), 0)
})

// 处理单元格点击
const handleCellClick = (row: any, column: any, cell: any, event: any) => {
  // 处理单元格点击事件
}

// 处理数量变化
const handleQuantityChange = (index: number) => {
  const item = oneTableData.value[index]
  if (item.quantity && item.price) {
    item.amount = item.quantity * item.price
  }
}

// 删除行
const handleDelete = (index: number) => {
  oneTableData.value.splice(index, 1)
  // 更新序号
  updateXH()
  ElMessage.success('删除成功')
}

// 添加行
const handleAdd = (newRow?: any) => {
  if (newRow) {
    oneTableData.value.push(newRow)
  } else {
    // 生成新的行号
    const newXH = oneTableData.value.length + 1
    oneTableData.value.push({
      XH: newXH,
      code: '',
      name: '',
      spec: '',
      unit: '',
      price: 0,
      quantity: 1,
      amount: 0
    })
  }
  // 更新序号
  updateXH()
}

// 更新序号
const updateXH = () => {
  oneTableData.value.forEach((item, index) => {
    item.XH = index + 1
  })
}

// 清空数据
const clearData = () => {
  oneTableData.value.splice(0)
  emit('clear', [])
  ElMessage.success('数据已清空')
}

// 暴露方法给父组件
defineExpose({
  handleAdd,
  clearData
})
</script>

<style scoped>
.edi-table-container {
  margin: 20px 0;
}

.edi-table-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
  padding: 10px 0;
  border-top: 1px solid #eee;
}

.total-info {
  font-size: 16px;
  font-weight: bold;
}

.total-amount {
  color: #f56c6c;
  margin-left: 10px;
}
</style>