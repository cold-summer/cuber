# Fork 说明 · cold-summer/cuber

这是 [huazhechen/cuber](https://github.com/huazhechen/cuber)（魔方栈）的 fork，用于对照学习。
上游是 MIT 协议，`LICENSE` 与作者署名均原样保留。

## 这个项目有三个版本，值得先弄清

排查过程中发现，魔方栈并不是一个版本，而是三个形态并存，这是最容易踩的坑：

| # | 版本 | 技术栈 | 在哪 | 状态 |
|---|---|---|---|---|
| 1 | **Vue 2 + Webpack 旧版** | Vue 2.6 + Vuetify 2.5 + Three 0.130 + Webpack 5 | [Gitee `huazhechen/cuber`](https://gitee.com/huazhechen/cuber) | 源码完整可构建 |
| 2 | **旧版线上站** | 同上（构建产物） | <http://cuber.90175.com/> | 能访问，但**不是最新代码** |
| 3 | **React 19 + Vite 重写版** | React 19 + Vite 8 + Three 0.185 + lucide-react | 本仓库 `master`、<https://cuber.cheesefans.com> | 上游当前主力版本 |

**关键点**：`http://cuber.90175.com/` 看起来"就是那个网站"，但它跑的是**旧版 Vue 实现**，
而不是上游最新的代码。新代码在 <https://cuber.cheesefans.com>，界面已完全重写
（顶部计时器改成药丸形卡片、底部工具栏集中到一条白色圆角栏、图标从 Material Design 换成 lucide）。

所以「照着 cuber.90175.com 复刻」得到的是**版本 1 的形态**，不是上游最新状态。

## 本仓库做了什么

`master` 分支 = 上游 `master` + 一处修复：

```diff
- "typescript": "7.0.2",
+ "typescript": "^5.8.0",
```

原因：`package.json` 里钉的 `typescript@7.0.2` 与 `@typescript-eslint/parser@8.65.0` 的 peer 范围
（`>=4.8.4 <6.1.0`）冲突，导致**直接 `npm install` 会失败**：

```
npm error code ERESOLVE
npm error Could not resolve dependency:
npm error peer typescript@">=4.8.4 <6.1.0" from @typescript-eslint/parser@8.65.0
```

而且 TypeScript 当前稳定版是 5.8.x，不存在 7.0.2 这个版本号。改完之后：

```bash
npm install      # 不再需要 --legacy-peer-deps
npm run build    # tsc --noEmit && vite build 通过
npm run dev      # http://localhost:5173
```

其余文件与上游一致，未做任何改动。

## `vue-legacy` 分支

想看**版本 1（Vue 2 + Webpack）**的完整复刻，切到 `vue-legacy` 分支。那边包含：

- 与 `cuber.90175.com` 一致的 Vue 2 实现，可构建可运行
- `docs/源码导读.md`：约 5700 行的源码导读，覆盖 `src/` 全部 54 个文件，
  由多个 agent 通读后逐条对抗性核验生成
- 迁移到新环境所需的两处工程改动说明（Node 17+ 的 OpenSSL 3 与 webpack md4 冲突等）
- 与线上站点的逐项验证结果和对照截图

## 练魔方的话用哪个

学还原用哪个都行，公式库（F2L 41 + OLL 57 + PLL 21 = 119 条）两边一致。
想读源码、改造代码，`vue-legacy` 那套的中文导读更齐全；想用最新实现，用 `master`。
