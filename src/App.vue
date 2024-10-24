<template>
  <el-container class="chat-container">
    <!-- Hamburger menu button -->
    <el-button @click="toggleSidebar" class="btn-menu" icon="el-icon-menu"></el-button>

    <!-- Sidebar -->
    <el-aside width="300px" :class="{ 'active': isSidebarOpen }" id="leftSidebar">
      <el-container direction="vertical">
        <el-header height="auto">
          <div class="sidebar-header">
            <h1 class="sidebar-title">ChatFUN</h1>
            <el-button @click="startNewChat" circle>
              <img src="/new.svg" alt="新建对话" style="width: 16px; height: 16px;">
            </el-button>
          </div>
        </el-header>

        <!-- Search input -->
        <el-header height="auto">
          <el-input class="search-input" placeholder="搜索对话..." @input="handleSearch"></el-input>
        </el-header>

        <el-main class="sidebar-content">
          <el-menu :default-active="activeChatId">
            <el-menu-item v-for="(ssid, index) in displaySSIDs" :key="index" :index="ssid"
              @click="switchChat(ssid, `对话${uniqueSSIDs.indexOf(ssid) + 1}`)">
              对话{{ uniqueSSIDs.indexOf(ssid) + 1 }}
            </el-menu-item>
          </el-menu>
        </el-main>
      </el-container>
    </el-aside>

    <!-- Main content -->
    <el-main class="main-content">
      <el-header>
        <h2>{{ activeTitle }}</h2>
      </el-header>

      <el-main class="message-list">
        <el-card v-for="(message, index) in filteredMessages" :key="index"
          :class="{ 'user-message': message.user === 'user' }">
          {{ message.content }}
        </el-card>
      </el-main>

      <el-footer height="auto" class="input-area">
        <el-input v-model="inputMessage" placeholder="输入消息..." @keyup.enter.native="handleSendMessage">
          <el-button slot="append" icon="el-icon-s-promotion" @click="handleSendMessage"></el-button>
        </el-input>
      </el-footer>
    </el-main>
  </el-container>
</template>

<script>
import { ref, computed } from 'vue'

export default {
  setup() {
    const bjtime = new Date().toLocaleString()
    const uuid = ref(Date.now().toString())
    const sequenceRef = ref(0)
    const titleCount = ref(1)

    const messages = ref([{ ssid: uuid.value, seq: sequenceRef.value, user: 'ai', content: "有什么可以帮你的？", timestamp: bjtime }])
    const isProcessing = ref(false)
    const activeChatId = ref(uuid.value)
    const activeTitle = ref("对话1")
    const searchQuery = ref('')
    const filteredSSIDs = ref([])
    const inputMessage = ref('')
    const isSidebarOpen = ref(true)

    const uniqueSSIDs = computed(() => [...new Set(messages.value.map(message => message.ssid))])
    const displaySSIDs = computed(() => searchQuery.value ? filteredSSIDs.value : uniqueSSIDs.value)
    const filteredMessages = computed(() => messages.value.filter(message => message.ssid === activeChatId.value))

    const startNewChat = () => {
      const newUuid = Date.now().toString()
      uuid.value = newUuid
      activeChatId.value = newUuid
      activeTitle.value = `对话${++titleCount.value}`
      messages.value.push({ ssid: newUuid, seq: 0, user: 'ai', content: "有什么可以帮你的？", timestamp: new Date().toLocaleString() })
    }

    const handleSearch = (query) => {
      filteredSSIDs.value = uniqueSSIDs.value.filter(ssid =>
        messages.value.some(message =>
          message.ssid === ssid && message.content.toLowerCase().includes(query.toLowerCase())
        )
      )
    }

    const switchChat = (ssid, title) => {
      activeChatId.value = ssid
      activeTitle.value = title
    }

    const handleSendMessage = () => {
      if (inputMessage.value.trim()) {
        messages.value.push({
          ssid: activeChatId.value,
          seq: ++sequenceRef.value,
          user: 'user',
          content: inputMessage.value,
          timestamp: new Date().toLocaleString()
        })
        inputMessage.value = ''
        // Here you would typically call your API to get the AI response
      }
    }

    const toggleSidebar = () => {
      isSidebarOpen.value = !isSidebarOpen.value
    }

    return {
      messages,
      activeChatId,
      activeTitle,
      searchQuery,
      filteredSSIDs,
      inputMessage,
      isSidebarOpen,
      uniqueSSIDs,
      displaySSIDs,
      filteredMessages,
      startNewChat,
      handleSearch,
      switchChat,
      handleSendMessage,
      toggleSidebar
    }
  }
}
</script>

<style scoped>
.chat-container {
  height: 100vh;
  width: 100vw;
}

.sidebar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 5px 10px 5px;
}

.sidebar-title {
  font-size: 1.5rem;
  font-weight: 700;
  font-family: Arial, sans-serif;
  color: #ec4899;
  margin: 0;
}

.sidebar-content {
  height: calc(100% - 120px);
  overflow-y: auto;
}

.message-list {
  overflow-y: auto;
  padding: 20px;
}

.user-message {
  background-color: #f0f9ff;
  margin-left: auto;
}

.input-area {
  border-top: 1px solid #e5e7eb;
  padding: 20px;
}

.btn-menu {
  display: none;
  position: fixed;
  top: 1rem;
  left: 1rem;
  z-index: 20;
}

@media (max-width: 767px) {
  .el-aside {
    position: fixed;
    left: -300px;
    top: 0;
    bottom: 0;
    z-index: 10;
    transition: left 0.3s ease-in-out;
  }

  .el-aside.active {
    left: 0;
  }

  .btn-menu {
    display: block;
  }
}
</style>