# Unlock Studio

> 本地音乐解锁台：把已下载的加密音乐还原成可播放的原始音频。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/license/mit)
[![Single File](https://img.shields.io/badge/Build-Single%20HTML-2ea44f.svg)](#项目结构)
[![No Dependencies](https://img.shields.io/badge/Dependencies-0-blue.svg)](#实现说明)
[![Chrome Edge](https://img.shields.io/badge/Recommended-Chrome%20%2F%20Edge-important.svg)](#浏览器兼容性)

Unlock Studio 是一个**单文件、纯前端、零依赖**的本地音乐解封装工具。支持网易云 NCM、酷狗 KGM/KGMA/KGG/VPR，以及 QQ 音乐 MFLAC/MGG/QMC 系列加密文件。

所有解析与解密都在浏览器本地完成：**没有后端、没有上传、没有统计，也不会重新编码音频**。

仓库地址：<https://github.com/eto9854/Unlock-Studio>

---

## 目录

- [功能特性](#功能特性)
- [支持格式](#支持格式)
- [快速开始](#快速开始)
- [使用说明](#使用说明)
- [密钥库](#密钥库)
- [隐私与安全](#隐私与安全)
- [浏览器兼容性](#浏览器兼容性)
- [实现说明](#实现说明)
- [已知限制](#已知限制)
- [常见问题](#常见问题)
- [项目结构](#项目结构)
- [免责声明](#免责声明)
- [License](#license)
- [致谢](#致谢)

---

## 功能特性

- **单文件运行**：整个应用就是一个 `index.html`，下载后直接打开即可，无需安装、无需构建。
- **多平台格式**：网易云 NCM；酷狗 KGM/KGMA/KGG/VPR；QQ 音乐 MFLAC/MGG/QMC 系列。
- **批量处理**：支持多选文件、拖拽文件、拖拽文件夹，递归扫描并自动去重。
- **队列工作台**：搜索、全部/完成/失败/进行中筛选，逐项进度与错误提示。
- **试听与播放**：详情面板支持播放/暂停、进度拖动、上一首/下一首；可开启“选中自动试听”和“播完自动下一首”。
- **导出与保存**：
  - 默认通过浏览器下载；
  - 可一键选择输出文件夹，完成文件自动写入；
  - 同名文件自动追加 `(1)`、`(2)`，避免覆盖；
  - 可随时改回浏览器下载。
- **失败处理**：单个重试 + “重试失败 N”一键重新排队。
- **封面与标签**：读取 NCM 内嵌封面；对 FLAC/MP3 自动提取 Vorbis Comment / ID3v2 标签和封面，支持单独下载封面。
- **桌面与移动端**：桌面左右工作台；移动端列表 + 底部详情抽屉；支持 Studio Night / Studio Day · Warm 双主题。
- **完全本地**：不上传文件、不上传密钥、不发起任何业务网络请求。

---

## 支持格式

| 平台 | 文件后缀 | 是否需要密钥 | 说明 |
| --- | --- | --- | --- |
| 网易云音乐 | `.ncm` | 不需要 | AES-128-ECB + RC4 变体，支持内嵌封面与元信息 |
| 酷狗音乐 | `.kgm` / `.kgma` / `.vpr` | 通常不需要 | 支持 v2 XOR、v3 MD5 派生；v5 需要酷狗密钥库 |
| 酷狗音乐 | `.kgg` | 需要 | v5 加密，需要 `KGMusicV3.db` 或 `kgg.key` |
| QQ 音乐 | `.mflac` / `.mflac0` / `.mflac1` | 多数不需要 | 支持 QMCv2 Map / RC4、QTag 内嵌 EKey |
| QQ 音乐 | `.mgg` / `.mgg0` / `.mgg1` | 多数不需要 | 同上 |
| QQ 音乐 | `.qmcflac` / `.qmcogg` / `.qmc0` / `.qmc2` / `.qmc3` / `.qmc4` / `.qmc6` / `.qmc8` | 视文件而定 | QMCv1 静态密钥、QMCv2 Map / RC4 |
| QQ 音乐 | `.tkm` / `.bkc*` | 视文件而定 | 旧版 QMC / 静态密钥系列 |

> 输出格式由解密后的音频头自动识别：MP3、FLAC、M4A、WAV、OGG、APE、AAC、WMA、DFF 等。  
> 工具只做“解封装”，不做有损转码；源文件是 FLAC 时会原样导出 `.flac`。

---

## 快速开始

### 方式一：直接使用

1. 下载仓库中的 [`index.html`](./index.html)；
2. 用 Chrome 或 Edge 打开；
3. 拖入加密音乐，等待解密完成；
4. 点击“全部保存”下载，或选择输出文件夹自动写入。

### 方式二：克隆仓库 / 本地服务

```bash
git clone https://github.com/eto9854/Unlock-Studio.git
cd Unlock-Studio
python -m http.server 8080
```

浏览器访问 `http://localhost:8080`。

> 直接双击 `index.html`（`file://`）也可以使用基础解密与下载功能；但目录选择、输出文件夹、自动保存等依赖 File System Access API，部分浏览器会限制。需要完整功能时，建议通过本地 HTTP 服务打开，并使用 Chrome / Edge。

---

## 使用说明

### 1. 导入文件

- 点击“添加文件”选择单个或多个加密文件；
- 直接拖拽文件到页面；
- 点击“文件夹”选择整个目录；
- 拖拽整个文件夹到页面（Chrome / Edge 支持目录拖拽，其他浏览器可能只能读取文件）。

队列会按文件路径排序、过滤不支持的后缀，并按“文件名 + 大小 + 修改时间”去重。

### 2. 选择文件查看详情

点击队列中的文件，右侧会显示：

- 封面、标题、歌手、专辑；
- 输出格式、码率、时长、文件大小；
- 密钥来源、容器版本、保存状态；
- 播放器、上一首/下一首、下载/保存、封面下载、重试、移除。

### 3. 保存

- **默认**：点击“全部保存”直接走浏览器下载；
- **保存到文件夹**：点击队列底部的“保存到文件夹”，选择一个目录后：
  - 底部提示会显示当前保存位置；
  - “全部保存”会写入该文件夹；
  - 开启自动保存后，完成的文件会自动写入；
  - 同名文件自动追加 `(1)`、`(2)`；
- 想恢复下载模式：设置 → 输出 → “改回浏览器下载”。

### 4. 试听

- 选中已完成文件会自动试听（可在设置关闭）；
- 支持播放/暂停、进度拖动、上一首/下一首；
- 开启“播完自动下一首”后会连续播放队列。

### 5. 失败重试

- 单项失败：详情面板点击“重试”；
- 批量失败：队列底部“重试失败 N”一键重新排队；
- 导入密钥库后再重试，通常可以解决 KGG / 部分 QQ 文件的密钥缺失问题。

---

## 密钥库

以下文件格式需要或可能需要密钥库。工具**不附带任何密钥数据**，请使用你自己合法拥有的本地密钥库。

### 酷狗

- Windows 客户端密钥库：`%APPDATA%\KuGou8\KGMusicV3.db`
- 或文本密钥文件 `kgg.key`，每行格式：`id$ekey`

在“设置 → 密钥库 → 导入密钥库”中选择上述文件。

### QQ 音乐

部分 MFLAC/MGG 文件没有内嵌 EKey（`STag` / `MusicEx` 尾部结构），需要 QQ 音乐的本地密钥库：

- `player_process_db`（SQLite）

在“设置 → 密钥库 → 导入密钥库”中选择该文件。

### 说明

- 密钥**只保存在当前页面的内存中**，不会上传，也不会写入本地存储；
- 刷新页面后需要重新导入；
- 如果提示“密钥库中没有该文件的密钥”，通常需要先在对应客户端播放一次该歌曲，让密钥库更新，再重新导出并导入；
- 密钥库格式可能随客户端版本变化，本项目不对未来版本做保证。

---

## 隐私与安全

- 没有服务器、没有后端接口、没有业务网络请求；
- 文件只在你选择的本地文件中读取，解密结果只存在于浏览器内存与 Blob URL 中；
- 密钥库解析结果只保存在当前页面内存中；
- 设置、主题、命名格式保存在 `localStorage`；
- 输出目录句柄保存在浏览器 `IndexedDB`，刷新后可能需要重新授权；
- 没有统计、没有埋点、没有第三方脚本。

> 页面唯一的外部链接是页脚的 GitHub 仓库链接，点击后才会打开外部页面。

---

## 浏览器兼容性

| 功能 | Chrome / Edge | Firefox / Safari |
| --- | --- | --- |
| 解密与下载 | 支持 | 支持 |
| 拖拽文件 | 支持 | 支持 |
| 拖拽文件夹 | 支持（优先使用 File System Handle，失败回退旧接口） | 取决于浏览器；失败时请用“文件夹”按钮 |
| 选择输出文件夹 / 自动保存 | 支持（File System Access API） | 可能不支持，自动退化为浏览器下载 |
| 移动端布局 | 支持 | 支持 |

建议使用 **Chrome / Edge** 以获得完整的目录与自动保存能力。

---

## 实现说明

- **纯前端**：HTML + CSS + 原生 JavaScript，无框架、无第三方运行库、无构建步骤；
- **分块处理**：按 4MB 分块读取并解密，最后组装为 Blob；超大文件仍受浏览器内存限制；
- **格式识别**：解密首块后按魔数识别 MP3 / FLAC / M4A / WAV / OGG / APE / AAC / WMA / DFF；
- **NCM**：AES-128-ECB 解 Key 与元信息，RC4 变体流式解密音频；
- **酷狗**：KGM v2 XOR 掩码、v3 MD5 派生、v5 EKey + QMC2；
- **酷狗密钥库**：`KGMusicV3.db` 的 AES-CBC 分页解密，内置最小 SQLite 读取器解析 `ShareFileItems`；
- **QQ 音乐**：TEA 解密 EKey，QMCv2 Map / RC4，QMCv1 静态密钥，QTag / STag / MusicEx 尾部解析；
- **旁路能力**：FLAC Vorbis Comment / PICTURE、MP3 ID3v2 / APIC 标签提取。

算法实现参考了公开的开源实现与公开格式资料，但本项目是独立重写的单文件版本，没有直接打包第三方库，也没有内置任何真实歌曲、密钥库或密钥数据。

### 验证

- QMC 系列使用 unlock-music 的公开测试向量进行字节级对照；
- NCM、KGM v2/v3、KGG v5 使用自建加密样本做往返验证；
- 酷狗 `KGMusicV3.db` 使用合成的加密分页库做整库解密与 SQLite 解析验证；
- QQ `player_process_db` 解析、STag 查找、QMC 内嵌 EKey 均已验证；
- 桌面 / 移动端布局、主题、队列、搜索筛选、失败重试、密钥导入、同名保存已做页面测试。

---

## 已知限制

- **不转码**：只还原原始音频流；FLAC 不会转 MP3，MP3 也不会转 FLAC；
- **不写标签**：可以读取标签和封面，但不会把标签写回音频；
- **不提供密钥**：KGG 与部分 QQ 文件必须由用户自行导入合法拥有的密钥库；
- **版本差异**：客户端未来若更改加密方式或密钥库格式，可能需要更新支持；
- **内存限制**：超大文件受浏览器内存与 Blob 限制影响，建议分批处理；
- **浏览器能力**：输出文件夹、自动保存、目录拖拽依赖 File System Access API；
- **不是下载器**：不抓取在线歌曲，只处理你本地已有的文件。

---

## 常见问题

### 为什么拖拽文件夹提示没有可处理的文件？

不同浏览器对目录拖拽支持不同。Chrome / Edge 会优先读取目录句柄，失败后回退旧接口；如果浏览器限制目录访问，请点击页面上的“文件夹”按钮选择目录。

### 为什么 KGG 提示缺少酷狗密钥？

需要先导入 `KGMusicV3.db` 或 `kgg.key`。如果密钥库已导入仍找不到对应密钥，请先在酷狗客户端播放该歌曲，让密钥库更新后再导出。

### 为什么某些 MFLAC / MGG 提示缺少密钥？

这类文件通常是 `STag` / `MusicEx` 结构，没有内嵌 EKey，需要 QQ 音乐的 `player_process_db` 密钥库。

### “全部保存”是下载还是写入文件夹？

默认没有选择输出文件夹时，走浏览器下载。点击“保存到文件夹”选择目录后，“全部保存”和自动保存会写入该目录。可在设置中改回浏览器下载。

### 为什么浏览器拦截了多个下载？

批量下载会触发浏览器的多文件下载限制。请在浏览器地址栏允许本站“下载多个文件”，或改用“保存到文件夹”。

### 支持转 MP3 吗？

不支持转码。工具只把加密容器解开，输出原始 MP3 / FLAC / OGG / M4A 等。如果原始音频是 FLAC，输出仍是 FLAC。

### 会不会上传我的文件或密钥？

不会。所有处理都在浏览器本地完成；密钥只存在当前页面内存中。

### 为什么自动试听没有声音？

浏览器可能阻止了自动播放。点击播放按钮即可；也可以在设置中关闭“选中自动试听”。

---

## 项目结构

```text
Unlock-Studio/
├─ index.html   # 单文件应用：UI + CSS + 解密核心
└─ README.md
```

无需 `npm install`，无需构建。

---

## 免责声明

- 本项目仅用于解密你本人合法拥有、且有权使用的本地音乐文件；
- 请遵守你所在地区的法律法规以及相关平台的服务条款；
- 本项目不提供绕过付费权益的能力，不附带密钥数据库，也不提供获取密钥的方法；
- 项目与网易云音乐、酷狗音乐、QQ 音乐及其关联公司无任何隶属或合作关系；
- 使用本项目产生的任何后果由使用者自行承担，本项目不对数据丢失、兼容性问题或法律风险作任何担保。

---

## License

本项目以 MIT License 开源。

```text
MIT License

Copyright (c) 2026 @Aran987

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

Crafted by @Aran987

GitHub: <https://github.com/eto9854/Unlock-Studio>

---

## 致谢

格式算法与测试向量参考了以下公开项目与研究资料：

- [unlock-music](https://github.com/unlock-music)
- [AudioDecrypt](https://github.com/0x77fe/AudioDecrypt)
- [Kugo-Music-Converter](https://github.com/skxxxkx666/Kugo-Music-Converter)
- [qqmusic-decrypt](https://github.com/mzj3920/qqmusic-decrypt)
- [music-unlock](https://github.com/inkwise/music-unlock)

感谢所有公开分享格式资料与测试样本的开发者。

如果这个项目对你有帮助，欢迎在 GitHub 上点一个 Star。
