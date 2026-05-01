---
title: RobotJS+Sharp：Windows桌面自动化的兼容地狱
date: 2026-05-01
tags:
  - RobotJS
  - Sharp
  - Windows自动化
  - Node.js
  - 踩坑记录
---

# RobotJS+Sharp：Windows桌面自动化的兼容地狱

## 前言

做Windows桌面自动化，绕不开RobotJS。做图片处理，绕不开Sharp。但当这两个"老大哥"凑到一起，在Windows的各种版本上运行，那可就是一场"兼容地狱"。

## 为什么选RobotJS

### 竞品对比

| 方案 | 优点 | 缺点 |
|------|------|------|
| RobotJS | 功能全面、成熟稳定 | 仅支持Windows/macOS |
| playwright | 跨平台、现代化 | 不能控制桌面应用 |
| puppeteer | Chrome控制强大 | 体积大、资源占用高 |
| AutoHotkey | 原生高效 | 学习曲线陡峭 |

**我的选择理由**：
1. RobotJS是纯Node.js方案
2. 可以与Sharp等Node图片库无缝配合
3. 功能覆盖：鼠标、键盘、截图、剪贴板

## Windows不同版本的兼容性差异

### 版本对照表

| Windows版本 | RobotJS | Sharp | Node.js |
|------------|---------|-------|---------|
| Win7 x64 | ⚠️ 需0.6.2 | ✅ 0.30.7 | 14.x |
| Win10 x64 | ✅ 0.6.2 | ✅ 0.33.x | 16.x+ |
| Win11 | ✅ 0.6.2 | ✅ 最新 | 18.x+ |

## GDI+截图性能对比

### RobotJS截图

```javascript
const robot = require('robotjs');
const bitmap = robot.screen.capture(0, 0, 1920, 1080);
```

**性能测试**：
- 1920x1080截图：约50-80ms
- 4K截图：约200-300ms

### Sharp处理

```javascript
const sharp = require('sharp');
await sharp(bitmap.buffer)
  .resize(800)
  .jpeg({ quality: 85 })
  .toFile('screenshot.jpg');
```

## 中文路径问题的解决方案

```javascript
const path = require('path');
const fs = require('fs');

// 使用ASCII临时路径
const tempPath = path.join(process.env.TEMP, 'temp_capture.png');
const bitmap = robot.screen.capture(0, 0, 100, 100);
const buffer = fs.readFileSync(tempPath);
await sharp(buffer).toFile('中文路径/输出.png');
```

## better-sqlite3的替代方案

### 问题

better-sqlite3编译经常失败。

### 解决方案：使用sql.js

sql.js是SQLite的WebAssembly版本，完全不依赖编译：

```bash
npm install sql.js
```

```javascript
const initSqlJs = require('sql.js');
const SQL = await initSqlJs();
const db = new SQL.Database();
db.run("CREATE TABLE test (id, name)");
```

## Puppeteer替代方案

### 体积问题

Puppeteer（+ Chromium）安装后非常大：150-200MB。

### 替代方案：纯Sharp图片处理

```javascript
const sharp = require('sharp');
await sharp('input.jpg')
  .resize(1920, 1080)
  .toFile('output.jpg');
```

## 总结

桌面自动化的"兼容地狱"是客观存在的现实：

1. **明确支持的最小环境**
2. **准备好降级方案**
3. **做好充分的错误处理**
4. **准备好多种备选技术栈**

让产品跑起来比什么都重要。
