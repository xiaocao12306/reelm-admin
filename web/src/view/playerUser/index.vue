<template>
  <div class="player-user-container">
    <!-- 搜索和操作区域 -->
    <div class="search-actions mb-4">
      <el-form :inline="true" :model="searchForm" class="demo-form-inline">
        <el-form-item label="邮箱">
          <el-input
            v-model="searchForm.email"
            placeholder="请输入邮箱"
            clearable
          />
        </el-form-item>
        <el-form-item label="昵称">
          <el-input
            v-model="searchForm.nickname"
            placeholder="请输入昵称"
            clearable
          />
        </el-form-item>
        <el-form-item label="实名状态">
          <el-select
            v-model="searchForm.verificationStatus"
            placeholder="请选择"
            clearable
          >
            <el-option label="已实名" value="verified" />
            <el-option label="未实名" value="unverified" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleSearch">搜索</el-button>
          <el-button @click="resetSearch">重置</el-button>
        </el-form-item>
      </el-form>

      <div class="action-buttons">
        <el-button type="primary" @click="handleAdd">新增玩家</el-button>
      </div>
    </div>

    <!-- 表格区域 -->
    <el-table v-loading="loading" :data="tableData" style="width: 100%" border>
      <el-table-column prop="id" label="ID" width="80" />
      <el-table-column prop="email" label="邮箱">
        <template #default="scope">
          <el-link
            :underline="false"
            type="primary"
            @click="handleView(scope.row)"
            >{{ scope.row.email }}
          </el-link>
        </template>
      </el-table-column>
      <el-table-column prop="nickname" label="昵称" />
      <el-table-column prop="verificationStatus" label="实名状态" width="100">
        <template #default="scope">
          <el-tag
            :type="
              scope.row.verificationStatus === 'verified'
                ? 'success'
                : 'warning'
            "
          >
            {{
              scope.row.verificationStatus === 'verified' ? '已实名' : '未实名'
            }}
          </el-tag>
        </template>
      </el-table-column>
      <!-- 服务器数量 -->
      <el-table-column prop="servers.length" label="服务器数量">
        <template #default="scope">
          <span>{{ scope.row.servers.length }} 个</span>
        </template>
      </el-table-column>
      <!-- 订单数量 -->
      <el-table-column prop="orders.length" label="订单数量">
        <template #default="scope">
          <span>{{ scope.row.orders.length }} 个</span>
        </template>
      </el-table-column>
      <el-table-column prop="createdAt" label="创建时间" width="180" />
      <el-table-column prop="lastLoginTime" label="最后登录时间" width="180" />
      <el-table-column label="操作" fixed="right" width="250">
        <template #default="scope">
          <el-button size="small" @click="handleView(scope.row)"
            >查看</el-button
          >
          <el-button size="small" type="primary" @click="handleEdit(scope.row)"
            >编辑</el-button
          >
          <el-button size="small" type="danger" @click="handleDelete(scope.row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <div class="pagination-container">
      <el-pagination
        v-model:current-page="pagination.currentPage"
        v-model:page-size="pagination.pageSize"
        :page-sizes="[10, 20, 50, 100]"
        :total="pagination.total"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>

    <!-- 新增/编辑对话框 -->
    <el-dialog
      v-model="dialogVisible"
      :title="dialogType === 'add' ? '新增玩家' : '编辑玩家'"
      width="500px"
    >
      <el-form ref="formRef" :model="form" :rules="rules" label-width="100px">
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="form.email" placeholder="请输入邮箱" />
        </el-form-item>
        <el-form-item label="昵称" prop="nickname">
          <el-input v-model="form.nickname" placeholder="请输入昵称" />
        </el-form-item>
        <el-form-item label="实名状态" prop="verificationStatus">
          <el-select
            v-model="form.verificationStatus"
            placeholder="请选择实名状态"
          >
            <el-option label="已实名" value="verified" />
            <el-option label="未实名" value="unverified" />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="handleSubmit">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
  import { ref, reactive, onMounted } from 'vue'
  import { ElMessage, ElMessageBox } from 'element-plus'
  import { useRoute } from 'vue-router'
  import router from '@/router'
  // 模拟数据
  const mockPlayers = [
    {
      id: 1001,
      email: 'player1@example.com',
      nickname: '玩家一号',
      verificationStatus: 'verified',
      createdAt: '2024-03-17 10:00:00',
      lastLoginTime: '2024-03-18 15:30:00',
      servers: [
        { id: 1, name: '我的世界服务器1', status: 'online' },
        { id: 2, name: '我的世界服务器2', status: 'offline' }
      ],
      orders: [
        {
          orderId: 'ORD001',
          amount: '99.00',
          status: 'paid',
          createdAt: '2024-03-17 10:30:00'
        },
        {
          orderId: 'ORD002',
          amount: '199.00',
          status: 'unpaid',
          createdAt: '2024-03-18 14:20:00'
        }
      ]
    },
    {
      id: 1002,
      email: 'player2@example.com',
      nickname: '玩家二号',
      verificationStatus: 'unverified',
      createdAt: '2024-03-16 09:00:00',
      lastLoginTime: '2024-03-17 20:15:00',
      servers: [{ id: 3, name: '我的世界服务器3', status: 'online' }],
      orders: [
        {
          orderId: 'ORD003',
          amount: '299.00',
          status: 'refunded',
          createdAt: '2024-03-16 09:30:00'
        }
      ]
    },
    {
      id: 1003,
      email: 'player3@example.com',
      nickname: '玩家三号',
      verificationStatus: 'verified',
      createdAt: '2024-03-15 14:00:00',
      lastLoginTime: '2024-03-18 08:45:00',
      servers: [],
      orders: [
        {
          orderId: 'ORD004',
          amount: '159.00',
          status: 'paid',
          createdAt: '2024-03-15 14:30:00'
        },
        {
          orderId: 'ORD005',
          amount: '259.00',
          status: 'failed',
          createdAt: '2024-03-17 16:20:00'
        }
      ]
    }
  ]

  // 搜索表单
  const searchForm = reactive({
    email: '',
    nickname: '',
    verificationStatus: ''
  })
  // 增加跳转到当前页面自动填充内容并搜索
  const route = useRoute()
  const query = route.query
  if (query.id) {
    searchForm.email = query.email
    searchForm.nickname = query.nickname
  }
  // 表格数据
  const loading = ref(false)
  const tableData = ref([])

  // 分页
  const pagination = reactive({
    currentPage: 1,
    pageSize: 10,
    total: 0
  })

  // 表单数据
  const dialogVisible = ref(false)
  const dialogType = ref('add')
  const formRef = ref(null)
  const form = reactive({
    email: '',
    nickname: '',
    verificationStatus: ''
  })

  // 表单验证规则
  const rules = {
    email: [
      { required: true, message: '请输入邮箱', trigger: 'blur' },
      { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
    ],
    nickname: [{ required: true, message: '请输入昵称', trigger: 'blur' }],
    verificationStatus: [
      { required: true, message: '请选择实名状态', trigger: 'change' }
    ]
  }

  // 详情抽屉
  const drawerVisible = ref(false)
  const detailData = ref({})

  // 搜索方法
  const handleSearch = () => {
    loading.value = true
    setTimeout(() => {
      // 模拟搜索过滤
      const filteredData = mockPlayers.filter((player) => {
        const emailMatch =
          !searchForm.email || player.email.includes(searchForm.email)
        const nicknameMatch =
          !searchForm.nickname || player.nickname.includes(searchForm.nickname)
        const statusMatch =
          !searchForm.verificationStatus ||
          player.verificationStatus === searchForm.verificationStatus
        return emailMatch && nicknameMatch && statusMatch
      })
      tableData.value = filteredData
      pagination.total = filteredData.length
      loading.value = false
    }, 500) // 模拟加载延迟
  }

  // 重置搜索
  const resetSearch = () => {
    Object.keys(searchForm).forEach((key) => {
      searchForm[key] = ''
    })
    handleSearch()
  }

  // 新增玩家
  const handleAdd = () => {
    dialogType.value = 'add'
    Object.keys(form).forEach((key) => {
      form[key] = ''
    })
    dialogVisible.value = true
  }

  // 编辑玩家
  const handleEdit = (row) => {
    dialogType.value = 'edit'
    Object.keys(form).forEach((key) => {
      form[key] = row[key]
    })
    dialogVisible.value = true
  }

  // 查看详情
  const handleView = async (row) => {
    router.push({
      path: '/layout/playerUser/playerDetail',
      query: {
        id: row.id
      }
    })
  }

  // 删除玩家
  const handleDelete = (row) => {
    ElMessageBox.confirm('确认删除该玩家吗？', '警告', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    })
      .then(() => {
        // TODO: 实现删除逻辑
        ElMessage.success('删除成功')
      })
      .catch(() => {})
  }

  // 提交表单
  const handleSubmit = async () => {
    if (!formRef.value) return
    await formRef.value.validate((valid) => {
      if (valid) {
        // TODO: 实现提交逻辑
        console.log('表单数据：', form)
        dialogVisible.value = false
        ElMessage.success(dialogType.value === 'add' ? '添加成功' : '修改成功')
      }
    })
  }

  // 分页方法
  const handleSizeChange = (val) => {
    pagination.pageSize = val
    handleSearch()
  }

  const handleCurrentChange = (val) => {
    pagination.currentPage = val
    handleSearch()
  }

  // 订单状态处理方法
  const getOrderStatusType = (status) => {
    const statusMap = {
      paid: 'success',
      unpaid: 'warning',
      refunded: 'info',
      failed: 'danger'
    }
    return statusMap[status] || 'info'
  }

  const getOrderStatusText = (status) => {
    const statusMap = {
      paid: '已支付',
      unpaid: '未支付',
      refunded: '已退款',
      failed: '失败'
    }
    return statusMap[status] || status
  }

  // 组件挂载时加载数据
  onMounted(() => {
    handleSearch()
  })
</script>

<style lang="scss" scoped>
  .player-user-container {
    padding: 20px;

    .search-actions {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      margin-bottom: 20px;
    }

    .pagination-container {
      margin-top: 20px;
      display: flex;
      justify-content: flex-end;
    }

    .drawer-content {
      padding: 20px;

      h3 {
        margin: 20px 0;
        font-size: 16px;
        font-weight: bold;
      }
    }
  }
</style>
