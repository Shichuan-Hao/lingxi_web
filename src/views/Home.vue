<template>
  <div class="app-container" :class="userStore.theme">
    <!-- 侧边栏 -->
    <div class="sidebar" :class="{ 'sidebar-collapsed': isSidebarCollapsed }">
      <div class="sidebar-header">
        <div class="logo-wrapper">
          <span class="logo-text"><span class="logo-highlight">灵犀</span><span class="logo-pulse"></span></span>
          <button class="collapse-btn" @click="toggleSidebar">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M6 2L2 8L6 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
          </button>
        </div>
      </div>
      
      <!-- 添加历史聊天列表 -->
      <div class="chat-history">
        <button class="new-chat-btn" @click="startNewChat">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path d="M8 3.33334V12.6667" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            <path d="M12.6667 8L3.33333 8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          开启新对话
        </button>
        
        <div class="history-list">
          <div v-for="conversation in conversationStore.conversations" 
               :key="conversation.id"
               :class="['history-item', { active: conversation.id === conversationStore.currentConversationId }]">
            <div class="history-content" @click="handleConversationClick(conversation.id)">
              <span class="history-title">{{ conversation.title }}</span>
              <span class="history-time">{{ formatTime(new Date(conversation.created_at)) }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 底部个人信息 -->
      <div class="user-section">
        <!-- 添加电商客服按钮 -->
        <div class="ecommerce-link" @click="goToEcommerce">
          <div class="ecommerce-icon">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <rect width="24" height="24" rx="12" fill="#fff" opacity="0.15"/>
              <path d="M6 6H7.5L8 9M8 9L9.5 16H16L17.5 9H8Z" stroke="#fff" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
              <circle cx="10.5" cy="19" r="1" fill="#fff"/>
              <circle cx="15.5" cy="19" r="1" fill="#fff"/>
            </svg>
          </div>
          <span class="ecommerce-text">电商客服</span>
          <div class="ecommerce-badge">新</div>
        </div>
        
        <div class="user-menu" @click.stop="showUserMenu = !showUserMenu">
          <div class="user-avatar">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
              <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="1.5"/>
              <circle cx="12" cy="9" r="3" stroke="currentColor" stroke-width="1.5"/>
              <path d="M17.9691 20C17.81 17.1085 16.9247 15 12 15C7.07527 15 6.18997 17.1085 6.03087 20" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
          </div>
          <span class="user-text">{{ userStore.username }}</span>
        </div>
        
        <!-- 用户菜单 -->
        <div v-if="showUserMenu" class="user-dropdown">
          <div class="menu-item" @click="handleLogout">
            <svg width="16" height="16" viewBox="0 0 16 16">
              <path d="M6 14H3C2.44772 14 2 13.5523 2 13V3C2 2.44772 2.44772 2 3 2H6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
              <path d="M10 11L14 8L10 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
              <path d="M14 8H6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
            退出登录
          </div>
        </div>
      </div>
    </div>

    <!-- 添加展开按钮 -->
    <button v-if="isSidebarCollapsed" 
            class="expand-btn" 
            @click="toggleSidebar">
      <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M10 2L14 8L10 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
      </svg>
    </button>

    <!-- 主聊天区域 -->
    <div class="chat-container" id="chat-container">
      <div class="chat-content">
        <!-- 初始状态：欢迎消息和输入框在一起居中 -->
        <div v-if="!messages.length && !uploadedFile" class="initial-state">
          <div class="welcome-message">
            <h2>我是 <span>灵犀</span>, 很高兴见到你!</h2>
            <p>我可以帮你写代码、读文件、写作各种创意内容，请把你的任务交给我吧~</p>
          </div>
          <div class="chat-input">
            <div class="input-wrapper">
              <input
                v-model="userInput"
                @keyup.enter="sendMessage"
                type="text"
                placeholder="给 灵犀 发送消息"
              />
              <div class="button-group">
                <div class="left-buttons">
                  <button 
                    class="tool-btn"
                    :class="{ 'tool-btn-active': isDeepThinking }"
                    @click="toggleDeepThinking"
                  >
                    <div class="icon">🔄</div>
                    深度思考
                  </button>
                  <button 
                    class="tool-btn"
                    :class="{ 'tool-btn-active': isSearching }"
                    @click="toggleSearch"
                  >
                    <div class="icon">🌐</div>
                    {{ isSearching ? '取消搜索' : '联网搜索' }}
                  </button>
                  <button 
                    class="tool-btn"
                    :class="{ 'tool-btn-active': isRagMode }"
                    @click="isRagMode = !isRagMode"
                  >
                    <div class="icon">📚</div>
                    知识库问答
                  </button>
                </div>
                <div class="right-buttons">
                  <button class="tool-btn" @click="$refs.fileInput?.click()">
                    <div class="icon">📎</div>
                  </button>
                  <button 
                    class="send-btn" 
                    :class="{ 'send-btn-active': userInput.trim() }" 
                    :disabled="!userInput.trim()"
                    @click="sendMessage"
                  >
                    <div class="icon">
                      <svg width="14" height="16" viewBox="0 0 14 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <path fill-rule="evenodd" clip-rule="evenodd" d="M7 16c-.595 0-1.077-.462-1.077-1.032V1.032C5.923.462 6.405 0 7 0s1.077.462 1.077 1.032v13.936C8.077 15.538 7.595 16 7 16z" fill="currentColor"/>
                        <path fill-rule="evenodd" clip-rule="evenodd" d="M.315 7.44a1.002 1.002 0 0 1 0-1.46L6.238.302a1.11 1.11 0 0 1 1.523 0c.421.403.421 1.057 0 1.46L1.838 7.44a1.11 1.11 0 0 1-1.523 0z" fill="currentColor"/>
                        <path fill-rule="evenodd" clip-rule="evenodd" d="M13.685 7.44a1.11 1.11 0 0 1-1.523 0L6.238 1.762a1.002 1.002 0 0 1 0-1.46 1.11 1.11 0 0 1 1.523 0l5.924 5.678c.42.403.42 1.056 0 1.46z" fill="currentColor"/>
                      </svg>
                    </div>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <!-- 对话状态或有文件上传时：消息列表在上，输入框在底部 -->
        <div v-else class="chat-state">
          <!-- 侧边栏收起时的顶部 Logo -->
          <div v-if="isSidebarCollapsed" class="chat-header-collapsed">
            <span class="logo-text-mini"><span class="logo-highlight">灵犀</span></span>
          </div>
          <div class="chat-messages">
            <div v-for="(message, index) in messages" 
                 :key="index"
                 :class="['message', message.role === 'user' ? 'user-message' : 'assistant-message']">
              <!-- 助手头像（左侧） -->
              <div v-if="message.role === 'assistant'" class="message-avatar assistant-avatar">
                <svg width="24" height="24" viewBox="0 0 24 24">
                  <path d="M12 2L20 7V17L12 22L4 17V7L12 2Z" fill="#00d4ff" opacity="0.2"/>
                  <path d="M12 2L20 7V17L12 22L4 17V7L12 2Z" stroke="#00d4ff" stroke-width="1.5"/>
                  <circle cx="12" cy="12" r="3" fill="#00d4ff" opacity="0.2" stroke="#00d4ff" stroke-width="1.5"/>
                </svg>
              </div>
              <!-- 消息内容区域 -->
              <div class="message-body">
                <div class="message-content" v-html="renderMessage(message)" @click="handleMessageClick(message, $event)"></div>
                <!-- 用户消息操作按钮 -->
                <div v-if="message.role === 'user'" class="message-actions user-actions">
                  <button class="msg-action-btn" title="复制" @click="copyMessage(message.content)">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                      <rect x="9" y="9" width="13" height="13" rx="2" ry="2"/>
                      <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
                    </svg>
                  </button>
                  <button class="msg-action-btn" title="编辑" @click="editMessage(index)">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                      <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/>
                      <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/>
                    </svg>
                  </button>
                </div>
                <!-- 助手消息操作按钮 -->
                <div v-if="message.role === 'assistant' && !message.isLoading" class="message-actions">
                  <button class="msg-action-btn" title="复制" @click="copyMessage(message.content)">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                      <rect x="9" y="9" width="13" height="13" rx="2" ry="2"/>
                      <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
                    </svg>
                  </button>
                  <button class="msg-action-btn" title="重新生成" @click="regenerateMessage(index)">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                      <polyline points="1 4 1 10 7 10"/>
                      <path d="M3.51 15a9 9 0 1 0 2.13-9.36L1 10"/>
                    </svg>
                  </button>
                  <button class="msg-action-btn" title="点赞" @click="likeMessage(index)">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                      <path d="M14 9V5a3 3 0 0 0-3-3l-4 9v11h12.28a2 2 0 0 0 2-1.7l1.38-9a2 2 0 0 0-2-2.3zM7 22H4a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h3"/>
                    </svg>
                  </button>
                  <button class="msg-action-btn" title="点踩" @click="dislikeMessage(index)">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                      <path d="M10 15v4a3 3 0 0 0 3 3l4-9V2H5.28a2 2 0 0 0-2 2.3l1.38 9a2 2 0 0 0 2 1.7h9.72"/>
                      <path d="M17 2h3a2 2 0 0 1 2 2v7a2 2 0 0 1-2 2h-3"/>
                    </svg>
                  </button>
                  <button class="msg-action-btn" title="分享" @click="shareMessage(message.content)">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                      <circle cx="18" cy="5" r="3"/>
                      <circle cx="6" cy="12" r="3"/>
                      <circle cx="18" cy="19" r="3"/>
                      <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"/>
                      <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"/>
                    </svg>
                  </button>
                </div>
              </div>
              <!-- 用户头像（右侧） -->
              <div v-if="message.role === 'user'" class="message-avatar user-avatar">
                <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="12" r="10" stroke="#999" stroke-width="1.5"/>
                  <circle cx="12" cy="9" r="3" stroke="#999" stroke-width="1.5"/>
                  <path d="M17.9691 20C17.81 17.1085 16.9247 15 12 15C7.07527 15 6.18997 17.1085 6.03087 20" stroke="#999" stroke-width="1.5" stroke-linecap="round"/>
                </svg>
              </div>
            </div>
          </div>
          <div class="chat-input-container">
            <!-- 文件上传状态显示 -->
            <div v-if="uploadedFile" class="cefa5c26 d5e44c7a">
              <div class="a4380d7b">
                <div class="cd190a50 e5931f90">
                  <div class="d2d04dae">
                    <div class="ds-icon b3a5d6c1" style="font-size: 32px; width: 32px; height: 32px;">
                      <svg width="32" height="32" viewBox="0 0 32 32" fill="none" xmlns="http://www.w3.org/2000/svg"
                           :data-type="getFileType(uploadedFile.type)">
                        <path d="M7 9C7 6.79086 8.79086 5 11 5L18.6383 5C19.1906 5 19.6383 5.44772 19.6383 6V6.92308C19.6383 9.13222 21.4292 10.9231 23.6383 10.9231H24C24.5523 10.9231 25 11.3708 25 11.9231V23C25 25.2091 23.2091 27 21 27H11C8.79086 27 7 25.2091 7 23V9Z"/>
                      </svg>
                    </div>
                    <div class="aea7ca45">
                      <div class="f3a54b52">{{ uploadedFile.name }}</div>
                      <div class="ee357eab">
                        {{ uploadedFile.status === 'uploading' ? '上传中...' : 
                           uploadedFile.status === 'error' ? '上传失败' :
                           `${formatFileSize(uploadedFile.size)}` }}
                      </div>
                    </div>
                    <button class="delete-btn" @click="handleDeleteFile">
                      <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
                        <path d="M12 4L4 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                        <path d="M4 4L12 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                      </svg>
                    </button>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- 原有的聊天输入框 -->
            <div class="chat-input">
              <div class="input-wrapper">
                <input
                  v-model="userInput"
                  @keyup.enter="sendMessage"
                  type="text"
                  placeholder="给 灵犀 发送消息"
                />
                <div class="button-group">
                  <div class="left-buttons">
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isDeepThinking }"
                      @click="toggleDeepThinking"
                    > 
                      <div class="icon">🔄</div>
                      深度思考
                    </button>
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isSearching }"
                      @click="toggleSearch"
                    >
                      <div class="icon">🌐</div>
                      {{ isSearching ? '取消搜索' : '联网搜索' }}
                    </button>
                    <button 
                      class="tool-btn"
                      :class="{ 'tool-btn-active': isRagMode }"
                      @click="isRagMode = !isRagMode"
                    >
                      <div class="icon">📚</div>
                      知识库问答
                    </button>
                  </div>
                  <div class="right-buttons">
                    <button class="tool-btn" @click="$refs.fileInput?.click()">
                      <div class="icon">📎</div>
                    </button>
                    <button 
                      class="send-btn" 
                      :class="{ 'send-btn-active': userInput.trim() }" 
                      :disabled="!userInput.trim()"
                      @click="sendMessage"
                    >
                      <div class="icon">
                        <svg width="14" height="16" viewBox="0 0 14 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                          <path fill-rule="evenodd" clip-rule="evenodd" d="M7 16c-.595 0-1.077-.462-1.077-1.032V1.032C5.923.462 6.405 0 7 0s1.077.462 1.077 1.032v13.936C8.077 15.538 7.595 16 7 16z" fill="currentColor"/>
                          <path fill-rule="evenodd" clip-rule="evenodd" d="M.315 7.44a1.002 1.002 0 0 1 0-1.46L6.238.302a1.11 1.11 0 0 1 1.523 0c.421.403.421 1.057 0 1.46L1.838 7.44a1.11 1.11 0 0 1-1.523 0z" fill="currentColor"/>
                          <path fill-rule="evenodd" clip-rule="evenodd" d="M13.685 7.44a1.11 1.11 0 0 1-1.523 0L6.238 1.762a1.002 1.002 0 0 1 0-1.46 1.11 1.11 0 0 1 1.523 0l5.924 5.678c.42.403.42 1.056 0 1.46z" fill="currentColor"/>
                        </svg>
                      </div>
                    </button>
                  </div>
                </div>
              </div>
            </div>
            <div class="disclaimer-text">内容由 AI 生成，请仔细甄别</div>
          </div>
        </div>
      </div>
    </div>

    <!-- 添加搜索结果面板 -->
    <div v-if="showSearchPanel" class="search-panel">
      <div class="search-panel-header">
        <h3>搜索结果</h3>
        <button class="close-btn" @click="showSearchPanel = false">
          <span>×</span>
        </button>
      </div>
      <div class="search-results-list">
        <div v-for="(result, index) in searchResults" 
             :key="index"
             class="search-result-item"
             :class="{ active: result.isExpanded }">
          <div class="result-header" @click="toggleResultExpand(result)">
            <div class="result-source">
              <img :src="getSourceIcon(result.source)" class="source-icon" />
              <span class="source-name">{{ result.source }}</span>
              <span class="result-date">{{ result.date }}</span>
            </div>
            <div class="result-title">{{ result.title }}</div>
          </div>
          <div v-if="result.isExpanded" class="result-content">
            <div class="result-snippet">{{ result.snippet }}</div>
            <a :href="result.url" target="_blank" class="result-link" @click.stop>查看原文</a>
          </div>
        </div>
      </div>
    </div>

    <!-- 添加搜索结果详情面板 -->
    <div v-if="selectedResult" class="result-detail-panel">
      <div class="detail-header">
        <button class="back-btn" @click="selectedResult = null">
          <span>←</span>
        </button>
        <h3>{{ selectedResult.title }}</h3>
      </div>
      <div class="detail-content">
        <div class="detail-meta">
          <span class="detail-source">{{ selectedResult.source }}</span>
          <span class="detail-date">{{ selectedResult.date }}</span>
        </div>
        <div class="detail-text">{{ selectedResult.snippet }}</div>
        <a :href="selectedResult.url" target="_blank" class="detail-link">查看原文</a>
      </div>
    </div>

    <!-- 在 app-container div 内部的最后添加 -->
    <input 
      type="file" 
      ref="fileInput"
      @change="handleFileUpload"
      style="display: none"
      multiple
    />
    
    <!-- 重命名对话框 -->
    <div v-if="showRenameModal" class="modal-overlay">
      <div class="modal-content">
        <h3>重命名对话</h3>
        <input 
          v-model="newConversationName" 
          type="text" 
          class="rename-input" 
          placeholder="请输入新名称"
          @keyup.enter="confirmRename"
        />
        <div class="modal-actions">
          <button class="cancel-btn" @click="cancelRename">取消</button>
          <button class="confirm-btn" @click="confirmRename">确认</button>
        </div>
      </div>
    </div>
    
    <!-- 删除确认对话框 -->
    <div v-if="showDeleteConfirm" class="modal-overlay">
      <div class="modal-content">
        <h3>确认删除</h3>
        <p>确定要删除这个对话吗？此操作无法撤销。</p>
        <div class="modal-actions">
          <button class="cancel-btn" @click="cancelDelete">取消</button>
          <button class="delete-btn" @click="executeDelete">删除</button>
        </div>
      </div>
    </div>
    
    <!-- 添加悬浮聊天按钮 -->
    <div class="chat-float-button" @click="toggleChatPopup" v-if="!showChatPopup">
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M20 2H4C2.9 2 2 2.9 2 4V22L6 18H20C21.1 18 22 17.1 22 16V4C22 2.9 21.1 2 20 2Z" fill="#00d4ff"/>
        <path d="M7 9H17M7 13H14" stroke="white" stroke-width="1.5" stroke-linecap="round"/>
      </svg>
    </div>
    
    <!-- 聊天弹窗 -->
    <div class="chat-popup" v-if="showChatPopup">
      <div class="chat-popup-header">
        <h3>客服助手</h3>
        <button class="chat-popup-close" @click="toggleChatPopup">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path d="M12 4L4 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            <path d="M4 4L12 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
        </button>
      </div>
      <div class="chat-popup-messages">
        <div v-for="(message, index) in popupMessages" 
             :key="index"
             :class="['popup-message', message.role === 'user' ? 'popup-user-message' : 'popup-assistant-message']">
          <div class="popup-message-content">
            <template v-if="message.isLoading">
              <div class="loading-dots">
                <span></span>
                <span></span>
                <span></span>
              </div>
            </template>
            <template v-else>
              <div v-html="md.render(message.content)"></div>
            </template>
          </div>
        </div>
      </div>
      <div class="chat-popup-input">
        <input 
          v-model="popupInput" 
          @keyup.enter="sendPopupMessage"
          type="text" 
          placeholder="请输入您的问题..." 
        />
        <button class="popup-send-btn" @click="sendPopupMessage">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path d="M15 1L7.5 8.5M15 1L10 15L7.5 8.5M15 1L1 6L7.5 8.5" stroke="currentColor" stroke-width="1.5"/>
          </svg>
        </button>
      </div>
    </div>
  </div>

  <!-- 复制成功提示 -->
  <transition name="toast">
    <div v-if="showCopyToast" class="copy-toast">
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <polyline points="20 6 9 17 4 12"/>
      </svg>
      <span>复制</span>
    </div>
  </transition>
</template>

<script setup lang="ts">
import { ref, nextTick, onMounted, onUnmounted, watch } from 'vue'
import { marked, Renderer } from 'marked'
import DOMPurify from 'dompurify'
import MarkdownIt from 'markdown-it'
import { ApiService, type Conversation } from '../services/api'
import { AuthService } from '../services/api'
import { useUserStore } from '../stores/user'
import { useConversationStore } from '../stores/conversation'
import type { ChatMessage } from '../types'
import { useRouter } from 'vue-router'

interface StreamResponse {
  content: string
  done: boolean
}

interface ExecutionResponse {
  type: 'conclusion' | 'process' | 'files' | 'cost' | 'error' | 'done'
  content: any
}

interface ExecutionRequest {
  prompt: string
  taskId?: string
}

interface ChatRequest {
  messages: {
    role: string
    content: string
  }[]
}

interface MessageUI {
  role: string
  content: string
  thinking?: string
  conclusion?: string
  process?: string
  files?: {
    files: string[]
    urls: string[]
  }
  cost?: {
    total: number
    details: Record<string, number>
  }
  hasDetails?: boolean
  showDetails?: boolean
  isSearching?: string
}

interface ChatHistory {
  id: number
  title: string
  time: Date
  messages: MessageUI[]
}

interface SearchResult {
  title: string
  url: string
  snippet: string
  date: string
  source: string
  isExpanded?: boolean
}

interface SearchResponse {
  type: 'search_results'
  total: number
  query: string
  results: SearchResult[]
  error?: string
}

// 定义消息类型
interface PopupMessage {
  role: string;
  content: string;
  isLoading?: boolean;
}

const userInput = ref('')
const messages = ref<MessageUI[]>([])
const isSidebarCollapsed = ref(false)
const chatHistory = ref<ChatHistory[]>([])
const currentChatIndex = ref(0)
const currentAgent = ref('openai')
const currentResponse = ref('')
const ws = ref<WebSocket | null>(null)
const isDeepThinking = ref(false)
const thinkStartTime = ref<number | null>(null)
const isSearching = ref(false)
const searchResults = ref<SearchResult[]>([])
const showSearchPanel = ref(false)
const showCopyToast = ref(false)
const selectedResult = ref<SearchResult | null>(null)
const fileInput = ref<HTMLInputElement | null>(null)
const uploadedFileInfo = ref<{
  name: string;
  type: string;
  size: number;
} | null>(null)
const uploadStatus = ref<{
  isUploading: boolean;
  fileName: string;
} | null>(null)
const uploadedFile = ref<{
  name: string;
  size: number;
  type: string;
  status: 'uploading' | 'success' | 'error';
} | null>(null)
const isRagMode = ref(false)
const currentIndexId = ref<string | null>(null)
const currentConversationId = ref<number | null>(null)

const md = new MarkdownIt({
  breaks: true,
  linkify: true,
  typographer: true
})

const scrollToBottom = async () => {
  await nextTick()
  const container = document.querySelector('.chat-container')
  if (container) {
    // 使用平滑滚动
    container.scrollTo({
      top: container.scrollHeight,
      behavior: 'smooth'
    })
  }
}

const userStore = useUserStore()
const conversationStore = useConversationStore()

// 对话重命名相关变量
const showRenameModal = ref(false)
const conversationToRename = ref<Conversation | null>(null)
const newConversationName = ref('')

// 对话删除相关变量
const showDeleteConfirm = ref(false)
const conversationIdToDelete = ref<number | null>(null)

// 添加聊天弹窗相关变量
const showChatPopup = ref(false)
const popupMessages = ref<PopupMessage[]>([
  { role: 'assistant', content: '您好！我是智能客服助手，有什么可以帮您的吗？' }
])
const popupInput = ref('')

const router = useRouter()

// 前往电商客服页面
const goToEcommerce = () => {
  router.push('/ecommerce')
}

onMounted(async () => {
  try {
    const userInfo = await AuthService.getUserInfo()
    userStore.setUserInfo(userInfo)
    // 加载用户的会话列表
    await conversationStore.loadUserConversations()
    // 创建新会话
    if (conversationStore.isNewConversation) {
      await conversationStore.createNewConversation()
    }
  } catch (error) {
    console.error('Failed to fetch user info:', error)
  }
})

// 修改开启新会话的方法
const startNewChat = async () => {
  try {
    await conversationStore.createNewConversation()
    // 创建新会话后刷新会话列表
    await conversationStore.loadUserConversations()
    messages.value = []
    currentChatIndex.value = 0
    scrollToBottom()
  } catch (error) {
    console.error('Failed to start new chat:', error)
  }
}

const loadChat = (index: number) => {
  currentChatIndex.value = index
  messages.value = [...chatHistory.value[index].messages]
  scrollToBottom()
}

const formatTime = (date: Date) => {
  return new Intl.DateTimeFormat('zh-CN', {
    month: 'numeric',
    day: 'numeric',
    hour: 'numeric',
    minute: 'numeric'
  }).format(date)
}

const renderMessage = (message: MessageUI) => {
  // 清理内容中的转义字符和多余的引号
  const cleanContent = (text: string) => {
    if (!text) return '';
    
    // 移除可能存在的data:前缀
    text = text.replace(/^data:\s*/g, '');
    
    // 移除多余的引号
    // 如果文本以引号开始和结束，移除它们
    text = text.replace(/^["']|["']$/g, '');
    
    // 移除每行开头的引号
    text = text.split('\n').map(line => line.replace(/^["']\s*/, '')).join('\n');
    
    // 移除每行结尾的引号
    text = text.split('\n').map(line => line.replace(/\s*["']$/, '')).join('\n');
    
    // 移除文本中连续的两个引号
    text = text.replace(/""/g, '');
    
    // 替换转义的换行符
    text = text.replace(/\\n/g, '\n');
    
    // 替换转义的引号
    text = text.replace(/\\"/g, '"');
    
    return text;
  };

  // 如果是搜索状态的消息，直接返回原始 HTML
  if (message.isSearching) {
    return message.content;
  }

  // 如果包含思考过程（使用Markdown格式）
  if (message.content.includes('### 思考过程')) {
    // 分割内容为思考过程和回复
    const parts = message.content.split('---');
    
    // 清理并渲染内容
    const cleanParts = parts.map(part => cleanContent(part));
    
    // 渲染为Markdown
    if (parts.length > 1) {
      // 有思考过程和回复
      return `
        <div class="message-wrapper">
          <details class="thinking-details">
            <summary class="thinking-summary">
              <span class="thinking-icon">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M12 2L20 7V17L12 22L4 17V7L12 2Z"/>
                </svg>
              </span>
              <span class="thinking-title">已思考</span>
              <span class="thinking-arrow">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="6 9 12 15 18 9"/>
                </svg>
              </span>
            </summary>
            <div class="thinking-content">
              ${md.render(cleanParts[0])}
            </div>
          </details>
          <div class="message-text">
            ${md.render(cleanParts[1])}
          </div>
        </div>
      `;
    } else {
      // 只有思考过程
      return `
        <div class="message-wrapper">
          <details class="thinking-details">
            <summary class="thinking-summary">
              <span class="thinking-icon">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M12 2L20 7V17L12 22L4 17V7L12 2Z"/>
                </svg>
              </span>
              <span class="thinking-title">已思考</span>
              <span class="thinking-arrow">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="6 9 12 15 18 9"/>
                </svg>
              </span>
            </summary>
            <div class="thinking-content">
              ${md.render(cleanParts[0])}
            </div>
          </details>
        </div>
      `;
    }
  }

  // 如果包含搜索加载状态
  if (message.content.includes('search-loading-container')) {
    const modelResponseDiv = message.content.indexOf('<div class="model-response">');
    if (modelResponseDiv !== -1) {
      const searchStatus = message.content.slice(0, modelResponseDiv);
      const response = message.content.slice(modelResponseDiv);
      return searchStatus + md.render(cleanContent(response));
    }
  }
  
  // 处理普通消息内容
  return md.render(cleanContent(message.content));
};

// 修改处理流式响应的函数，在每次内容更新时都滚动
const handleStreamResponse = async (reader: ReadableStreamDefaultReader<Uint8Array>, 
                                  onData: (content: string) => void) => {
  const decoder = new TextDecoder()
  
  while (true) {
    const { done, value } = await reader.read()
    if (done) break
    
    const text = decoder.decode(value)
    const lines = text.split('\n')
    
    for (const line of lines) {
      if (!line.startsWith('data: ')) continue
      
      const content = line.slice(6)
      if (content === '[DONE]') continue

      onData(content)
      // 每次更新内容后滚动
      scrollToBottom()
    }
  }
}

// 处理搜索请求
const handleSearch = async (reader: ReadableStreamDefaultReader<Uint8Array>) => {
  if (!reader) throw new Error('No reader available')

  let currentContent = ''
  const decoder = new TextDecoder()

  try {
    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      const chunk = decoder.decode(value)
      const lines = chunk.split('\n')

      for (const line of lines) {
        if (!line.startsWith('data: ')) continue
        
        const content = line.slice(6).trim()
        if (!content || content === '[DONE]') continue

        try {
          const data = JSON.parse(content)
          
          switch (data.type) {
            case 'search_start':
              // 显示"正在联网检索"
              messages.value.push({
                role: 'assistant' as const,
                content: `<div class="search-loading-container">
                  <div class="search-loading-text">🔍 正在联网检索...</div>
                </div>
                <div class="model-response"></div>`,
                isSearching: 'true'
              })
              break

            case 'search_results':
              // 处理搜索结果
              searchResults.value = data.results.map((result: any) => ({
                ...result,
                date: new Date().toLocaleDateString('zh-CN'),
                source: new URL(result.url).hostname,
                isExpanded: false
              }))
              
              // 更新搜索状态消息
              const lastMessage = messages.value[messages.value.length - 1]
              if (lastMessage && lastMessage.isSearching === 'true') {
                const modelResponseDiv = lastMessage.content.indexOf('<div class="model-response">')
                if (modelResponseDiv !== -1) {
                  const beforeResponse = lastMessage.content.slice(0, modelResponseDiv)
                  lastMessage.content = beforeResponse.replace(
                    '正在联网检索...',
                    '联网检索完成，点击查看结果'
                  ) + lastMessage.content.slice(modelResponseDiv)
                }
              }
              break

            case 'direct_answer':
              // 隐藏搜索提示，显示直接回答
              messages.value.push({
                role: 'assistant' as const,
                content: ''
              })
              break

            case 'direct_content':
              // 更新最后一条消息的内容
              const message = messages.value[messages.value.length - 1]
              if (message && message.role === 'assistant') {
                message.content += data.content
              }
              break

            default:
              // 处理普通文本内容
              const cleanContent = content.replace(/^"|"$/g, '').replace(/\\n/g, '\n')
              currentContent += cleanContent

              // 更新消息内容
              const currentMessage = messages.value[messages.value.length - 1]
              if (currentMessage && currentMessage.isSearching === 'true') {
                const modelResponseDiv = currentMessage.content.indexOf('<div class="model-response">')
                if (modelResponseDiv !== -1) {
                  const beforeResponse = currentMessage.content.slice(0, modelResponseDiv + '<div class="model-response">'.length)
                  currentMessage.content = beforeResponse + md.render(currentContent) + '</div>'
                }
              }
          }

          scrollToBottom()
        } catch (e) {
          console.error('Error parsing message:', e)
        }
      }
    }
  } catch (error) {
    console.error('Error:', error)
    messages.value[messages.value.length - 1].content = '抱歉，搜索时发生了错误，请稍后重试。'
  }
}

// 修改 sendMessage 中的搜索部分
const sendMessage = async () => {
  if (!userInput.value.trim()) return

  try {
    const userMessage = {
      role: 'user' as const,
      content: userInput.value
    }
    messages.value.push(userMessage)
    userInput.value = ''

    if (isSearching.value) {
      const reader = await ApiService.search([userMessage], conversationStore.currentConversationId!)
  if (!reader) throw new Error('No reader available')
      await handleSearch(reader)
    } else if (isDeepThinking.value) {
      const reader = await ApiService.reason([userMessage], conversationStore.currentConversationId!)
      if (!reader) throw new Error('No reader available')
      await handleChatStream(reader)
    } else {
      const reader = await ApiService.chat([userMessage], conversationStore.currentConversationId!)
      if (!reader) throw new Error('No reader available')
      await handleChatStream(reader)
    }

    await scrollToBottom()
    // 发送消息后刷新会话列表
    await conversationStore.loadUserConversations()
  } catch (error) {
    console.error('Error:', error)
    messages.value.push({
      role: 'assistant' as const,
      content: '抱歉，发生了错误，请稍后重试。'
    })
  }
}

const selectAgent = (agent: string) => {
  if (agent === 'TwoAgentChat') {
    messages.value = []
    messages.value.push({
      role: 'assistant',
      content: '已切换到 TwoAgentChat 模式。请描述您的任务，我会协助您完成。'
    })
  }
  currentAgent.value = agent
}

const toggleSidebar = () => {
  isSidebarCollapsed.value = !isSidebarCollapsed.value;
}

const toggleDeepThinking = () => {
  isDeepThinking.value = !isDeepThinking.value;
  if (isDeepThinking.value) {
    // 如果开启深度思考，自动关闭搜索
    isSearching.value = false;
  }
}

const toggleSearch = () => {
  isSearching.value = !isSearching.value
  if (!isSearching.value) {
    // 如果取消搜索，重置状态
    isDeepThinking.value = false
  }
}

// 监听消息变化
watch(messages, () => {
  scrollToBottom()
}, { deep: true })

// 添加 ResizeObserver 来监听内容高度变化
onMounted(() => {
  const container = document.querySelector('.chat-container')
  if (container) {
    const resizeObserver = new ResizeObserver(() => {
      scrollToBottom()
    })
    
    resizeObserver.observe(container)
    
    // 组件卸载时清理
    onUnmounted(() => {
      resizeObserver.disconnect()
    })
  }
})

// 添加获取图标的方法
const getSourceIcon = (source: string) => {
  // 这里可以根据不同来源返回不同的图标URL
  return `https://www.google.com/s2/favicons?domain=${source}`
}

// 修改处理搜索结果的逻辑
const handleSearchResults = (results: SearchResult[]) => {
  searchResults.value = results.map(result => ({
    ...result,
    date: new Date().toLocaleDateString('zh-CN'),
    source: new URL(result.url).hostname,
    isExpanded: false
  }));
  
  // 更新搜索状态消息
  const lastMessage = messages.value[messages.value.length - 1];
  if (lastMessage && lastMessage.isSearching === 'true') {
    const modelResponseDiv = lastMessage.content.indexOf('<div class="model-response">');
    if (modelResponseDiv !== -1) {
      const beforeResponse = lastMessage.content.slice(0, modelResponseDiv);
      lastMessage.content = beforeResponse.replace(
        '正在联网检索...',
        '联网检索完成，点击查看结果'
      ) + lastMessage.content.slice(modelResponseDiv);
    }
    lastMessage.isSearching = 'false';
  }
  
  // 不自动显示搜索面板
  showSearchPanel.value = false;
};

// 添加切换展开状态的方法
const toggleResultExpand = (result: SearchResult) => {
  result.isExpanded = !result.isExpanded
}

// 修改消息点击事件处理
const handleMessageClick = (message: MessageUI, event: MouseEvent) => {
  // 检查点击的元素是否包含搜索完成的文本
  const clickedElement = event.target as HTMLElement;
  const searchContainer = clickedElement.closest('.search-loading-container');
  
  if (searchContainer && searchResults.value.length > 0) {
    showSearchPanel.value = true;
  }
};

// 复制消息内容
const copyMessage = async (content: string) => {
  try {
    // 移除 HTML 标签，只复制纯文本
    const tempDiv = document.createElement('div');
    tempDiv.innerHTML = content;
    const text = tempDiv.textContent || tempDiv.innerText || '';
    await navigator.clipboard.writeText(text);
    showCopyToast.value = true;
    setTimeout(() => { showCopyToast.value = false; }, 1500);
  } catch (err) {
    console.error('复制失败:', err);
  }
};

// 重新生成消息
const regenerateMessage = async (index: number) => {
  // 找到对应用户消息并重新发送
  let userIndex = index - 1;
  while (userIndex >= 0 && messages.value[userIndex].role !== 'user') {
    userIndex--;
  }
  if (userIndex >= 0) {
    // 移除当前助手消息及之后的消息
    messages.value.splice(index);
    // 重新发送用户消息
    const userMessage = messages.value[userIndex];
    await sendMessage(userMessage.content);
  }
};

// 点赞
const likeMessage = (index: number) => {
  console.log('点赞消息', index);
};

// 点踩
const dislikeMessage = (index: number) => {
  console.log('点踩消息', index);
};

// 分享消息
const shareMessage = (content: string) => {
  console.log('分享消息', content);
};

// 编辑消息
const editMessage = (index: number) => {
  const message = messages.value[index];
  if (message && message.role === 'user') {
    userInput.value = message.content;
    messages.value.splice(index);
    nextTick(() => {
      const textarea = document.querySelector('.chat-input textarea') as HTMLTextAreaElement;
      if (textarea) textarea.focus();
    });
  }
};

const handleFileUpload = async (event: Event) => {
  const input = event.target as HTMLInputElement
  if (!input.files?.length) return
  
  const userId = localStorage.getItem('user_id')
  if (!userId) {
    console.error('No user ID found')
    return
  }

  // 处理多个文件
  for (const file of input.files) {
    const formData = new FormData()
    formData.append('file', file)
    formData.append('user_id', userId)
    
    // 设置上传中状态
    uploadedFile.value = {
      name: file.name,
      size: file.size,
      type: file.type,
      status: 'uploading'
    }
    
    try {
      const response = await fetch('http://localhost:8000/api/upload', {
        method: 'POST',
        body: formData
      })
      
      if (!response.ok) {
        throw new Error('Upload failed')
      }
      
      const result = await response.json()
      
      if (result.status === 'success') {
        // 保存 index_id
        currentIndexId.value = result.index_id
        
        // 更新上传成功状态
        uploadedFile.value = {
          name: result.original_name,
          size: result.size,
          type: result.type,
          status: 'success'
        }
      }
      
    } catch (err: any) {
      console.error('Upload failed:', err)
      if (uploadedFile.value) {
        uploadedFile.value.status = 'error'
      }
      currentIndexId.value = null
    }
  }
  
  input.value = ''
}

// 格式化文件大小的辅助函数
const formatFileSize = (bytes: number) => {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB'
}

// 添加删除文件的方法
const handleDeleteFile = () => {
  uploadedFile.value = null
  currentIndexId.value = null
}

// 添加获取文件类型的方法
const getFileType = (mimeType: string) => {
  if (mimeType.includes('pdf')) return 'pdf'
  if (mimeType.includes('word')) return 'doc'
  if (mimeType.includes('excel') || mimeType.includes('spreadsheet')) return 'xls'
  if (mimeType.includes('powerpoint') || mimeType.includes('presentation')) return 'ppt'
  if (mimeType.includes('text')) return 'txt'
  if (mimeType.includes('image')) return 'image'
  return 'default'
}

const handleLogout = () => {
  userStore.reset()
  conversationStore.reset()
  AuthService.logout()
}

const showUserMenu = ref(false)

// 关闭菜单的点击外部处理
const handleClickOutside = (event: MouseEvent) => {
  const target = event.target as HTMLElement
  if (!target.closest('.user-menu')) {
    showUserMenu.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
})

const handleSubmit = async () => {
  if (!userInput.value.trim()) return

  try {
    // 如果没有当前会话ID，先创建新会话
    if (!currentConversationId.value) {
      currentConversationId.value = await ApiService.createConversation()
    }

    // 添加用户消息
    const userMessage = {
      role: 'user' as const,
      content: userInput.value
    }
    messages.value.push(userMessage)
    userInput.value = ''

    // 如果开启了搜索模式
    if (isSearching.value) {
      const reader = await ApiService.search([userMessage], currentConversationId.value)
      if (!reader) throw new Error('No reader available')
      await handleSearch(reader)
    } else if (isDeepThinking.value) {
      const reader = await ApiService.reason([userMessage], currentConversationId.value)
      if (!reader) throw new Error('No reader available')
      await handleChatStream(reader)
    } else {
      const reader = await ApiService.chat([userMessage], currentConversationId.value)
      if (!reader) throw new Error('No reader available')
      await handleChatStream(reader)
    }

    // 发送消息后刷新会话列表
    await conversationStore.loadUserConversations()
  } catch (error) {
    console.error('Error:', error)
    messages.value.push({
      role: 'assistant' as const,
      content: '抱歉，发生了错误，请稍后重试。'
    })
  }
}

// 处理新会话按钮点击
const handleNewConversation = async () => {
  try {
    // 创建新会话
    currentConversationId.value = await ApiService.createConversation()
    // 清空消息
    messages.value = []
    userInput.value = ''
  } catch (error) {
    console.error('Error creating new conversation:', error)
  }
}

const handleChatStream = async (reader: ReadableStreamDefaultReader<Uint8Array>) => {
  try {
    // 添加助手消息
    messages.value.push({
      role: 'assistant',
      content: '',
      thinking: '' // 添加thinking字段用于存储思考过程
    })
    
    let hasResponse = false;
    
    // 使用ApiService处理流
    await ApiService.handleChatStream(reader, (chunk) => {
      // 清理内容的函数
      const cleanChunkContent = (content: string) => {
        if (!content) return '';
        
        // 移除可能的data:前缀
        content = content.replace(/^data:\s*/g, '').trim();
        
        // 移除多余的引号
        // 如果文本以引号开始和结束，移除它们
        content = content.replace(/^["']|["']$/g, '');
        
        // 移除每行开头的引号
        content = content.split('\n').map(line => line.replace(/^["']\s*/, '')).join('\n');
        
        // 移除每行结尾的引号
        content = content.split('\n').map(line => line.replace(/\s*["']$/, '')).join('\n');
        
        // 移除文本中连续的两个引号
        content = content.replace(/""/g, '');
        
        // 替换转义的换行符
        content = content.replace(/\\n/g, '\n');
        
        // 替换转义的引号
        content = content.replace(/\\"/g, '"');
        
        return content;
      };
      
      const lastMessage = messages.value[messages.value.length - 1]
      
      if (chunk.type === 'think') {
        // 存储思考过程，但不修改content字段
        lastMessage.thinking = cleanChunkContent(chunk.content);
        
        // 只有在深度思考模式下才显示思考过程
        if (isDeepThinking.value) {
          // 如果已经有普通响应，则将思考过程放在前面
          if (hasResponse) {
            lastMessage.content = `### 思考过程\n\n${lastMessage.thinking}\n\n---\n\n${lastMessage.content}`
          } else {
            // 否则只显示思考过程
            lastMessage.content = `### 思考过程\n\n${lastMessage.thinking}`
          }
        }
      } else if (chunk.type === 'response') {
        // 清理响应内容
        const cleanResponse = cleanChunkContent(chunk.content);
        hasResponse = true;
        
        // 如果是普通响应，检查是否在深度思考模式下且有思考过程
        if (isDeepThinking.value && lastMessage.thinking) {
          // 深度思考模式：在回复前添加思考过程
          lastMessage.content = `### 思考过程\n\n${lastMessage.thinking}\n\n---\n\n${cleanResponse}`
        } else {
          // 普通模式：只显示回复内容
          lastMessage.content = cleanResponse;
        }
      }
    })
    
    await scrollToBottom()
  } catch (error) {
    console.error('Error handling chat stream:', error)
    const lastMessage = messages.value[messages.value.length - 1]
    lastMessage.content = '抱歉，发生了错误，请稍后重试。'
  }
}

// 处理会话点击
const handleConversationClick = async (conversationId: number) => {
  try {
    await conversationStore.loadConversationMessages(conversationId)
    // 转换消息格式以匹配现有的渲染逻辑
    messages.value = conversationStore.currentMessages.map(msg => ({
      role: msg.sender === 'user' ? 'user' : 'assistant',
      content: msg.content
    } as MessageUI))
  } catch (error) {
    console.error('Failed to load conversation:', error)
  }
}

// 显示重命名对话框
const showRenameDialog = (conversation: Conversation) => {
  conversationToRename.value = conversation
  newConversationName.value = conversation.title
  showRenameModal.value = true
}

// 确认重命名对话
const confirmRename = async () => {
  if (!conversationToRename.value || !newConversationName.value.trim()) return
  
  try {
    await conversationStore.updateConversationName(
      conversationToRename.value.id,
      newConversationName.value.trim()
    )
    showRenameModal.value = false
    conversationToRename.value = null
    newConversationName.value = ''
  } catch (error) {
    console.error('Failed to rename conversation:', error)
  }
}

// 取消重命名对话
const cancelRename = () => {
  showRenameModal.value = false
  conversationToRename.value = null
  newConversationName.value = ''
}

// 显示删除确认对话框
const confirmDelete = (conversationId: number) => {
  conversationIdToDelete.value = conversationId
  showDeleteConfirm.value = true
}

// 确认删除对话
const executeDelete = async () => {
  if (!conversationIdToDelete.value) return
  
  try {
    await conversationStore.deleteConversation(conversationIdToDelete.value)
    showDeleteConfirm.value = false
    conversationIdToDelete.value = null
    
    // 如果删除的是当前会话，创建新会话
    if (conversationStore.isNewConversation) {
      await conversationStore.createNewConversation()
      messages.value = []
    }
  } catch (error) {
    console.error('Failed to delete conversation:', error)
  }
}

// 取消删除对话
const cancelDelete = () => {
  showDeleteConfirm.value = false
  conversationIdToDelete.value = null
}

// 切换聊天弹窗显示
const toggleChatPopup = () => {
  showChatPopup.value = !showChatPopup.value
}

// 发送弹窗消息
const sendPopupMessage = async () => {
  if (!popupInput.value.trim()) return
  
  // 添加用户消息
  popupMessages.value.push({
    role: 'user',
    content: popupInput.value
  })
  
  const userQuestion = popupInput.value
  popupInput.value = ''
  
  try {
    // 显示加载状态
    popupMessages.value.push({
      role: 'assistant',
      content: '正在思考...',
      isLoading: true
    })
    
    // 获取用户ID
    const userId = localStorage.getItem('user_id')
    if (!userId) {
      throw new Error('用户ID不存在')
    }
    
    // 调用LangGraph接口
    const formData = new FormData()
    formData.append('query', userQuestion)
    formData.append('user_id', parseInt(userId))
    if (conversationStore.currentConversationId) {
      formData.append('conversation_id', conversationStore.currentConversationId)
    }
    
    const response = await fetch('/api/langgraph/query', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: formData
    })
    
    if (!response.ok) {
      throw new Error('请求失败: ' + response.statusText)
    }
    
    // 移除加载消息
    popupMessages.value.pop()
    
    // 检查响应类型
    const contentType = response.headers.get('Content-Type')
    
    // 如果是流式响应
    if (contentType && contentType.includes('text/event-stream')) {
      // 添加一个空的助手消息
      popupMessages.value.push({
        role: 'assistant',
        content: ''
      })
      
      // 获取reader
      const reader = response.body?.getReader()
      if (!reader) throw new Error('无法读取响应流')
      
      // 处理流
      await handlePopupChatStream(reader)
    } else {
      // 非流式响应，处理JSON
      const result = await response.json()
      popupMessages.value.push({
        role: 'assistant',
        content: result.response || '抱歉，我无法处理您的请求。'
      })
    }
  } catch (error) {
    console.error('发送消息失败:', error)
    
    // 移除加载消息
    const loadingMsgIndex = popupMessages.value.findIndex(msg => msg.isLoading)
    if (loadingMsgIndex !== -1) {
      popupMessages.value.splice(loadingMsgIndex, 1)
    }
    
    // 显示错误消息
    popupMessages.value.push({
      role: 'assistant',
      content: '抱歉，发生了错误，请稍后再试。'
    })
  }
}

// 处理弹窗的聊天流
const handlePopupChatStream = async (reader: ReadableStreamDefaultReader<Uint8Array>) => {
  const decoder = new TextDecoder()
  let currentContent = ''
  
  try {
    while (true) {
      const { done, value } = await reader.read()
      if (done) break
  
      const chunk = decoder.decode(value)
      const lines = chunk.split('\n')
      
      for (const line of lines) {
        if (!line.startsWith('data: ')) continue
        
        const content = line.slice(6) // 移除 'data: ' 前缀
        if (content === '[DONE]') continue
        
        // 处理所有可能的换行符形式
        if (content === '"\\n\\n"' || content === '"\n\n"' || content === '\n\n') {
          currentContent += '\n\n'
          continue
        }
        
        // 移除引号
        let cleanedContent = content
        if (cleanedContent.startsWith('"') && cleanedContent.endsWith('"')) {
          cleanedContent = cleanedContent.slice(1, -1)
        }
        
        // 处理转义的换行符
        cleanedContent = cleanedContent.replace(/\\n\\n/g, '\n\n').replace(/\\n/g, '\n')
  
        // 添加到当前内容
        currentContent += cleanedContent
        
        // 更新最后一条消息的内容
        const lastMessage = popupMessages.value[popupMessages.value.length - 1]
        lastMessage.content = currentContent
      }
    }
  } catch (error) {
    console.error('Error handling chat stream:', error)
    const lastMessage = popupMessages.value[popupMessages.value.length - 1]
    lastMessage.content = '抱歉，发生了错误，请稍后重试。'
  }
}

// 格式化弹窗消息 - 使用pre-wrap样式确保换行显示
const formatPopupMessage = (content: string) => {
  if (!content) return ''
  return content
}
</script>

<style>
/* 添加全局样式 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

/* 复制成功 Toast */
.copy-toast {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(0, 0, 0, 0.75);
  color: #fff;
  padding: 10px 20px;
  border-radius: 24px;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 6px;
  z-index: 9999;
  pointer-events: none;
}

.copy-toast svg {
  color: #4ade80;
  flex-shrink: 0;
}

/* Toast 过渡动画 */
.toast-enter-active,
.toast-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translate(-50%, -50%) scale(0.9);
}

.toast-enter-to,
.toast-leave-from {
  opacity: 1;
  transform: translate(-50%, -50%) scale(1);
}

/* 侧边栏收起时的顶部 Logo */
.chat-header-collapsed {
  position: sticky;
  top: 0;
  text-align: center;
  padding: 12px 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
  background: #fff;
  z-index: 5;
}

.chat-header-collapsed .logo-text-mini {
  font-size: 18px;
  font-weight: 600;
  letter-spacing: 2px;
}
</style>

<style scoped>
/* 根容器 */
.app-container {
  display: flex;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: #fff;
}

/* 浅色主题全局覆盖 */
.app-container .sidebar,
.app-container .chat-container,
.app-container .input-wrapper,
.app-container .message,
.app-container .welcome-message h2,
.app-container .message-content,
.app-container input,
.app-container .history-title,
.app-container .user-menu,
.app-container .menu-item,
.app-container .user-text,
.app-container .agent-name,
.app-container .logo-text,
.app-container .empty-state-content h3,
.app-container .detail-header h3,
.app-container .detail-source,
.app-container .detail-text,
.app-container .result-title,
.app-container .source-name,
.app-container .chat-popup-header h3,
.app-container .popup-assistant-message,
.app-container .file-name,
.app-container .upload-header,
.app-container .response-content,
.app-container ::deep(.message-content),
.app-container ::deep(.message-content) h1,
.app-container ::deep(.message-content) h2,
.app-container ::deep(.message-content) h3,
.app-container ::deep(.message-content) h4,
.app-container ::deep(.message-content) h5,
.app-container ::deep(.message-content) h6,
.app-container ::deep(.message-content) p,
.app-container ::deep(.message-content) ul,
.app-container ::deep(.message-content) ol,
.app-container ::deep(.message-content) li,
.app-container ::deep(.thinking-content) p,
.app-container ::deep(.thinking-content) code,
.app-container ::deep(.file-info-content) p {
  color: #333;
}

.app-container .chat-container {
  background: transparent;
}

.app-container .sidebar {
  background: #fff;
  border-right: 1px solid rgba(0, 0, 0, 0.06);
}

.app-container .sidebar-header {
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
}

.app-container .logo-wrapper {
  background: rgba(255, 255, 255, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}

.app-container .logo-text {
  color: #333;
}

.app-container .history-item {
  background: rgba(0, 0, 0, 0.02);
}

.app-container .history-item:hover {
  background: rgba(0, 212, 255, 0.06);
  border-color: rgba(0, 212, 255, 0.2);
}

.app-container .history-item.active {
  background: rgba(0, 212, 255, 0.1);
  border-color: rgba(0, 212, 255, 0.3);
}

.app-container .history-title {
  color: #333;
}

.app-container .history-time {
  color: #999;
}

.app-container .user-section {
  border-top: 1px solid rgba(0, 0, 0, 0.06);
}

.app-container .user-menu {
  background: rgba(255, 255, 255, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.6);
  color: #333;
}

.app-container .user-menu:hover {
  background: rgba(255, 255, 255, 0.8);
  border-color: rgba(0, 212, 255, 0.3);
}

.app-container .user-dropdown {
  background: rgba(255, 255, 255, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.7);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1), 0 0 20px rgba(0, 212, 255, 0.08);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
}

.app-container .menu-item {
  color: #333;
}

.app-container .menu-item:hover {
  background: rgba(0, 212, 255, 0.06);
}

.app-container .input-wrapper {
  background: #fff;
  border: 1px solid rgba(0, 0, 0, 0.1);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.06);
}

.app-container .input-wrapper:focus-within {
  border-color: rgba(0, 212, 255, 0.6);
  box-shadow: 0 0 20px rgba(0, 212, 255, 0.15), 0 0 40px rgba(0, 212, 255, 0.08), 0 4px 20px rgba(0, 0, 0, 0.06);
}

.app-container input {
  color: #333;
}

.app-container input::placeholder {
  color: #999;
}

.app-container .tool-btn {
  color: #888;
}

.app-container .tool-btn:hover {
  color: #333;
  background: rgba(0, 212, 255, 0.08);
}

.app-container .send-btn {
  color: #999;
}

.app-container .send-btn-active {
  color: #00d4ff;
}

.app-container .welcome-message h2 {
  color: #333;
}

.app-container .welcome-message p {
  color: #888;
}

.app-container .message {
  border-bottom: 1px solid rgba(0, 0, 0, 0.04);
}

.app-container .user-message {
  background: #fff;
}

.app-container .assistant-message {
  background: #fff;
}

.app-container .user-message .message-avatar {
  color: #999;
}

.app-container .disclaimer-text {
  color: #999;
}

.app-container .chat-input-container::before {
  background: linear-gradient(to bottom, transparent, rgba(255, 255, 255, 0.8));
}

.app-container .agent-item:hover {
  background-color: rgba(0, 212, 255, 0.06);
}

.app-container .agent-item.active {
  background-color: rgba(0, 212, 255, 0.1);
}

.app-container .collapse-btn {
  color: #999;
}

.app-container .collapse-btn:hover {
  color: #333;
  background: rgba(0, 0, 0, 0.05);
}

.app-container .expand-btn {
  color: #999;
}

.app-container .expand-btn:hover {
  color: #333;
}

.app-container .search-panel {
  background: rgba(255, 255, 255, 0.85);
  border-right: 1px solid rgba(0, 0, 0, 0.08);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
}

.app-container .search-panel-header {
  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.app-container .search-panel-header h3 {
  color: #333;
}

.app-container .search-result-item {
  background: rgba(0, 0, 0, 0.02);
  border: 1px solid rgba(0, 0, 0, 0.06);
}

.app-container .search-result-item:hover {
  background: rgba(0, 0, 0, 0.04);
}

.app-container .search-result-item.active {
  background: rgba(0, 212, 255, 0.06);
  border-color: rgba(0, 212, 255, 0.2);
}

.app-container .result-title {
  color: #333;
}

.app-container .result-content {
  border-top: 1px solid rgba(0, 0, 0, 0.06);
}

.app-container .result-snippet {
  color: #666;
}

.app-container .result-url {
  color: #999;
}

.app-container .result-detail-panel {
  background: rgba(240, 244, 248, 0.95);
}

.app-container .detail-header {
  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.app-container .back-btn {
  color: #999;
}

.app-container .back-btn:hover {
  color: #333;
}

.app-container .detail-source {
  color: #333;
}

.app-container .detail-date {
  color: #999;
}

.app-container .detail-text {
  color: #333;
}

.app-container .detail-link {
  color: #00d4ff;
  border-color: #00d4ff;
}

.app-container .detail-link:hover {
  background: rgba(0, 212, 255, 0.1);
}

.app-container .thinking-details {
  background: rgba(0, 212, 255, 0.04);
  border: 1px solid rgba(0, 212, 255, 0.1);
}

.app-container .thinking-summary {
  color: #4a90d9;
}

.app-container .thinking-icon {
  color: #4a90d9;
}

.app-container .thinking-content {
  color: #666;
}

.app-container .upload-status {
  background: rgba(255, 255, 255, 0.7);
  border: 1px solid rgba(0, 0, 0, 0.06);
}

.app-container .upload-header {
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
  color: #333;
}

.app-container .file-status {
  color: #999;
}

.app-container .modal-content {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.7);
}

.app-container .modal-content h3 {
  color: #333;
}

.app-container .modal-content p {
  color: #666;
}

.app-container .rename-input {
  background: rgba(255, 255, 255, 0.7);
  border: 1px solid rgba(0, 0, 0, 0.1);
  color: #333;
}

.app-container button.cancel-btn {
  border-color: rgba(0, 0, 0, 0.15);
  color: #666;
}

.app-container button.cancel-btn:hover {
  background: rgba(0, 0, 0, 0.05);
}

.app-container .chat-popup {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.7);
}

.app-container .chat-popup-header {
  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.app-container .chat-popup-header h3 {
  color: #333;
}

.app-container .chat-popup-close {
  color: #999;
}

.app-container .chat-popup-close:hover {
  color: #333;
  background: rgba(0, 0, 0, 0.05);
}

.app-container .popup-assistant-message {
  background: rgba(0, 0, 0, 0.05);
  color: #333;
}

.app-container .chat-popup-input {
  border-top: 1px solid rgba(0, 0, 0, 0.08);
}

.app-container .chat-popup-input input {
  background: rgba(0, 0, 0, 0.05);
  color: #333;
}

.app-container .chat-popup-input input::placeholder {
  color: #999;
}

.app-container .popup-send-btn:disabled {
  background-color: #ccc;
}

.app-container ::deep(.thinking-content blockquote) {
  background: rgba(0, 0, 0, 0.04);
  color: #555;
}

.app-container ::deep(.thinking-content strong) {
  color: #00d4ff;
}

.app-container ::deep(.thinking-content pre) {
  background: rgba(0, 0, 0, 0.06);
}

.app-container ::deep(.search-result) {
  background: rgba(0, 212, 255, 0.03);
  border: 1px solid rgba(0, 212, 255, 0.1);
}

.app-container ::deep(.search-result:hover) {
  background: rgba(0, 212, 255, 0.06);
  border-color: rgba(0, 212, 255, 0.2);
}

.app-container ::deep(.search-snippet) {
  color: #666;
}

.app-container ::deep(.search-url) {
  color: #999;
}

.app-container ::deep(.file-message) {
  background: rgba(0, 212, 255, 0.03);
  border: 1px solid rgba(0, 212, 255, 0.1);
}

.app-container ::deep(.file-name) {
  color: #333;
}

.app-container ::deep(.file-status) {
  color: #00d4ff;
}

.app-container ::deep(.error-message) {
  color: #ff4b4b;
  background: rgba(255, 75, 75, 0.08);
  border: 1px solid rgba(255, 75, 75, 0.2);
}

.app-container ::deep(.cefa5c26) {
  background: rgba(255, 255, 255, 0.7);
  box-shadow: 0 -4px 12px rgba(0, 0, 0, 0.05);
}

.app-container ::deep(.f3a54b52) {
  color: #333;
}

.app-container ::deep(.ee357eab) {
  color: #999;
}

.app-container ::deep(.delete-btn) {
  color: #999;
}

.app-container ::deep(.delete-btn:hover) {
  color: #ff4b4b;
  background: rgba(255, 75, 75, 0.08);
}

.app-container .logout-btn {
  color: #999;
}

.app-container .logout-btn:hover {
  color: #ff4b4b;
}

.app-container .action-btn {
  color: #999;
}

.app-container .action-btn:hover {
  background: rgba(0, 0, 0, 0.05);
}

.app-container .close-btn {
  color: #999;
}

.app-container .close-btn:hover {
  color: #333;
}

.app-container ::deep(.tooltip) {
  background-color: #333;
  color: #fff;
}

.app-container ::deep(.tooltip::after) {
  border-color: #333 transparent transparent transparent;
}

.app-container .search-loading-container {
  background: rgba(0, 212, 255, 0.08);
  border: 1px solid rgba(0, 212, 255, 0.2);
}

.app-container .search-loading-container:hover {
  background: rgba(0, 212, 255, 0.12);
  border-color: rgba(0, 212, 255, 0.3);
}

.app-container .search-loading-text {
  color: #00d4ff;
}

.app-container .file-info-container {
  background: rgba(0, 212, 255, 0.03);
  border: 1px solid rgba(0, 212, 255, 0.1);
}

.app-container .file-info-header {
  color: #00d4ff;
}

.app-container .file-info-content {
  color: #333;
}

.app-container .search-loading-container.error {
  background: rgba(255, 75, 75, 0.08);
  border-color: rgba(255, 75, 75, 0.2);
}

.app-container .search-loading-container.error .search-loading-text {
  color: #ff4b4b;
}

/* 左侧栏 */
.sidebar {
  width: 260px;
  height: 100vh;
  background: linear-gradient(180deg, rgba(13, 17, 23, 0.92) 0%, rgba(10, 13, 20, 0.92) 100%);
  border-right: 1px solid rgba(0, 212, 255, 0.12);
  display: flex;
  flex-direction: column;
  transition: width 0.3s ease;
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  overflow: hidden;
  min-width: 0;
}

.sidebar-collapsed {
  width: 0;
}

/* 右侧聊天区域 */
.chat-container {
  flex: 1;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #fff;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: #c8d0d8 #e0e7f0;
  position: relative;
}

/* 聊天内容区域 */
.chat-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  transition: all 0.3s ease;
  overflow-y: visible;
  position: relative;
  background: #fff;
}

.chat-container::-webkit-scrollbar {
  width: 8px;
  position: absolute;
  right: 0;
}

.chat-container::-webkit-scrollbar-track {
  background: #0a0d14;
}

.chat-container::-webkit-scrollbar-thumb {
  background-color: #4a4a4a;
  border-radius: 4px;
  border: 2px solid #0a0d14;
}

.chat-container::-webkit-scrollbar-thumb:hover {
  background-color: #666;
}

/* 初始状态样式 */
.initial-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 40px;
  margin: auto;
  width: 100%;
  max-width: 600px;
  opacity: 1;
  transform: translateY(0);
  transition: all 0.3s ease;
}

/* 对话状态样式 */
.chat-state {
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100%;
  position: relative;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 消息区域样式 */
.chat-messages {
  flex: 1;
  padding-bottom: 280px; /* 增加更多底部padding，确保内容不被遮挡 */
  background: #fff;
}

.chat-input-container {
  position: fixed;
  bottom: 0;
  left: calc(50% + 130px); /* 考虑侧边栏宽度的一半 (260px / 2 = 130px) */
  transform: translateX(-50%);
  width: 100%;
  max-width: 800px;
  background: #fff;
  padding: 0 20px 20px;
  z-index: 10;
  transition: left 0.3s ease;
}

/* 当侧边栏收起时的样式 */
.sidebar-collapsed ~ .chat-container .chat-input-container {
  left: 50%; /* 侧边栏收起时回到中间 */
}

.chat-input {
  width: 100%;
  margin-bottom: 8px;
  transition: all 0.3s ease;
}

.chat-input .input-wrapper {
  max-width: 800px;
}

.disclaimer-text {
  text-align: center;
  color: #666;
  font-size: 12px;
  padding: 4px 0;
}

/* 聊天框样式 */
.center-chat-box {
  display: flex;
  flex-direction: column;
  gap: 40px;
  width: 100%;
  max-width: 600px;
}

/* 欢迎消息样式 */
.welcome-message {
  text-align: center;
}

.welcome-message h2 {
  color: #333;
  font-size: 24px;
  margin: 0;
  font-weight: normal;
}

.welcome-message h2 span {
  background: linear-gradient(135deg, #00d4ff 0%, #6366f1 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 700;
}

.welcome-message p {
  color: #888;
  font-size: 13px;
  margin: 0;
  margin-top: 2px;
}

/* 消息样式 */
.message {
  padding: 12px 20px;
  display: flex;
  align-items: flex-start;
  gap: 10px;
  width: 100%;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 消息体 */
.message-body {
  display: flex;
  flex-direction: column;
  gap: 6px;
  max-width: calc(100% - 40px);
}

/* 用户消息样式 */
.user-message {
  justify-content: flex-end;
}

.user-message .message-body {
  align-items: flex-end;
}

.user-message .message-content {
  background: #fff;
  border-radius: 18px 18px 4px 18px;
  padding: 10px 16px;
  max-width: 80%;
}

/* AI消息样式 */
.assistant-message {
  justify-content: flex-start;
}

.assistant-message .message-body {
  align-items: flex-start;
  max-width: calc(100% - 40px);
}

.assistant-message .message-content {
  padding-top: 2px;
}

/* 头像样式 */
.message-avatar {
  width: 28px;
  height: 28px;
  flex-shrink: 0;
  margin-top: 2px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.user-avatar {
  order: 1;
}

/* 消息内容样式 */
.message-content {
  font-size: 14px;
  line-height: 1.6;
  color: #333;
}

/* 消息操作按钮 */
.message-actions {
  display: flex;
  align-items: center;
  gap: 2px;
  opacity: 1;
}

.user-actions {
  justify-content: flex-end;
  padding-right: 4px;
}

.msg-action-btn {
  background: none;
  border: none;
  color: #bbb;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.msg-action-btn:hover {
  color: #666;
  background: rgba(0, 0, 0, 0.04);
}

/* 底部个人信息样式 */
.user-section {
  padding: 16px;
  border-top: 1px solid rgba(0, 212, 255, 0.12);
  margin-top: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
  position: relative;
}

.user-menu {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 12px;
  cursor: pointer;
  border-radius: 10px;
  transition: all 0.2s;
  color: #333;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.06);
}

.user-menu:hover {
  background: rgba(255, 255, 255, 0.08);
  border-color: rgba(0, 212, 255, 0.2);
}

.arrow-icon {
  transition: transform 0.2s;
}

.user-menu:hover .arrow-icon {
  transform: rotate(180deg);
}

.user-dropdown {
  position: fixed;
  bottom: 80px;
  left: 16px;
  width: 228px;
  background: rgba(20, 24, 35, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4), 0 0 20px rgba(0, 212, 255, 0.08);
  z-index: 1000;
  animation: dropdownSlide 0.2s ease;
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
}

@keyframes dropdownSlide {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.menu-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  color: #333;
  cursor: pointer;
  transition: all 0.2s;
}

.menu-item:hover {
  background: rgba(255, 255, 255, 0.08);
}

.menu-item:last-child {
  color: #ff6b6b;
}

.menu-item:last-child:hover {
  background: rgba(255, 107, 107, 0.12);
}

.user-avatar {
  width: 24px;
  height: 24px;
  color: #888;
}

.user-text {
  font-size: 14px;
  color: #333;
}

/* 侧边栏样式 */
.sidebar-header {
  padding: 16px;
  border-bottom: 1px solid rgba(0, 212, 255, 0.08);
  position: relative;
}

.sidebar-header::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 16px;
  width: 60px;
  height: 1px;
  background: linear-gradient(90deg, rgba(0, 212, 255, 0.3), transparent);
}

.logo-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  gap: 8px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 10px;
  margin: 0 16px 16px;
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
}

.logo-text {
  color: #333;
  font-size: 20px;
  font-weight: 600;
  letter-spacing: 2px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.logo-highlight {
  background: linear-gradient(135deg, #00d4ff 0%, #6366f1 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 700;
}

/* 灵犀在线状态指示 */
.logo-pulse {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #00d4ff;
  box-shadow: 0 0 6px rgba(0, 212, 255, 0.6), 0 0 12px rgba(0, 212, 255, 0.3);
  animation: logoPulse 2s ease-in-out infinite;
  flex-shrink: 0;
}

@keyframes logoPulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(1.8); }
}

.collapse-btn {
  background: none;
  border: none;
  color: #666;
  padding: 8px;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.collapse-btn:hover {
  color: #333;
  background: rgba(255, 255, 255, 0.1);
}

.agents-list {
  padding: 1rem 0;
  overflow-y: auto;
}

.agent-item {
  display: flex;
  align-items: center;
  padding: 0.8rem 1rem;
  cursor: pointer;
  transition: background-color 0.2s;
  gap: 0.8rem;
}

.agent-item:hover {
  background-color: #363636;
}

.agent-item.active {
  background-color: #4a4eff33;
}

.agent-icon {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.2rem;
}

.agent-name {
  font-size: 0.9rem;
  color: #333;
}

.new-chat-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin: 0 16px;
  padding: 8px 12px;
  font-size: 13px;
  color: #fff;
  background: linear-gradient(135deg, #00d4ff 0%, #6366f1 100%);
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 4px 16px rgba(0, 212, 255, 0.2);
}

.new-chat-btn:hover {
  box-shadow: 0 6px 24px rgba(0, 212, 255, 0.35);
  transform: translateY(-1px);
}

.new-chat-btn svg {
  width: 14px;
  height: 14px;
  color: #fff;
  opacity: 1;
}

/* 输入框容器样式 */
.input-wrapper {
  width: 100%;
  background: rgba(20, 24, 35, 0.55);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 25px;
  padding: 12px 16px;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.input-wrapper:focus-within {
  border-color: rgba(0, 212, 255, 0.5);
  box-shadow: 0 0 20px rgba(0, 212, 255, 0.12), 0 0 40px rgba(0, 212, 255, 0.06);
}

input {
  width: 100%;
  background: none;
  border: none;
  color: #333;
  font-size: 14px;
  outline: none;
  padding: 8px 0;
  margin-bottom: 4px;
}

input::placeholder {
  color: #666;
}

/* 按钮组样式 */
.button-group {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 8px;
}

.left-buttons, .right-buttons {
  display: flex;
  gap: 8px;
  align-items: center;
}

.tool-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  background: none;
  border: none;
  color: #888;
  padding: 4px 8px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: 4px;
}

.tool-btn:hover {
  color: #333;
  background: rgba(0, 212, 255, 0.1);
}

.tool-btn-active {
  color: #00d4ff !important;
  background: rgba(0, 212, 255, 0.1);
}

.tool-btn-active:hover {
  background: rgba(0, 212, 255, 0.2);
}

.send-btn {
  background: none;
  border: none;
  padding: 8px;
  cursor: pointer;
  color: #666;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.send-btn:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

.send-btn-active {
  color: #00d4ff;
}

.send-btn-active:hover {
  color: #00b8e6;
  filter: drop-shadow(0 0 6px rgba(0, 212, 255, 0.4));
}

.send-btn .icon {
  width: 16px;
  height: 16px;
}

/* 空状态样式调整 */
.empty-state {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  padding: 1rem;
}

.empty-state-content {
  background: none;
  padding: 0;
  box-shadow: none;
  width: 100%;
  max-width: 600px;
}

.empty-state-content h3 {
  color: #333;
  font-size: 1rem;
  margin-bottom: 0.3rem;
}

.empty-state-content p {
  color: #666;
  font-size: 0.9rem;
  margin-bottom: 1.5rem;
}

.empty-state-content .input-wrapper {
  margin-top: 0;
}

/* 工具提示样式 */
.tool-btn .tooltip {
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background-color: #4a4a4a;
  color: #fff;
  padding: 0.3rem 0.6rem;
  border-radius: 4px;
  font-size: 0.8rem;
  white-space: nowrap;
  opacity: 0;
  visibility: hidden;
  transition: all 0.2s;
  margin-bottom: 0.5rem;
  pointer-events: none;
}

.tool-btn:hover .tooltip {
  opacity: 1;
  visibility: visible;
}

.tool-btn .tooltip::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border-width: 4px;
  border-style: solid;
  border-color: #4a4a4a transparent transparent transparent;
}

/* 修改空状态下的输入框样式 */
.empty-state-content .input-wrapper {
  margin-top: 1.5rem;
  width: 100%;
}

/* 修改历史聊天列表样式 */
.chat-history {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.chat-history::-webkit-scrollbar {
  display: none;
}

.history-list {
  margin-top: 16px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.history-item {
  padding: 12px 16px;
  cursor: pointer;
  transition: all 0.2s ease;
  background-color: transparent;
  border-radius: 0;
}

.history-item:hover {
  background-color: #141823;
}

.history-item.active {
  background-color: #343541;
  border-left: none;
}

.history-content {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.history-title {
  color: #ececf1;
  font-size: 13px;
  line-height: 1.4;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.history-time {
  color: #666;
  font-size: 12px;
}

/* Markdown 样式优化 */
:deep(.message-content) {
  color: #333;
  font-size: 14px;
  line-height: 1.6;

  h1, h2, h3, h4, h5, h6 {
    color: #333;
    margin: 1em 0;
  }

  p {
    color: #333;
    margin: 0.5em 0;
  }

  a {
    text-decoration: none;
    transition: all 0.2s ease;
  }

  a:hover {
    text-decoration: underline;
    opacity: 0.8;
  }

  ul, ol {
    color: #333;
    margin: 0.5em 0;
    padding-left: 2em;
  }

  li {
    margin: 0.25em 0;
  }

  blockquote {
    margin: 1em 0;
    padding: 1em;
    background: rgba(51, 51, 51, 0.6);
    border-left: 4px solid #00d4ff;
    border-radius: 4px;
    color: #666;
  }

  blockquote strong {
    color: #00d4ff;
  }
}

/* 修改展开按钮样式 */
.expand-btn {
  position: fixed;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: #666;
  padding: 8px;
  cursor: pointer;
  transition: all 0.2s;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
}

.expand-btn:hover {
  color: #333;
}

/* 搜索结果样式 */
.search-result {
  background: rgba(0, 212, 255, 0.05);
  border: 1px solid rgba(0, 212, 255, 0.1);
  border-radius: 8px;
  padding: 16px;
  margin: 12px 0;
  transition: all 0.2s ease;
}

.search-result:hover {
  background: rgba(0, 212, 255, 0.1);
  border-color: rgba(0, 212, 255, 0.2);
}

.search-title {
  color: #00d4ff;
  font-size: 16px;
  font-weight: 500;
  text-decoration: none;
  display: block;
  margin-bottom: 8px;
  line-height: 1.4;
}

.search-title:hover {
  text-decoration: underline;
}

.search-snippet {
  color: #ccc;
  font-size: 14px;
  line-height: 1.6;
  margin-bottom: 8px;
}

.search-url {
  color: #666;
  font-size: 12px;
  word-break: break-all;
}

:deep(.message-content) {
  .search-result {
    background: rgba(0, 212, 255, 0.05);
    border: 1px solid rgba(0, 212, 255, 0.1);
    border-radius: 8px;
    padding: 16px;
    margin: 12px 0;
    transition: all 0.2s ease;
  }

  .search-result:hover {
    background: rgba(0, 212, 255, 0.1);
    border-color: rgba(0, 212, 255, 0.2);
  }

  .search-title {
    color: #00d4ff;
    font-size: 16px;
    font-weight: 500;
    text-decoration: none;
    display: block;
    margin-bottom: 8px;
    line-height: 1.4;
  }

  .search-title:hover {
    text-decoration: underline;
  }

  .search-snippet {
    color: #ccc;
    font-size: 14px;
    line-height: 1.6;
    margin-bottom: 8px;
  }

  .search-url {
    color: #666;
    font-size: 12px;
    word-break: break-all;
  }
}

/* 搜索面板样式 */
.search-panel {
  width: 320px;
  height: 100vh;
  background: #141823;
  border-right: 1px solid #333;
  display: flex;
  flex-direction: column;
}

.search-panel-header {
  padding: 16px;
  border-bottom: 1px solid #333;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.search-panel-header h3 {
  color: #333;
  font-size: 16px;
  margin: 0;
}

.close-btn {
  background: none;
  border: none;
  color: #666;
  font-size: 20px;
  cursor: pointer;
  padding: 4px 8px;
}

.close-btn:hover {
  color: #333;
}

.search-results-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
}

.search-result-item {
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  margin-bottom: 8px;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.search-result-item:hover {
  background: rgba(255, 255, 255, 0.05);
}

.search-result-item.active {
  background: rgba(0, 212, 255, 0.05);
  border-color: rgba(0, 212, 255, 0.2);
}

.result-header {
  cursor: pointer;
}

.result-source {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.source-name {
  color: #333;
  font-size: 13px;
}

.result-date {
  color: #666;
  font-size: 12px;
  margin-left: auto;
}

.result-title {
  color: #333;
  font-size: 14px;
  line-height: 1.4;
  margin-bottom: 4px;
}

.result-content {
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  animation: slideDown 0.2s ease;
}

.result-snippet {
  color: #ccc;
  font-size: 13px;
  line-height: 1.6;
  margin-bottom: 8px;
}

.result-link {
  display: inline-block;
  color: #00d4ff;
  text-decoration: none;
  font-size: 13px;
  transition: all 0.2s ease;
}

.result-link:hover {
  text-decoration: underline;
}

/* 搜索结果详情面板 */
.result-detail-panel {
  flex: 1;
  height: 100vh;
  background: #0a0d14;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.detail-header {
  padding: 16px;
  border-bottom: 1px solid #333;
  display: flex;
  align-items: center;
  gap: 16px;
}

.back-btn {
  background: none;
  border: none;
  color: #666;
  font-size: 20px;
  cursor: pointer;
  padding: 4px 8px;
}

.back-btn:hover {
  color: #fff;
}

.detail-header h3 {
  color: #fff;
  font-size: 18px;
  margin: 0;
  line-height: 1.4;
}

.detail-content {
  padding: 24px;
}

.detail-meta {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;
}

.detail-source {
  color: #fff;
  font-size: 14px;
}

.detail-date {
  color: #666;
  font-size: 14px;
}

.detail-text {
  color: #fff;
  font-size: 14px;
  line-height: 1.8;
  margin-bottom: 24px;
}

.detail-link {
  display: inline-block;
  color: #00d4ff;
  text-decoration: none;
  font-size: 14px;
  padding: 8px 16px;
  border: 1px solid #00d4ff;
  border-radius: 4px;
  transition: all 0.2s ease;
}

.detail-link:hover {
  background: rgba(0, 212, 255, 0.1);
}

/* 更新搜索加载状态样式 */
:deep(.search-loading-container) {
  background: rgba(0, 212, 255, 0.1);
  border: 1px solid rgba(0, 212, 255, 0.3);
  border-radius: 20px;
  padding: 0px 0px;
  margin: 12px 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  position: sticky;
  top: 0;
  z-index: 10;
  cursor: pointer;
  transition: all 0.2s ease;
  width: auto;
}

:deep(.search-loading-container:hover) {
  background: rgba(0, 212, 255, 0.2);
  border-color: rgba(0, 212, 255, 0.4);
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

:deep(.search-loading-text) {
  color: #00d4ff;
  font-size: 13px;
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
  padding: 4px 8px;
}

.thinking-details {
  background: rgba(0, 212, 255, 0.04);
  border: 1px solid rgba(0, 212, 255, 0.1);
  border-radius: 8px;
  margin-bottom: 12px;
  overflow: hidden;
}

.thinking-summary {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  cursor: pointer;
  list-style: none;
  font-size: 14px;
  color: #4a90d9;
  font-weight: 500;
  user-select: none;
}

.thinking-summary::-webkit-details-marker {
  display: none;
}

.thinking-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #4a90d9;
}

.thinking-arrow {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: auto;
  transition: transform 0.2s ease;
  color: #999;
}

.thinking-details[open] .thinking-arrow {
  transform: rotate(180deg);
}

.thinking-content {
  color: #666;
  font-size: 14px;
  line-height: 1.6;
  padding: 0 14px 14px;
}

.response-content {
  color: #fff;
  font-size: 14px;
  line-height: 1.6;
}

/* 添加渐变遮罩效果 */
.chat-input-container::before {
  content: '';
  position: absolute;
  top: -40px;
  left: 0;
  right: 0;
  height: 40px;
  background: linear-gradient(to bottom, transparent, #fff);
  pointer-events: none;
  z-index: -1;
}

/* 在已有样式中添加 */
.file-info-container {
  margin-top: 12px;
  padding: 12px;
  background: rgba(0, 212, 255, 0.05);
  border: 1px solid rgba(0, 212, 255, 0.1);
  border-radius: 8px;
}

.file-info-header {
  color: #00d4ff;
  font-size: 14px;
  font-weight: 500;
  margin-bottom: 8px;
}

.file-info-content {
  color: #fff;
  font-size: 14px;
  line-height: 1.6;
}

.file-info-content p {
  margin: 4px 0;
}

.search-loading-container.error {
  background: rgba(255, 75, 75, 0.1);
  border-color: rgba(255, 75, 75, 0.3);
}

.search-loading-container.error .search-loading-text {
  color: #ff4b4b;
}

/* 文件消息样式 */
:deep(.file-message) {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: rgba(0, 212, 255, 0.05);
  border: 1px solid rgba(0, 212, 255, 0.1);
  border-radius: 8px;
  margin: 8px 0;
}

:deep(.file-message.error) {
  background: rgba(255, 75, 75, 0.05);
  border-color: rgba(255, 75, 75, 0.1);
}

:deep(.file-icon) {
  font-size: 24px;
}

:deep(.file-info) {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

:deep(.file-name) {
  color: #fff;
  font-size: 14px;
  font-weight: 500;
}

:deep(.file-status) {
  color: #00d4ff;
  font-size: 12px;
}

:deep(.file-message.error .file-status) {
  color: #ff4b4b;
}

.upload-status-container {
  margin-bottom: 16px;
}

.upload-status {
  background: #141823;
  border-radius: 8px;
  overflow: hidden;
}

.upload-header {
  padding: 12px 16px;
  color: #fff;
  font-size: 14px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.upload-content {
  padding: 12px 16px;
}

.upload-file {
  display: flex;
  align-items: center;
  gap: 12px;
}

.file-icon {
  width: 32px;
  height: 32px;
}

.file-info {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.file-name {
  color: #fff;
  font-size: 14px;
  font-weight: 500;
}

.file-status {
  color: #888;
  font-size: 12px;
}

/* 文件上传消息样式 */
:deep(.cefa5c26) {
  background: #141823;
  border-radius: 6px;
  overflow: hidden;
  margin-bottom: 8px;
  box-shadow: 0 -4px 12px rgba(0, 0, 0, 0.1);
  width: 220px;
  z-index: 11;
  position: relative;  /* 将相对定位移到父容器 */
}

:deep(.ca114c67) {
  display: none;
}

:deep(.cd190a50) {
  display: flex;
  align-items: center;
  gap: 12px;
}

:deep(.d2d04dae) {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 10px;
}

:deep(.aea7ca45) {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

:deep(.f3a54b52) {
  color: #fff;
  font-size: 13px;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 120px;  /* 减小文件名显示宽度 */
}

:deep(.ee357eab) {
  color: #888;
  font-size: 11px;
}

:deep(.error-message) {
  color: #ff4b4b;
  padding: 12px;
  background: rgba(255, 75, 75, 0.1);
  border: 1px solid rgba(255, 75, 75, 0.3);
  border-radius: 8px;
  margin: 8px 0;
}

:deep(.header-content) {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

:deep(.delete-btn) {
  background: none;
  border: none;
  color: #666;
  padding: 3px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: all 0.2s;
  position: absolute;  /* 保持绝对定位 */
  top: 4px;
  right: 4px;
  z-index: 12;  /* 确保按钮始终在最上层 */
}

:deep(.delete-btn:hover) {
  color: #ff4b4b;
  background: rgba(255, 75, 75, 0.1);
}

:deep(.ds-icon) {
  svg {
    &[data-type="pdf"] path {
      fill: #F8CA27;
    }
    &[data-type="doc"] path, &[data-type="docx"] path {
      fill: #4B8BF4;
    }
    &[data-type="xls"] path, &[data-type="xlsx"] path {
      fill: #34A853;
    }
    &[data-type="ppt"] path, &[data-type="pptx"] path {
      fill: #EA4335;
    }
    &[data-type="txt"] path {
      fill: #9AA0A6;
    }
    &[data-type="image"] path {
      fill: #4B8BF4;
    }
  }
}

:deep(.ds-icon.b3a5d6c1) {
  font-size: 24px;  /* 减小图标大小 */
  width: 24px;
  height: 24px;
}

.logout-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  transition: all 0.2s;
}

.logout-btn:hover {
  color: #ff4b4b;
}

.history-title {
  font-size: 14px;
  color: #fff;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 180px;  /* 限制宽度，防止标题太长 */
}

.history-time {
  font-size: 12px;
  color: #888;
}

.history-item {
  padding: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  border-radius: 8px;
  margin: 4px 0;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid transparent;
}

.history-item:hover {
  background: rgba(255, 255, 255, 0.06);
  border-color: rgba(0, 212, 255, 0.15);
}

.history-item.active {
  background: rgba(0, 212, 255, 0.12);
  border-color: rgba(0, 212, 255, 0.25);
}

.history-actions {
  display: flex;
  gap: 8px;
}

.action-btn {
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: 4px;
}

.action-btn:hover {
  background: rgba(255, 255, 255, 0.1);
}

.rename-btn {
  display: flex;
  align-items: center;
  gap: 8px;
}

.delete-btn {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* 对话框样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background-color: #141823;
  border-radius: 8px;
  padding: 20px;
  width: 100%;
  max-width: 400px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.app-container.light .modal-content {
  background-color: #ffffff;
  color: #333333;
}

.modal-content h3 {
  margin-bottom: 16px;
  font-size: 18px;
  font-weight: 500;
}

.modal-content p {
  margin-bottom: 20px;
  color: #aaa;
}

.app-container.light .modal-content p {
  color: #666;
}

.rename-input {
  width: 100%;
  padding: 10px;
  border: 1px solid #555;
  background-color: #333;
  color: #fff;
  border-radius: 4px;
  font-size: 14px;
  margin-bottom: 20px;
}

.app-container.light .rename-input {
  background-color: #f5f5f5;
  color: #333;
  border-color: #ddd;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

button.cancel-btn {
  background-color: transparent;
  border: 1px solid #555;
  color: #aaa;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

button.cancel-btn:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

button.confirm-btn {
  background-color: #0070f3;
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

button.confirm-btn:hover {
  background-color: #0051b3;
}

button.delete-btn {
  background-color: #e53e3e;
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

button.delete-btn:hover {
  background-color: #c53030;
}

.app-container.light button.cancel-btn {
  border-color: #ddd;
  color: #666;
}

.app-container.light button.cancel-btn:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

/* 悬浮聊天按钮样式 */
.chat-float-button {
  position: fixed;
  right: 30px;
  bottom: 30px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: #00d4ff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 1000;
  transition: all 0.3s ease;
}

.chat-float-button:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

/* 聊天弹窗样式 */
.chat-popup {
  position: fixed;
  right: 30px;
  bottom: 30px;
  width: 320px;
  height: 400px;
  background-color: #141823;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
  z-index: 1000;
  overflow: hidden;
  animation: popupFadeIn 0.3s ease;
}

.app-container.light .chat-popup {
  background-color: #ffffff;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

@keyframes popupFadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.chat-popup-header {
  padding: 12px 16px;
  border-bottom: 1px solid #333;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.app-container.light .chat-popup-header {
  border-bottom-color: #e0e0e0;
}

.chat-popup-header h3 {
  color: #fff;
  font-size: 16px;
  margin: 0;
}

.app-container.light .chat-popup-header h3 {
  color: #333;
}

.chat-popup-close {
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.chat-popup-close:hover {
  color: #fff;
  background: rgba(255, 255, 255, 0.1);
}

.app-container.light .chat-popup-close:hover {
  color: #333;
  background: rgba(0, 0, 0, 0.05);
}

.chat-popup-messages {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.popup-message {
  max-width: 80%;
  padding: 8px 12px;
  border-radius: 12px;
  font-size: 14px;
  line-height: 1.4;
}

.popup-user-message {
  align-self: flex-end;
  background-color: #00d4ff;
  color: white;
  border-bottom-right-radius: 4px;
}

.popup-assistant-message {
  align-self: flex-start;
  background-color: #3a3a3a;
  color: white;
  border-bottom-left-radius: 4px;
}

.app-container.light .popup-assistant-message {
  background-color: #f0f0f0;
  color: #333;
}

.chat-popup-input {
  padding: 12px;
  border-top: 1px solid #333;
  display: flex;
  gap: 8px;
}

.app-container.light .chat-popup-input {
  border-top-color: #e0e0e0;
}

.chat-popup-input input {
  flex: 1;
  background-color: #3a3a3a;
  border: none;
  color: white;
  padding: 8px 12px;
  border-radius: 20px;
  font-size: 14px;
  outline: none;
}

.app-container.light .chat-popup-input input {
  background-color: #f0f0f0;
  color: #333;
}

.chat-popup-input input::placeholder {
  color: #999;
}

.popup-send-btn {
  background-color: #00d4ff;
  border: none;
  color: white;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
}

.popup-send-btn:hover {
  background-color: #00b8e6;
}

.popup-send-btn:disabled {
  background-color: #555;
  cursor: not-allowed;
}

/* 在 popup-message 样式下添加加载状态样式 */
.popup-message.popup-assistant-message:has(.loading-dots) {
  background-color: #00d4ff20;
}

.loading-dots {
  display: inline-flex;
  align-items: center;
}

.loading-dots span {
  display: inline-block;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: currentColor;
  margin: 0 2px;
  animation: loadingDots 1.4s infinite ease-in-out both;
}

.loading-dots span:nth-child(1) {
  animation-delay: -0.32s;
}

.loading-dots span:nth-child(2) {
  animation-delay: -0.16s;
}

@keyframes loadingDots {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1); }
}

/* 电商客服按钮样式 */
.ecommerce-btn {
  background: linear-gradient(135deg, #00d4ff 0%, #6366f1 100%) !important;
  color: white !important;
  transition: all 0.3s ease;
  border: none !important;
}

.ecommerce-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 212, 255, 0.4);
}

.ecommerce-link {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  background: linear-gradient(135deg, #00d4ff 0%, #6366f1 100%);
  border-radius: 12px;
  cursor: pointer;
  margin-bottom: 16px;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 4px 12px rgba(0, 212, 255, 0.3);
  position: relative;
  overflow: hidden;
}

.ecommerce-link:before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0) 70%);
  z-index: 1;
}

.ecommerce-link:hover {
  transform: translateY(-4px) scale(1.02);
  box-shadow: 0 8px 16px rgba(0, 212, 255, 0.4);
}

.ecommerce-link:active {
  transform: translateY(0) scale(0.98);
  box-shadow: 0 2px 8px rgba(0, 212, 255, 0.3);
}

.ecommerce-icon {
  width: 28px;
  height: 28px;
  margin-right: 12px;
  position: relative;
  z-index: 2;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.1));
}

.ecommerce-text {
  color: white;
  font-size: 15px;
  font-weight: 600;
  letter-spacing: 0.5px;
  position: relative;
  z-index: 2;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.ecommerce-badge {
  position: absolute;
  top: -6px;
  right: -6px;
  background-color: #fff;
  color: #6366f1;
  font-size: 12px;
  font-weight: bold;
  padding: 2px 6px;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 212, 255, 0.3);
  z-index: 3;
  transform: rotate(15deg);
}

/* 思考过程样式 */
:deep(.message-wrapper) {
  display: flex;
  flex-direction: column;
  gap: 12px;
  width: 100%;
}

:deep(.thinking-details) {
  background: rgba(0, 212, 255, 0.04);
  border: 1px solid rgba(0, 212, 255, 0.1);
  border-radius: 8px;
  margin-bottom: 12px;
  overflow: hidden;
}

:deep(.thinking-summary) {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  cursor: pointer;
  list-style: none;
  font-size: 14px;
  color: #4a90d9;
  font-weight: 500;
  user-select: none;
}

:deep(.thinking-summary::-webkit-details-marker) {
  display: none;
}

:deep(.thinking-icon) {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #4a90d9;
}

:deep(.thinking-arrow) {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: auto;
  transition: transform 0.2s ease;
  color: #999;
}

:deep(.thinking-details[open] .thinking-arrow) {
  transform: rotate(180deg);
}

:deep(.thinking-content) {
  font-size: 14px;
  line-height: 1.6;
  padding: 0 14px 14px;
  color: #666;
}

:deep(.thinking-content blockquote) {
  margin: 0;
  padding: 16px;
  background: rgba(0, 212, 255, 0.06);
  border-left: 4px solid #00d4ff;
  border-radius: 0 8px 8px 0;
  color: #555;
}

:deep(.thinking-content strong) {
  color: #4a90d9;
  font-weight: 600;
  display: block;
  margin-bottom: 8px;
}

:deep(.thinking-content p) {
  margin: 8px 0;
  color: #666;
}

:deep(.thinking-content code) {
  background: rgba(0, 212, 255, 0.08);
  padding: 2px 4px;
  border-radius: 4px;
  color: #4a90d9;
  font-family: monospace;
}

:deep(.thinking-content pre) {
  background: rgba(0, 0, 0, 0.04);
  padding: 12px;
  border-radius: 4px;
  overflow-x: auto;
  margin: 10px 0;
}

:deep(.thinking-content pre code) {
  background: transparent;
  padding: 0;
  color: #333;
}

:deep(.message-text) {
  font-size: 14px;
  line-height: 1.6;
}
</style>