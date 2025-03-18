<template>
  <div class="server-list-container">
    <!-- 搜索和操作区域 -->
    <div class="search-actions mb-4">
      <el-form :inline="true" :model="searchForm" class="demo-form-inline">
        <el-form-item label="服务器ID">
          <el-input
            v-model="searchForm.serverId"
            placeholder="请输入服务器ID"
            clearable
          />
        </el-form-item>
        <el-form-item label="服务器名称">
          <el-input
            v-model="searchForm.serverName"
            placeholder="请输入服务器名称"
            clearable
          />
        </el-form-item>
        <el-form-item label="玩家ID">
          <el-input
            v-model="searchForm.playerId"
            placeholder="请输入玩家ID"
            clearable
          />
        </el-form-item>
        <el-form-item label="玩家邮箱">
          <el-input
            v-model="searchForm.playerEmail"
            placeholder="请输入玩家邮箱"
            clearable
          />
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model="searchForm.status" placeholder="请选择" clearable>
            <el-option label="在线" value="online" />
            <el-option label="离线" value="offline" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleSearch">搜索</el-button>
          <el-button @click="resetSearch">重置</el-button>
        </el-form-item>
      </el-form>

      <div class="action-buttons">
        <el-button type="primary" @click="handleAdd">新增服务器</el-button>
      </div>
    </div>

    <!-- 表格区域 -->
    <el-table v-loading="loading" :data="tableData" style="width: 100%" border>
      <el-table-column prop="id" label="ID" width="80" />
      <el-table-column prop="name" label="服务器名称" width="180" />
      <el-table-column prop="nodeInfo" label="节点信息" width="150" />
      <el-table-column prop="owner" label="拥有者" width="180">
        <template #default="scope">
          <el-button link type="primary" @click="viewPlayerDetail(scope.row)">
            {{ scope.row.owner }}
          </el-button>
        </template>
      </el-table-column>
      <el-table-column prop="ip" label="IP地址" width="150" />
      <el-table-column prop="port" label="端口" width="100" />
      <el-table-column prop="status" label="状态" width="100">
        <template #default="scope">
          <el-tag :type="scope.row.status === 'online' ? 'success' : 'danger'">
            {{ scope.row.status === 'online' ? '在线' : '离线' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="createdAt" label="创建时间" width="180" />
      <el-table-column label="基本操作" width="200">
        <template #default="scope">
          <el-button
            size="small"
            :type="scope.row.status === 'online' ? 'danger' : 'success'"
            :disabled="scope.row.processing"
            @click="
              handleServerAction(
                scope.row,
                scope.row.status === 'online' ? 'stop' : 'start'
              )
            "
          >
            {{ scope.row.status === 'online' ? '停止' : '启动' }}
          </el-button>
          <el-button
            size="small"
            type="warning"
            :disabled="scope.row.status === 'offline' || scope.row.processing"
            @click="handleServerAction(scope.row, 'restart')"
          >
            重启
          </el-button>
        </template>
      </el-table-column>
      <el-table-column label="高级操作" width="300">
        <template #default="scope">
          <el-dropdown
            @command="(command) => handleAdvancedAction(scope.row, command)"
          >
            <el-button size="small">
              高级操作<el-icon class="el-icon--right"><arrow-down /></el-icon>
            </el-button>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item command="upgrade">一键升级</el-dropdown-item>
                <el-dropdown-item command="backup">备份</el-dropdown-item>
                <el-dropdown-item command="pause">暂停</el-dropdown-item>
                <el-dropdown-item command="destroy" divided
                  >销毁</el-dropdown-item
                >
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          <el-button size="small" type="primary" @click="handleEdit(scope.row)"
            >编辑</el-button
          >
          <el-button size="small" @click="handleView(scope.row)"
            >详情</el-button
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
      :title="dialogType === 'add' ? '新增服务器' : '编辑服务器'"
      width="500px"
    >
      <el-form ref="formRef" :model="form" :rules="rules" label-width="100px">
        <el-form-item label="服务器名称" prop="name">
          <el-input v-model="form.name" placeholder="请输入服务器名称" />
        </el-form-item>
        <el-form-item label="节点信息" prop="nodeInfo">
          <el-input v-model="form.nodeInfo" placeholder="请输入节点信息" />
        </el-form-item>
        <el-form-item label="拥有者" prop="ownerId">
          <el-select
            v-model="form.ownerId"
            placeholder="请选择拥有者"
            filterable
            remote
          >
            <el-option
              v-for="player in playerOptions"
              :key="player.id"
              :label="player.nickname + ' (' + player.email + ')'"
              :value="player.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="IP地址" prop="ip">
          <el-input v-model="form.ip" placeholder="请输入IP地址" />
        </el-form-item>
        <el-form-item label="端口" prop="port">
          <el-input v-model="form.port" placeholder="请输入端口" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="handleSubmit">确定</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 查看详情抽屉 -->
    <el-drawer
      v-model="drawerVisible"
      title="服务器详情"
      direction="rtl"
      size="500px"
    >
      <template #default>
        <div class="drawer-content">
          <el-descriptions :column="1" border>
            <el-descriptions-item label="ID">{{
              detailData.id
            }}</el-descriptions-item>
            <el-descriptions-item label="服务器名称">{{
              detailData.name
            }}</el-descriptions-item>
            <el-descriptions-item label="节点信息">{{
              detailData.nodeInfo
            }}</el-descriptions-item>
            <el-descriptions-item label="拥有者">{{
              detailData.owner
            }}</el-descriptions-item>
            <el-descriptions-item label="IP地址">{{
              detailData.ip
            }}</el-descriptions-item>
            <el-descriptions-item label="端口">{{
              detailData.port
            }}</el-descriptions-item>
            <el-descriptions-item label="状态">
              <el-tag
                :type="detailData.status === 'online' ? 'success' : 'danger'"
              >
                {{ detailData.status === 'online' ? '在线' : '离线' }}
              </el-tag>
            </el-descriptions-item>
            <el-descriptions-item label="创建时间">{{
              detailData.createdAt
            }}</el-descriptions-item>
            <el-descriptions-item label="最后操作时间">{{
              detailData.lastOperationTime
            }}</el-descriptions-item>
          </el-descriptions>

          <!-- 系统资源使用情况 -->
          <div class="mt-4">
            <h3>系统资源使用情况</h3>
            <el-row :gutter="20">
              <el-col :span="12">
                <div class="resource-item">
                  <div class="resource-title">CPU使用率</div>
                  <el-progress
                    :percentage="detailData.resources?.cpu || 0"
                    :color="getCpuProgressColor(detailData.resources?.cpu || 0)"
                  />
                </div>
              </el-col>
              <el-col :span="12">
                <div class="resource-item">
                  <div class="resource-title">内存使用率</div>
                  <el-progress
                    :percentage="detailData.resources?.memory || 0"
                    :color="
                      getMemoryProgressColor(detailData.resources?.memory || 0)
                    "
                  />
                </div>
              </el-col>
            </el-row>
            <el-row :gutter="20" class="mt-3">
              <el-col :span="12">
                <div class="resource-item">
                  <div class="resource-title">磁盘使用率</div>
                  <el-progress
                    :percentage="detailData.resources?.disk || 0"
                    :color="
                      getDiskProgressColor(detailData.resources?.disk || 0)
                    "
                  />
                </div>
              </el-col>
              <el-col :span="12">
                <div class="resource-item">
                  <div class="resource-title">网络流量</div>
                  <div>
                    {{
                      formatNetworkTraffic(detailData.resources?.network || 0)
                    }}
                  </div>
                </div>
              </el-col>
            </el-row>
          </div>

          <!-- 操作日志 -->
          <div class="mt-4">
            <h3>操作日志</h3>
            <el-timeline>
              <el-timeline-item
                v-for="(log, index) in detailData.logs || []"
                :key="index"
                :timestamp="log.time"
                :type="getLogTypeIcon(log.action)"
              >
                {{ log.action }}: {{ log.description }}
              </el-timeline-item>
            </el-timeline>
          </div>
        </div>
      </template>
    </el-drawer>
  </div>
</template>

<script setup>
  import { ref, reactive, onMounted, computed } from 'vue'
  import { ElMessage, ElMessageBox } from 'element-plus'
  import { useRoute, useRouter } from 'vue-router'
  import { ArrowDown } from '@element-plus/icons-vue'

  const route = useRoute()
  const router = useRouter()

  // 模拟数据 - 服务器列表
  const mockServers = [
    {
      id: 1,
      name: '我的世界服务器1',
      nodeInfo: '节点A',
      ownerId: 1001,
      owner: '玩家一号',
      ip: '192.168.1.101',
      port: '25565',
      status: 'online',
      createdAt: '2024-03-15 10:00:00',
      lastOperationTime: '2024-03-18 15:30:00',
      processing: false,
      email: 'player1@example.com',
      resources: {
        cpu: 45,
        memory: 60,
        disk: 30,
        network: 1024 * 1024 * 5 // 5MB
      },
      logs: [
        {
          time: '2024-03-18 15:30:00',
          action: '启动',
          description: '服务器启动成功'
        },
        {
          time: '2024-03-17 20:15:00',
          action: '停止',
          description: '服务器正常停止'
        },
        {
          time: '2024-03-17 10:30:00',
          action: '重启',
          description: '服务器重启成功'
        }
      ]
    },
    {
      id: 2,
      name: '我的世界服务器2',
      nodeInfo: '节点B',
      ownerId: 1001,
      owner: '玩家一号',
      ip: '192.168.1.102',
      port: '25566',
      status: 'offline',
      createdAt: '2024-03-16 11:00:00',
      lastOperationTime: '2024-03-17 18:20:00',
      email: 'player1@example.com',
      processing: false,
      resources: {
        cpu: 0,
        memory: 0,
        disk: 25,
        network: 0
      },
      logs: [
        {
          time: '2024-03-17 18:20:00',
          action: '停止',
          description: '服务器正常停止'
        },
        {
          time: '2024-03-17 09:15:00',
          action: '启动',
          description: '服务器启动成功'
        }
      ]
    },
    {
      id: 3,
      name: '我的世界服务器3',
      nodeInfo: '节点A',
      ownerId: 1002,
      owner: '玩家二号',
      ip: '192.168.1.103',
      port: '25567',
      status: 'online',
      createdAt: '2024-03-14 14:00:00',
      email: 'player1@example.com',
      lastOperationTime: '2024-03-18 09:10:00',
      processing: false,
      resources: {
        cpu: 75,
        memory: 80,
        disk: 65,
        network: 1024 * 1024 * 10 // 10MB
      },
      logs: [
        {
          time: '2024-03-18 09:10:00',
          action: '升级',
          description: '服务器升级到最新版本'
        },
        {
          time: '2024-03-16 16:30:00',
          action: '备份',
          description: '创建服务器备份'
        },
        {
          time: '2024-03-15 11:20:00',
          action: '启动',
          description: '服务器启动成功'
        }
      ]
    }
  ]

  // 模拟数据 - 玩家列表（用于选择服务器拥有者）
  const mockPlayers = [
    { id: 1001, nickname: '玩家一号', email: 'player1@example.com' },
    { id: 1002, nickname: '玩家二号', email: 'player2@example.com' },
    { id: 1003, nickname: '玩家三号', email: 'player3@example.com' }
  ]

  // 搜索表单
  const searchForm = reactive({
    serverId: '',
    serverName: '',
    playerId: '',
    playerEmail: '',
    status: ''
  })

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
    name: '',
    nodeInfo: '',
    ownerId: '',
    ip: '',
    port: ''
  })

  // 表单验证规则
  const rules = {
    name: [{ required: true, message: '请输入服务器名称', trigger: 'blur' }],
    nodeInfo: [{ required: true, message: '请输入节点信息', trigger: 'blur' }],
    ownerId: [{ required: true, message: '请选择拥有者', trigger: 'change' }],
    ip: [
      { required: true, message: '请输入IP地址', trigger: 'blur' },
      {
        pattern: /^(\d{1,3}\.){3}\d{1,3}$/,
        message: 'IP地址格式不正确',
        trigger: 'blur'
      }
    ],
    port: [
      { required: true, message: '请输入端口', trigger: 'blur' },
      { pattern: /^\d+$/, message: '端口必须为数字', trigger: 'blur' }
    ]
  }

  // 详情抽屉
  const drawerVisible = ref(false)
  const detailData = ref({})

  // 玩家选项
  const playerOptions = ref(mockPlayers)

  // 搜索方法
  const handleSearch = () => {
    loading.value = true
    setTimeout(() => {
      // 模拟搜索过滤
      const filteredData = mockServers.filter((server) => {
        const serverIdMatch =
          !searchForm.serverId || server.id.toString() === searchForm.serverId
        const serverNameMatch =
          !searchForm.serverName || server.name.includes(searchForm.serverName)
        const playerIdMatch =
          !searchForm.playerId ||
          server.ownerId.toString() === searchForm.playerId
        const playerEmailMatch =
          !searchForm.playerEmail ||
          mockPlayers
            .find((p) => p.id === server.ownerId)
            ?.email.includes(searchForm.playerEmail)
        const statusMatch =
          !searchForm.status || server.status === searchForm.status

        return (
          serverIdMatch &&
          serverNameMatch &&
          playerIdMatch &&
          playerEmailMatch &&
          statusMatch
        )
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

  // 新增服务器
  const handleAdd = () => {
    dialogType.value = 'add'
    Object.keys(form).forEach((key) => {
      form[key] = ''
    })
    dialogVisible.value = true
  }

  // 编辑服务器
  const handleEdit = (row) => {
    dialogType.value = 'edit'
    Object.keys(form).forEach((key) => {
      form[key] = row[key]
    })
    dialogVisible.value = true
  }

  // 查看详情
  const handleView = async (row) => {
    loading.value = true
    // 模拟API请求延迟
    setTimeout(() => {
      // 从模拟数据中查找完整的服务器信息
      const serverDetail = mockServers.find((s) => s.id === row.id)
      detailData.value = serverDetail || row
      drawerVisible.value = true
      loading.value = false
    }, 500)
  }

  // 服务器基本操作（启动、停止、重启）
  const handleServerAction = (server, action) => {
    const actionText = {
      start: '启动',
      stop: '停止',
      restart: '重启'
    }

    ElMessageBox.confirm(`确认${actionText[action]}该服务器吗？`, '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    })
      .then(() => {
        // 模拟操作执行
        const targetServer = tableData.value.find((s) => s.id === server.id)
        if (targetServer) {
          targetServer.processing = true

          // 模拟操作延迟
          setTimeout(() => {
            if (action === 'start') {
              targetServer.status = 'online'
            } else if (action === 'stop') {
              targetServer.status = 'offline'
            } else if (action === 'restart') {
              // 重启操作不改变状态，保持在线
            }

            targetServer.processing = false
            targetServer.lastOperationTime = new Date().toLocaleString()

            ElMessage.success(`服务器${actionText[action]}操作已完成`)
          }, 2000)
        }
      })
      .catch(() => {})
  }

  // 服务器高级操作
  const handleAdvancedAction = (server, action) => {
    const actionText = {
      upgrade: '升级',
      backup: '备份',
      pause: '暂停',
      destroy: '销毁'
    }

    // 销毁操作需要特殊确认
    if (action === 'destroy') {
      ElMessageBox.confirm(
        '销毁操作将永久删除该服务器及其所有数据，无法恢复。确认继续吗？',
        '危险操作',
        {
          confirmButtonText: '确认销毁',
          cancelButtonText: '取消',
          type: 'danger'
        }
      )
        .then(() => {
          executeAdvancedAction(server, action, actionText[action])
        })
        .catch(() => {})
    } else {
      ElMessageBox.confirm(
        `确认对服务器执行${actionText[action]}操作吗？`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      )
        .then(() => {
          executeAdvancedAction(server, action, actionText[action])
        })
        .catch(() => {})
    }
  }

  // 执行高级操作
  const executeAdvancedAction = (server, action, actionText) => {
    const targetServer = tableData.value.find((s) => s.id === server.id)
    if (targetServer) {
      targetServer.processing = true

      // 模拟操作延迟
      setTimeout(() => {
        if (action === 'destroy') {
          // 从列表中移除
          tableData.value = tableData.value.filter((s) => s.id !== server.id)
          pagination.total -= 1
        } else {
          // 其他操作只更新最后操作时间
          targetServer.lastOperationTime = new Date().toLocaleString()
          targetServer.processing = false
        }

        ElMessage.success(`服务器${actionText}操作已完成`)
      }, 2000)
    }
  }

  // 查看玩家详情
  const viewPlayerDetail = (row) => {
    router.push({
      path: '/layout/playerUser/userList',
      query: {
        email: row.email,
        autoView: 'true'
      }
    })
  }

  // 提交表单
  const handleSubmit = async () => {
    if (!formRef.value) return
    await formRef.value.validate((valid) => {
      if (valid) {
        // 模拟提交
        if (dialogType.value === 'add') {
          // 模拟新增
          const newServer = {
            id: Math.max(...mockServers.map((s) => s.id)) + 1,
            ...form,
            owner:
              mockPlayers.find((p) => p.id === form.ownerId)?.nickname ||
              '未知',
            status: 'offline',
            createdAt: new Date().toLocaleString(),
            lastOperationTime: new Date().toLocaleString(),
            processing: false,
            resources: {
              cpu: 0,
              memory: 0,
              disk: 0,
              network: 0
            },
            logs: [
              {
                time: new Date().toLocaleString(),
                action: '创建',
                description: '服务器创建成功'
              }
            ]
          }

          mockServers.push(newServer)
          tableData.value.push(newServer)
          pagination.total += 1

          ElMessage.success('服务器创建成功')
        } else {
          // 模拟编辑
          const targetServer = tableData.value.find(
            (s) => s.id === detailData.value.id
          )
          if (targetServer) {
            Object.keys(form).forEach((key) => {
              targetServer[key] = form[key]
            })
            targetServer.owner =
              mockPlayers.find((p) => p.id === form.ownerId)?.nickname || '未知'
            targetServer.lastOperationTime = new Date().toLocaleString()

            ElMessage.success('服务器信息更新成功')
          }
        }

        dialogVisible.value = false
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

  // 资源使用进度条颜色
  const getCpuProgressColor = (percentage) => {
    if (percentage < 50) return '#67C23A'
    if (percentage < 80) return '#E6A23C'
    return '#F56C6C'
  }

  const getMemoryProgressColor = (percentage) => {
    if (percentage < 60) return '#67C23A'
    if (percentage < 85) return '#E6A23C'
    return '#F56C6C'
  }

  const getDiskProgressColor = (percentage) => {
    if (percentage < 70) return '#67C23A'
    if (percentage < 90) return '#E6A23C'
    return '#F56C6C'
  }

  // 格式化网络流量
  const formatNetworkTraffic = (bytes) => {
    if (bytes < 1024) return `${bytes} B/s`
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(2)} KB/s`
    if (bytes < 1024 * 1024 * 1024)
      return `${(bytes / (1024 * 1024)).toFixed(2)} MB/s`
    return `${(bytes / (1024 * 1024 * 1024)).toFixed(2)} GB/s`
  }

  // 获取日志图标类型
  const getLogTypeIcon = (action) => {
    const actionMap = {
      启动: 'success',
      停止: 'warning',
      重启: 'warning',
      升级: 'primary',
      备份: 'info',
      暂停: 'warning',
      销毁: 'danger',
      创建: 'success'
    }
    return actionMap[action] || 'info'
  }

  // 组件挂载时处理路由参数并加载数据
  onMounted(() => {
    // 检查是否有查询参数
    const { playerId, playerEmail, autoSearch } = route.query

    if (autoSearch === 'true') {
      if (playerId) {
        searchForm.playerId = playerId
      }
      if (playerEmail) {
        searchForm.playerEmail = playerEmail
      }
    }

    // 加载数据
    handleSearch()
  })
</script>

<style lang="scss" scoped>
  .server-list-container {
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

      .resource-item {
        margin-bottom: 10px;

        .resource-title {
          margin-bottom: 5px;
          font-size: 14px;
          color: #606266;
        }
      }
    }
  }
</style>
