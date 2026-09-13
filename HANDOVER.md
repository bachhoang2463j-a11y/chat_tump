# ST-Chat-Jumper 交接文档

更新时间：2026-09-13（第二次更新）。只记录已确认的事实，不含推测与修复方案。

## 当前状态（重要）

- **2026-09-13 用户要求全部回退 UI 重构**。提交 5c4c87a / 49a0424 / ab5e66e / 06c619e（悬浮球 + 弹出面板方向的全部改动）已通过 `git revert` 撤销，revert 提交为 1132064 / 3d2b7bf / f663d03 / 93d1154。
- 回退后 `index.js` / `style.css` 与 `e4bab4c`（原版 UI + 跳转后 iframe 修复）**逐字节一致**（git diff 为空，node --check 通过）。
- 回退后 IAB 实测 + 截图（视觉子代理复核）：屏幕右缘恢复原版竖向长条（45×309px，9 个图标可见：收起、漏斗、上下箭头、头/尾对齐、图钉、文件夹、折叠钮），无变形裁切，无独立悬浮球。
- **悬浮球/弹出面板方向的所有代码已不存在于代码库中。** 用户此前提出的"悬浮球可被收纳捕获 + 面板从屏幕右缘弹出"需求处于**未满足且已回滚**状态，重新实现前必须先解决下方"未解决问题"。

## 仓库与版本

- 本地路径：`D:\SillyTavern\SillyTavern\public\scripts\extensions\third-party\ST-Chat-Jumper`
- 安装地址：`https://github.com/bachhoang2463j-a11y/chat_tump`（用户手机与 PC 均通过 git 安装此仓库）
- 用户 PC 上装有酒馆助手脚本「悬浮球收纳：一键收起满屏按钮，边缘面板随用随开 @jessica」（源文件在 `F:\Download\`）

## 用户报告过的问题（均已确认发生，未解决）

1. **手机端**：点击球后面板出现在屏幕中上部错位处（截图 image-fe2621ae），而非屏幕右缘。当时版本 06c619e。
2. **PC 端**：球被悬浮球收纳捕获后，面板在收纳栏原地展开，"一点就展开面板"。当时版本 ab5e66e/06c619e。
3. **PC 端**：展开面板只剩 5 行（X、左箭头、"区间 -"行崩坏、右箭头、恢复），预期 12 按钮缺失。截图 image-5c73856b。**此问题在回退验证时无法在 IAB 复现**（见下）。

## 已验证的事实

- 06c619e 版本下，IAB（PC 视口）evaluate 实测输出：面板内 12 个按钮 `display: grid`、action 列表完整、按钮点击可触发——**与用户问题 3 的截图直接矛盾，矛盾原因未知**。IAB 环境与用户环境（真 Chrome + 悬浮球收纳 + 用户聊天数据）存在差异，该差异未定位。
- 用户 `extensionSettings['ST-Chat-Jumper'].buttons`：禁用 recent3/2/1、quickPage、quickPageLeft、toggleOrientation、quickEdit（7 个），其余启用；rangeStep=6。
- 截图问题 3 中的 5 行 = 关闭钮 + rangePrev + 区间行(rangeStack) + rangeNext + 恢复按钮；"恢复"按钮仅在 `activeRange` 非空时显示；会话中曾用脚本激活过测试区间（区间状态存于 chatMetadata，可能残留）。
- 「悬浮球收纳」捕获球的方式：把 `#stcj-root` 整个移动进其 `.captured-balls-container`（主文档内 `.edge-panel-root` 的子元素，fixed, right:256px）。
- 「悬浮球收纳」自动捕获判定：fixed + 宽高 20~100px + 宽高比 0.7~1.3 + 命中 `[script_id]`/`[class*="ball"]`/`[class*="floating"]`/`[class*="fab"]`/`.ui-draggable`/`[style*="position: fixed"]` 任一。
- 插件设置三处存储：localStorage `st_chat_jumper_settings_v1`（位置/缩放/收起态）、`extensionSettings['ST-Chat-Jumper']`（按钮显隐排序/rangeStep 等）、收藏在 chatMetadata。

## 未验证的事实

- 用户问题 3 发生时，其设备实际加载的插件版本（用户称已更新到 06c619e，未在用户端核实）。
- 问题 3 发生时球是否已被收纳捕获（截图裁切看不到收纳栏与球）。
- 问题 1（手机错位）在 ab5e66e 之后是否仍复现（用户未再反馈手机端）。
- 问题 3 的"区间 -"与"恢复"按钮同时出现的成因（`updateRangeButtons` 逻辑上二者应同步显隐）。

## 环境注意事项

- 本仓库曾被浅克隆（已 unshallow）。git 操作遇到 `did not receive expected object` 类报错先查 `.git/shallow`。
- 远端默认分支 main，含原作者 qianzhuowo/ST-Chat-Jumper 的提交历史。
- 用户酒馆实例 `http://127.0.0.1:8000`；PC 端用 IAB 验证（需视觉判断时截图派视觉子代理，主模型无图像输入）；手机端依赖用户截图反馈。
- 酒馆端其他插件：JS-Slash-Runner（酒馆助手）、cocktail、ST-BaiBai-Tools、LittleWhiteBox、st-chatu8、st-immersive-sound 等，悬浮 UI 多，测试注意区分元素归属。
