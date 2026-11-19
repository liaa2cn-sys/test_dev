# 长安一页 · 西安旅游攻略单页站点

一个可直接双击打开、可离线阅读、可直接分享的西安自由行攻略单页。纯静态 HTML + CSS + 原生 JS，CSS/JS/内联 favicon 全部写在 `index.html` 中，无构建步骤、无 npm 依赖、无 CDN、无远程字体、无图片与外部请求。

## 打开方式

### 方式一：直接双击

双击仓库根目录的 `index.html`，用最新版 Chrome / Edge / Firefox / Safari 打开即可。`file://` 下不需要启动任何服务。

### 方式二：本地静态预览（可选）

```bash
cd test_dev
python3 -m http.server 8000
```

浏览器访问 `http://localhost:8000/`。预览结束后按 `Ctrl+C` 停止。

## 文件结构

```text
test_dev/
├── index.html   # 单页站点：结构、样式、交互、全部内容
└── README.md    # 本说明
```

不包含 `node_modules/`、`dist/`、构建配置、环境变量或任何凭据。

## 内容范围

页面包含 6 个一级板块，均可通过顶部导航或移动端折叠菜单跳转：

1. **城市概览**：城市定位、历史气质、三条游览主线、行前快速事实。
2. **必去景点**：9 个景点卡片，含兵马俑、大雁塔、西安城墙、钟鼓楼、华清宫、大唐不夜城 6 个必去项，以及陕西历史博物馆、碑林博物馆 + 书院门、小雁塔 + 西安博物院 3 个补充推荐。
3. **经典路线**：1 日东线精华、2 日东线 + 老城曲江、3 日东线 + 中轴 + 博物馆，含上午/下午/晚上、交通衔接与预约重点。
4. **美食推荐**：回民街街区说明 + 肉夹馍、羊肉泡馍、biangbiang 面、凉皮、甑糕、胡辣汤、水盆羊肉、葫芦鸡 8 种美食。
5. **交通与住宿**：机场/高铁到达、市内交通、去兵马俑的常见方式，以及钟楼/回民街、大雁塔/曲江、西安北站、永宁门/南门里 4 类住宿区域对比。
6. **最佳季节与避坑**：四季建议、人流提示、10 条避坑、应急电话与行前清单。

## 信息边界与免责声明

- 信息基准：**2026-11**；事实核对日期：**2026-11-19**。
- 核对范围：需求与设计中的稳定事实清单（景点所属朝代、地理位置、相对方位、所属关系等）；**未联网抓取景区官方实时公告**，动态信息请在出行前按官方渠道复核。
- 景点所属朝代、地理位置、相对方位等稳定事实经整理核对；**票价、开放时间、演出、预约规则、地铁/公交首末班、索道运营、临时闭馆**等均为动态信息，统一标注“参考，以景区公告为准”或“以官方公告为准”，页面不写精确票价与精确时刻。
- 本站为静态旅行参考，不提供预订、支付、实时票价、实时客流或导航服务，不构成任何服务承诺。
- 出行前请通过秦始皇帝陵博物院、大慈恩寺/大雁塔景区、西安城墙景区、西安钟鼓楼博物馆、华清宫、陕西历史博物馆等官方渠道复核最新信息。
- 美食价格与营业时间以商家实际为准；回民街为清真街区，请尊重饮食与习俗。

## 自检方法

### 1. 静态结构与锚点检查（Python 标准库）

```bash
cd test_dev
python3 - <<'PY'
import re
from html.parser import HTMLParser
from pathlib import Path

VOID = {"area","base","br","col","embed","hr","img","input","link","meta",
        "param","source","track","wbr"}

class Doc(HTMLParser):
    def __init__(self):
        super().__init__(convert_charrefs=True)
        self.stack, self.errors, self.ids, self.hrefs = [], [], [], []
    def handle_starttag(self, tag, attrs):
        a = dict(attrs)
        if a.get("id"): self.ids.append(a["id"])
        if tag == "a" and (a.get("href") or "").startswith("#"):
            self.hrefs.append(a["href"][1:])
        if tag not in VOID: self.stack.append(tag)
    def handle_endtag(self, tag):
        if not self.stack:
            self.errors.append("unexpected </%s>" % tag)
        elif self.stack[-1] == tag:
            self.stack.pop()
        else:
            self.errors.append("mismatch </%s>" % tag)

html = Path("index.html").read_text(encoding="utf-8")
d = Doc(); d.feed(html); d.close()
ids = set(d.ids)
print("unclosed:", d.stack)
print("errors:", d.errors[:10])
print("duplicate ids:", sorted({i for i in d.ids if d.ids.count(i) > 1}))
print("missing anchors:", sorted({h for h in d.hrefs if h and h not in ids}))
PY
```

通过条件：`unclosed`、`errors`、`duplicate ids`、`missing anchors` 均为空。

### 2. 浏览器检查

1. 用 Chromium 打开 `index.html`（`file://` 或本地 HTTP 均可），确认页面完整渲染，DevTools Console 无报错，Network 无外部请求。
2. 375×812 视口：确认 `document.documentElement.scrollWidth <= 375` 且 `document.body.scrollWidth <= 375`；点击“菜单”能展开/关闭，点击任一板块链接能跳转。
3. 320×640 视口：确认无横向滚动、文字不重叠。
4. 1440×900 视口：确认内容居中、卡片布局正常、导航横向可用。
5. 键盘：Tab 第一个焦点为“跳到主要内容”；菜单、导航、回到顶部可操作；焦点可见；移动端菜单打开后按 Esc 关闭并回到按钮。
6. 模拟 `prefers-reduced-motion: reduce`：入场动画关闭或明显弱化，正文完整可见。
7. 禁用 JavaScript 后重载：正文可读、导航仍可见（菜单按钮隐藏），无横向溢出。

### 3. 外部依赖检查

```bash
cd test_dev
grep -nE 'https?://|@import|<script[^>]+src=|<link[^>]+rel="?stylesheet|<img\b|@font-face' index.html || echo "no external dependencies"
```

通过条件：无匹配输出。页面不使用 `<img>`，主视觉与装饰全部由 CSS 渐变、纹理和内联 favicon 完成，因此不存在破图与图床依赖。

## 浏览器支持与已知限制

- 已在 Chromium（Playwright）中实测：`file://` 与本地 HTTP 均 0 console error、0 失败请求；375px / 320px 无横向溢出；600×320、568×320 短横屏下移动菜单可内部滚动到最后一项；1440px 三列布局正常；6/6 导航锚点跳转后不被 sticky header 遮挡；reduced-motion 与无 JS 场景正文完整可见。
- Firefox / Safari 未在本环境实测；代码仅使用广泛支持的经典脚本、CSS Grid、CSS 变量与 `clamp()`，不支持 IE。
- `file://` 下不依赖 `fetch`/XHR/ES Module/Service Worker，因此双击打开与通过静态服务器访问的行为一致。
- 页面不包含真实照片，主视觉采用唐风配色、CSS 渐变/纹理与内联 favicon；如需配图，应使用有授权的本地图片并补充 `alt` 与加载失败占位。

## 维护提示

- 更新内容时，请同步检查页脚“信息基准 2026-11”，并重跑本 README 的静态检查。
- 动态信息（票价、开放时间、演出、班次、预约规则）不要在页面写死精确值；保持“参考，以景区公告为准/以官方公告为准”的标注。
- 导航锚点 `top`、`main`、`overview`、`sights`、`routes`、`food`、`transport`、`tips` 为页面契约，改版时不要删除或重命名。
