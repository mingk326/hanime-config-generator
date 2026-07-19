# Hanime config.yaml 生成器

一个纯前端可视化工具，用于生成 [hanime-dl](https://github.com/mingjiezxc/hanime-dl) 的 `config.yaml` 配置文件，同时可参数化生成配套的 Python 抓取脚本 `hanime_get_code.py`。

整个工具是一个独立的 HTML 文件，不依赖任何后端、CDN 或外部库。双击打开即可使用，所有数据在浏览器本地处理。

![license](https://img.shields.io/badge/license-MIT-blue)
![size](https://img.shields.io/badge/size-42KB-brightgreen)
![deps](https://img.shields.io/badge/dependencies-zero-success)
![preview](https://img.shields.io/badge/preview-online-success)

## 在线预览

无需下载，直接在浏览器中使用：

**[https://hanime-config-generator.1768432482.workers.dev](https://hanime-config-generator.1768432482.workers.dev)**

预览站点通过 Cloudflare Workers 部署，与仓库 `index.html` 内容同步。打开后即可使用全部功能，数据仍在本地处理，不上传到任何服务器。

## 功能概览

工具围绕 hanime-dl 的完整工作流设计，分为三个环节：

1. **生成抓取脚本** — 参数化生成 `hanime_get_code.py`，用 Playwright 抓取视频 ID 列表
2. **管理视频 ID** — 导入 `SingleCode.txt`、手动添加、批量编辑
3. **生成 config.yaml** — 与 hanime-dl 官方字段完全对齐的配置文件

## 生成 config.yaml

配置文件字段与 [hanime-dl 官方 config.yaml](https://github.com/mingjiezxc/hanime-dl#配置) 完全对齐，共 10 个字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `chromeRemoteURL` | string | Chrome 远程调试 URL |
| `CacheDir` | string | 缓存目录 |
| `DownDir` | string | 下载目录 |
| `HttpProxy` | string | HTTP 代理（可选，留空不输出） |
| `DirectDownloadFirst` | bool | 是否优先尝试直接下载 |
| `MaxDownloadWorkers` | int | 最大并发下载线程数 |
| `ListCode` | []string | 播放列表 ID 列表 |
| `SingleCode` | []string | 单个视频 ID 列表 |
| `ClearCache` | bool | 下载后清除缓存 |
| `VideoResolution` | string | 视频分辨率 |

输出采用 YAML plain scalar 风格，字符串默认不加引号，仅在包含特殊字符时用单引号包裹，避免 Windows 路径反斜杠转义问题：

```yaml
chromeRemoteURL: http://localhost:9222/json/version

CacheDir: E:\ai work\cache
DownDir: H:\
VideoResolution: 1080p
MaxDownloadWorkers: 3
DirectDownloadFirst: true
SingleCode:
- 407112
- 407213
- 407240
- 407324234
ClearCache: true
```

## 生成 hanime_get_code.py

通过表单参数化生成完整的 Playwright 抓取脚本，避免每次手动改代码。

可配置项：

- 起始页 / 结束页
- 排序方式（最新上片 / 最多點擊 / 最多收藏 / 最高評分 / 最多發表）
- 输出文件路径（填目录或 `.txt` 均可，目录会自动补全 `SingleCode.txt`）
- Chrome 可执行文件路径
- 请求间隔、页面超时、Cloudflare 验证等待时间
- User-Agent
- 无头模式 / tqdm 进度条 / 首页手动确认 三个开关

生成的脚本相比原版的改进：

- 所有硬编码值提取为顶部常量，便于修改
- 路径使用 raw string（`r"..."`）避免反斜杠转义问题
- 运行时路径自愈：若 OUTPUT 是目录，自动 `os.path.join` 补全文件名
- 写文件前自动创建不存在的父目录
- 用 `main()` 函数 + `if __name__ == "__main__"` 结构化封装

## ID 管理

工具提供多种方式导入和管理视频 ID：

- **文件导入** — 选择或拖拽 `SingleCode.txt`
- **粘贴解析** — 兼容 `SingleCode:` 头、`- xxx` 列表、逗号/空格/换行混排等多种格式
- **手动添加** — 多个 ID 用任意分隔符
- **列表操作** — 搜索、升降序、去重、复制、单条删除

ListCode（播放列表 ID）单独配置，不与 SingleCode 混淆。

## 配置预设

支持将当前所有配置（包括 ID 列表）保存为命名预设，存储在浏览器 localStorage 中。下次打开工具时一键加载，免去重复填写路径和参数。

预设场景示例：

- 不同下载位置（本地磁盘 vs NAS 仓库）
- 不同分辨率组合（1080p 本地 vs 720p NAS）
- 不同并发数（白天高并发 vs 夜间低并发）

## 使用方法

### 在线使用

推荐访问 Cloudflare Workers 预览站点（与仓库内容同步）：

**[https://hanime-config-generator.1768432482.workers.dev](https://hanime-config-generator.1768432482.workers.dev)**

或者直接访问 raw 文件地址（无样式渲染需手动保存为 .html 后打开）：

```
https://raw.githubusercontent.com/mingk326/hanime-config-generator/main/index.html
```

将页面保存到本地后双击打开即可。

### 本地使用

```bash
git clone https://github.com/mingk326/hanime-config-generator.git
cd hanime-config-generator
# 双击 index.html 在浏览器中打开
```

推荐使用 Chrome 或 Edge，File System Access API 需要安全上下文（HTTPS 或 localhost）。

### 典型工作流

1. 在「④ 生成 hanime_get_code.py」卡片配置抓取参数，生成并下载 `.py` 脚本
2. 运行脚本得到 `SingleCode.txt`（包含视频 ID 列表）
3. 在「① ID 管理」卡片导入 `SingleCode.txt`
4. 在「② 配置项」卡片填写 CacheDir、DownDir 等路径
5. 点击「生成 config.yaml」，下载并放到 hanime-dl 同目录
6. 运行 `./hanime-dl` 开始下载

## 技术细节

### YAML 生成

工具不依赖任何 YAML 库，通过字符串拼接生成。`yamlVal()` 函数根据 YAML plain scalar 规则智能判断何时必须加引号：

- 空字符串 → `''`
- 以特殊字符开头（`-`、`?`、`:`、`[`、`{`、`#`、`&` 等）→ 单引号包裹
- 含 `: `（冒号+空格）或 ` #`（空格+井号）→ 单引号包裹
- 是 YAML 保留字（true/false/null/yes/no）→ 单引号包裹
- 其他情况 → 不加引号

这样既保证了输出的简洁性，又避免了 YAML 解析错误。经过 Python `yaml.safe_load` 验证，所有字段类型与官方定义一致。

### Python 脚本生成

生成的 Python 代码经过 `python -m py_compile` 语法检查，确保可直接运行。脚本内嵌路径自愈逻辑，即使用户误填目录路径也能正常输出文件。

### 数据安全

所有数据在浏览器本地处理，不上传任何信息到服务器。预设数据存储在 localStorage，清除浏览器数据会同步清除预设。

## 浏览器兼容性

| 浏览器 | 支持情况 | 备注 |
|--------|----------|------|
| Chrome 86+ | 完整支持 | File System Access API 可用 |
| Edge 86+ | 完整支持 | File System Access API 可用 |
| Firefox | 基本支持 | 目录浏览回退到 webkitdirectory |
| Safari | 基本支持 | 目录浏览回退到 webkitdirectory |

## 许可证

MIT License

## 致谢

- [hanime-dl](https://github.com/mingjiezxc/hanime-dl) — 本工具生成的配置文件供其使用
- [Playwright](https://playwright.dev/) — 生成的抓取脚本基于其 API
