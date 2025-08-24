# EFB Telegram Master 增强版

[![PyPI release](https://img.shields.io/pypi/v/efb-telegram-master.svg)](https://pypi.org/project/efb-telegram-master/)
[![Tests status](https://github.com/ehForwarderBot/efb-telegram-master/workflows/Tests/badge.svg)](https://github.com/ehForwarderBot/efb-telegram-master/actions)
[![Downloads per month](https://pepy.tech/badge/efb-telegram-master/month)](https://pepy.tech/project/efb-telegram-master)

## 🚀 主要增强功能

### ✨ 自动创建群组功能
- **智能群组管理**：自动为不同类型的对话创建对应的 Telegram 群组
- **灵活分类**：支持私聊、群聊、系统消息、公众号等不同类型的自动分组
- **自动化操作**：支持自动静音、归档、添加到文件夹等后续操作
- **便捷管理**：无需手动创建和链接群组，一切自动化完成

### 🌐 完整代理支持
- **双重代理配置**：同时支持主要功能和自动创建群组功能的代理设置
- **多协议支持**：完整支持 HTTP 和 SOCKS5 代理
- **API 域名代理**：支持自定义 Telegram API 域名，突破网络限制
- **统一配置**：一处配置，全功能生效

## 安装和配置

### 系统要求
- Python >= 3.6
- EH Forwarder Bot >= 2.0.0
- ffmpeg, libmagic, libwebp

### 快速安装
```bash
pip3 install efb-telegram-master
```

### 基本配置
配置文件位置：`~/.ehforwarderbot/profiles/default/blueset.telegram/config.yaml`

## 🔧 自动创建群组配置

### 聊天类型说明
- `1`: 私人聊天
- `2`: 群组聊天  
- `3`: 系统消息
- `4`: 公众号/频道

### 完整配置示例

```yaml
# 基本配置
token: "你的bot token"
admins:
  - 你的用户ID

flags:
  # 启用自动群组管理
  auto_manage_tg_config:
    # 启用自动创建群组功能
    auto_manage_tg: true
    
    # Telegram API 凭证（从 https://my.telegram.org/apps 获取）
    tg_api_id: 12345
    tg_api_hash: "your_api_hash"
    bot_name: "@your_bot_name"
    
    # 自动创建群组的聊天类型
    auto_create_tg_group: [1, 2]  # 私聊和群聊
    
    # 自动静音创建的群组
    auto_mute_created_tg_group: [2]  # 静音群聊
    
    # 自动添加到文件夹
    auto_add_group_to_folder:
      1: "微信-私聊"
      2: "微信-群聊"
      3: "微信-系统"
      4: "微信-公众号"
    
    # 自动归档群组
    auto_archive_create_tg_group: [2, 3, 4]
    
    # 公众号消息统一群组ID（可选）
    mq_auto_link_group_id: "-xxxxxxxx"
    
    # 🌟 新增：Pyrogram 代理配置
    pyrogram_proxy:
      type: socks5  # 或 http
      hostname: 127.0.0.1
      port: 1080
      # 可选：代理认证
      username: proxy_user
      password: proxy_pass

# 主要功能代理配置
request_kwargs:
  proxy_url: socks5://127.0.0.1:1080/
  # HTTP 代理示例：
  # proxy_url: http://127.0.0.1:8080/

# 可选：自定义 API 域名
flags:
  api_base_url: "https://your-api-proxy.com/bot"
  api_base_file_url: "https://your-api-proxy.com/file/bot"
```

## 🌐 代理配置方案

### 方案一：传统代理
```yaml
request_kwargs:
  proxy_url: socks5://127.0.0.1:1080/
  # 或 HTTP 代理
  # proxy_url: http://127.0.0.1:8080/
```

### 方案二：API 域名代理
```yaml
flags:
  api_base_url: "https://api-proxy.example.com/bot"
  api_base_file_url: "https://api-proxy.example.com/file/bot"
```

### 方案三：组合配置（推荐）
```yaml
flags:
  api_base_url: "https://api-proxy.example.com/bot"
  auto_manage_tg_config:
    pyrogram_proxy:
      type: socks5
      hostname: 127.0.0.1
      port: 1080

request_kwargs:
  proxy_url: socks5://127.0.0.1:1080/
```

## 💡 使用场景

### 自动群组管理场景
1. **多平台整合**：微信、QQ、Discord 等多平台消息自动分类
2. **工作流优化**：重要联系人自动创建专用群组，杂乱消息自动归档
3. **隐私保护**：敏感对话自动静音，避免通知泄露

### 代理使用场景
1. **网络限制**：在受限网络环境中正常使用 Telegram Bot
2. **服务器部署**：VPS 服务器通过代理访问 Telegram API
3. **高可用性**：多重代理配置确保服务稳定运行

## 🎯 核心功能

### 消息转发
- 支持所有 Telegram 消息类型（文本、图片、语音、文件等）
- 智能引用回复和消息编辑
- 双向消息同步

### 群组管理
- `/link` - 手动链接对话到群组
- `/unlink_all` - 取消群组链接
- `/update_info` - 自动更新群组信息
- `/chat` - 生成对话头用于快速回复

### 高级功能
- `/extra` - 访问从属频道的额外功能
- `/react` - 消息表情回应
- `/rm` - 删除远程消息
- 消息过滤和搜索

## 📋 基本命令

```
help - 显示命令列表
link - 链接远程对话到群组
unlink_all - 取消群组的所有链接
info - 显示当前群组信息
chat - 生成对话头
extra - 访问从属频道功能
update_info - 更新群组信息
react - 消息表情回应
rm - 删除远程消息
```

## 🔧 高级配置选项

### 消息处理
```yaml
flags:
  # 每页显示的对话数量
  chats_per_page: 10
  
  # 允许多个对话链接到同一群组
  multiple_slave_chats: true
  
  # 快速回复设置
  send_to_last_chat: "warn"  # enabled/warn/disabled
  
  # 媒体消息占位符
  default_media_prompt: "emoji"  # emoji/text/disabled
```

### 网络配置
```yaml
request_kwargs:
  read_timeout: 6
  connect_timeout: 7
  proxy_url: socks5://127.0.0.1:1080/
```

## 🚨 重要说明

### 安全提示
1. **Token 安全**：妥善保管 Bot Token，不要泄露给他人
2. **API 凭证**：Telegram API ID 和 Hash 仅用于自动群组创建功能
3. **代理安全**：使用可信的代理服务器，避免敏感信息泄露

### 功能限制
- 部分 Telegram 消息类型不支持（游戏、支付、投票等）
- 文件大小限制：发送 50MB，接收 20MB
- 无法处理来自其他 Bot 的消息

## 🆕 版本更新

### v2.x 增强功能
- ✅ 完整的 Pyrogram 客户端代理支持
- ✅ 自动创建群组功能优化
- ✅ 统一的代理配置管理
- ✅ 改进的网络连接稳定性

## 📄 许可证

本项目基于 GNU Affero General Public License 3.0 许可证开源。

---

**项目维护**：本分支专注于中国大陆用户的使用体验优化，包括网络连接问题解决和自动化功能增强。

**反馈与建议**：如有问题或建议，欢迎提交 Issue。