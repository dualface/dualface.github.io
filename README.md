# dualface.github.io

## Languages

- English is at the site root; Chinese is under `/cn/` (language key `cn`, locale `zh-CN`).
- A post is one page bundle with one file per language:
  - `content/posts/<slug>/index.md`: English, at `/posts/<slug>/`
  - `content/posts/<slug>/index.cn.md`: Chinese, at `/cn/posts/<slug>/`
  - Images in the bundle are shared by both languages.
  - PNG files in the bundle are converted to WebP at build time, and posts embed the WebP URL. Keep linking `images/x.png` in the source; do not convert by hand.
- `content/posts/` is written by the dualface-writing publish script; do not edit posts here.
- Other pages follow the same rule: `about.md` / `about.cn.md`, `projects.md` / `projects.cn.md`.
- The home page shows the three newest posts and the first `##` section of the Projects page, so keep the current projects in that section.
- The newsletter box after a post, and on the English About page (`newsletter = true`), is English only (Buttondown); Chinese posts show the WeChat QR code `static/images/wechat-qr.png`.
