<template>
  <div class="app-container">
    <el-form :model="queryParams" :inline="true">
      <el-form-item label="设备编码">
        <el-select
            v-model="queryParams.status"
            placeholder="请选择设备编码"
            clearable
        >
          <!--          <el-option :value="0" label="离线"/>-->
          <!--          <el-option :value="1" label="在线"/>-->
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search">查询</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="pageList" style="width: 100%">
      <el-table-column prop="id" label="序号" align="center" width="110" sortable/>
      <el-table-column prop="code" label="设备编码" align="center"/>
      <el-table-column prop="name" label="设备名称" align="center"/>
      <el-table-column prop="date" label="信息时间" align="center"/>
      <el-table-column prop="type" label="信息类型" align="center"/>
    </el-table>
    <div style="display: flex; justify-content: right; margin-top: 20px">
      <el-pagination
          v-model:current-page="queryParams.current"
          v-model:page-size="queryParams.page"
          :page-sizes="[100, 200, 300, 400]"
          :small="true"
          :background="true"
          layout="sizes, prev, pager, next,jumper"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
      />
    </div>
  </div>
</template>
<script setup>
import {ref, getCurrentInstance} from "vue";

const queryParams = ref({
  current: 1,
  page: 100
})
const total = ref(1000)
const pageList = ref([
  {
    id: 1,
    code: '860287000113045',
    name: '赵军峰',
    date: '2024-02-27 00:57:26',
    type: '心率'
  },
  {
    id: 4,
    code: '860287000113045',
    name: '赵军峰',
    date: '2024-02-27 00:57:26',
    type: '心率'
  },
  {
    id: 3,
    code: '860287000113045',
    name: '赵军峰',
    date: '2024-02-27 00:57:26',
    type: '心率'
  },
])

function handleSizeChange(value) {
  console.log(value)
  queryParams.value.page = value
}

function handleCurrentChange(value) {
  console.log(value, 'current')
  queryParams.value.current = value
}

</script>

<style scoped lang="scss">
</style>
