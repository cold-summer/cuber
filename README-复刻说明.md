# 魔方栈 Cuber · 本地复刻版

这是 [魔方栈 Cuber](http://cuber.90175.com/) 的完整本地副本，可以直接构建、离线运行，用来练魔方和读源码。

## 来源与授权

- 上游项目：**魔方栈 Cuber**，作者 **华哲辰**
  - Gitee：<https://gitee.com/huazhechen/cuber>
  - 在线演示：<http://cuber.90175.com/>
- 授权：**MIT License**（见 `LICENSE`，Copyright (c) 2018 huazhechen）

MIT 允许自由使用、修改、分发，条件是**保留原始版权声明与许可声明**。本目录中的 `LICENSE` 与源码中的作者署名均已保留，请勿删除。如果要再分发，请一并带上 `LICENSE`。

## 快速开始

```bash
cd /Users/icc/cc/cuber

npm install          # 安装依赖（首次较慢，见下方「npm 太慢」）
npm run build        # 产出到 dist/
npx serve dist       # 或者任意静态服务器
```

自带一个更省事的预览方式（当前已在跑）：

```bash
cd dist && python3 -m http.server 8099 --bind 127.0.0.1
# 打开 http://127.0.0.1:8099/
```

想在**手机/平板上练**（这个站支持多点触控，手机上体验其实更好），把绑定地址换成 `0.0.0.0`，
然后用电脑的局域网 IP 访问（`ipconfig getifaddr en0` 查看）：

```bash
cd dist && python3 -m http.server 8099 --bind 0.0.0.0
# 手机浏览器打开 http://<电脑局域网IP>:8099/
```

注意这会把页面暴露在同一个局域网内，家里/自己的网络再用。

开发模式（热更新）：

```bash
npm run watch        # http://localhost:8080
```

## 本地已验证可用

在真实浏览器里逐项点过，确认与线上站点一致：

| 能力 | 验证结果 |
| --- | --- |
| 四种模式渲染 | `?mode=` 首页 / `algs` / `director` / `helper` 全部正常，页面可见文本与线上逐字一致 |
| 3D 魔方 | three.js 正常出图，配色、透视、阴影与线上一致 |
| 键盘转层 | 按 `I`（=R）后 `history.moves` 由 0 变 1 |
| 撤销 | 点底部 backspace 按钮后 `moves` 回到 0 |
| 菜单 | 打开后为 `游戏 / 助手 / 教学 / 动画 / 阶数 / 控制 / 显示 / 镜头 / 配色 / 关于` |
| 公式播放 | `?mode=algs` 点播放，F2L-01 实际执行 `U R U' R'`，进度条与步进按钮联动 |
| 初始状态 | 与线上一致：开局随机打乱记在 `history.init`，不计入 `moves` |
| **求解器（Kociemba）** | 端到端跑通：`setup(打乱)` → `solver.solve()` → `push(解法)`，魔方回到六面纯色。20 步解法耗时约 25ms |
| 求解器校验能力 | 未着色（`"?"`）的输入被正确拒绝为 `error: invalid cube`，非法状态不会给出假解法 |

对照截图在 `verify/02-old-site-90175/`（线上）与 `verify/01-vue-webpack-local/`（本地）。

## 为在新环境跑起来做的改动

源码逻辑**一行未动**，只改了工程配置。全部改动如下：

1. **`package.json`** — `build` / `watch` 前面加了 `NODE_OPTIONS=--openssl-legacy-provider`。
   原因：Node 17+ 自带的 OpenSSL 3 移除了 `md4`，而 webpack 5.45 内部有十几处硬编码用 `md4` 算哈希，直接构建会抛
   `ERR_OSSL_EVP_UNSUPPORTED`。这个环境变量让 Node 重新启用 legacy provider（本机 Node v24.21.0 实测可用）。
   注意 `output.hashFunction` **改不掉**这个问题，因为 `md4` 在 webpack 内部多处硬编码。

2. **`webpack.config.js`** — 增加一个 12 行的内联 `ManifestPlugin`，把 `resource/manifest.json` 发到 `dist/`。
   用内联插件是为了不引入 `copy-webpack-plugin` 这类新依赖，保持依赖树和上游一致。

3. **`resource/manifest.json`（新增）+ `resource/index.html`** — 补上 PWA 清单和 `<link rel="manifest">`，
   对齐线上站点（线上就是靠这个支持「添加到主屏幕」的）。

## 与线上站点的差异

已知且仅有以下几处，均不影响使用：

| 项 | 说明 |
| --- | --- |
| OLL-18 公式 | 仓库里是 `(RU2'R2'FRF') (U2'M'URU'r')`，线上部署的是 `RD (r'U'rD') R'U' (R2'FRF'R)`。仓库版是作者后来的修订，本复刻用仓库版 |
| 打包产物 | 线上把 vendor 和 index 打成一个文件，仓库配置是 `splitChunks` 分成两个；功能无差别 |
| `index.html` | 线上是手写的 favicon 链接，本复刻用 html-webpack-plugin 注入，效果相同 |
| 百度统计 | `src/index.ts` 第 13–21 行会在页面加载时注入百度统计脚本上报到原作者的统计账号。源码保持原样，**本地使用建议注释掉这 9 行** |

## 源码导读

想读源码的话，先看 **[`docs/源码导读.md`](docs/源码导读.md)**（约 5700 行）。它由 21 个 agent 分两轮生成：
10 个读者各通读一个子系统写档案，再由独立核验者**打开源码逐条查证**每一句技术断言。
覆盖 `src/` 全部 54 个文件，写的是「谁调用谁、数据长什么样、循环怎么走」，并附跨模块接线图与完整性核查。

其中已经验证过的两个有价值的结论：

- `twister.finish()` 存在**潜在死循环**：若动作队列非空而当前没有任何补间动画
  （例如用户正按住某一层不放，`group.drag()` 已通过 `hold()` 抢到锁但不产生 tween），
  队列里的动作会因抢锁失败被 `unshift` 回去，外层 `while` 永不结束 → **浏览器卡死**。
  首页冷启动与 `Playbar.init()` 都会走到这段逻辑。
- `Viewport` 的滚轮缩放**不是**死代码。`wheel` 虽写作原型方法，但 `vue-class-component` 会把它
  收进 `options.methods`，Vue 的 `initMethods` 在 `initData` 之前就把它 bind 成真实 vm 的自有属性，
  所以构造期取到的是已绑定版本。这一条是核验者推翻档案初稿结论后确认的。

## 目录结构

```
src/
  index.ts            入口：解析 ?mode= 决定挂载哪个组件，初始化 Vuetify
  index.css           全局微调（Vuetify 组件尺寸/滑块过渡等）
  data.ts             偏好设置(PreferanceData)与配色(PaletteData)，存 localStorage
  cuber/              魔方本体（与界面无关）
    define.ts         面/颜色常量：FACE 枚举 + COLORS 调色板
    cubelet.ts        单个小方块：位置与朝向
    group.ts          按层/面把方块分组，支撑「一次转一层」
    cube.ts           整个魔方：阶数、状态、是否复原
    twister.ts        动作解析与执行（R U' Rw2 这类记号 → 方块旋转）
    tweener.ts        补间缓动与动画队列
    history.ts        操作历史：记录/合并同类步/撤销/计算步数
    controller.ts     交互控制：拖拽转视角、拖动方块转层
    world.ts          three.js 场景容器：相机、灯光、尺寸自适应
  solver/             Kociemba 式求解器
    CubieCube.ts      角块/棱块表示、move 表、乘法与逆元
    CoordCube.ts      各类坐标与剪枝表
    Solver.ts         分阶段迭代加深搜索
    Util.ts           打乱生成、facelet 解析、状态合法性校验
  common/             通用库
    gif.ts            自己实现的 GIF 编码（LZW + 调色板），用于导出动画
    zip.ts            打包导出
    bytes.ts          字节/位运算工具
    color.ts          配色体系（WCA 标准色与自定义）
    util.ts           杂项工具
  vue/                界面层（Vue 2 + Vuetify 2，TS 装饰器写法）
    Playground/       首页：计时器 + 打乱 + 历史 + 撤销
    Algs/             公式库：F2L/OLL/PLL 共 119 条，带 3D 演示与播放器
    Director/         动画制作：编排动作、截图、导出 GIF、生成分享链接
    Helper/           着色求解：照着实物涂色后求解
    Player/           播放器：打开别人分享的链接（?mode=player&data=...）
    Setting/          顶部导航与模式切换
    Viewport/         three.js 画布 + 鼠标/多点触控
    Playbar/          公式播放控制条
    Menu/             Order 阶数 / Control 控制 / Appear 显示 / Camera 镜头 / Palette 配色 / About 关于
```

## 四种模式

| 地址 | 用途 |
| --- | --- |
| `/` | 首页练习：计时器、随机打乱、历史、撤销、分享 |
| `/?mode=algs` | 公式库：119 条速拧公式，3D 演示、分步播放、可改公式 |
| `/?mode=director` | 动画制作：写动作脚本、截图、导出 GIF |
| `/?mode=helper` | 着色求解：按实物涂色后求解；若报错说明魔方扭角或装错了 |
| `/?mode=player&data=...` | 播放分享链接 |
| `/?mode=reset` | 清空 localStorage 并回到首页 |

公式库共 **119 条**：F2L 41 + OLL 57 + PLL 21。你改过的公式会存进 localStorage 的 `algs`。

## 键盘映射（从 `src/vue/Playground/index.ts` 的 keymap 提取）

| 按键 | 动作 | 按键 | 动作 | 按键 | 动作 |
| :---: | :--- | :---: | :--- | :---: | :--- |
| `I` | R | `J` | U | `D` | L |
| `K` | R' | `F` | U' | `E` | L' |
| `W` | B | `H` | F | `S` | D |
| `O` | B' | `G` | F' | `L` | D' |
| `U` | r | `V` | l | `Z` | d |
| `M` | r' | `R` | l' | `/` | d' |
| `C` | u' | `,` | u | `X` / `.` | M' |
| `5` / `6` | M | `T` / `Y` | x | `N` / `B` | x' |
| `P` | z | `A` | y' | `;` | y |
| `Q` | z' | `↑` | R | `↓` | R' |
| `←` | U | `→` | U' | `Backspace` | 撤销 |

宽层（多层一起转）用 `3`/`7` 把层数减一、`4`/`8` 加一，屏幕上会显示当前层数前缀，例如层数 3 时按 `I` 得到 `3R`。

## 练魔方的建议路线

1. 先在首页按空格启动计时、`🎲` 打乱，用键盘把魔方还原，把基本记号（R U F L D B 及带 `'` 的逆操作）练成肌肉记忆。
2. 进 `?mode=algs`，锁定 **F2L** 标签，逐条看 3D 演示，重点看「棱角块怎么配对、怎么插入」。41 条不用背，理解前 10 条的思路即可。
3. 再刷 **OLL 57 条** 和 **PLL 21 条**，用播放器的单步/快进把每条公式的方块轨迹看清。
4. 想改公式就在公式框里直接编辑，会自动保存到本地，改错了刷新即回到原公式（因为改的是 localStorage）。
5. 想分享或做教学 GIF，用 `?mode=director`。

## 常见问题

**npm 太慢** — 默认走 `registry.npmjs.org`，国内可能每个包几秒到几十秒。换镜像：

```bash
npm config set registry https://registry.npmmirror.com
npm install
```

**构建报 `ERR_OSSL_EVP_UNSUPPORTED`** — 说明 `NODE_OPTIONS=--openssl-legacy-provider` 没生效。
确认用的是本目录 `package.json` 里的脚本，或者手动加：

```bash
NODE_OPTIONS=--openssl-legacy-provider npx webpack --mode=production
```

用 Node 16 则不需要这个变量。

**`webpack server` 起不来** — webpack-dev-server 3.x 在 Node 17+ 同样受 `md4` 问题影响，`npm run watch` 已带好环境变量。
若想改用 Node 16（对这套 2021 年的依赖最友好），可以用 `nvm use 16`。

**改完没生效** — `dist/` 是构建产物，改 `src/` 后必须重新 `npm run build`，或者在 `npm run watch` 下改。

**想彻底重置** — 访问 `/?mode=reset` 清空 localStorage，或直接删浏览器站点数据。

## 一个容易踩的 API 语义（读源码时有用）

`Cube.twister` 有两个长得很像但语义完全不同的方法，写脚本或改造时极易搞混：

- `setup(exp)` —— **重置整个魔方**，再把 `exp` 当作"初始场景"应用上去，同时清空 `history` 并把 `exp` 记进 `history.init`。
  首页开局的随机打乱、公式库里切换公式用的都是它。所以连续调两次 `setup` 并不是"叠加两步"，
  而是"第二次把第一次的结果抹掉重新开始"。
- `push(exp)` —— 把动作**追加到当前状态**之上，走动画队列。要在已有魔方上执行一串公式，用这个。

`twist(action, fast, force)` 是底层单步执行，`fast`/`force` 决定是否跳过动画与是否等动画结束再重试。
判断魔方是否"忙"，可以看 `cube.locks` 里各轴的锁数量加上 `twister.queue.length` 是否为 0。
