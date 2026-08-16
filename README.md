# 📚 出版刷题（Publishing Exam Prep）

**出版专业技术人员职业资格考试（初级）刷题软件** —— 手机 App（Android APK）+ 网页版（PWA）双端支持，内置 2013–2019、2021、2022 年真题与 2026 年教材新增知识点新编题，支持在线更新。

> ⚠️ 本项目为个人学习工具，仅供个人学习使用。**未经作者允许，不得用于商业用途，不得二次转载、二次发布。**

---

## ✨ 功能一览

| 功能 | 说明 |
| --- | --- |
| 双科目题库 | 出版专业基础（1066 题）/ 出版专业实务（1131 题） |
| 知识点分类 | 按章 → 节 → 知识点组织，可按知识点选题 |
| 选题 / 背题模式 | 选题模式答题判分；背题模式直接看答案记忆 |
| 加权出题系统 | 错题权重更高、已掌握题目权重降低，智能复习 |
| 历年试卷 | 按年份做整套真题（2013–2019、2021、2022），按题号顺序 |
| 真题浏览 | 全量浏览 + 年份/类型/关键字筛选 + 收藏 + 纠错反馈 |
| 错题本 / 收藏 | 自动收录错题，可重做；题目可收藏 |
| 刷题历史 | 正确率趋势、用时统计、答对/答错/半对统计 |
| 未完成练习 | 中途退出自动保存，首页/历史一键继续（离开期间不计时） |
| 深浅主题 | 深色 / 浅色 / 跟随系统 |
| 在线更新 | App 内检查更新：题库增量同步 + APK 自动下载（系统下载器） |
| 反馈直达 | 用户纠错/备注 → Cloudflare Worker → GitHub Issues |

## 📱 使用方法（普通用户）

1. 直接安装 APK：`出版刷题-vX.X.apk`（Android 7.0+，首次安装需允许"安装未知应用"）
2. 或浏览器打开网页版（PWA，可"添加到主屏幕"）
3. 打开后无需任何配置：题库内置、更新源内置、自动检查更新

## 🖥 开发 / 部署方法

### 环境要求

- Node.js ≥ 20（本地服务器 / 构建脚本）
- Python 3 + 依赖（题库数据处理，可选）
- JDK 21 + Android SDK（构建 APK 用）
- PowerShell（Windows 构建脚本）

### 本地运行（网页版 + 数据接口）

```bash
cd app
node server.js        # 默认 https://127.0.0.1:8443（自签名证书）
```

### 构建 Android APK

```powershell
powershell -ExecutionPolicy Bypass -File E:\文档刷题\app\scripts\build-apk.ps1 -Version 2.5 -Notes "更新说明，多行用 `n"
```

构建产物：`出版刷题-vX.X.apk`（同时生成 `app/public/version.json` 与 `app/public/download/*.apk`）。

### 发布更新站点（GitHub Pages）

```powershell
powershell -ExecutionPolicy Bypass -File E:\文档刷题\app\scripts\deploy-site.ps1          # 生成 update-site/ 目录
powershell -ExecutionPolicy Bypass -File E:\文档刷题\app\scripts\deploy-github.ps1 -Version 2.5  # 上传到 GitHub Pages
```

更新站点结构（静态托管，任意静态空间均可作为更新源）：

```
version.json            # 版本信息：appVersion / apkUrl / notes / bank(各文件 sha1+size)
data/knowledge.json     # 知识树
data/questions-jc.json  # 基础题库（含解析、审核状态）
data/questions-sw.json  # 实务题库
download/*.apk          # 安装包
```

App 启动/手动检查更新时：拉取 `version.json` → 对比 `bank` 的 sha1 → 变化则下载新题库存入 IndexedDB；`appVersion` 不同则提示下载新 APK。

## 🔌 接口说明（本地服务器 `server.js`）

| 接口 | 方法 | 说明 |
| --- | --- | --- |
| `/api/state` | GET/POST | 用户数据（历史/错题/收藏/设置/qstats）同步 |
| `/api/questions?subject=jc\|sw[&includeDrafts=1]` | GET | 题库（含审核合并结果） |
| `/api/knowledge` | GET | 知识树 |
| `/api/bank?name=...` | GET | 题库静态副本（更新用） |
| `/api/app-version` | GET | 版本信息（等同 version.json） |
| `/api/review` | POST | 审核/纠错/备注（`{qid, action, note}`） |
| `/api/feedback` | GET | 查看收到的反馈（作者用） |
| 静态文件 | GET | `public/` 目录（App 前端 + 题库 + APK） |

> 用户反馈公网链路：App → `DEFAULT_FEEDBACK_ENDPOINT`（Cloudflare Worker）→ GitHub Issues API → 仓库 Issues。

## 🔧 其他人拿到源码，需要修改什么才能正常使用

1. **`app/public/app.js`** 顶部常量：
   - `APP_VERSION`：与 `version.json` 保持一致（构建脚本自动更新）
   - `DEFAULT_UPDATE_BASE`：改成你自己的更新源地址（如 `https://你的用户名.github.io/你的仓库名`）
   - `DEFAULT_FEEDBACK_ENDPOINT`：改成你自己的反馈接收地址（Cloudflare Worker 或其它 POST 端点）
2. **`app/scripts/deploy-github.ps1`**：
   - `$token`：换成你自己的 GitHub fine-grained token（仅授权更新仓库 + `Contents: Read and write`，**切勿使用全权限 token，切勿提交到公开仓库**）
   - `$owner` / `$repo`：你的 GitHub 用户名 / 更新仓库名
3. **反馈 Worker**（Cloudflare）：
   - 环境变量 `GH_OWNER` / `GH_REPO` / `GH_TOKEN`（fine-grained token，仅 `Issues: Read and write`）
   - Worker 代码见 `feedback-worker.js`
4. **`app/capacitor.config.json`**：`appId`（包名）改成你自己的，如 `com.你的域名.出版刷题`
5. **`app/android/app/build.gradle`**：`applicationId` 与 `appId` 一致（构建脚本自动更新 versionCode/versionName）
6. **题库数据**：替换 `app/public/data/*.json`（或重新运行 `pipeline/` 数据处理流程生成）
7. **本地服务器端口**：`app/server.js` 中的端口（默认 8443），改端口后同步改防火墙/部署配置

## ⚠️ 注意事项

- **Token 安全**：任何 GitHub token 一旦出现在公开仓库（包括 APK 等二进制文件内），GitHub 的 secret scanning 会自动吊销。因此反馈 token 必须放在云端（Worker 环境变量），**绝不写进 App 代码或仓库文件**。
- **更新源变更**：更换更新源地址后，旧版本 App 内置的旧地址将无法再检查更新，需重新安装新版。
- **自签名证书**：本地服务器使用自签名 HTTPS 证书，浏览器首次访问需手动信任。
- **题库说明**：真题为扫描件 AI 转录，可能存在个别转录误差；解析中 AI 补写部分已在题库内标注。发现错误可在 App「真题浏览 → 纠错/备注」提交反馈。
- **数据备份**：App 内「我的 → 导出备份」可导出全部学习数据（历史/错题/收藏），建议定期导出。
- **版权**：题库内容版权归原作者/出版方所有，仅供个人学习交流。**未经作者允许，本软件不得用于商业用途，不得二次转载、二次发布、二次打包分发。**

## 📦 更新历史

### v2.5
- 项目仓库更名与 README 完善
- 更新源地址迁移（旧地址自动跳转）

### v2.4
- 反馈接收改为云端转发（Cloudflare Worker → GitHub Issues），不再内置 token
- 上传失败提示具体原因，支持复制反馈手动转发

### v2.3
- 修复：继续答题后计时从离开时刻继续（离开期间不计时）
- 更新说明逐条展示

### v2.2
- 新增：未完成练习自动保存，首页/历史一键继续
- 新增：历年试卷（按年份整套真题）
- 真题浏览加入 2026 新编题（标注非真题），纠错反馈直达作者
- 修复：App 内更新下载改用系统下载器，不再跳转浏览器

### v2.1
- 修复：新编题无法在真题浏览显示（数据文件损坏问题）
- 修复：更新下载监听器挂载时机（不再跳转浏览器）

### v2.0
- 反馈链路打通：纠错/备注 → GitHub Issues
- 真题浏览卡片直接展示选项/答案/解析 + 星标收藏
- 移除新编题审核入口

## 🙏 致谢

- 题库数据基于历年考试真题整理（AI 转录 + 人工校对）
- 感谢所有通过"纠错/备注"提交反馈的热心用户
