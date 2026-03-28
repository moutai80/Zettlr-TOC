# Zettlr 目录折叠功能增强版

## 📌 项目说明

本项目是基于 [Zettlr](https://github.com/Zettlr/Zettlr) 进行的二次开发，添加了目录折叠功能。

## 🔗 原项目

- **原项目**: https://github.com/Zettlr/Zettlr
- **原作者**: Hendrik Erz
- **原许可证**: GPL-3.0

## 📜 许可证

本项目继承原项目的 **GPL-3.0** 许可证。

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

## ✨ 新增功能

### 1. 目录折叠/展开
- 有子标题的章节显示折叠按钮
- 没有子标题的章节不留空白（文字对齐）
- 点击 ▼/▶ 折叠/展开章节

### 2. 工具栏
- 📍 **自动滚动开关** - 控制当前章节高亮跟随
- 📁 **全部折叠/展开** - 一键操作所有章节
- 🔍 **搜索** - 过滤目录内容

### 3. 优化
- 所有条目文字对齐
- 折叠按钮独立状态，不受手动操作影响
- CSS 旋转动画，流畅过渡

## 📦 使用方法

### 安装依赖
```bash
yarn install
```

### 开发模式
```bash
yarn dev
```

### 打包构建
```bash
yarn package:linux-x64
yarn release:linux-x64
```

### 运行
```bash
chmod +x release/Zettlr-4.3.0-x86_64.AppImage
./Zettlr-4.3.0-x86_64.AppImage
```

## 🔧 修改的文件

```
source/win-main/sidebar/ToCTab.vue
```

## 📝 修改历史

- 2026-03-28: 添加目录折叠功能
- 2026-03-28: 修复对齐问题
- 2026-03-28: 添加工具栏
- 2026-03-28: 优化折叠逻辑

## ⚠️ 重要声明

1. 本项目是基于 Zettlr 的修改版本，不是官方版本
2. 所有修改遵循 GPL-3.0 许可证
3. 原项目的版权和许可证必须保留
4. 如有任何版权问题，请联系原作者

## 📧 联系方式

- 原项目：https://www.zettlr.com/
- 原项目论坛：https://forum.zettlr.com/
- 原项目 Discord: https://go.zettlr.com/discord

---

**感谢 Zettlr 原作者的优秀作品！** 🙏
