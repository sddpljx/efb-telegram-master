# EFB Telegram Master 代理配置增强

## 概述

本文档记录了为 EFB Telegram Master 添加 Pyrogram 客户端代理支持的修改，以及如何配置自定义 API 域名的完整指南。

## 问题背景

EFB Telegram Master 使用了两个不同的 Telegram 客户端：

1. **主要功能**：使用 `python-telegram-bot` 库（已支持代理）
2. **自动创建群组功能**：使用 `pyrogram` 库（原本不支持代理）

当用户在环境变量中设置代理后，自动创建群组功能仍然无法通过代理访问 Telegram 服务器。

## 解决方案

### 1. Pyrogram 客户端代理支持

#### 修改的文件
- `efb_telegram_master/auto_tg_manager.py`

#### 修改内容

**原代码：**
```python
self.tg_client = pyrogram.Client(name='efb_telegram_auto_create_group_client',
                                 api_id=self.tg_config.get('tg_api_id'),
                                 api_hash=self.tg_config.get('tg_api_hash'),
                                 workdir=ehforwarderbot.utils.get_data_path(channel.channel_id))
```

**修改后：**
```python
# 构建 Pyrogram 客户端参数
client_params = {
    'name': 'efb_telegram_auto_create_group_client',
    'api_id': self.tg_config.get('tg_api_id'),
    'api_hash': self.tg_config.get('tg_api_hash'),
    'workdir': ehforwarderbot.utils.get_data_path(channel.channel_id)
}

# 添加代理支持
proxy_config = self.tg_config.get('pyrogram_proxy')
if proxy_config:
    proxy_type = proxy_config.get('type', '').lower()
    if proxy_type == 'http':
        client_params['proxy'] = {
            'scheme': 'http',
            'hostname': proxy_config.get('hostname'),
            'port': proxy_config.get('port'),
            'username': proxy_config.get('username'),
            'password': proxy_config.get('password')
        }
    elif proxy_type == 'socks5':
        client_params['proxy'] = {
            'scheme': 'socks5',
            'hostname': proxy_config.get('hostname'),
            'port': proxy_config.get('port'),
            'username': proxy_config.get('username'),
            'password': proxy_config.get('password')
        }
        
# 从字典中移除 None 值
if 'proxy' in client_params and client_params['proxy']:
    client_params['proxy'] = {k: v for k, v in client_params['proxy'].items() if v is not None}
    if not client_params['proxy'].get('hostname') or not client_params['proxy'].get('port'):
        self.logger.warning("Pyrogram proxy configuration incomplete, proxy disabled.")
        client_params.pop('proxy')
        
self.tg_client = pyrogram.Client(**client_params)
```

#### 配置示例更新

**修改的文件：**
- `README.rst`

**添加的配置示例：**
```yaml
flags:
  auto_manage_tg_config:
    # 其他现有配置...
    auto_manage_tg: true
    tg_api_id: 12345
    tg_api_hash: xxxxxxxxxxxxxxxxxxxxxxxxx
    bot_name: '@bot_name'
    # 新增：Pyrogram 代理配置
    pyrogram_proxy:
      type: socks5  # or http
      hostname: 127.0.0.1
      port: 1080
      # Optional authentication
      username: proxy_user
      password: proxy_pass
```

### 2. API 域名代理配置

EFB Telegram Master 已经内置支持自定义 API 域名，无需修改代码。

#### 现有功能
- `api_base_url`: 自定义 Telegram Bot API 基础 URL
- `api_base_file_url`: 自定义 Telegram Bot API 文件 URL
- `request_kwargs`: 支持 HTTP/SOCKS5 代理配置

#### 配置方法

```yaml
# ~/.ehforwarderbot/profiles/default/blueset.telegram/config.yaml

token: "你的bot token"
admins:
- 你的用户ID

# 方案1：自定义 API 域名
flags:
  api_base_url: "https://你的代理域名/bot"
  api_base_file_url: "https://你的代理域名/file/bot"

# 方案2：使用代理
request_kwargs:
  # HTTP 代理
  proxy_url: http://127.0.0.1:8080/
  # 或者 SOCKS5 代理
  # proxy_url: socks5://127.0.0.1:1080/
  # 可选：代理认证
  # username: proxy_user
  # password: proxy_pass

# 方案3：结合使用
flags:
  api_base_url: "https://你的代理域名/bot"
request_kwargs:
  proxy_url: socks5://127.0.0.1:1080/
```

## 完整配置示例

### 基本配置
```yaml
token: "012345678:1Aa2Bb3Vc4Dd5Ee6Gg7Hh8Ii9Jj0Kk1Ll2M"
admins:
- 102938475

# 自定义 API 域名（可选）
flags:
  api_base_url: "https://tg-api.example.com/bot"
  api_base_file_url: "https://tg-api.example.com/file/bot"

# 主要功能代理设置
request_kwargs:
  proxy_url: socks5://127.0.0.1:1080/
```

### 自动创建群组 + 代理配置
```yaml
token: "012345678:1Aa2Bb3Vc4Dd5Ee6Gg7Hh8Ii9Jj0Kk1Ll2M"
admins:
- 102938475

flags:
  # 自动创建群组配置
  auto_manage_tg_config:
    auto_manage_tg: true
    tg_api_id: 12345
    tg_api_hash: "your_api_hash_here"
    bot_name: '@your_bot_name'
    auto_create_tg_group: [1, 2]  # 私聊和群聊
    auto_mute_created_tg_group: [2]  # 静音群聊
    
    # Pyrogram 代理配置
    pyrogram_proxy:
      type: socks5
      hostname: 127.0.0.1
      port: 1080
      # username: proxy_user  # 可选
      # password: proxy_pass  # 可选

# 主要功能代理设置
request_kwargs:
  proxy_url: socks5://127.0.0.1:1080/
```

## 使用说明

### 1. 配置 Pyrogram 代理

1. 确保自动创建群组功能已启用（`auto_manage_tg: true`）
2. 在 `auto_manage_tg_config` 中添加 `pyrogram_proxy` 配置
3. 支持 HTTP 和 SOCKS5 代理类型
4. 用户名和密码为可选配置

### 2. 配置 API 域名代理

1. **域名代理**：使用 `api_base_url` 和 `api_base_file_url`
2. **传统代理**：使用 `request_kwargs.proxy_url`
3. **组合使用**：可以同时配置域名和代理

### 3. 注意事项

- 重启 EFB 服务后配置生效
- 代理配置错误会在日志中显示警告
- 支持代理认证（用户名/密码）
- 可以分别为不同功能配置不同的代理策略

## 测试验证

1. **主要功能测试**：发送消息，检查是否通过代理
2. **自动创建群组测试**：收到新消息时检查群组创建是否通过代理
3. **日志检查**：查看是否有代理相关的错误信息

## 兼容性

- 支持 EFB 2.x 版本
- 需要 `pyrogram` 库支持
- 支持 HTTP 和 SOCKS5 代理协议
- 向后兼容现有配置

---

**修改日期：** 2025-08-24  
**修改内容：** 为 Pyrogram 客户端添加代理支持，完善 API 域名代理配置文档