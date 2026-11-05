# 生命线 · 网页版

《生命线》（Lifeline, 3 Minute Games / Big Fish Games）的同人网页移植：把原作的“消息式叙事”搬到浏览器里，纯静态、无后端、无追踪，可直接托管在 GitHub Pages。

## 结构

```
index.html            入口（开场 + 聊天界面）
css/app.css           深空主题样式
js/engine.js          DSL 解析与求值（无 DOM 依赖，可单测）
js/app.js             逐段执行、选项、存档、音频
data/story_cn.txt     中文剧本（来自原包 assets/StoryData_cn.txt）
data/strings_cn.xml   界面文案（来自原包 assets/strings_cn.xml）
assets/fonts/         LetterGothic 字体
assets/img/           背景、边框、按钮、通知图标
assets/audio/         环境音与消息提示音
```

## 运行

```bash
cd lifeline-web
python -m http.server 8099
# 打开 http://127.0.0.1:8099/
```

也可以直接部署到 GitHub Pages：仓库根目录即站点根目录，无需构建。

## 实现要点

- **逐段执行**：解析 `:: 段落名`，遇到 `<<choice …>>` 停下来等玩家选择，遇到 `[[delay 30m|目标]]` 真实等待（“快进”模式会压缩到最多 4 秒），其余文本按打字节奏逐条出现。
- **变量与条件**：支持 `<<set $x = …>>`、`<<if $x == 1>> <<elseif>> <<else>> <<endif>>`、`&&`/`||`、`=`/`gt`/`lt` 等比较；文本里的 `$变量` 会被替换。
- **原文未实现的标记**：`<<textinput>>`、`<<sendemail>>`、`<<netcheck>>` 在原 APK 里没有处理代码（DEX 中 0 命中）。这里 `<<textinput>>` 提供了输入框，邮件发送与联网核验不实现——CN/ES/KO 版那段“Big Fish 邮件订阅 + 年龄核验”流程会走到“处理中”然后无后续。
- **存档**：localStorage 记录变量、当前段落与滚动记录，刷新后可继续。
- **无后端**：不收集任何数据，音频与字体均本地加载。

## 版权

剧情、人物、美术、音频版权归 3 Minute Games LLC / Big Fish Games 所有。本仓库是非商业同人移植，仅供学习与个人体验，不包含破解、去广告或内购绕过。若版权方提出异议，会立即下架。
