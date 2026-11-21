# 蓝屿手记 · 静态博客（GitHub Pages）

一套零服务器、零数据库、零月费的个人博客：用户端负责阅读与下载，管理端负责发文章、管资源、改设置。
所有内容都是仓库里的三个 JSON 文件，保存即提交，GitHub Pages 自动构建。

## 目录结构

```
.
├── index.html            首页
├── blog.html             文章列表 / 详情（#/post/<slug>）
├── downloads.html        下载专区（提取码校验）
├── login.html            用户注册 / 登录（本机账号）
├── admin.html            管理端（GitHub 令牌登录）
├── 404.html              404 页
├── assets/css/style.css  液态玻璃蓝色主题
├── assets/js/core.js     数据加载、Markdown 渲染、加密、GitHub 接口
├── assets/js/*.js        各页面逻辑
├── data/site.json        站点设置
├── data/posts.json       文章
├── data/downloads.json   下载资源（提取码只存加盐哈希）
├── files/                站内下载文件放这里
├── .nojekyll             关闭 Jekyll 处理，确保 assets 正常
└── README.md
```

## 一、部署到 GitHub Pages（约 5 分钟）

1. 新建仓库，例如 `boke`（公开仓库即可，Pages 免费）。
2. 把本目录全部内容推送到仓库默认分支（`main`）。
   ```bash
   git init
   git add .
   git commit -m "init blog"
   git branch -M main
   git remote add origin https://github.com/<你的用户名>/boke.git
   git push -u origin main
   ```
3. 仓库 → Settings → Pages：
   - Source 选择 `Deploy from a branch`
   - Branch 选择 `main`，目录选 `/ (root)`
   - 保存后等待 1 分钟左右
4. 访问 `https://<你的用户名>.github.io/boke/`
5. 如果仓库名就是 `<你的用户名>.github.io`，则直接访问 `https://<你的用户名>.github.io/`。

> 本工程全部使用相对路径，放在仓库根目录或 `docs/` 目录下都能正常打开。

## 二、创建管理端登录用的令牌（关键）

不要用账号密码登录管理端——静态站点没有服务端，密码校验没有意义。这里用 GitHub 细粒度令牌：

1. GitHub → Settings → Developer settings → **Personal access tokens → Fine-grained tokens**
2. Generate new token：
   - Repository access：Only select repositories → 选择你的博客仓库
   - Permissions → Repository permissions → **Contents: Read and write**
   - 其余权限一律不给
3. 复制令牌（`github_pat_…`，只显示一次）
4. 打开 `admin.html`，填写：
   - 仓库所有者：你的 GitHub 用户名
   - 仓库名：`boke`
   - 分支：`main`
   - 访问令牌：粘贴令牌
5. 登录后即可在页面上管理文章、下载资源与站点设置。

令牌只存在当前标签页的 `sessionStorage`，关闭标签页即失效，两小时后自动过期。
怀疑泄露时，在 GitHub 上直接吊销该令牌即可。

管理员账号密码：登录表单里还有「管理员账号 / 密码」两栏，**首次登录可以留空**；
登录成功后到「站点设置 → 管理员账号」设置即可。设置后每次登录都需要账号密码，
它作为界面入口的第二道门禁，与令牌共同生效。密码同样只保存 PBKDF2 加盐哈希。

## 三、发布文章

管理端 → 文章 → 新建文章：

- 标题、摘要、标签、封面色、日期、状态（已发布 / 草稿）
- 正文支持 Markdown 子集：`##` 标题、`-` 列表、`>` 引用、`**加粗**`、`` `代码` ``、` ``` ` 代码块、`[文字](https://…)`
- 右侧实时预览；保存后自动提交到 `data/posts.json`

## 四、下载专区与提取码

管理端 → 下载资源 → 新建资源：

- 下载方式：**站内文件**（文件先放到仓库 `files/`，路径写 `files/xxx.zip`）或 **外部链接**（填 https 地址）
- 提取码：需要时填写一个码；系统只保存 `sha256(salt:code)`，保存后无法查看原文，只能重置
- 访客在下载专区点击「获取下载」，输入正确提取码后才显示下载按钮

初始示例：
- `files/sample-notes.txt`，提取码 `boke2026`
- 外链示例，提取码 `glass2026`

### 能力边界（务必了解）

静态站点无法真正隐藏文件直链，提取码挡的是「顺手点开、被搜索引擎索引、随手转发」。
如需强保护，请把文件放到需要鉴权的对象存储，而不是依赖前端校验。

## 五、背景音乐

管理端 → 站点设置 → 背景音乐：

- 网易云：歌曲 / 歌单页面 →「生成外链播放器」→ 复制 iframe 的 `src`，形如
  `https://music.163.com/outchain/player?type=0&id=3778678&auto=0&height=66`
- QQ 音乐同理，域名需为 `y.qq.com` / `music.qq.com` / `i.y.qq.com`

前台右下角会出现「音乐」浮窗。`auto=0` 表示不自动播放，符合浏览器自动播放策略。

## 六、站点设置

站点名称、副标题、作者、头像文字、公告、页脚、每页文章数、社交外链、背景音乐，
全部在管理端「站点设置」里改，保存后写入 `data/site.json`，不需要碰代码。

## 七、安全设计

| 风险 | 处理 |
| --- | --- |
| XSS | 所有内容先转义再渲染；Markdown 只支持安全子集，链接仅允许 http/https/mailto |
| 令牌泄露 | 令牌只存 `sessionStorage`，2 小时过期，关闭标签页失效；不写入仓库、不写 localStorage |
| 越权写入 | 令牌用 Fine-grained，仅单仓库 Contents 读写，泄露影响范围最小 |
| 点击劫持 / 注入 | 全站 CSP：脚本仅同源、禁止 object、限制 iframe 域名、禁止内联脚本 |
| 密码泄露 | 用户密码 PBKDF2-SHA256（15 万次迭代 + 随机盐），本地只存哈希 |
| 暴力尝试 | 登录失败 5 次锁定 60 秒 |
| 外部跳转 | 全部 `rel="noopener noreferrer"`，`referrer` 策略收紧 |
| 搜索索引管理页 | `admin.html` 带 `noindex, nofollow` |
| 管理员账号 | 账号密码经 PBKDF2 加盐哈希写入 `data/admin.json`，只存哈希；连续 5 次失败锁定 60 秒 |
| 双层门禁 | 账号密码控制界面入口，GitHub 令牌控制实际写入；两者缺一不可发布内容 |

需要说明：用户注册是**本机账号**，只在当前浏览器生效，不具备跨设备能力——这是静态托管的固有限制，
不是故障。管理员身份与用户账号完全独立。

## 八、本地预览

```bash
python3 -m http.server 8080
# 打开 http://localhost:8080/
```

`crypto.subtle` 需要安全上下文，`localhost` 与 `https` 都满足。
用 `file://` 双击打开时，页面会用 `assets/js/seed.js` 的离线数据渲染排版，
但哈希相关功能（用户密码、提取码校验）不可用——请用本地服务器或线上地址测试这些功能。
`seed.js` 由当前 `data/*.json` 生成，仅作兜底；正式内容以 `data/` 为准。

管理端在没有令牌时，可点「本地预览模式」，修改只保存在当前浏览器，用于先看效果。

## 九、常见问题

**保存后线上没变化？**
Pages 构建通常 1 分钟内完成。到仓库 Actions / Pages 面板确认构建成功，再强制刷新页面。

**提示「令牌无效或已过期」？**
令牌过期、权限不足（需 Contents 读写）或仓库名写错。重新生成令牌后登录。

**音乐不显示？**
域名不在允许列表，或 `enabled` 未开启。检查站点设置里的播放器地址。

**文章页 404？**
本项目使用 `blog.html#/post/<slug>` 的哈希路由，不需要服务端重写规则；直接访问带 hash 的地址即可。