<template>
  <div class="app-container scroll-y">
    <div class="mt-10px mb-10px">本地excel导出</div>
    <div class="mb-10px rowSS">
      <el-input v-model="fileName" style="width: 300px" placeholder="文件名" />
      <el-button class="ml" type="primary" @click="handleExportExcel">
        <el-icon style="vertical-align: middle">
          <Download />
        </el-icon>
        <span style="vertical-align: middle">导出</span>
      </el-button>
    </div>

    <el-table v-loading="listLoading" :data="list" element-loading-text="Loading" border highlight-current-row>
      <el-table-column align="center" label="ID" width="95">
        <template #default="scope">
          {{ scope.$index }}
        </template>
      </el-table-column>
      <el-table-column label="order_no">
        <template #default="scope">
          {{ scope.row.order_no }}
        </template>
      </el-table-column>

      <el-table-column label="price" align="center">
        <template #default="scope">
          <span>{{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="username" align="center">
        <template #default="scope">
          {{ scope.row.username }}
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script setup>
import { Download } from '@element-plus/icons-vue'
import { onBeforeMount, toRefs } from 'vue'
import { transactionList } from '@/api/remote-search'
import { aoaToSheetXlsx } from './excel'
import { ElMessage } from 'element-plus'
const statusFilter = (status) => {
  const statusMap = {
    published: 'success',
    draft: 'info',
    deleted: 'danger'
  }
  return statusMap[status]
}
const state = reactive({
  list: null,
  listLoading: true
})

onBeforeMount(() => {
  fetchData()
})

const fetchData = () => {
  state.listLoading = true
  transactionList().then(({ data }) => {
    state.list = data.data?.items
    state.listLoading = false
  })
}
// 导出Excel表格
const fileName = ref('')
const handleExportExcel = () => {
  if (!unref(fileName).trim()) {
    ElMessage({
      showClose: true,
      message: '请输入文件名',
      type: 'warning'
    })
    return
  }
  const table = unref(state.list)
  const header = ['ID', '订单号', '价格', '用户名']
  const data = table.map((item, index) => {
    // eslint-disable-next-line camelcase
    const { id, order_no, price, username } = item
    // eslint-disable-next-line camelcase
    return [index, order_no, price, username]
  })
  aoaToSheetXlsx({
    data,
    header,
    filename: `${unref(fileName)}.xlsx`
  })
}

//导出属性到页面中使用
const { list, listLoading } = toRefs(state)
</script>

<style scoped lang="scss"></style>
