# dualface.github.io

## Languages

- English is at the site root; Chinese is under `/cn/` (language key `cn`, locale `zh-CN`).
- A post is one page bundle with one file per language:
  - `content/posts/<slug>/index.md`: English, at `/posts/<slug>/`
  - `content/posts/<slug>/index.cn.md`: Chinese, at `/cn/posts/<slug>/`
  - Images in the bundle are shared by both languages.
- `content/posts/` is written by the dualface-writing publish script; do not edit posts here.
- Other pages follow the same rule: `about.md` / `about.cn.md`, `projects.md` / `projects.cn.md`.
- The newsletter box after a post is English only (Buttondown); the Chinese page shows the WeChat QR code `static/images/wechat-qr.png`.
