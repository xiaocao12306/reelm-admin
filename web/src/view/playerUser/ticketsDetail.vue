<template>
  <div class="ticket-detail-container">
    <!-- 工单头部信息 -->
    <el-card class="ticket-header mb-4">
      <div class="ticket-header-content">
        <div class="ticket-title-section">
          <div class="title-badges">
            <h1 class="ticket-title">{{ ticket.title }}</h1>
            <el-tag
              :type="getStatusType(ticket.status)"
              size="small"
              class="ml-2"
            >
              {{ getStatusText(ticket.status) }}
            </el-tag>
            <el-tag
              :type="getPriorityType(ticket.priority)"
              size="small"
              class="ml-2"
            >
              <span
                class="priority-dot"
                :class="getPriorityDotClass(ticket.priority)"
              ></span>
              {{ getPriorityText(ticket.priority) }}优先级
            </el-tag>
          </div>
          <p class="ticket-meta">
            工单号: {{ ticket.id }} | 创建于: {{ ticket.createdAt }} | 最后更新:
            {{ ticket.updatedAt }}
          </p>
        </div>
        <div class="ticket-actions">
          <el-button type="primary" @click="handleProcessTicket"
            >处理工单</el-button
          >
          <el-button type="success" @click="handleResolveTicket"
            >解决工单</el-button
          >
        </div>
      </div>

      <div class="ticket-info-grid">
        <!-- 玩家信息 -->
        <div class="info-section">
          <h3 class="info-title">玩家信息</h3>
          <div class="player-info">
            <el-avatar :size="32" :src="ticket.player.avatar"></el-avatar>
            <div class="player-details">
              <p class="player-name">{{ ticket.player.name }}</p>
              <p class="player-email">{{ ticket.player.email }}</p>
            </div>
          </div>
        </div>

        <!-- 服务器信息 -->
        <div class="info-section">
          <h3 class="info-title">服务器信息</h3>
          <div class="server-info">
            <p class="server-name">{{ ticket.server.name }}</p>
            <div class="server-details">
              <span class="server-id">ID: {{ ticket.server.id }}</span>
              <el-tag
                :type="ticket.server.status === 'online' ? 'success' : 'danger'"
                size="small"
                effect="light"
              >
                {{ ticket.server.status === 'online' ? '在线' : '离线' }}
              </el-tag>
            </div>
          </div>
        </div>

        <!-- 处理人员 -->
        <div class="info-section">
          <h3 class="info-title">处理人员</h3>
          <div class="staff-info">
            <el-avatar :size="32" :src="ticket.assignedTo.avatar"></el-avatar>
            <div class="staff-details">
              <p class="staff-name">{{ ticket.assignedTo.name }}</p>
              <p class="staff-last-action">最后操作: {{ ticket.updatedAt }}</p>
            </div>
          </div>
        </div>
      </div>
    </el-card>

    <!-- 主体内容 - 对话和侧边栏 -->
    <div class="ticket-content">
      <!-- 左侧对话区域 -->
      <el-card class="conversation-card">
        <template #header>
          <div class="card-header">
            <span>工单对话</span>
          </div>
        </template>

        <!-- 消息列表 -->
        <div class="message-list" ref="messageList">
          <div
            v-for="message in messages"
            :key="message.id"
            class="message-item"
            :class="{
              'player-message': message.sender === 'player',
              'staff-message': message.sender === 'staff'
            }"
          >
            <div class="message-content">
              <div class="message-header">
                <div class="sender-info">
                  <el-avatar
                    :size="24"
                    :src="
                      message.sender === 'player'
                        ? ticket.player.avatar
                        : message.staffAvatar
                    "
                  ></el-avatar>
                  <span class="sender-name">
                    {{
                      message.sender === 'player'
                        ? ticket.player.name
                        : message.staffName
                    }}
                  </span>
                </div>
                <span class="message-time">{{ message.timestamp }}</span>
              </div>
              <div class="message-text">{{ message.content }}</div>

              <!-- 附件 -->
              <div
                v-if="message.attachments && message.attachments.length > 0"
                class="message-attachments"
              >
                <div
                  v-for="(attachment, index) in message.attachments"
                  :key="index"
                  class="attachment-item"
                >
                  <img
                    v-if="attachment.type === 'image'"
                    :src="attachment.url"
                    :alt="attachment.name"
                    class="attachment-image"
                  />
                  <div v-else class="attachment-file">
                    <el-icon><Document /></el-icon>
                    <span>{{ attachment.name }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 回复区域 -->
        <div class="reply-section">
          <div class="reply-header">
            <span class="reply-title">回复玩家</span>
            <el-select
              v-model="selectedTemplate"
              placeholder="使用模板"
              @change="useTemplate"
              size="small"
              class="template-select"
            >
              <el-option
                v-for="template in replyTemplates"
                :key="template.id"
                :label="template.name"
                :value="template.id"
              />
            </el-select>
          </div>

          <el-input
            v-model="newReply"
            type="textarea"
            :rows="4"
            placeholder="输入回复内容..."
            resize="none"
          />

          <div class="reply-actions">
            <div class="attachment-buttons">
              <el-button
                circle
                :icon="Paperclip"
                @click="handleAttachFile"
              ></el-button>
              <el-button
                circle
                :icon="Picture"
                @click="handleAttachImage"
              ></el-button>
            </div>
            <div class="send-buttons">
              <el-button @click="simulateUserReply">模拟用户回复</el-button>
              <el-button
                type="primary"
                @click="handleSendReply"
                :disabled="!newReply.trim()"
                >发送回复</el-button
              >
            </div>
          </div>
        </div>
      </el-card>

      <!-- 右侧信息栏 -->
      <el-card class="sidebar-card">
        <!-- 内部备注 -->
        <template #header>
          <div class="card-header">
            <span>内部备注</span>
            <span class="header-subtitle">仅对内部员工可见</span>
          </div>
        </template>

        <div class="notes-list">
          <div v-for="note in internalNotes" :key="note.id" class="note-item">
            <div class="note-header">
              <span class="note-author">{{ note.staffName }}</span>
              <span class="note-time">{{ note.timestamp }}</span>
            </div>
            <p class="note-content">{{ note.content }}</p>
          </div>
        </div>

        <!-- 添加内部备注 -->
        <div class="add-note-section">
          <el-input
            v-model="newNote"
            type="textarea"
            :rows="3"
            placeholder="添加内部备注..."
            resize="none"
          />
          <el-button
            type="primary"
            class="add-note-button"
            @click="handleAddNote"
            :disabled="!newNote.trim()"
            >添加备注</el-button
          >
        </div>

        <!-- 工单操作 -->
        <div class="ticket-operations">
          <h3 class="operations-title">工单操作</h3>

          <div class="operation-item">
            <span class="operation-label">修改状态</span>
            <el-select v-model="ticket.status" class="operation-select">
              <el-option label="待处理" value="open" />
              <el-option label="处理中" value="processing" />
              <el-option label="已关闭" value="closed" />
            </el-select>
          </div>

          <div class="operation-item">
            <span class="operation-label">修改优先级</span>
            <el-select v-model="ticket.priority" class="operation-select">
              <el-option label="低" value="low" />
              <el-option label="中" value="medium" />
              <el-option label="高" value="high" />
            </el-select>
          </div>

          <div class="operation-item">
            <span class="operation-label">重新分配给</span>
            <el-select v-model="ticket.assignedTo.id" class="operation-select">
              <el-option label="张客服" value="staff001" />
              <el-option label="李客服" value="staff002" />
              <el-option label="王客服" value="staff003" />
              <el-option label="赵客服" value="staff004" />
            </el-select>
          </div>
        </div>
      </el-card>
    </div>
  </div>
</template>

<script setup>
  import { ref, reactive, nextTick, onMounted } from 'vue'
  import { ElMessage } from 'element-plus'
  import { Document, Picture, Paperclip } from '@element-plus/icons-vue'

  // 模拟工单数据
  const ticket = reactive({
    id: 'TK-2025031',
    title: '无法访问我的Minecraft服务器',
    status: 'processing', // open, processing, closed
    priority: 'high',
    createdAt: '2025-03-17 09:12:43',
    updatedAt: '2025-03-17 15:42:18',
    player: {
      id: 'player123',
      name: 'MineKing',
      email: 'player123@example.com',
      avatar: '/api/placeholder/36/36'
    },
    server: {
      id: 'MC-2025042',
      name: 'MineKingdom',
      status: 'offline'
    },
    assignedTo: {
      id: 'staff001',
      name: '张客服',
      avatar: '/api/placeholder/36/36'
    }
  })

  // 模拟对话消息
  const messages = ref([
    {
      id: 1,
      sender: 'player',
      content:
        '我今天早上尝试登录我的Minecraft服务器(MineKingdom)，但是一直无法连接。显示"无法连接到服务器"的错误。我已经尝试了重启客户端，但问题依然存在。昨天晚上服务器还能正常访问，今天突然就连不上了。服务器应该还在订阅期内，我上个月刚续费了3个月。请帮我查看一下是什么问题，谢谢！',
      timestamp: '2025-03-17 09:12:43',
      attachments: []
    },
    {
      id: 2,
      sender: 'staff',
      content:
        '您好，感谢您联系我们的客服团队。我们注意到您的Minecraft服务器目前无法访问的问题。我已经查看了您的服务器状态，发现服务器确实处于离线状态，可能是由于系统资源使用过高导致服务器崩溃。\n\n我已经执行了服务器重启操作，现在应该可以正常访问了。请您尝试重新连接，看看问题是否解决。',
      timestamp: '2025-03-17 09:25:17',
      staffName: '张客服',
      staffAvatar: '/api/placeholder/36/36'
    },
    {
      id: 3,
      sender: 'player',
      content:
        '谢谢回复。我刚刚尝试连接，确实可以进入服务器了。但是我发现我的一些存档似乎丢失了，上次我建造的城堡不见了。这是怎么回事？能恢复吗？',
      timestamp: '2025-03-17 09:42:05',
      attachments: [
        {
          name: 'screenshot.jpg',
          url: '/api/placeholder/200/100',
          type: 'image'
        }
      ]
    }
  ])

  // 内部备注
  const internalNotes = ref([
    {
      id: 1,
      staffName: '系统',
      content: '检测到该服务器在过去24小时内有3次高负载警报',
      timestamp: '2025-03-17 09:15:22'
    },
    {
      id: 2,
      staffName: '张客服',
      content: '检查了日志，发现是内存溢出导致的崩溃。已重启并调整了内存分配。',
      timestamp: '2025-03-17 09:22:10'
    }
  ])

  // 回复模板
  const replyTemplates = [
    {
      id: 1,
      name: '服务器重启回复',
      content:
        '亲爱的玩家，\n\n我们已经检测到您的服务器出现了问题。我们已经执行了服务器重启操作，现在服务器应该已经恢复正常。请尝试再次连接。\n\n如果您仍然遇到问题，请回复此工单，我们将进一步协助您解决问题。\n\n游戏托管服务团队'
    },
    {
      id: 2,
      name: '数据恢复回复',
      content:
        '亲爱的玩家，\n\n关于您提到的数据丢失问题，我们可以尝试从最近的备份中恢复您的数据。我们的系统会每24小时自动进行一次备份。\n\n请告知您希望恢复到哪个时间点的数据，我们将尽快为您处理。请注意，恢复操作可能会覆盖当前的游戏数据。\n\n游戏托管服务团队'
    },
    {
      id: 3,
      name: '技术支持回复',
      content:
        '亲爱的玩家，\n\n感谢您提供的详细信息。为了更好地解决您的问题，我们需要查看服务器日志。我已经安排技术团队检查相关日志并分析问题原因。\n\n我们会在24小时内给您回复调查结果。如果您有任何其他问题，请随时告知我们。\n\n游戏托管服务团队'
    }
  ]

  // 表单数据
  const newReply = ref('')
  const newNote = ref('')
  const selectedTemplate = ref('')
  const messageList = ref(null)

  // 获取状态对应的文本
  const getStatusText = (status) => {
    const statusMap = {
      open: '待处理',
      processing: '处理中',
      closed: '已关闭'
    }
    return statusMap[status] || status
  }

  // 获取状态对应的类型
  const getStatusType = (status) => {
    const typeMap = {
      open: 'info',
      processing: 'warning',
      closed: 'success'
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

  // 处理发送回复
  const handleSendReply = () => {
    if (!newReply.value.trim()) return

    const newMessage = {
      id: messages.value.length + 1,
      sender: 'staff',
      content: newReply.value,
      timestamp: new Date()
        .toLocaleString('zh-CN', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit',
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit',
          hour12: false
        })
        .replace(/\//g, '-'),
      staffName: '张客服',
      staffAvatar: '/api/placeholder/36/36'
    }

    messages.value.push(newMessage)
    newReply.value = ''
    selectedTemplate.value = ''

    // 更新工单最后更新时间
    ticket.updatedAt = newMessage.timestamp

    // 滚动到底部
    scrollToBottom()

    ElMessage.success('回复已发送')
  }

  // 处理添加内部备注
  const handleAddNote = () => {
    if (!newNote.value.trim()) return

    const note = {
      id: internalNotes.value.length + 1,
      staffName: '张客服',
      content: newNote.value,
      timestamp: new Date()
        .toLocaleString('zh-CN', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit',
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit',
          hour12: false
        })
        .replace(/\//g, '-')
    }

    internalNotes.value.push(note)
    newNote.value = ''

    ElMessage.success('备注已添加')
  }

  // 使用模板
  const useTemplate = (templateId) => {
    const template = replyTemplates.find((t) => t.id === templateId)
    if (template) {
      newReply.value = template.content
    }
  }

  // 模拟用户发送新消息
  const simulateUserReply = () => {
    const userReply = {
      id: messages.value.length + 1,
      sender: 'player',
      content:
        '感谢您的帮助！如果可以从备份恢复，请使用昨天（2025-03-16）的备份，那时我的城堡还在。另外，请问如何防止服务器再次崩溃？我计划邀请更多朋友加入服务器，担心会有稳定性问题。',
      timestamp: new Date()
        .toLocaleString('zh-CN', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit',
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit',
          hour12: false
        })
        .replace(/\//g, '-'),
      attachments: []
    }

    messages.value.push(userReply)

    // 更新工单最后更新时间
    ticket.updatedAt = userReply.timestamp

    // 滚动到底部
    scrollToBottom()

    ElMessage.info('收到用户新消息')
  }

  // 处理工单
  const handleProcessTicket = () => {
    ticket.status = 'processing'
    ElMessage.success('工单已标记为处理中')
  }

  // 解决工单
  const handleResolveTicket = () => {
    ticket.status = 'closed'
    ElMessage.success('工单已标记为已解决')
  }

  // 附件处理
  const handleAttachFile = () => {
    ElMessage.info('文件上传功能待实现')
  }

  const handleAttachImage = () => {
    ElMessage.info('图片上传功能待实现')
  }

  // 滚动到底部
  const scrollToBottom = () => {
    nextTick(() => {
      if (messageList.value) {
        messageList.value.scrollTop = messageList.value.scrollHeight
      }
    })
  }

  // 组件挂载时滚动到底部
  onMounted(() => {
    scrollToBottom()
  })
</script>

<style lang="scss" scoped>
  .ticket-detail-container {
    padding: 20px;

    .ticket-header {
      .ticket-header-content {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;
        margin-bottom: 16px;

        @media (max-width: 768px) {
          flex-direction: column;

          .ticket-actions {
            margin-top: 12px;
            align-self: flex-end;
          }
        }

        .ticket-title-section {
          .title-badges {
            display: flex;
            align-items: center;
            flex-wrap: wrap;

            .ticket-title {
              font-size: 18px;
              font-weight: bold;
              margin: 0;
              margin-right: 8px;
            }

            .priority-dot {
              display: inline-block;
              width: 8px;
              height: 8px;
              border-radius: 50%;
              margin-right: 4px;

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

          .ticket-meta {
            font-size: 12px;
            color: #909399;
            margin: 4px 0 0 0;
          }
        }
      }

      .ticket-info-grid {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 16px;
        padding-top: 16px;
        border-top: 1px solid #ebeef5;

        @media (max-width: 768px) {
          grid-template-columns: 1fr;
        }

        .info-section {
          .info-title {
            font-size: 13px;
            color: #606266;
            margin: 0 0 8px 0;
            font-weight: normal;
          }

          .player-info,
          .server-info,
          .staff-info {
            display: flex;
            align-items: center;

            .player-details,
            .server-details,
            .staff-details {
              margin-left: 12px;

              .player-name,
              .server-name,
              .staff-name {
                font-size: 14px;
                font-weight: 500;
                margin: 0;
                color: #303133;
              }

              .player-email,
              .server-id,
              .staff-last-action {
                font-size: 12px;
                color: #909399;
                margin: 4px 0 0 0;
              }

              .server-details {
                display: flex;
                align-items: center;
                gap: 8px;
              }
            }
          }
        }
      }
    }

    .ticket-content {
      display: flex;
      gap: 20px;

      @media (max-width: 992px) {
        flex-direction: column;
      }

      .conversation-card {
        flex: 1;

        .card-header {
          font-weight: bold;
        }

        .message-list {
          max-height: 500px;
          overflow-y: auto;
          padding: 0 0 16px 0;

          .message-item {
            margin-bottom: 16px;
            display: flex;

            &.player-message {
              justify-content: flex-start;

              .message-content {
                background-color: #f5f7fa;
              }
            }

            &.staff-message {
              justify-content: flex-end;

              .message-content {
                background-color: #ecf5ff;
              }
            }

            .message-content {
              max-width: 80%;
              border-radius: 8px;
              padding: 12px;

              .message-header {
                display: flex;
                justify-content: space-between;
                align-items: center;
                margin-bottom: 8px;

                .sender-info {
                  display: flex;
                  align-items: center;

                  .sender-name {
                    margin-left: 8px;
                    font-weight: 500;
                    font-size: 14px;
                  }
                }

                .message-time {
                  font-size: 12px;
                  color: #909399;
                }
              }

              .message-text {
                font-size: 14px;
                line-height: 1.5;
                white-space: pre-line;
              }

              .message-attachments {
                margin-top: 12px;

                .attachment-item {
                  display: inline-block;
                  margin-right: 8px;
                  margin-bottom: 8px;

                  .attachment-image {
                    max-width: 200px;
                    border-radius: 4px;
                    border: 1px solid #ebeef5;
                  }

                  .attachment-file {
                    display: flex;
                    align-items: center;
                    padding: 6px 10px;
                    background-color: #f5f7fa;
                    border-radius: 4px;
                    border: 1px solid #ebeef5;

                    .el-icon {
                      margin-right: 6px;
                      color: #909399;
                    }

                    span {
                      font-size: 12px;
                    }
                  }
                }
              }
            }
          }
        }

        .reply-section {
          border-top: 1px solid #ebeef5;
          padding-top: 16px;

          .reply-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;

            .reply-title {
              font-size: 14px;
              font-weight: 500;
            }

            .template-select {
              width: 150px;
            }
          }

          .reply-actions {
            display: flex;
            justify-content: space-between;
            margin-top: 12px;

            .attachment-buttons {
              display: flex;
              gap: 8px;
            }

            .send-buttons {
              display: flex;
              gap: 8px;
            }
          }
        }
      }

      .sidebar-card {
        width: 300px;

        @media (max-width: 992px) {
          width: 100%;
        }

        .card-header {
          font-weight: bold;
          display: flex;
          align-items: center;

          .header-subtitle {
            font-size: 12px;
            color: #909399;
            margin-left: 8px;
            font-weight: normal;
          }
        }

        .notes-list {
          max-height: 300px;
          overflow-y: auto;
          margin-bottom: 16px;

          .note-item {
            background-color: #f5f7fa;
            border-radius: 6px;
            padding: 12px;
            margin-bottom: 12px;

            .note-header {
              display: flex;
              justify-content: space-between;
              margin-bottom: 8px;

              .note-author {
                font-weight: 500;
                font-size: 13px;
              }

              .note-time {
                font-size: 12px;
                color: #909399;
              }
            }

            .note-content {
              font-size: 13px;
              margin: 0;
              line-height: 1.5;
            }
          }
        }

        .add-note-section {
          margin-bottom: 20px;

          .add-note-button {
            width: 100%;
            margin-top: 8px;
          }
        }

        .ticket-operations {
          border-top: 1px solid #ebeef5;
          padding-top: 16px;

          .operations-title {
            font-size: 14px;
            font-weight: 500;
            margin: 0 0 12px 0;
          }

          .operation-item {
            margin-bottom: 12px;

            .operation-label {
              display: block;
            }
          }
        }
      }
    }
  }
</style>
