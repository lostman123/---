---
name: learnin-download
description: Download video courses from 复旦大学终身教育学院 learning platform (learnin.com.cn / media.o-learn.cn). Use when the user wants to save an m3u8-encrypted course video from learnin.com.cn for offline viewing. Covers extracting the m3u8 URL, getting the AES key, and downloading with N_m3u8DL-RE.
---

# learnin.com.cn 视频下载

## 前置条件

确保已安装 `N_m3u8DL-RE`（命令行工具）。

## 工作流

### 1. 获取 m3u8 播放列表 URL

打开浏览器开发者工具 → Network 面板 → 过滤 `m3u8` → 找到 `.../api/video/play/m3u8/xxx.m3u8` 请求，复制完整 URL。

或者从页面 HTML/JS 中提取 `vid`，构造 URL：

```
https://media.o-learn.cn/api/video/play/m3u8/{vid}.m3u8
```

### 2. 下载视频

```bash
N_m3u8DL-RE "https://media.o-learn.cn/api/video/play/m3u8/{vid}.m3u8" \
  -H "Referer: https://www.learnin.com.cn/" \
  --save-dir /root/downloads \
  --save-name "课程名称" \
  --auto-select \
  --del-after-done
```

关键点：
- **只需要 `Referer` 头**，无需 cookie/token
- `--auto-select` 自动选择最佳画质
- `--del-after-done` 完成后删除临时文件
- `--save-name` 指定输出文件名（不含扩展名）
- `--save-dir` 指定输出目录

### 3. 持久化 cookies（可选）

如果 `Referer` 不够，可使用浏览器 cookie：

```bash
N_m3u8DL-RE "m3u8_url" \
  -H "Referer: https://www.learnin.com.cn/" \
  -H "Cookie: accessToken=xxx; SESSION=xxx" \
  --save-dir /root/downloads \
  --save-name "课程名称" \
  --auto-select \
  --del-after-done
```

### 4. 手动指定 KEY（如果自动获取失败）

先下载 key：

```bash
curl -s -H "Referer: https://www.learnin.com.cn/" \
  "https://media.o-learn.cn/api/video/play/key/{vid}?ts=xxx&sign=xxx" \
  -o /tmp/video.key
```

然后传给 N_m3u8DL-RE：

```bash
N_m3u8DL-RE "m3u8_url" \
  -H "Referer: https://www.learnin.com.cn/" \
  --key "KID:$(xxd -p /tmp/video.key | tr -d '\n')" \
  --save-dir /root/downloads \
  --save-name "课程名称" \
  --auto-select \
  --del-after-done
```

## 说明

- `media.o-learn.cn` 仅校验 `Referer: https://www.learnin.com.cn/`
- 视频使用 AES-128 加密，N_m3u8DL-RE 自动处理解密
- m3u8 URL 中的 `ts` 和 `sign` 参数会过期，需获取最新的 m3u8 文件
- 输出为 mp4 格式（通过 ffmpeg 合并）
