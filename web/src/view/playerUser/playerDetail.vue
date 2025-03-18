<template>
  <div class="player-detail-container">
    <!-- 头部区域 -->
    <div class="header-section">
      <div class="header-content">
        <div class="header-left">
          <el-button icon="Back" @click="handleBack" class="back-button"
            >返回</el-button
          >
          <h1 class="page-title">玩家详情</h1>
          <el-tag
            :type="
              playerData.verificationStatus === 'verified'
                ? 'success'
                : 'warning'
            "
          >
            {{
              playerData.verificationStatus === 'verified' ? '已实名' : '未实名'
            }}
          </el-tag>
        </div>
        <div class="header-actions">
          <el-button type="primary" @click="handleEdit">编辑</el-button>
          <el-button type="danger" @click="handleDelete">删除</el-button>
        </div>
      </div>
    </div>

    <!-- 玩家基本信息卡片 -->
    <el-card class="info-card">
      <div class="player-profile">
        <div class="avatar">
          {{
            playerData.nickname
              ? playerData.nickname.charAt(0).toUpperCase()
              : '?'
          }}
        </div>
        <div class="player-info">
          <h2 class="player-name">{{ playerData.nickname }}</h2>
          <p class="player-email">{{ playerData.email }}</p>
        </div>
        <div class="player-meta">
          <p>ID: {{ playerData.id }}</p>
          <p>注册时间: {{ playerData.createdAt }}</p>
          <p>最后登录: {{ playerData.lastLoginTime }}</p>
        </div>
      </div>
    </el-card>

    <!-- 导航标签 -->
    <el-card class="tabs-card">
      <el-tabs v-model="activeTab">
        <el-tab-pane label="基本信息" name="basic"></el-tab-pane>
        <el-tab-pane
          :label="`服务器(${playerServers.length})`"
          name="servers"
        ></el-tab-pane>
        <el-tab-pane
          :label="`订单(${playerOrders.length})`"
          name="orders"
        ></el-tab-pane>
        <el-tab-pane label="日志" name="logs"></el-tab-pane>
      </el-tabs>

      <!-- 基本信息标签内容 -->
      <div v-if="activeTab === 'basic'" class="tab-content">
        <div class="info-grid">
          <!-- 账户信息卡片 -->
          <div class="info-section">
            <h3 class="section-title">账户信息</h3>
            <div class="info-grid-content">
              <div class="info-item">
                <p class="info-label">邮箱</p>
                <p class="info-value">{{ playerData.email }}</p>
              </div>
              <div class="info-item">
                <p class="info-label">昵称</p>
                <p class="info-value">{{ playerData.nickname }}</p>
              </div>
              <div class="info-item">
                <p class="info-label">注册时间</p>
                <p class="info-value">{{ playerData.createdAt }}</p>
              </div>
              <div class="info-item">
                <p class="info-label">最后登录</p>
                <p class="info-value">{{ playerData.lastLoginTime }}</p>
              </div>
              <div class="info-item">
                <p class="info-label">实名状态</p>
                <el-tag
                  size="small"
                  :type="
                    playerData.verificationStatus === 'verified'
                      ? 'success'
                      : 'warning'
                  "
                >
                  {{
                    playerData.verificationStatus === 'verified'
                      ? '已实名'
                      : '未实名'
                  }}
                </el-tag>
              </div>
              <div class="info-item">
                <p class="info-label">账户状态</p>
                <el-tag size="small" type="success">正常</el-tag>
              </div>
            </div>
          </div>

          <!-- 账户统计卡片 -->
          <div class="info-section">
            <h3 class="section-title">账户统计</h3>
            <div class="stats-grid">
              <div class="stat-item">
                <p class="stat-label">服务器数量</p>
                <p class="stat-value stat-blue">{{ playerServers.length }}</p>
              </div>
              <div class="stat-item">
                <p class="stat-label">订单总数</p>
                <p class="stat-value stat-green">{{ playerOrders.length }}</p>
              </div>
              <div class="stat-item">
                <p class="stat-label">消费总额</p>
                <p class="stat-value stat-orange">
                  ¥{{ calculateTotalSpending().toFixed(2) }}
                </p>
              </div>
              <div class="stat-item">
                <p class="stat-label">最近活跃</p>
                <p class="stat-value stat-purple">
                  {{ playerData.lastLoginTime }}
                </p>
              </div>
            </div>
          </div>

          <!-- 备注区域 -->
          <div class="remark-section">
            <h3 class="section-title">备注</h3>
            <div v-if="savedRemark" class="saved-remark">
              <p>{{ savedRemark }}</p>
            </div>
            <el-input
              v-model="remark"
              type="textarea"
              :rows="3"
              placeholder="添加备注信息..."
            />
            <div class="remark-actions">
              <el-button type="primary" @click="handleSaveRemark"
                >保存备注</el-button
              >
            </div>
          </div>
        </div>
      </div>

      <!-- 服务器标签内容 -->
      <div v-if="activeTab === 'servers'" class="tab-content">
        <div class="tab-header">
          <h3 class="section-title">关联服务器</h3>
          <el-button type="primary">添加服务器</el-button>
        </div>

        <div v-if="playerServers.length > 0" class="servers-grid">
          <el-card
            v-for="server in playerServers"
            :key="server.id"
            class="server-card"
          >
            <div class="server-header">
              <div class="server-title">
                <h4>{{ server.name }}</h4>
                <el-tag
                  size="small"
                  :type="server.status === 'online' ? 'success' : 'danger'"
                >
                  {{ server.status === 'online' ? '在线' : '离线' }}
                </el-tag>
              </div>
              <div class="server-actions">
                <el-button
                  size="small"
                  type="primary"
                  @click="handleServerAction(server.id, '控制台')"
                >
                  控制台
                </el-button>
                <el-dropdown
                  @command="(command) => handleServerAction(server.id, command)"
                >
                  <el-button size="small">
                    更多<el-icon class="el-icon--right"><arrow-down /></el-icon>
                  </el-button>
                  <template #dropdown>
                    <el-dropdown-menu>
                      <el-dropdown-item command="启动">启动</el-dropdown-item>
                      <el-dropdown-item command="停止">停止</el-dropdown-item>
                      <el-dropdown-item command="重启">重启</el-dropdown-item>
                      <el-dropdown-item divided command="删除"
                        >删除</el-dropdown-item
                      >
                    </el-dropdown-menu>
                  </template>
                </el-dropdown>
              </div>
            </div>
            <div class="server-details">
              <div class="detail-item">
                <span class="detail-label">IP:</span> {{ server.ip }}
              </div>
              <div class="detail-item">
                <span class="detail-label">端口:</span> {{ server.port }}
              </div>
              <div class="detail-item">
                <span class="detail-label">创建日期:</span>
                {{ server.createdAt }}
              </div>
              <div class="detail-item">
                <span class="detail-label">最后活跃:</span>
                {{ server.lastActive }}
              </div>
            </div>
          </el-card>
        </div>
        <el-empty v-else description="该玩家没有关联的服务器"></el-empty>
      </div>

      <!-- 订单标签内容 -->
      <div v-if="activeTab === 'orders'" class="tab-content">
        <div class="tab-header">
          <h3 class="section-title">订单历史</h3>
          <div class="tab-actions">
            <el-button>筛选</el-button>
            <el-button type="primary">创建订单</el-button>
          </div>
        </div>

        <el-table
          v-if="playerOrders.length > 0"
          :data="playerOrders"
          border
          style="width: 100%"
        >
          <el-table-column prop="id" label="订单号" />
          <el-table-column label="金额">
            <template #default="scope">
              ¥{{ scope.row.amount.toFixed(2) }}
            </template>
          </el-table-column>
          <el-table-column label="状态">
            <template #default="scope">
              <el-tag :type="getOrderStatusType(scope.row.status)" size="small">
                {{ getOrderStatusText(scope.row.status) }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="serverName" label="服务器" />
          <el-table-column prop="createTime" label="创建时间" />
          <el-table-column label="操作" width="100">
            <template #default="scope">
              <el-button link type="primary" @click="viewOrderDetail(scope.row)"
                >详情</el-button
              >
            </template>
          </el-table-column>
        </el-table>
        <el-empty v-else description="该玩家没有订单记录"></el-empty>
      </div>

      <!-- 日志标签内容 -->
      <div v-if="activeTab === 'logs'" class="tab-content">
        <div class="tab-header">
          <h3 class="section-title">操作日志</h3>
          <div class="tab-actions">
            <el-button>筛选</el-button>
            <el-button>导出</el-button>
          </div>
        </div>

        <div class="logs-list">
          <el-card v-for="log in playerLogs" :key="log.id" class="log-item">
            <div class="log-header">
              <span class="log-type">{{ log.type }}</span>
              <span class="log-time">{{ log.time }}</span>
            </div>
            <div class="log-details">
              <div v-if="log.detail">操作: {{ log.detail }}</div>
              <div v-if="log.amount">金额: ¥{{ log.amount.toFixed(2) }}</div>
              <div>IP: {{ log.ip }}</div>
              <div v-if="log.device">设备: {{ log.device }}</div>
            </div>
          </el-card>
        </div>
      </div>
    </el-card>

    <!-- 底部操作按钮 -->
    <el-card class="footer-card">
      <div class="footer-content">
        <div class="last-updated">
          <span>最后更新: {{ playerData.lastUpdated || '无更新记录' }}</span>
        </div>
        <div class="footer-actions">
          <el-button @click="handleBack">返回列表</el-button>
          <el-button type="primary" @click="handleManageServers"
            >管理服务器</el-button
          >
        </div>
      </div>
    </el-card>
  </div>
</template>

<script setup>
  import { ref, computed, reactive } from 'vue'
  import { useRouter } from 'vue-router'
  import { ElMessage, ElMessageBox } from 'element-plus'
  import { ArrowDown } from '@element-plus/icons-vue'

  const router = useRouter()

  // 当前激活的标签页
  const activeTab = ref('basic')

  // 模拟数据 - 玩家基本信息
  const playerData = reactive({
    id: 1003,
    email: 'player3@example.com',
    nickname: '风暴使者',
    verificationStatus: 'verified',
    createdAt: '2025-02-18',
    lastLoginTime: '2025-03-15',
    servers: 3,
    orders: 7,
    lastUpdated: '2025-03-17 16:30:45'
  })

  // 模拟数据 - 玩家关联的服务器
  const playerServers = ref([
    {
      id: 1,
      name: 'Minecraft生存服务器',
      status: 'online',
      ip: '192.168.1.1',
      port: 25565,
      createdAt: '2025-01-20',
      lastActive: '2025-03-17'
    },
    {
      id: 2,
      name: 'Rust沙盒世界',
      status: 'offline',
      ip: '192.168.1.2',
      port: 28015,
      createdAt: '2025-02-05',
      lastActive: '2025-03-12'
    },
    {
      id: 3,
      name: 'ARK恐龙岛',
      status: 'online',
      ip: '192.168.1.3',
      port: 27015,
      createdAt: '2025-02-25',
      lastActive: '2025-03-16'
    }
  ])

  // 模拟数据 - 玩家订单
  const playerOrders = ref([
    {
      id: 'ORD-2025-001',
      amount: 199.0,
      status: 'paid',
      createTime: '2025-01-20',
      payTime: '2025-01-20',
      serverName: 'Minecraft生存服务器'
    },
    {
      id: 'ORD-2025-023',
      amount: 299.0,
      status: 'paid',
      createTime: '2025-02-05',
      payTime: '2025-02-05',
      serverName: 'Rust沙盒世界'
    },
    {
      id: 'ORD-2025-085',
      amount: 199.0,
      status: 'refunded',
      createTime: '2025-03-01',
      payTime: '2025-03-01',
      serverName: 'Minecraft生存服务器'
    },
    {
      id: 'ORD-2025-112',
      amount: 399.0,
      status: 'paid',
      createTime: '2025-03-10',
      payTime: '2025-03-10',
      serverName: 'ARK恐龙岛'
    }
  ])

  // 模拟数据 - 操作日志
  const playerLogs = ref([
    {
      id: 1,
      type: '登录',
      time: '2025-03-17 15:30:22',
      ip: '123.45.67.89',
      device: 'Windows Chrome'
    },
    {
      id: 2,
      type: '服务器控制',
      time: '2025-03-17 15:35:12',
      ip: '123.45.67.89',
      detail: '启动Minecraft服务器'
    },
    {
      id: 3,
      type: '系统操作',
      time: '2025-03-17 16:05:47',
      ip: '123.45.67.89'
    },
    {
      id: 4,
      type: '登录',
      time: '2025-03-16 09:12:45',
      ip: '123.45.67.89',
      device: 'iOS Safari'
    },
    {
      id: 5,
      type: '账户设置修改',
      time: '2025-03-16 09:20:33',
      ip: '123.45.67.89',
      detail: '修改密码'
    }
  ])

  // 备注状态
  const remark = ref('')
  const savedRemark = ref('')

  // 计算总消费
  const calculateTotalSpending = () => {
    return playerOrders.value.reduce((total, order) => {
      return total + (order.status !== 'refunded' ? order.amount : 0)
    }, 0)
  }

  // 获取订单状态对应的类型
  const getOrderStatusType = (status) => {
    const typeMap = {
      paid: 'success',
      unpaid: 'warning',
      refunded: 'info',
      failed: 'danger'
    }
    return typeMap[status] || 'info'
  }

  // 获取订单状态对应的文本
  const getOrderStatusText = (status) => {
    const statusMap = {
      paid: '已支付',
      unpaid: '未支付',
      refunded: '已退款',
      failed: '失败'
    }
    return statusMap[status] || status
  }

  // 处理保存备注
  const handleSaveRemark = () => {
    if (!remark.value.trim()) {
      ElMessage.warning('备注内容不能为空')
      return
    }

    savedRemark.value = remark.value
    ElMessage.success('备注已保存')
  }

  // 处理服务器操作
  const handleServerAction = (serverId, action) => {
    ElMessage.info(`对服务器 ${serverId} 执行 ${action} 操作`)
  }

  // 查看订单详情
  const viewOrderDetail = (order) => {
    ElMessage.info(`查看订单详情: ${order.id}`)
  }

  // 返回列表
  const handleBack = () => {
    router.go(-1)
  }

  // 编辑玩家信息
  const handleEdit = () => {
    ElMessage.info('编辑玩家信息')
  }

  // 删除玩家
  const handleDelete = () => {
    ElMessageBox.confirm('确定要删除该玩家账户吗？此操作不可恢复。', '警告', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    })
      .then(() => {
        ElMessage.success('玩家已删除')
        handleBack()
      })
      .catch(() => {})
  }

  // 管理服务器
  const handleManageServers = () => {
    router.push({
      path: '/playerUser/serverList',
      query: {
        playerId: playerData.id,
        playerEmail: playerData.email,
        autoSearch: 'true'
      }
    })
  }
</script>

<style lang="scss" scoped>
  .player-detail-container {
    padding: 20px;

    .header-section {
      background-color: #fff;
      box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
      border-radius: 4px;
      margin-bottom: 20px;

      .header-content {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 16px 20px;

        .header-left {
          display: flex;
          align-items: center;

          .back-button {
            margin-right: 16px;
          }

          .page-title {
            font-size: 20px;
            font-weight: bold;
            margin: 0 16px 0 0;
          }
        }

        .header-actions {
          display: flex;
          gap: 10px;
        }
      }
    }

    .info-card {
      margin-bottom: 20px;

      .player-profile {
        display: flex;
        align-items: center;

        .avatar {
          width: 64px;
          height: 64px;
          border-radius: 50%;
          background-color: #e6f1fc;
          color: #409eff;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 24px;
          font-weight: bold;
          margin-right: 20px;
        }

        .player-info {
          flex-grow: 1;

          .player-name {
            font-size: 20px;
            font-weight: bold;
            margin: 0 0 4px 0;
          }

          .player-email {
            color: #606266;
            margin: 0;
          }
        }

        .player-meta {
          text-align: right;

          p {
            margin: 4px 0;
            color: #909399;
          }
        }
      }
    }

    .tabs-card {
      margin-bottom: 20px;

      .tab-content {
        padding: 20px 0;

        .tab-header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 20px;

          .tab-actions {
            display: flex;
            gap: 10px;
          }
        }

        .info-grid {
          display: grid;
          grid-template-columns: 1fr 1fr;
          gap: 20px;

          @media (max-width: 992px) {
            grid-template-columns: 1fr;
          }

          .remark-section {
            grid-column: span 2;

            @media (max-width: 992px) {
              grid-column: span 1;
            }

            .saved-remark {
              background-color: #fdf9e8;
              border: 1px solid #faecd8;
              border-radius: 4px;
              padding: 16px;
              margin-bottom: 16px;
            }

            .remark-actions {
              display: flex;
              justify-content: flex-end;
              margin-top: 12px;
            }
          }
        }

        .info-section {
          background-color: #f5f7fa;
          border-radius: 4px;
          padding: 16px;

          .section-title {
            font-size: 16px;
            font-weight: bold;
            margin: 0 0 16px 0;
            color: #303133;
          }

          .info-grid-content {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 16px;

            .info-item {
              .info-label {
                font-size: 13px;
                color: #909399;
                margin: 0 0 4px 0;
              }

              .info-value {
                font-size: 14px;
                color: #303133;
                margin: 0;
              }
            }
          }

          .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;

            .stat-item {
              background-color: #fff;
              border-radius: 4px;
              padding: 12px;
              box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);

              .stat-label {
                font-size: 13px;
                color: #909399;
                margin: 0 0 4px 0;
              }

              .stat-value {
                font-size: 20px;
                font-weight: bold;
                margin: 0;

                &.stat-blue {
                  color: #409eff;
                }

                &.stat-green {
                  color: #67c23a;
                }

                &.stat-orange {
                  color: #e6a23c;
                }

                &.stat-purple {
                  color: #9c27b0;
                }
              }
            }
          }
        }

        .servers-grid {
          display: grid;
          grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
          gap: 20px;

          @media (max-width: 576px) {
            grid-template-columns: 1fr;
          }

          .server-card {
            .server-header {
              display: flex;
              justify-content: space-between;
              align-items: center;
              margin-bottom: 16px;

              .server-title {
                display: flex;
                align-items: center;
                gap: 8px;

                h4 {
                  margin: 0;
                  font-size: 16px;
                  font-weight: bold;
                }
              }

              .server-actions {
                display: flex;
                gap: 8px;
              }
            }

            .server-details {
              display: grid;
              grid-template-columns: 1fr 1fr;
              gap: 8px;

              .detail-item {
                font-size: 13px;

                .detail-label {
                  color: #909399;
                }
              }
            }
          }
        }

        .logs-list {
          display: flex;
          flex-direction: column;
          gap: 12px;

          .log-item {
            .log-header {
              display: flex;
              justify-content: space-between;
              margin-bottom: 8px;

              .log-type {
                font-weight: 500;
              }

              .log-time {
                font-size: 13px;
                color: #909399;
              }
            }

            .log-details {
              font-size: 13px;
              color: #606266;

              div {
                margin-bottom: 4px;
              }
            }
          }
        }
      }
    }

    .footer-card {
      .footer-content {
        display: flex;
        justify-content: space-between;
        align-items: center;

        .last-updated {
          font-size: 13px;
          color: #909399;
        }

        .footer-actions {
          display: flex;
          gap: 10px;
        }
      }
    }
  }
</style>
