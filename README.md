# 面相算命 (Face Fortune)

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Vue](https://img.shields.io/badge/Vue-3.x-brightgreen.svg)
![Vite](https://img.shields.io/badge/Vite-6.x-purple.svg)

## 项目介绍

面相算命是一个基于人工智能的面部分析应用，通过分析用户面部特征，生成有趣的性格、事业和婚姻运势分析结果。该项目仅供娱乐，将人脸识别技术与传统面相学相结合，创造了一种新颖的交互体验。

## 功能特点

- **实时人脸检测**：使用摄像头实时捕捉用户面部
- **面部特征分析**：分析面部轮廓、表情等特征
- **性格特点评估**：通过雷达图直观展示性格特点分析
- **运势预测**：包含事业、婚姻等方面的运势分析
- **每日宜忌**：提供个性化的日常建议
- **响应式设计**：适配各种屏幕尺寸的设备

## 技术栈

- **前端框架**：Vue 3 + Vite
- **UI组件库**：Element Plus
- **图表可视化**：ECharts
- **人脸识别**：face-api.js
- **开发语言**：JavaScript

## 项目截图

*（项目截图待添加）*

## 安装使用

### 环境要求

- Node.js 16.0+
- npm 或 yarn

### 安装步骤

1. 克隆项目到本地

```bash
git clone https://github.com/your-username/face-fortune.git
cd face-fortune
```

2. 安装依赖

```bash
npm install
# 或
yarn
```

3. 启动开发服务器

```bash
npm run dev
# 或
yarn dev
```

4. 构建生产版本

```bash
npm run build
# 或
yarn build
```

## 项目结构

```
face-fortune/
├── public/               # 静态资源
│   └── models/           # face-api.js 模型文件
├── src/                  # 源代码
│   ├── assets/           # 项目资源文件
│   ├── components/       # 组件
│   │   └── FaceFortune.vue  # 主要功能组件
│   ├── App.vue           # 根组件
│   ├── main.js           # 入口文件
│   └── style.css         # 全局样式
├── index.html            # HTML 模板
├── vite.config.js        # Vite 配置
└── package.json          # 项目依赖
```

## 使用说明

1. 点击「打开摄像头」按钮，授予摄像头访问权限
2. 确保面部在摄像头视野范围内，光线充足
3. 点击「开始算命」按钮进行面相分析
4. 查看分析结果，包括性格特点、事业分析和婚姻分析
5. 可以查看今日宜忌建议

## 注意事项

- 本应用需要摄像头权限，请确保浏览器已获得授权
- 面部识别效果受光线、角度等因素影响
- 分析结果仅供娱乐，请勿当真

## 未来计划

- [ ] 增加更多面相分析维度
- [ ] 支持历史记录保存
- [ ] 添加更多面部特征识别模型
- [ ] 优化移动端体验
- [ ] 支持多语言

## 贡献指南

欢迎贡献代码、提出问题或建议！请遵循以下步骤：

1. Fork 本仓库
2. 创建您的特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交您的更改 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 打开一个 Pull Request

## 许可证

本项目采用 MIT 许可证 - 详情请参阅 [LICENSE](LICENSE) 文件

## 联系方式

如有任何问题或建议，请通过 GitHub Issues 与我们联系。

---

**免责声明**：本应用仅供娱乐，分析结果不具有任何科学依据，请勿将结果用于任何重要决策。
