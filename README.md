# qq-bot
基于deepseek和qwen模型的qq机器人，可改人设，会盗图，会发图，可认为编写关系网



# NapCat‑QQ‑Bot
> 🤖 基于个人QQ号 + NapCatQQ(OneBot11) + DeepSeek 大模型的聊天机器人，支持看图理解、语义表情包、自动收藏表情、人物关系记忆、联网搜索、聊天日志记录。

[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![OneBot‑11](https://img.shields.io/badge/OneBot‑11-standard-green.svg)](https://onebot.dev/)
[![License](https://img.shields.io/badge/license-MIT‑orange.svg)](LICENSE)

> ⚠️ **重要声明**：本项目仅用于技术学习研究。使用个人QQ账号作为机器人存在账号风控、封禁风险，请自行承担全部使用风险；禁止用于骚扰、营销、牟利等违规场景。

## ✨ 功能特性
- 💬 **AI对话聊天**：接入 DeepSeek，支持会话记忆，记忆持久化保存，重启不丢失
- 🧠 **自动模式切换**：闲聊模式保持人设；遇到编程、数学、法律、事实查询自动切换**认真模式**输出严谨回答
- 🌐 **联网搜索**：遇到时效性、事实类问题自动调用Bing搜索获取资料再回答，无需额外API‑KEY
- 👤 **人物关系系统**：`people.json` 定义角色身份、备注、别名，识别好友备注、群名片，对不同人采用不同对话态度，内置管理员权限体系
- 🖼️ **图像理解**：接入Qwen‑VL视觉模型，识别图片内容，把图片转成文字上下文参与对话
- 🎭 **完整表情包体系**
  - 语义打分选表情包，根据对话情绪、场景自动匹配合适图片，情绪冲突时不强行发图
  - 自动收藏表情包：收到图片先AI判断是否表情包，语义去重后才入库
  - 批量打标脚本、图库清理脚本，方便维护本地表情包库
  - QQ原生表情face、超级表情mface语义识别
- 👥 **群聊增强**：群内旁听上下文，@才响应，限流防刷屏，冷却+每分钟最大回复数双重保护
- 📜 **聊天记录系统**：自动落盘JSONL日志，配套查看、检索、统计脚本
- ⚙️ **管理员指令**：状态查询、重载配置、刷新好友列表、清空会话记忆，无需重启程序
- 📝 **多段真人式回复**：一条回答拆分为多条消息气泡发送，模拟人类打字间隔；图文分离，表情包单独发送

## 🧱 系统架构
```
QQ客户端消息 ↔ NapCatQQ(OneBot11协议层) ↔ WebSocket ↔ bot.py
                                                         ↙ ↓ ↘
                                                DeepSeek聊天模型  通义VL视觉模型  Bing联网搜索
```

## 📋 前置依赖
1. Python >= 3.10
2. [NapCatQQ](https://github.com/NapNeko/NapCatQQ)：实现OneBot‑11协议，让个人QQ提供接口
3. DeepSeek API‑KEY（必填，对话核心）：前往 [DeepSeek开放平台](https://platform.deepseek.com/) 获取
4. 视觉模型KEY（可选，看图、表情包功能依赖）：阿里云百炼 DashScope Qwen‑VL系列

## 🚀 快速部署
### 1. 克隆项目
```bash
git clone [https://github.com/xxx/NapCat](https://github.com/xxx/NapCat)‑QQ‑Bot.git
cd NapCat‑QQ‑Bot
```

### 2. 安装Python依赖
```bash
pip install -r requirements.txt
```

### 3. NapCatQQ配置
1. 下载安装NapCatQQ，扫码登录你的QQ账号
2. WebUI打开网络配置，开启**正向WebSocket监听**
   - Host: `127.0.0.1`
   - Port: `3001`
   - Access Token 可选，设置后`.env`中需要同步填写

### 4. 环境配置
复制环境变量模板
```bash
# windows
copy .env.example .env
# linux/mac
cp .env.example .env
```
编辑 `.env`
```ini
DEEPSEEK_API_KEY=你的deepseek密钥
# 视觉功能，不需要可以留空
VISION_API_KEY=你的dashscope密钥
```

### 5. （可选）人物关系配置
复制示例配置
```bash
cp people.example.json people.json
```
编辑`people.json`填写角色、QQ号、身份描述，身份包含`创造者`自动拥有管理员权限。

### 6. 启动机器人
> 顺序：**先启动NapCatQQ，等待WebSocket服务启动成功，再运行bot.py**
```bash
python bot.py
```
看到 `已连接 ws://127.0.0.1:3001,机器人就绪。` 即为运行成功。

- 私聊：直接发送消息即可对话
- 群聊：需要`@机器人`才会回复

## 🎴 表情包使用
1. 将表情包图片放入`stickers/`目录，支持png/jpg/gif/webp
2. AI批量自动打标签（增量，仅处理未标记图片）
```bash
python label_stickers.py
```
- `--force`：全部重新打标覆盖旧标签
- `--limit 10`：仅处理前10张，用于测试
- `--dry-run`：预览待处理文件，不调用API

3. 清理图库中混入的照片、截图等非表情包
```bash
python clean_stickers.py        # 仅扫描报告，不会改动文件
python clean_stickers.py --apply # 移动非表情包至_rejected文件夹
```

## 📖 聊天日志查看
机器人会自动把聊天写入`chat_logs/*.jsonl`
```bash
python show_chat.py                     # 查看今日聊天
python show_chat.py --list              # 列出全部日志文件
python show_chat.py --date 2026‑01‑01   # 指定日期
python show_chat.py --grep 关键词       # 搜索关键词
python show_chat.py --stats             # 聊天数据统计
python show_chat.py --session group_123 # 指定群聊/私聊会话
```

## 🛠️ 管理员指令
> 群聊需要@机器人，私聊直接发送，不经过大模型，直接执行：

| 指令 | 功能 |
|---|---|
| `状态` | 查看表情包数量、好友数量、活跃会话统计 |
| `重载表情` | 重新加载表情包标签tags.json |
| `更新好友` | 强制刷新QQ好友备注列表 |
| `重载关系` | 重新读取people.json人物关系配置 |
| `清空记忆` | 清空当前会话的对话记忆 |

## 📂 项目文件说明
| 文件 | 说明 |
|---|---|
| `bot.py` | 机器人主程序 |
| `requirements.txt` | python依赖清单 |
| `.env.example` | 环境变量模板 |
| `people.example.json` | 人物关系配置模板 |
| `check_vision_key.py` | 视觉API密钥自检脚本 |
| `label_stickers.py` | 表情包AI批量打标 |
| `clean_stickers.py` | 表情包图库清理工具 |
| `show_chat.py` | 聊天记录查看分析工具 |
| `启停与运维.md` | 运维、排错完整文档 |

## ⚙️ 关键环境变量参考
| 变量 | 默认值 | 说明 |
|---|---|---|
| `MEMORY_SIZE` | 10 | 单会话最大记忆轮数 |
| `REPLY_MAX_PER_MIN` | 12 | 每分钟最大回复数量，防风控 |
| `REPLY_COOLDOWN_SEC` | 1.5 | 两次回复最小冷却秒数 |
| `SEARCH_ENABLED` | 1 | 是否开启联网搜索 |
| `CHAT_LOG_ENABLED` | 1 | 是否开启聊天记录落盘 |
| `ADMIN_QQ` | 空 | 额外管理员QQ，逗号分隔 |

## ❓ 常见问题
1. **一直打印重连**：NapCatQQ没有启动、正向WebSocket端口不是3001，检查端口连通性
2. **群聊不回复消息**：群聊必须@机器人才能触发响应
3. **看图/自动收藏表情无效**：检查`VISION_API_KEY`是否正确；运行`python check_vision_key.py`自检密钥与模型
4. **消息被吞**：触发限流，查看日志`[限流]`，可以适度调大`REPLY_MAX_PER_MIN`
5. **打标返回401鉴权失败**：视觉KEY与VISION_BASE_URL不匹配；公共dashscope密钥不要填专属端点地址

## 📌 安全提示
- `.env`、`people.json`、`state.json`、`chat_logs/`包含密钥、聊天、好友信息，**禁止上传到公开仓库**
- 如果API密钥发生泄露，请立刻前往对应平台作废旧密钥，生成新密钥
- 个人QQ号运行机器人极易触发风控，请勿高频刷屏、不要大量陌生群聊调用

## 📄 License
MIT License

> 本项目仅供学习交流，使用过程产生全部风险由使用者自行承担。
> 本项目与腾讯QQ、DeepSeek、阿里云无任何关联。

---

### 补充建议（你复制到github的时候）
1. 可以给仓库配一张演示截图，放在readme顶部
2. 把`启停与运维.md`放到仓库根目录，链接到readme里
3. 把`.env.example`放到仓库，不要上传真实`.env`文件，`.gitignore`配置忽略：
```
.env
state.json
people.json
stickers/_rejected
chat_logs/
__pycache__/
*.pyc
```

如果你需要，我还可以帮你写一份**github的issues模板、feature request模板**，或者精简一份短版README用于项目首页。
