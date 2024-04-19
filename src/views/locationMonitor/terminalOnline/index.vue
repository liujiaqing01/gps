<template>
  <div class="app-container">
    <el-form :model="queryParams" :inline="true">
      <el-form-item label="所属矿区">
        <el-select
            v-model="queryParams.status"
            placeholder="请选择所属矿区"
            clearable
        >
          <!--          <el-option :value="0" label="离线"/>-->
          <!--          <el-option :value="1" label="在线"/>-->
        </el-select>
      </el-form-item>
      <el-form-item label="所属类型">
        <el-select
            v-model="queryParams.name"
            placeholder="请选择所属类型"
            clearable
            filterable
        >
          <!--          <el-option :value="0">离线</el-option>-->
          <!--          <el-option :value="1">在线</el-option>-->
        </el-select>
      </el-form-item>
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
      <el-form-item>
        <el-button type="primary" icon="Search">查询</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="pageList" style="width: 100%">
      <el-table-column prop="id" label="序号" align="center" width="110" sortable/>
      <el-table-column prop="name" label="终端名称" align="center"/>
      <el-table-column prop="a" label="所属矿区" align="center"/>
      <el-table-column prop="phone" label="终端手机号" align="center"/>
      <el-table-column prop="type" label="所属类型" align="center"/>
      <el-table-column label="经度" align="center">
        <template #default="scope">
          <el-tag
              type="warning"
              effect="plain"
          >
            {{ scope.row.j }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="维度" align="center">
        <template #default="scope">
          <el-tag
              type="warning"
              effect="plain"
          >
            {{ scope.row.w }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="b" label="更新时间" align="center"/>
      <el-table-column label="在线状态" align="center">
        <template #default="scope">
          <el-tag
              :type="scope.row.status ? 'success' : 'danger'"
              effect="plain"
          >
            {{ scope.row.status ? '在线' : '离线' }}
          </el-tag>
        </template>
      </el-table-column>
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
    name: 'CC19',
    a: '万企煤矿',
    type: '铲车',
    phone: '1564123',
    j: '106.89309',
    w: '39.74177',
    b: '2024-04-19 14:32:30',
    status: 1
  },
  {
    id: 2,
    name: 'CC19',
    a: '万企煤矿',
    type: '铲车',
    phone: '1564123',
    j: '106.89309',
    w: '39.74177',
    b: '2024-04-19 14:32:30',
    status: 0
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
