# 生命线 · 网页版

《生命线》（Lifeline, 3 Minute Games / Big Fish Games）的同人网页移植：把原作的“消息式叙事”搬到浏览器里，纯静态、无后端、无追踪。

**线上地址：<https://yenchiu.com/lifeline-web/>**

## 嵌入到自己的网站

```html
<iframe
  src="https://yenchiu.com/lifeline-web/"
  title="生命线 · Lifeline"
  style="width:100%;max-width:520px;height:780px;border:0;border-radius:10px"
  loading="lazy"></iframe>
```

也可以直接给人发链接，或在站点里做一张入口卡片：

```html
<a href="https://yenchiu.com/lifeline-web/" target="_blank" rel="noopener">
  玩《生命线》网页版 →
</a>
```

## 结构

```
index.html            入口（开场 + 聊天界面）
css/app.css           深空主题样式
js/engine.js          DSL 解析与求值（无 DOM 依赖，可单测）
js/app.js             逐 token 执行、选项、存档、音频
data/story_cn.txt     中文剧本（来自原包 assets/StoryData_cn.txt）
data/strings_cn.xml   界面文案（来自原包 assets/strings_cn.xml）
assets/fonts/         LetterGothic 字体
assets/img/           背景、海报、边框、按钮、通知图标
assets/audio/         环境音与消息提示音
```

## 本地运行

```bash
cd lifeline-web
python -m http.server 8099
# 打开 http://127.0.0.1:8099/
```

也可以直接部署到 GitHub Pages：仓库根目录即站点根目录，无需构建。当前 Pages 分支为 `main`、路径 `/`。

## 实现要点

- **逐 token 执行**：把每个段落切成 `text / set / if / elseif / else / endif / silent / choices / link / input` 等 token。遇到选择停下来等玩家点击，遇到 `[[delay 30m|目标]]` 变成长等待（“快进”模式压缩到最多 4 秒，点击页脚可跳过），其余文本按打字节奏逐条出现。
- **变量与条件**：支持 `<<set $x = …>>`、`<<if $x == 1>>…<<elseif>>…<<else>>…<<endif>>`（含同一行内联写法）、`&&`/`and`、`||`/`or`、`=`/`==`/`eq`/`is`/`gt`/`gte`/`lt`/`lte` 比较；文本里的 `$变量` 会被替换，未知变量输出 `0`。
- **原文未实现的标记**：`<<textinput>>`、`<<sendemail>>`、`<<netcheck>>` 在原 APK 里没有处理代码（DEX 中 0 命中）。这里 `<<textinput>>` 提供输入框，`<<sendemail>>` 与 `<<netcheck>>` 被忽略——CN/ES/KO 版那段“Big Fish 邮件订阅 + 年龄核验”会走到“处理中”后停在 `[[gameover]]` 之前的空档，属于原作就残缺的分支。
- **存档**：localStorage 记录变量、当前段落与 token 下标、聊天记录，刷新后可“继续上次的故事”。
- **无后端**：不收集任何数据，字体、图片、音频全部本地加载。

## 存档与导入导出

进度全部保存在浏览器本地（localStorage），不上传任何服务器。顶部「存档」按钮打开存档面板：

- **自动存档**：每收到一条消息、每次变量变化都会写入，刷新或关掉页面后可用「继续上次的故事」恢复。
- **手动槽位**：3 个槽位，可保存 / 读取 / 删除，槽位行会显示当前段落、消息条数和保存时间。
- **导出文件**：下载 `lifeline-<时间>-<段落>.json`，包含变量、位置和完整聊天记录。
- **导入文件**：选择之前导出的 `.json` 即可恢复（含聊天记录）。
- **进度码**：`LL2z.` 开头的短码，压缩后约 500 字符，只含变量与位置、不含聊天记录，适合跨设备或发给别人接着玩；粘贴到文本框点「从进度码导入」即可。
- 旧的 v1 自动存档（`lifeline.web.save.v1`）仍可读取，会在下次写入时迁移为 v2 格式。
- 如果存档正好停在段落末尾（最后一条消息已显示、跳转还没发生），载入时会回退到该段最近的选项/跳转处，让玩家重新选择，而不是卡住。

存档文件结构：

```json
{
  "ver": 2,
  "app": "lifeline-web",
  "passage": "whois",
  "idx": 7,
  "vars": { "capalive": 1, "power": "none" },
  "log": [{ "k": "them", "t": "喂喂？" }],
  "savedAt": "2026-11-08T02:05:20.320Z"
}
```

## 已知限制

- 只有一个语言包（中文）。要加英文，把 `StoryData_en.txt` / `strings_en.xml` 放进 `data/`，在 `app.js` 的 `boot()` 里改成按 `?lang=` 选择即可。
- 原作的 Wear 手表端、通知栏推送、后台定时排程都没有对应实现；本作把“等待”放在前台页面里。
- 移动端 Safari 的自动播放限制：环境音在第一次点击后开始，静音按钮可随时关闭。

## 版权

剧情、人物、美术、音频版权归 3 Minute Games LLC / Big Fish Games 所有。本仓库是非商业同人移植，仅供学习与个人体验，不包含破解、去广告或内购绕过。若版权方提出异议，会立即下架。
