<template>
  <div class="app-container">
    <el-form :model="queryParams" :inline="true">
      <el-form-item label="在线状态">
        <el-select
            v-model="queryParams.status"
            placeholder="请选择在线状态"
            clearable
        >
          <el-option :value="0" label="离线"/>
          <el-option :value="1" label="在线"/>
        </el-select>
      </el-form-item>
      <el-form-item label="人员名称">
        <el-select
            v-model="queryParams.name"
            placeholder="请选择人员名称"
            clearable
            filterable
        >
          <!--          <el-option :value="0">离线</el-option>-->
          <!--          <el-option :value="1">在线</el-option>-->
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search">查询</el-button>
        <el-button type="info" icon="Refresh" plain>重置</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="pageList" style="width: 100%">
      <el-table-column prop="name" label="人员名称" align="center"/>
      <el-table-column label="状态" align="center">
        <template #default="scope">
          <el-tag
              :type="scope.row.status ? 'success' : 'danger'"
              effect="plain"
          >
            {{ scope.row.status ? '在线' : '离线' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="a" label="所属矿区" align="center"/>
      <el-table-column prop="b" label="部门" align="center"/>
      <el-table-column prop="g" label="时间" align="center"/>
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
    name: '久保田',
    status: 0,
    a: 'jiubaotian',
    b: '研发部',
    g: '2021-4-6 12:34:45',
  },
  {
    name: '凯斯',
    status: 0,
    a: 'kaisi',
    b: '研发部',
    g: '2021-4-6 12:34:45',
  },
  {
    name: '红岩',
    status: 1,
    a: 'hongyan',
    b: '信息部',
    g: '2021-4-6 12:34:45',
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
