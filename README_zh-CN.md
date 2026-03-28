<h1 align="center">
  <a href="https://github.com/Zettlr/Zettlr">
    <img src="https://raw.githubusercontent.com/Zettlr/Zettlr/master/resources/icons/png/256x256.png" alt="Zettlr"/>
  </a>
  <br/>
  Zettlr [<em>ˈset·lər</em>] - 目录折叠增强版
</h1>

<p align="center"><strong>一站式出版工作台（中文增强版）</strong></p>

<p align="center">
  <strong>⚠️ 本项目是基于 <a href="https://github.com/Zettlr/Zettlr">Zettlr</a> 的二次开发，不是官方版本</strong>
</p>

<p align="center">
  <a href="https://www.gnu.org/licenses/gpl-3.0">
    <img src="https://img.shields.io/badge/License-GPLv3-blue.svg" alt="License: GNU GPL v3">
  </a>
  <a href="https://github.com/Zettlr/Zettlr">
    <img alt="Original Project" src="https://img.shields.io/badge/Original-Zettlr-blue">
  </a>
</p>

<p align="center">
  <a href="#特性">特性</a> |
  <a href="#新增功能">新增功能</a> |
  <a href="#安装">安装</a> |
  <a href="#使用指南">使用指南</a> |
  <a href="#构建">构建</a> |
  <a href="#许可证">许可证</a>
</p>

---

## 📌 项目说明

**本项目是基于 [Zettlr](https://github.com/Zettlr/Zettlr) 的中文增强版本**，添加了目录折叠、自动滚动等实用功能，并优化了中文用户体验。

- **原项目**: https://github.com/Zettlr/Zettlr
- **原作者**: Hendrik Erz
- **原许可证**: GPL-3.0

---

## ✨ 新增功能（相比原版）

### 1. 🗂️ 目录折叠/展开

- ✅ 有子标题的章节显示折叠按钮
- ✅ 没有子标题的章节不留空白（文字对齐）
- ✅ 点击 ▼/▶ 折叠/展开章节
- ✅ CSS 旋转动画，流畅过渡

### 2. 🧭 智能工具栏

| 图标 | 功能 | 说明 |
|------|------|------|
| 📍 | **自动滚动** | 当前章节自动高亮跟随光标 |
| 📁 | **全部折叠/展开** | 一键操作所有章节 |
| 🔍 | **搜索** | 快速过滤目录内容 |

### 3. 🇨🇳 中文优化

- ✅ 完整中文界面翻译
- ✅ 符合中文用户习惯
- ✅ 中文文档支持

---

## 🎯 特性（原版）

- 🔒 **隐私优先**：你的笔记就是你的笔记
- 📚 **引用管理**：集成 Zotero、JabRef 等参考文献管理工具
- 🌍 **多语言支持**：十几种语言
- 📝 **LaTeX 和 Word 模板**：专业出版支持
- 🎨 **自定义 CSS**：主题、暗黑模式
- 🔍 **全文搜索**：快速找到任何内容
- 🗂️ **Zettelkasten**：支持卡片盒笔记法

---

## 📦 安装

### 方法 1：下载预编译版本

```bash
# Linux
chmod +x Zettlr-4.3.0-x86_64.AppImage
./Zettlr-4.3.0-x86_64.AppImage

# macOS
open Zettlr-4.3.0-arm64.dmg

# Windows
双击 Zettlr-Setup-4.3.0.exe
```

### 方法 2：从源码构建

```bash
# 1. 克隆仓库
git clone https://github.com/你的用户名/Zettlr-CN.git
cd Zettlr-CN

# 2. 安装依赖
yarn install

# 3. 开发模式运行
yarn dev

# 4. 打包构建
yarn package:linux-x64
yarn release:linux-x64
```

---

## 📖 使用指南

### 目录折叠功能

1. **打开目录面板** - 右侧边栏
2. **折叠章节** - 点击章节左侧的 ▼ 按钮
3. **展开章节** - 点击章节左侧的 ▶ 按钮
4. **全部折叠** - 点击工具栏的 📁 按钮
5. **搜索** - 在工具栏输入关键词过滤

### 自动滚动

- 📍 **开启** - 当前章节自动高亮，跟随光标移动
- ◌ **关闭** - 不自动高亮

---

## 🔧 构建命令

```bash
# 开发模式
yarn dev

# 打包（不构建安装包）
yarn package:linux-x64

# 构建安装包（AppImage、deb、rpm）
yarn release:linux-x64

# 其他平台
yarn package:win-x64      # Windows
yarn package:mac-x64      # macOS Intel
yarn package:mac-arm      # macOS Apple Silicon
```

---

## 📝 修改的文件

```
source/win-main/sidebar/ToCTab.vue    # 目录折叠核心功能
static/lang/zh-CN.po                   # 中文翻译
README-TOC-功能说明.md                 # 功能说明文档
```

---

## ⚖️ 许可证

**GPL-3.0**

```
Copyright (C) 2025 OpenClaw · 严谨专业版

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
```

---

## ⚠️ 重要声明

1. 本项目是基于 Zettlr 的修改版本，**不是官方版本**
2. 所有修改遵循 **GPL-3.0** 许可证
3. 原项目的版权和许可证必须保留
4. 如有任何版权问题，请联系原作者

---

## 🙏 致谢

感谢 **Zettlr 原作者** [Hendrik Erz](https://github.com/hendrik-erz) 的优秀作品！

- 原项目官网：https://www.zettlr.com/
- 原项目论坛：https://forum.zettlr.com/
- 原项目 Discord: https://go.zettlr.com/discord

---

## 📧 联系方式

- 问题反馈：https://github.com/你的用户名/Zettlr-CN/issues
- 原作者联系：info@zettlr.com

---

**🎉 享受更强大的 Zettlr！**
