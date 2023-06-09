<template>
  <el-card shadow="never" class="border-0">
    <!-- 搜索 -->
    <Search :model="searchForm" @search="getData" @reset="resetSearchForm">
      <SearchItem label="管理员名称">
        <el-input v-model="searchForm.keyword" placeholder="管理员名称" clearable />
      </SearchItem>
    </Search>

    <!-- 新增|刷新 -->
    <ListHeader @add="handleCreate" @refresh="getData"></ListHeader>

    <!-- 表格 -->
    <el-table :data="tableData" stripe style="width: 100%;" v-loading="loading">
      <el-table-column label="管理员" width="200">
        <template #default="{ row }">
          <div class="flex items-center">
            <el-avatar :size="40" :src="row.avatar">
              <img src="https://ts1.cn.mm.bing.net/th/id/R-C.987f582c510be58755c4933cda68d525?rik=C0D21hJDYvXosw&riu=http%3a%2f%2fimg.pconline.com.cn%2fimages%2fupload%2fupc%2ftx%2fwallpaper%2f1305%2f16%2fc4%2f20990657_1368686545122.jpg&ehk=netN2qzcCVS4ALUQfDOwxAwFcy41oxC%2b0xTFvOYy5ds%3d&risl=&pid=ImgRaw&r=0" />
            </el-avatar>
            <div class="ml-3">
              <h6>{{ row.username }}</h6>
              <small>ID:{{ row.id }}</small>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="所属角色" align="center">
        <template #default="{ row }">
          {{ row.role?.name || "-" }}
        </template>
      </el-table-column>
      <el-table-column label="状态" width="120">
        <template #default="{ row }">
          <el-switch :modelValue="row.status" :loading="row.loading" :disabled="row.super == 1" :active-value="1"
            :inactive-value="0" @change="handleChangeStatus($event, row)" />
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180" align="center">
        <template #default="{ row }">
          <small v-if="row.super == 1" class="text-sm text-gray-500">暂无操作</small>
          <div v-else>
            <el-button size="small" type="primary" @click="handleEdit(row)">修改</el-button>
            <el-popconfirm title="是否删除该管理员？" width="171" confirm-button-text="删除" cancel-button-text="取消"
              @confirm="handleDelete(row.id)">
              <template #reference>
                <el-button size="small" type="danger">删除</el-button>
              </template>
            </el-popconfirm>
          </div>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <div class="flex justify-center items-center mt-5">
      <el-pagination v-model:current-page="currentPage" v-model:page-size="searchForm.limit"
        :page-sizes="[10, 20, 30, 40]" background layout="total, sizes, prev, pager, next, jumper" :total="total"
        @size-change="handleSizeChange" @current-change="getData" />
    </div>
    <!-- 新增|修改 -->
    <FormDrawer ref="formDrawerRef" :title="drawerTitle" @submit="handleSubmit">
      <el-form :model="form" ref="formRef" :rules="rules" label-width="80px" style="width: 98%;">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="form.username" placeholder="用户名" />
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="form.password" placeholder="密码" />
        </el-form-item>
        <el-form-item label="图像" prop="avatar">
          <ChooseImage v-model="form.avatar"></ChooseImage>
        </el-form-item>
        <el-form-item label="所属角色" prop="role_id">
          <el-select v-model="form.role_id" placeholder="选择所属角色">
            <el-option v-for="item in roles" :key="item.id" :label="item.name" :value="item.id" />
          </el-select>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-switch v-model="form.status" :active-value="1" :inactive-value="0" />
        </el-form-item>
      </el-form>
    </FormDrawer>
  </el-card>
</template>

<script setup>
import { ref } from "vue"
import ChooseImage from "@/components/image/ChooseImage.vue"
import ListHeader from "@/components/table/ListHeader.vue"
import Search from "@/components/search/Search.vue"
import SearchItem from "@/components/search/SearchItem.vue"
import { getManagerList, updateManagerStatus, addManager, updateManager, deleteManager } from "@/api/manager.js"
import { useInitTable, useInitForm, useDeleteAndChange } from "@/hooks/useTable.js"

// 所属角色
const roles = ref([])

// 列表 | 分页 | 搜索
const optionTable = {
  searchForm: { keyword: null, limit: 10 },
  getList: getManagerList,
  onGetListSuccess: handleSuccess
}
const { searchForm, resetSearchForm, tableData, loading, currentPage, total,
  handleSizeChange, getData
} = useInitTable(optionTable)
// 处理响应结果
function handleSuccess(res) {
  tableData.value = res.list.map(item => {
    item.loading = false
    return item
  })
  total.value = res.totalCount
  roles.value = res.roles
}

// 新增 | 修改
const optionForm = {
  form: {
    username: "",
    password: "",
    role_id: null,
    status: 1,
    avatar: ""
  },
  getData,
  update: updateManager,
  add: addManager
}
const { formDrawerRef, formRef, form, rules, drawerTitle, handleSubmit, handleCreate, handleEdit } = useInitForm(optionForm)

// 删除 | 修改状态
const option = { loading, getData, updateStatus: updateManagerStatus, delete: deleteManager }
const { handleDelete, handleChangeStatus } = useDeleteAndChange(option)


</script>

