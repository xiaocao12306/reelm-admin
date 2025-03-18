<template>
  <div class="ticket-list-container">
    <!-- 统计卡片 -->
    <div class="stats-cards">
      <!-- 待处理工单 -->
      <el-card class="stat-card">
        <div class="stat-content">
          <div class="stat-icon yellow">
            <el-icon><Clock /></el-icon>
          </div>
          <div class="stat-info">
            <div class="stat-label">待处理工单</div>
            <div class="stat-value">{{ ticketStats.pending }}</div>
          </div>
        </div>
      </el-card>

      <!-- 处理中工单 -->
      <el-card class="stat-card">
        <div class="stat-content">
          <div class="stat-icon blue">
            <el-icon><Document /></el-icon>
          </div>
          <div class="stat-info">
            <div class="stat-label">处理中工单</div>
            <div class="stat-value">{{ ticketStats.processing }}</div>
          </div>
        </div>
      </el-card>

      <!-- 已解决工单 -->
      <el-card class="stat-card">
        <div class="stat-content">
          <div class="stat-icon green">
            <el-icon><Select /></el-icon>
          </div>
          <div class="stat-info">
            <div class="stat-label">已解决工单</div>
            <div class="stat-value">{{ ticketStats.resolved }}</div>
          </div>
        </div>
      </el-card>

      <!-- 平均响应时间 -->
      <el-card class="stat-card">
        <div class="stat-content">
          <div class="stat-icon purple">
            <el-icon><Timer /></el-icon>
          </div>
          <div class="stat-info">
            <div class="stat-label">平均响应时间</div>
            <div class="stat-value">{{ ticketStats.avgResponseTime }}</div>
          </div>
        </div>
      </el-card>
    </div>

    <!-- 搜索和操作栏 -->
    <div class="search-actions">
      <div class="search-box">
        <el-input
          v-model="searchQuery"
          placeholder="搜索工单号、标题或玩家名..."
          prefix-icon="Search"
          clearable
        />
      </div>

      <div class="action-buttons">
        <el-dropdown trigger="click" @command="handleFilterCommand">
          <el-button>
            <el-icon class="el-icon--left"><Filter /></el-icon>
            筛选
          </el-button>
          <template #dropdown>
            <el-dropdown-menu>
              <div class="filter-dropdown-content">
                <h4 class="filter-title">优先级</h4>
                <div class="filter-options">
                  <el-checkbox
                    v-model="filters.priority"
                    label="high"
                    @change="handleFilterChange"
                    >高</el-checkbox
                  >
                  <el-checkbox
                    v-model="filters.priority"
                    label="medium"
                    @change="handleFilterChange"
                    >中</el-checkbox
                  >
                  <el-checkbox
                    v-model="filters.priority"
                    label="low"
                    @change="handleFilterChange"
                    >低</el-checkbox
                  >
                </div>

                <h4 class="filter-title">分类</h4>
                <div class="filter-options">
                  <el-checkbox
                    v-model="filters.category"
                    label="technical"
                    @change="handleFilterChange"
                    >技术问题</el-checkbox
                  >
                  <el-checkbox
                    v-model="filters.category"
                    label="billing"
                    @change="handleFilterChange"
                    >账单问题</el-checkbox
                  >
                  <el-checkbox
                    v-model="filters.category"
                    label="account"
                    @change="handleFilterChange"
                    >账户问题</el-checkbox
                  >
                  <el-checkbox
                    v-model="filters.category"
                    label="config"
                    @change="handleFilterChange"
                    >配置问题</el-checkbox
                  >
                  <el-checkbox
                    v-model="filters.category"
                    label="performance"
                    @change="handleFilterChange"
                    >性能问题</el-checkbox
                  >
                </div>

                <div class="filter-actions">
                  <el-button type="primary" size="small" @click="clearFilters"
                    >清除筛选</el-button
                  >
                </div>
              </div>
            </el-dropdown-menu>
          </template>
        </el-dropdown>

        <el-button type="primary" @click="createDemoTicket">
          <el-icon class="el-icon--left"><Plus /></el-icon>
          创建工单
        </el-button>
      </div>
    </div>

    <!-- 工单列表区域 -->
    <el-card class="ticket-list-card">
      <!-- 标签页导航 -->
      <el-tabs v-model="activeTab" @tab-click="handleTabClick">
        <el-tab-pane label="所有工单" name="all"></el-tab-pane>
        <el-tab-pane name="pending">
          <template #label>
            <span>待处理</span>
            <el-tag size="small" type="warning" class="tab-count">{{
              ticketStats.pending
            }}</el-tag>
          </template>
        </el-tab-pane>
        <el-tab-pane name="processing">
          <template #label>
            <span>处理中</span>
            <el-tag size="small" type="primary" class="tab-count">{{
              ticketStats.processing
            }}</el-tag>
          </template>
        </el-tab-pane>
        <el-tab-pane name="resolved">
          <template #label>
            <span>已解决</span>
            <el-tag size="small" type="success" class="tab-count">{{
              ticketStats.resolved
            }}</el-tag>
          </template>
        </el-tab-pane>
      </el-tabs>

      <!-- 工单表格 -->
      <el-table
        :data="filteredTickets"
        style="width: 100%"
        v-loading="loading"
        label-align="center"
      >
        <el-table-column label="工单号/标题">
          <template #default="scope">
            <div class="ticket-id-title">
              <el-button
                type="text"
                class="ticket-id"
                @click="handleTicketClick(scope.row)"
                >{{ scope.row.id }}</el-button
              >
              <div class="ticket-title">{{ scope.row.title }}</div>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="玩家">
          <template #default="scope">
            <div class="player-info">
              <div class="player-details">
                <el-button
                  type="primary"
                  link
                  @click="handlePlayerClick(scope.row)"
                  class="player-name"
                  >{{ scope.row.player.name }}({{
                    scope.row.player.email
                  }})</el-button
                >
                <div class="player-id">ID: {{ scope.row.player.id }}</div>
              </div>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="优先级" width="100">
          <template #default="scope">
            <el-tag :type="getPriorityType(scope.row.priority)" effect="light">
              {{ getPriorityText(scope.row.priority) }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column label="状态" width="100">
          <template #default="scope">
            <el-tag :type="getStatusType(scope.row.status)" effect="light">
              {{ getStatusText(scope.row.status) }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column label="创建时间" width="150">
          <template #default="scope">
            {{ scope.row.createdAt }}
          </template>
        </el-table-column>

        <el-table-column label="最后更新" width="150">
          <template #default="scope">
            {{ scope.row.updatedAt }}
          </template>
        </el-table-column>

        <el-table-column label="操作" fixed="right" width="240">
          <template #default="scope">
            <div class="ticket-actions" @click.stop>
              <el-button
                v-if="scope.row.status !== 'resolved'"
                type="primary"
                size="small"
                @click="handleEditTicket(scope.row)"
              >
                编辑
              </el-button>

              <el-button
                v-if="scope.row.status === 'pending'"
                type="success"
                size="small"
                @click="handleAcceptTicket(scope.row)"
              >
                接受处理
              </el-button>

              <el-button
                v-if="scope.row.status === 'processing'"
                type="success"
                size="small"
                :icon="Select"
                @click="handleResolveTicket(scope.row)"
              >
                已解决
              </el-button>

              <el-dropdown
                trigger="click"
                @command="(command) => handleMoreActions(command, scope.row)"
              >
                <el-button size="small" :icon="ArrowDown">更多</el-button>
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item command="view">查看详情</el-dropdown-item>
                    <el-dropdown-item
                      v-if="scope.row.server"
                      command="viewServer"
                      >查看关联服务器</el-dropdown-item
                    >
                    <el-dropdown-item
                      divided
                      command="delete"
                      class="danger-item"
                    >
                      删除工单
                    </el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </div>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="pagination-container">
        <el-pagination
          v-model:current-page="pagination.currentPage"
          v-model:page-size="pagination.pageSize"
          :page-sizes="[10, 20, 50, 100]"
          :total="filteredTickets.length"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>
  </div>
</template>

<script setup>
  import { ref, reactive, computed, onMounted } from 'vue'
  import { ElMessage, ElMessageBox } from 'element-plus'
  import { useRouter } from 'vue-router'
  import {
    Clock,
    Document,
    Select,
    Timer,
    Filter,
    Plus,
    Search,
    Edit,
    ArrowDown
  } from '@element-plus/icons-vue'

  const router = useRouter()
  const loading = ref(false)

  // 模拟工单状态数据
  const ticketStats = reactive({
    pending: 8,
    processing: 12,
    resolved: 45,
    avgResponseTime: '4.2小时'
  })

  // 模拟工单数据
  const tickets = ref([
    {
      id: 'TK-2025031',
      title: '无法访问我的Minecraft服务器',
      player: {
        id: 'player123',
        name: 'MineKing',
        avatar: '/api/placeholder/36/36',
        email: 'mineking@example.com'
      },
      priority: 'high',
      status: 'pending',
      createdAt: '2025-03-17 09:12',
      updatedAt: '2025-03-17 09:12',
      category: 'technical',
      server: {
        id: 'MC-2025042',
        name: 'MineKingdom'
      }
    },
    {
      id: 'TK-2025030',
      title: '服务器备份功能不工作',
      player: {
        id: 'gamemaster42',
        name: 'GameMaster',
        avatar: '/api/placeholder/36/36',
        email: 'gamemaster@example.com'
      },
      priority: 'medium',
      status: 'processing',
      createdAt: '2025-03-16 17:45',
      updatedAt: '2025-03-17 08:30',
      category: 'technical',
      server: {
        id: 'MC-2024981',
        name: 'SurvivalWorld'
      }
    },
    {
      id: 'TK-2025029',
      title: '请求更改服务器配置',
      player: {
        id: 'dragonslayer99',
        name: 'DragonSlayer',
        avatar: '/api/placeholder/36/36',
        email: 'dragonslayer@example.com'
      },
      priority: 'low',
      status: 'resolved',
      createdAt: '2025-03-15 11:20',
      updatedAt: '2025-03-16 14:05',
      category: 'config',
      server: {
        id: 'ARK-2025008',
        name: 'DinoWorld'
      }
    },
    {
      id: 'TK-2025028',
      title: '无法进行在线支付',
      player: {
        id: 'gamer567',
        name: 'ProGamer',
        avatar: '/api/placeholder/36/36',
        email: 'gamer567@example.com'
      },
      priority: 'high',
      status: 'pending',
      createdAt: '2025-03-15 09:33',
      updatedAt: '2025-03-15 09:33',
      category: 'billing',
      server: null
    },
    {
      id: 'TK-2025027',
      title: 'CS2服务器延迟过高',
      player: {
        id: 'shooter123',
        name: 'HeadShooter',
        avatar: '/api/placeholder/36/36',
        email: 'shooter123@example.com'
      },
      priority: 'medium',
      status: 'processing',
      createdAt: '2025-03-14 20:15',
      updatedAt: '2025-03-15 10:42',
      category: 'performance',
      server: {
        id: 'CS-2024133',
        name: 'FragZone'
      }
    },
    {
      id: 'TK-2025026',
      title: '账户密码重置请求',
      player: {
        id: 'newplayer2025',
        name: 'NewPlayer',
        avatar: '/api/placeholder/36/36',
        email: 'newplayer2025@example.com'
      },
      priority: 'medium',
      status: 'resolved',
      createdAt: '2025-03-14 16:22',
      updatedAt: '2025-03-14 17:05',
      category: 'account',
      server: null
    }
  ])

  // 过滤和搜索状态
  const activeTab = ref('all')
  const searchQuery = ref('')
  const filters = reactive({
    priority: [],
    category: []
  })

  // 分页
  const pagination = reactive({
    currentPage: 1,
    pageSize: 10,
    total: 0
  })

  // 过滤工单
  const filteredTickets = computed(() => {
    return tickets.value.filter((ticket) => {
      // 基于标签过滤
      if (activeTab.value === 'pending' && ticket.status !== 'pending')
        return false
      if (activeTab.value === 'processing' && ticket.status !== 'processing')
        return false
      if (activeTab.value === 'resolved' && ticket.status !== 'resolved')
        return false

      // 基于搜索过滤
      if (searchQuery.value) {
        const query = searchQuery.value.toLowerCase()
        const matchesId = ticket.id.toLowerCase().includes(query)
        const matchesTitle = ticket.title.toLowerCase().includes(query)
        const matchesPlayer = ticket.player.name.toLowerCase().includes(query)

        if (!(matchesId || matchesTitle || matchesPlayer)) return false
      }

      // 基于优先级过滤
      if (
        filters.priority.length > 0 &&
        !filters.priority.includes(ticket.priority)
      )
        return false

      // 基于分类过滤
      if (
        filters.category.length > 0 &&
        !filters.category.includes(ticket.category)
      )
        return false

      return true
    })
  })

  // 映射状态为标签名
  const getStatusText = (status) => {
    const statusMap = {
      pending: '待处理',
      processing: '处理中',
      resolved: '已解决'
    }
    return statusMap[status] || status
  }

  // 获取状态对应的类型
  const getStatusType = (status) => {
    const typeMap = {
      pending: 'warning',
      processing: 'primary',
      resolved: 'success'
    }
    return typeMap[status] || 'info'
  }

  // 获取优先级对应的文本
  const getPriorityText = (priority) => {
    const priorityMap = {
      high: '高',
      medium: '中',
      low: '低'
    }
    return priorityMap[priority] || priority
  }

  // 获取优先级对应的类型
  const getPriorityType = (priority) => {
    const typeMap = {
      high: 'danger',
      medium: 'warning',
      low: 'success'
    }
    return typeMap[priority] || 'info'
  }

  // 获取优先级点的类名
  const getPriorityDotClass = (priority) => {
    return `priority-dot-${priority}`
  }

  // 处理标签页点击
  const handleTabClick = () => {
    pagination.currentPage = 1
  }

  // 处理筛选变化
  const handleFilterChange = () => {
    pagination.currentPage = 1
  }

  // 处理筛选命令
  const handleFilterCommand = (command) => {
    console.log('Filter command:', command)
  }

  // 清除所有过滤器
  const clearFilters = () => {
    filters.priority = []
    filters.category = []
    searchQuery.value = ''
  }

  // 创建演示工单
  const createDemoTicket = () => {
    const newTicket = {
      id: `TK-${2025032 + tickets.value.length}`,
      title: '新增游戏服务器请求',
      player: {
        id: 'newuser2025',
        name: 'GameEnthusiast',
        avatar: '/api/placeholder/36/36',
        email: 'gameenthusiast@example.com'
      },
      priority: 'medium',
      status: 'pending',
      createdAt: new Date()
        .toLocaleString('zh-CN', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit',
          hour: '2-digit',
          minute: '2-digit',
          hour12: false
        })
        .replace(/\//g, '-'),
      updatedAt: new Date()
        .toLocaleString('zh-CN', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit',
          hour: '2-digit',
          minute: '2-digit',
          hour12: false
        })
        .replace(/\//g, '-'),
      category: 'request',
      server: null
    }

    tickets.value.unshift(newTicket)

    // 更新统计
    ticketStats.pending += 1

    ElMessage.success('工单创建成功')
  }

  // 处理工单点击
  const handleTicketClick = (row) => {
    console.log('跳转到工单详情页:', row.id)
    // 实际应用中这里会导航到工单详情页
    router.push({
      path: '/layout/playerUser/ticketsDetail',
      query: {
        id: row.id
      }
    })
  }

  // 点击跳转 player
  const handlePlayerClick = (row) => {
    console.log('跳转到玩家详情页:', row.player.id)
    // 实际应用中这里会导航到玩家详情页
    router.push({
      path: '/layout/playerUser/userList',
      query: {
        email: row.player.email,
        nickname: row.player.nickname
      }
    })
  }

  // 处理编辑工单
  const handleEditTicket = (ticket) => {
    console.log('编辑工单:', ticket.id)
    // 实际应用中这里会打开编辑对话框或导航到编辑页面
  }

  // 处理接受工单
  const handleAcceptTicket = (ticket) => {
    ElMessageBox.confirm(`确认接受处理工单 ${ticket.id}?`, '确认', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'info'
    })
      .then(() => {
        // 更新工单状态
        ticket.status = 'processing'
        ticket.updatedAt = new Date()
          .toLocaleString('zh-CN', {
            year: 'numeric',
            month: '2-digit',
            day: '2-digit',
            hour: '2-digit',
            minute: '2-digit',
            hour12: false
          })
          .replace(/\//g, '-')

        // 更新统计
        ticketStats.pending -= 1
        ticketStats.processing += 1

        ElMessage.success('已接受处理工单')
      })
      .catch(() => {})
  }

  // 处理解决工单
  const handleResolveTicket = (ticket) => {
    ElMessageBox.confirm(`确认将工单 ${ticket.id} 标记为已解决?`, '确认', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'success'
    })
      .then(() => {
        // 更新工单状态
        ticket.status = 'resolved'
        ticket.updatedAt = new Date()
          .toLocaleString('zh-CN', {
            year: 'numeric',
            month: '2-digit',
            day: '2-digit',
            hour: '2-digit',
            minute: '2-digit',
            hour12: false
          })
          .replace(/\//g, '-')

        // 更新统计
        ticketStats.processing -= 1
        ticketStats.resolved += 1

        ElMessage.success('工单已标记为已解决')
      })
      .catch(() => {})
  }

  // 处理更多操作
  const handleMoreActions = (command, ticket) => {
    console.log(`执行命令 ${command} 于工单 ${ticket.id}`)

    switch (command) {
      case 'assign':
        ElMessage.info('打开分配工单对话框')
        break
      case 'view':
        handleTicketClick(ticket)
        break
      case 'viewServer':
        if (ticket.server) {
          console.log('查看服务器:', ticket.server.id)
          // router.push(`/servers/${ticket.server.id}`)
        }
        break
      case 'delete':
        ElMessageBox.confirm(`确认删除工单 ${ticket.id}?`, '警告', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            // 从列表中移除工单
            const index = tickets.value.findIndex((t) => t.id === ticket.id)
            if (index !== -1) {
              // 更新统计
              const status = tickets.value[index].status
              ticketStats[status] -= 1

              // 删除工单
              tickets.value.splice(index, 1)
              ElMessage.success('工单已删除')
            }
          })
          .catch(() => {})
        break
    }
  }

  // 分页方法
  const handleSizeChange = (val) => {
    pagination.pageSize = val
  }

  const handleCurrentChange = (val) => {
    pagination.currentPage = val
  }

  // 组件挂载时加载数据
  onMounted(() => {
    loading.value = true
    setTimeout(() => {
      loading.value = false
    }, 500)
  })
</script>

<style lang="scss" scoped>
  .ticket-list-container {
    padding: 20px;

    .stats-cards {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 16px;
      margin-bottom: 20px;

      .stat-card {
        .stat-content {
          display: flex;
          align-items: center;

          .stat-icon {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 48px;
            height: 48px;
            border-radius: 8px;
            margin-right: 16px;

            &.yellow {
              background-color: #fef9e7;
              color: #f39c12;
            }

            &.blue {
              background-color: #eaf2fa;
              color: #3498db;
            }

            &.green {
              background-color: #eafaf1;
              color: #2ecc71;
            }

            &.purple {
              background-color: #f4ecf7;
              color: #9b59b6;
            }

            :deep(.el-icon) {
              font-size: 24px;
            }
          }

          .stat-info {
            .stat-label {
              font-size: 14px;
              color: #606266;
              margin-bottom: 4px;
            }

            .stat-value {
              font-size: 20px;
              font-weight: bold;
              color: #303133;
            }
          }
        }
      }
    }

    .search-actions {
      display: flex;
      justify-content: space-between;
      margin-bottom: 20px;

      .search-box {
        width: 300px;
      }

      .action-buttons {
        display: flex;
        gap: 12px;
      }
    }

    .ticket-list-card {
      margin-bottom: 20px;

      :deep(.el-tabs__item) {
        display: flex;
        align-items: center;

        .tab-count {
          margin-left: 8px;
        }
      }

      .ticket-id-title {
        .ticket-id {
          font-weight: 500;
          margin-bottom: 4px;
        }

        .ticket-title {
          color: #606266;
          font-size: 13px;
        }
      }

      .player-info {
        display: flex;
        align-items: center;

        .player-details {
          margin-left: 12px;

          .player-name {
            font-weight: 500;
          }

          .player-id {
            color: #909399;
            font-size: 12px;
          }
        }
      }

      .priority-tag {
        display: flex;
        align-items: center;

        .priority-dot {
          width: 8px;
          height: 8px;
          border-radius: 50%;
          margin-right: 6px;

          &.priority-dot-high {
            background-color: #f56c6c;
          }

          &.priority-dot-medium {
            background-color: #e6a23c;
          }

          &.priority-dot-low {
            background-color: #67c23a;
          }
        }
      }

      .ticket-actions {
        display: flex;
        gap: 10px;
        align-items: center;
      }

      .pagination-container {
        margin-top: 20px;
        display: flex;
        justify-content: flex-end;
      }
    }
  }

  .filter-dropdown-content {
    padding: 12px;
    min-width: 200px;

    .filter-title {
      font-size: 13px;
      color: #606266;
      margin-bottom: 8px;
      font-weight: 500;
    }

    .filter-options {
      display: flex;
      flex-direction: column;
      gap: 8px;
      margin-bottom: 16px;
    }

    .filter-actions {
      display: flex;
      justify-content: flex-end;
      margin-top: 8px;
    }
  }

  :deep(.danger-item) {
    color: #f56c6c;
  }
</style>
