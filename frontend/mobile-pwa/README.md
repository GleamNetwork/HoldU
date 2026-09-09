# 同频移动端 PWA 前端

这是面向 iPhone 纵向比例的单页前端演示。首页以晨光房间和小频为主视觉，提供陪伴对话、感受记录和隐私说明的可操作抽屉。

## 本地打开

为使 Service Worker 和“添加到主屏幕”配置生效，请通过本地服务器或 HTTPS 访问，而不要直接双击 HTML 文件。

```bash
cd frontend/mobile-pwa
python3 -m http.server 8766
```

随后打开 `http://127.0.0.1:8766/`。部署到 HTTPS 后，可在 iPhone Safari 选择“分享 -> 添加到主屏幕”，以独立轻应用方式运行。

## 目录

- `index.html`：前端界面与交互
- `manifest.webmanifest`、`sw.js`：PWA 清单与离线缓存
- `assets/`：小频的安静/回应状态、场景背景和应用图标
- `DESIGN.md`：色彩语义、动效和安全文案边界

所有对话、感受和设备摘要均为比赛演示状态。页面不包含真实危机数据，也不替代专业心理或医疗服务。
