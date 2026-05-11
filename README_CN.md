# drawio-skill — From Text to Professional Diagrams

[English](README.md) | [Online Docs](https://agents365-ai.github.io/drawio-skill/)

## 功能说明

- 根据自然语言描述生成 `.drawio` XML 文件
- 使用 draw.io 桌面版原生 CLI 将图表导出为 PNG、SVG、PDF 或 JPG
- **6 种图表类型预设**：ERD、UML 类图、序列图、架构图、ML/深度学习、流程图 — 包含预设形状、样式和布局规范
- **动画连接线** (`flowAnimation=1`) 适用于数据流和管道图（在 SVG 和 draw.io 桌面版中可见）
- **ML 模型图支持** 带张量形状标注 `(B, C, H, W)` — 适合 NeurIPS/ICML/ICLR 论文
- **网格对齐布局** — 所有坐标对齐到 10px 倍数，确保整洁对齐
- **浏览器降级方案** — 当桌面 CLI 不可用时生成 diagrams.net URL
- 迭代设计：预览、获取反馈、反复优化直到图表满意为止
- **自动启动** draw.io 桌面版，导出后可直接精修
- 当图表有助于解释复杂系统时自动触发
- **样式预设（v1.3 新增）** — 用一个 `.drawio` 文件或图片"教会"Skill 你的风格，命名保存后可在后续图表中复用。详见 SKILL.md 的 `## Style Presets` 小节。
- **自定义输出目录（v1.4 新增）** — 在提示里指定任意路径（如 `./artifacts/`、`docs/images/`），Skill 会自动 `mkdir -p` 并导出到那里；适合 CI/CD 产物流水线。

## 对比

参见 [COMPARISON_CN.md](COMPARISON_CN.md) —— 与原生智能体、其他 draw.io skills/工具（jgraph/drawio-mcp、bahayonghang/drawio-skills、GBSOSS/ai-drawio）的对照表，以及核心优势小结。

## 支持的图表类型

- **架构图**：微服务架构、云架构（AWS/GCP/Azure）、网络拓扑、部署架构 — 带分层泳道和 hub 居中策略
- **ML / 深度学习**：Transformer、CNN、LSTM、GRU 架构 — 带张量形状标注和层类型配色
- **流程图**：业务流程、工作流、决策树、状态机 — 带语义形状（平行四边形 I/O、菱形判断）
- **UML**：类图（继承/组合/聚合箭头）、序列图（生命线、激活框）
- **数据图**：ER 图（表容器、PK/FK 标记）、数据流图（DFD）
- **其他**：组织架构图、思维导图、线框图

## 工作流程

<p align="center">
  <img src="assets/workflow-cn.png" width="420" alt="工作流程">
</p>

## 安装

两步 —— 先装 draw.io CLI,再把技能加载到 host:

1. **[安装 draw.io 桌面版](INSTALL_CLI_CN.md)** —— macOS / Windows / Linux 各平台配方。
2. **[安装技能](INSTALL_SKILL_CN.md)** —— 插件市场(推荐)、手动克隆、以及更新命令。

## 使用方式

参见 [USAGE_CN.md](USAGE_CN.md) —— 包含自然语言提示词、微服务示例,以及多种拓扑演示(星形 / 分层 / 环形)。

## 样式预设

样式预设让你捕获并复用视觉风格，跨多张图表保持一致。激活预设后，它会替代内置的配色、形状词汇、字体和连线默认值。

### 内置预设

| 名称 | 说明 |
|------|------|
| `default` | 干净的蓝/绿/黄配色，与内置规范保持一致 |
| `corporate` | 低饱和度专业配色，适合商务演示 |
| `handdrawn` | 手绘描边风格，适合非正式或白板式图表 |

### 将预设应用到图表

```
画一个微服务架构图，使用我的 "corporate" 样式
```

或者设置默认样式，让后续所有图表自动使用：

```
把 "corporate" 设为我的默认样式
```

### 从文件中学习样式

指向任意 `.drawio` 文件或图片：

```
从 ~/diagrams/brand.drawio 学习我的样式，保存为 "mybrand"
从 ~/diagrams/screenshot.png 学习我的样式，保存为 "mybrand"
```

Skill 会自动提取配色、形状、字体和连线风格，渲染样例图供预览，确认后才保存至 `~/.drawio-skill/styles/mybrand.json`。

### 管理预设

| 你说的话 | 发生什么 |
|---|---|
| "列出我的样式" | 以表格展示所有用户和内置预设 |
| "显示我的 `<name>` 样式" | 格式化输出预设 JSON |
| "把 `<name>` 设为默认" | 设为所有图表的默认激活预设 |
| "取消默认" | 清除默认（恢复内置规范） |
| "删除 `<name>`" | 删除用户预设（会请求确认） |
| "把 `<a>` 重命名为 `<b>`" | 重命名用户预设 |

## 开源协议

MIT

## 支持作者

如果这个 skill 对你有帮助，欢迎支持作者：

<table>
  <tr>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/wechat-pay.png" width="180" alt="微信支付">
      <br>
      <b>微信支付</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/alipay.png" width="180" alt="支付宝">
      <br>
      <b>支付宝</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/buymeacoffee.png" width="180" alt="Buy Me a Coffee">
      <br>
      <b>Buy Me a Coffee</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/awarding/award.gif" width="180" alt="打赏">
      <br>
      <b>打赏</b>
    </td>
  </tr>
</table>

## 作者

**Agents365-ai**

- Bilibili: https://space.bilibili.com/441831884
- GitHub: https://github.com/Agents365-ai
