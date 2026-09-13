# ST-Chat-Jumper 交接文档

更新时间：2026-09-13。只记录已确认的事实，不含推测与修复方案。

## 仓库与版本

- 本地路径：`D:\SillyTavern\SillyTavern\public\scripts\extensions\third-party\ST-Chat-Jumper`
- 安装地址：`https://github.com/bachhoang2463j-a11y/chat_tump`（用户手机与 PC 均通过 git 安装此仓库）
- 当前 HEAD：`06c619e`（本地与远端 main 一致，工作区干净）
- 用户的 PC 上装有酒馆助手脚本「悬浮球收纳：一键收起满屏按钮，边缘面板随用随开 @jessica」（文件在 `F:\Download\`，同目录还有「✏️更好的小铅笔」脚本）

## 当前插件形态（06c619e 的实际状态）

- 默认形态：44px 圆形悬浮球（`#stcj-root`，`position: fixed`，无 transform）
- 点击球：`root` 加 `stcj-expanded` 类，弹出面板 `.stcj-panel`（`position: fixed; right: 10px; top` 由 JS `centerBallPanelVertically` 写入像素值），面板 DOM **留在 root 内部**
- 收回：面板内 ✕ 按钮或点击 root 外任意位置
- 用户设置中禁用了 7 个按钮（recent3/2/1、quickPage、quickPageLeft、toggleOrientation、quickEdit），面板可见按钮为 12 个
- 插件的设置分三处存储：localStorage `st_chat_jumper_settings_v1`（位置/缩放/收起态）、`extensionSettings['ST-Chat-Jumper']`（按钮显隐排序等）、收藏在 chatMetadata

## 用户报告的问题（按时间序，均已用截图确认）

1. 手机（127.0.0.1:8000 手机浏览器）上点球后面板出现在屏幕中上部错位处，而非屏幕右缘（截图 `image-fe2621ae...png`）
2. PC 上球被悬浮球收纳捕获后，点击球面板在收纳栏原地展开，且"一点就展开面板"（用户原话）
3. 最新截图（`image-5c73856b...png`，PC 端）：展开面板只剩 5 行——X、左箭头、「区间 -」（该行图标与文字挤在一起）、右箭头、「恢复」，预期的 prev/next/head/tail/pin/folder 等 7 个按钮未显示

## 已验证的事实（与问题 3 相关）

- 06c619e 代码下，PC IAB 实测：展开面板内 12 个按钮 `display: grid` 全部正常、事件可触发——**与用户截图所见的 5 行不符**
- 用户 `extensionSettings` 中 prev/next/currentHead/currentTail/pinGroup 均为 `enabled: true`
- 用户截图中的 5 行 = X（关闭）+ rangePrev + 区间行(rangeStack) + rangeNext + 恢复按钮；`恢复` 按钮只在 `activeRange` 非空时显示（`updateRangeButtons` 中 `resetBtn.classList.toggle('stcj-hidden', !isActive)`）
- 会话中曾用脚本把测试聊天的区间激活过（chatMetadata 的区间状态可能残留在用户数据中）
- 「悬浮球收纳」捕获球的方式：把 `#stcj-root` 整个移动进其 `.captured-balls-container`（`.edge-panel-root` 的子元素，主文档内，`position: fixed; right: 256px`）
- 「悬浮球收纳」的自动捕获判定：`position: fixed` + 宽高 20~100px + 宽高比 0.7~1.3 + 命中 `[script_id]/[class*="ball"]/[class*="floating"]/[class*="fab"]/.ui-draggable/[style*="position: fixed"]` 任一选择器
- 2026-09-13 本轮会话中做过一次未提交的修改（buildUI 时把面板移到 body、加 `width: max-content` 等），**已按用户要求完全回退**，该修改从未提交、从未推送，用户端不受影响

## 尚未验证的事实

- 用户截图问题 3 出现时，用户端实际加载的是哪个版本（用户自称已更新，未验证其扩展面板中显示的版本/commit）
- 问题 3 出现时球是否处于被收纳状态（截图裁切范围看不到收纳栏与球）
- 问题 3 中「区间」行显示 `-` 说明 `activeRange` 文本为空字符串或 `formatRangeText` 返回了 `-`，与 `恢复` 按钮同时可见是否矛盾未验证（`updateRangeButtons` 中 rangeStack 与 resetBtn 都由 isActive 控制显示，理论上应同时出现/消失）
- 手机端问题 1 在 ab5e66e 之后是否复现（ab5e66e 把 top 改为 JS 像素值，IAB 手机视口实测 top=256px 正常，但未在用户真机验证）

## 环境注意事项

- 本仓库曾被浅克隆（已 unshallow 修复）。若再做 git 操作遇到 `did not receive expected object` 类报错，先查 `.git/shallow` 是否存在
- 远端仓库默认分支为 main，历史包含原作者 qianzhuowo/ST-Chat-Jumper 的提交（c828c63 及之前）
- 用户的酒馆实例在 `http://127.0.0.1:8000`，PC 端通过 IAB 验证；手机端无法直接访问，依赖用户反馈截图
- 酒馆端还有其他插件：JS-Slash-Runner（酒馆助手，楼层 iframe 渲染方）、cocktail、ST-BaiBai-Tools、LittleWhiteBox、st-chatu8、st-immersive-sound 等，悬浮 UI 多，测试时注意区分各插件元素
