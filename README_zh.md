# 离线 PDB 链接生成器

一款专为结构生物学研究人员设计的静态网络工具。输入 PDB 编号即可立即生成对应结构文件及相关数据库页面的直接下载链接。

---

## 预览

![image](public/e1.jpg)

---

English version here:[English version](README.md)

---

## 在线试用

https://pdbdownloader.netlify.app

---

## 本地运行

### 环境要求

- **Node.js**: 16.0.0 或更高版本
- **npm**: 7.0.0 或更高版本（或 yarn/pnpm）

### 安装步骤

1. 克隆仓库

   ```bash
   git clone https://github.com/Perspicace123/PDB_downloader.git
   cd PDB_downloader
   ```

2. 安装依赖

   ```bash
   npm install
   ```

3. 启动开发服务器

   ```bash
   npm run dev
   ```

4. 构建生产版本（可选）

   ```bash
   npm run build
   ```

### 注意事项

- Vite 默认使用端口 5173，请确保该端口可用
- 如需修改配置，请编辑 `vite.config.js`

---

## 项目结构

```
├── public/          # 静态资源
├── src/             # 源代码
├── App.vue          # 根组件
├── index.html       # 入口 HTML
├── package.json     # 项目配置
└── vite.config.js   # Vite 配置
```

---

## 主要优势

- **离线且隐私安全**：完全在浏览器中运行，数据不会发送到任何服务器
- **快速且稳定**：即时响应，无需网络延迟
- **研究高效**：直接访问分子数据，加速研究工作流程

---

## 功能特性

- **即时链接生成**：为任意 PDB 编号生成完整的 URL 集合
- **多种文件格式**：支持 PDB、mmCIF、XML 文件及其压缩版本
- **数据库集成**：直接链接至 RCSB PDB、PDBe、PDBsum 和 UniProt
- **响应式设计**：支持桌面和移动设备
- **一键复制**：将生成的任何链接复制到剪贴板
