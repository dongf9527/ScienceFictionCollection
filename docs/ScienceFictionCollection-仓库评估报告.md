# ScienceFictionCollection 及 Web 仓库评估报告

| 项目 | 内容 |
| --- | --- |
| 评估对象 | [VeejaLiu/ScienceFictionCollection](https://github.com/VeejaLiu/ScienceFictionCollection) · [VeejaLiu/ScienceFictionCollection-Web](https://github.com/VeejaLiu/ScienceFictionCollection-Web) |
| 评估基线 | 主仓库 `master@faf3afc`（2026-10-21）、Web 仓库 `master@c9340a0`（2026-10-21），均为完整克隆 |
| 评估目的 | 了解两个仓库的结构、内容及作用，研究二次开发可行性 |
| 评估方式 | 完整克隆后基于实际代码与 Git 历史分析（192 + 28 个提交全部纳入统计） |
| 报告日期 | 2026-10-31 |

---

## 1. 执行摘要

这是一套**"内容 + 阅读器"双仓库架构**的开源科幻小说在线阅读项目：主仓库存放小说原文与集中维护的元数据，Web 仓库是配套的静态阅读站，发布于 GitHub Pages。两者通过**兄弟目录约定**和 **raw.githubusercontent.com 按需取正文**两条纽带协作，设计克制、边界清晰。

- **内容规模**：17 位作者 + 1 种期刊（《科幻世界》），224 组作品、228 个阅读文件，213 个 TXT、14 个 EPUB、1 个 PDF；从刘慈欣、阿西莫夫、凡尔纳到特德·姜、王晋康，中英文本兼收。
- **工程质量**：高于典型的"网盘式小说仓库"——有元数据 schema（`schemaVersion 1`）、同步/生成脚本、严格可逆的编码转换工具、8 个单元测试文件、钉死的 Node/pnpm 版本、带双仓库 checkout 的 CI 发布流水线和完善的 SEO。
- **二开可行性：高**。Web 端是标准 Next.js 16 + React 19 静态导出应用，内容与展示通过 `metadata/` 彻底解耦，替换品牌/仓库常量即可自立门户；主要风险点是正文按需加载依赖 `raw.githubusercontent.com`（境内访问不稳定）以及小说正文的版权属性。
- **Clone 机制结论**：**保持两个独立仓库、以兄弟目录方式克隆**（项目官方即如此设计），不建议合并为单一仓库，详见第 6 节。

---

## 2. 仓库概览与相互关系

| 维度 | ScienceFictionCollection（主仓库/内容库） | ScienceFictionCollection-Web（阅读站） |
| --- | --- | --- |
| 角色 | 小说原文 + 元数据中心 | 在线阅读器 + 发布渠道 |
| 可见性 | 公开 | 公开 |
| 来源 | 独立仓库（非 fork） | **fork 自 [BiscuitCoder/github_novel_read](https://github.com/BiscuitCoder/github_novel_read)** |
| 创建时间 | 2021-12-02 | 2026-10-17（fork 后深度改造） |
| 最近推送 | 2026-10-21 | 2026-10-21 |
| 默认分支 | `master` | `master` |
| 体积 | 124 MB（其中 `.git` 60 MB） | 15 MB（其中 `.git` 13 MB） |
| 纳管文件 | 375 个 | 170 个 |
| 提交数 | 192 | 28 |
| 贡献者 | 10 人（VeejaLiu ~93%，其余零星贡献） | 3 人（keylen 15 / Veeja Liu 12 / BiscuitCoder 1） |
| Stars / Forks | 1171 / 151 | 0 / 0 |
| 许可 | MIT（代码与结构）；正文/封面版权另见 `LICENSE-SCOPE.md` | `LICENSE-SCOPE.md` + `licenses/`（第三方素材授权记录） |
| 主语言 | Python（仅工具脚本，内容为 txt/epub） | TypeScript 84.3% |

**两仓库关系**：不是主从模块关系，而是**生产者/消费者**关系——主仓库是唯一的"事实源"（single source of truth），Web 仓库在构建时读取其元数据、在运行时按需读取其正文。官方分工定义见主仓库 `docs/PROJECT.md` 的"两个仓库的分工"表格。

**提交时间线**（主仓库）：2021-12 建库（33 次提交）→ 2023-02 集中整理（25）→ 2024-11 起进入元数据治理期（49）→ 2026-10 冲刺（31）。Web 端：2026-03 上游改造期（15）→ 2026-10 发布期（12）。

---

## 3. 仓库结构（条目 1）

### 3.1 主仓库

```text
ScienceFictionCollection/
├── 001 - 刘慈欣(Cixin Liu)/            # 17 个作者目录：编号 + 中文名(英文名)
│   └── [年份]《书名》.txt               # 正文文件命名规范：[出版年]《标题》(别名).txt
│   └── 0-刘慈欣汇总.md                 # 作者小传/汇总页
├── 002 ~ 017 …（赫伯特/阿西莫夫/凡尔纳/雪莱/布拉德伯里/卡德/特德·姜/
│              斯万维克/王晋康/安迪·威尔/克拉克/柯南·道尔/威尔斯/
│              斯塔普尔顿/菲利普·K.迪克/雷·内勒）
├── 018 - Science Fiction World(科幻世界杂志社)/   # 期刊目录（2026 年第 5、6 期 EPUB）
├── CATALOG.md                          # 完整作品目录（script.py 自动生成）
├── README.md                           # 首页目录（script.py 自动生成）
├── LICENSE (MIT) + LICENSE-SCOPE.md    # 代码 MIT；正文/封面权利单独界定
├── convert.sh                          # GBK→UTF-8 严格可逆转换（原子写入）
├── script.py                           # 由目录+元数据生成 README/CATALOG
├── docs/PROJECT.md                     # 项目说明：两仓库分工、维护指南
└── metadata/                           # ★ 数据中枢（133 个文件，占仓库文件数 35%）
    ├── authors.json                    # 作者注册表（id/slug/多语言名/bio/来源）
    ├── periodicals.json                # 期刊注册表
    ├── books/<作者slug>.json ×17       # 每作者的作品清单（文件级元数据）
    ├── issues/<刊物slug>.json ×1       # 期刊分期清单
    ├── avatars/ ×19                    # 头像 WebP（最长边 160px）
    ├── covers/                         # 封面 WebP（≤600×900），按作者/作品/版本归档
    ├── publication-years/              # 出版年核对记录（verification.json 等）
    ├── acquisition/                    # 文本来源与权利记录
    ├── sync.py                         # 元数据同步（校验文件名/路径/大小一致性）
    └── README.md                       # 元信息维护指南
```

结构要点：

1. **编号目录即分类**：`NNN - 作者名` 双语命名，排序稳定、人读机读两便；作者下允许系列子目录（如阿西莫夫"基地三部曲"三层嵌套）。
2. **`metadata/` 是全仓库最有价值的部分**：228 个阅读文件每一条都有稳定 ID（如 `011-f00713b822e3`）、`workId`（跨语言版本关联）、slug、格式、语言、封面路径、来源链接。文件名与元数据的一致性由 `sync.py` 校验。
3. **README/CATALOG 不手工维护**：`script.py` 读取目录与 JSON 生成，保证目录即文档。
4. 正文文件命名带 `[年份]` 前缀并区分译本/版本，空文件（正文缺失）会被生成器显式标注。

### 3.2 Web 仓库

```text
ScienceFictionCollection-Web/
├── app/                    # Next.js App Router
│   ├── page.tsx            # 书架首页（默认作者作品 + 作者侧栏）
│   ├── authors/[slug]/     # 作者页（SSG 真实 HTML）
│   ├── books/[slug]/       # 作品/阅读页
│   ├── periodicals/        # 期刊入口与刊物页（按年、期号排序）
│   └── sitemap.ts / robots.ts
├── components/             # shadcn/ui 组件库 + 书架/阅读器业务组件
├── lib/                    # 核心逻辑
│   ├── site.ts             # ★ 内容仓库常量（CONTENT_REPO_OWNER/NAME）
│   ├── metadata.ts         # ★ 正文 URL 拼装（raw.githubusercontent.com）
│   ├── collection.ts       # 读取 catalog.json
│   └── i18n / reader-settings / reading-progress / text-chapters …
├── scripts/                # 10 个构建/守护脚本（.mjs）
│   ├── prepare-collection.mjs   # ★ 从兄弟目录拷贝 metadata → public/collection
│   ├── build-pages.mjs / finalize-export.mjs / verify-static-export.mjs
│   └── check-runtime.mjs        # Node/pnpm 版本守卫
├── tests/                  # 8 个 node --test 测试文件
├── docs/PROJECT.md         # 开发指南（含上游来源与授权）
├── licenses/ + ASSET_NOTICES.txt   # 第三方素材授权记录
├── .github/workflows/pages.yml     # ★ Pages 发布流水线
└── .env.example / .nvmrc / .node-version / pnpm-lock.yaml
```

---

## 4. 技术栈（条目 2）

### 4.1 主仓库

| 层 | 技术 | 说明 |
| --- | --- | --- |
| 内容 | TXT / EPUB / PDF / WebP | 213 txt + 14 epub + 1 pdf + 94 webp |
| 元数据 | JSON（自定 schema，`schemaVersion: 1`） | 作者/作品/期刊/头像/封面/核对记录 |
| 工具链 | Python 3 标准库（无第三方依赖） | `script.py` 文档生成、`sync.py` 元数据同步、`convert.sh` 内嵌 Python 做 GBK→UTF-8 严格可逆转换 |
| 质量 | 原子写入 + 回读校验 | 两个脚本均采用临时文件 + `os.replace`，转换前做 GBK 逐字节回环校验 |
| CI | 无 | 主仓库无工作流（纯内容库，合理） |

### 4.2 Web 仓库

| 层 | 选型 | 版本 |
| --- | --- | --- |
| 框架 | Next.js（App Router，`output` 静态导出） | 16.1.6 |
| UI 运行时 | React | 19.2.4 |
| 语言 | TypeScript（strict，`tsc --noEmit` 纳入脚本） | 5.7.3 |
| 样式 | Tailwind CSS + tw-animate-css | 4.2 |
| 组件库 | shadcn/ui（Radix UI 全家桶 + components.json） | — |
| EPUB 阅读 | epubjs（pnpm overrides 修补依赖） | 0.3.93 |
| 其他运行时 | zod、react-hook-form、recharts（上游遗留）、lucide-react | — |
| 包管理 | pnpm（`--frozen-lockfile`） | 9.15.2（钉死） |
| Node | `.nvmrc`/`.node-version`/`check-runtime.mjs` 三重约束 | 24.22 LTS（`>=24.22 <25`） |
| 测试 | Node 内置 test runner，8 个测试文件 | — |
| 发布 | GitHub Actions → GitHub Pages（push master 自动；支持 `workflow_dispatch` 指定内容分支） | — |
| SEO | SSG 真实 HTML、Person/Book/Periodical JSON-LD、sitemap、canonical、Open Graph | — |

**评价**：技术选型新而克制（Next 16 / React 19 / Tailwind 4 都是 2026 年当前主线版本）；版本钉得很死，换机成本高但复现性极好；`check-runtime.mjs` 在每个 npm script 前置版本守卫，是少见的细节。

---

## 5. 内容及作用（条目 3 / 主目的）

### 5.1 主仓库 = 可维护的"科幻文库"

不只是小说文件堆放：每一本书都有登记（格式、语言、译本、封面、来源、出版年核对状态）。官方统计（`docs/PROJECT.md`，与实际文件交叉验证一致）：**17 位作者、1 种期刊、224 组作品、2 期期刊、228 个阅读文件**；228 条记录中 78 条引用 76 张已核对封面，无封面的作品由阅读器即时绘制替代图。内容以中文文本为主（含简繁体登记），另有英文原版（威尔斯、菲利普·K.迪克、部分克拉克短篇等）。

### 5.2 Web 仓库 = 零后端的静态阅读站

- 书架/作者页/作品页全部构建期生成静态 HTML；打开 TXT/Markdown 时浏览器**按需请求该一本书的正文**并恢复本机阅读进度（按稳定文件 ID 存 localStorage，无账号）；EPUB 页内阅读（目录/翻页/位置恢复）；PDF 提供原始文件链接。
- 界面支持 8 种语言（简中/繁中/英/法/德/俄/日/韩），"界面语言 ≠ 正文翻译"，同一作品的多语言文件靠 `workId` 关联，版本切换策略为"当前界面语言 → 英文 → 任一已有"。

### 5.3 两条协作纽带（数据流）

```text
【构建期】主仓库 metadata/ ──(兄弟目录约定 ../ScienceFictionCollection 或
           COLLECTION_METADATA_DIR)──▶ prepare-collection.mjs
           ──▶ public/collection/（仅 JSON + 头像/封面，明确不读正文）
           ──▶ next build 静态导出 ──▶ GitHub Pages

【运行期】阅读页 ──▶ raw.githubusercontent.com/VeejaLiu/ScienceFictionCollection/
           master/<登记路径>   （lib/site.ts 常量 + lib/metadata.ts:144 拼装）

【CI 期】pages.yml 双 checkout：Web 全量 + 主仓库 sparse-checkout 只拉 metadata/
         → pnpm test → build:pages → check:export → deploy-pages
```

**评估结论**：这是一套完成度相当高的个人数字图书馆方案。内容治理（元数据、来源记录、版权界定、自动文档）做得比多数同类项目认真；Web 端把"静态站也能做阅读器"的边界推得比较远（EPUB 页内阅读、8 语言界面、JSON-LD、期刊 schema）。

---

## 6. Clone 机制分析（条目 4）

**结论：克隆到本地应保持两个独立仓库（兄弟目录并排放置），不要合并。**

这是项目**官方设计**而非权宜之计，证据有四处：

1. `ScienceFictionCollection-Web/docs/PROJECT.md` 的开发环境一节给出的命令就是**先后 clone 两个仓库**，然后进 Web 目录 `pnpm install`；
2. `Web/.env.example` 明示默认值 `../ScienceFictionCollection/metadata`（兄弟目录相对路径）；
3. `pages.yml` 在 CI 里也是双 checkout（主仓库还只 sparse-checkout `metadata`，说明官方刻意不把 124MB 正文拖进构建）；
4. 两个仓库都没有 `.gitmodules`——耦合只靠**路径约定**和 `lib/site.ts` 里两个常量，没有更深的机制。

### 三种方案对比

| 方案 | 做法 | 优点 | 缺点 | 评价 |
| --- | --- | --- | --- | --- |
| **A. 兄弟目录双 clone**（官方） | 见下方命令 | 零改动；与 CI、`.env.example` 默认值、文档全部对齐 | 需要记住"必须放同一父目录" | ★★★★★ 推荐 |
| B. 双 clone + 引导脚本 | 写一个 `bootstrap.sh` 一键 clone 并校验兄弟目录 | 消除人为失误；可顺带校验 Node/pnpm | 增加一个自维护脚本 | ★★★★ 可选增强 |
| C. git submodule | 把主仓库挂为 Web 的 submodule | 一条 `clone --recursive` 拉全 | 上游无此设计，需改 CI 与文档；submodule 心智负担高；主仓库 60MB 历史会被嵌进工作流 | ★★ 不建议 |
| D. 合并为 monorepo | 两仓库历史并入一个仓库 | 单仓库单发布流 | 破坏 fork 关系与上游同步能力；需重写 CI 双 checkout、路径常量、`.env.example`；主仓库 124MB 体积拖累 Web 仓库 clone | ★ 不建议 |

### 推荐操作（方案 A）

```bash
mkdir sfc && cd sfc
git clone https://github.com/VeejaLiu/ScienceFictionCollection.git
git clone https://github.com/VeejaLiu/ScienceFictionCollection-Web.git

cd ScienceFictionCollection-Web
nvm install && nvm use          # Node 24.22（.nvmrc）
pnpm install --frozen-lockfile  # pnpm 9.15.2（packageManager 钉死）
pnpm dev                        # 自动：check-runtime → prepare-collection → next dev (端口 3014)
```

正文按需拉取不需要本地主仓库存在（走 raw.githubusercontent），但**本地改内容时必须保留兄弟目录结构**，否则 `prepare-collection` 找不到 metadata。若主仓库不在旁边，用 `COLLECTION_METADATA_DIR=/绝对路径/ScienceFictionCollection/metadata` 覆盖。

---

## 7. 二次开发方向与可行性（条目 5）

按"投入产出比"排序，工作量与风险均标注：

### 方向一：品牌化自有部署（低成本，立即可行）★ 推荐起点

- **改动点**：`lib/site.ts` 两个常量（正文仓库 owner/name）、`.env.example` 的 `SITE_URL`、`components.json` 之外的少量品牌文案、Pages 工作流中的仓库地址。
- **路线**：fork 或新建自己的 Web 仓库 → 改常量指向自己 fork 的内容仓库 → 保留 GitHub Pages 或迁 Vercel/Cloudflare Pages（纯静态导出，迁移零障碍）。
- **可行性**：高。内容与展示的解耦就是为这种场景设计的。
- **工作量**：0.5～1 天（含验证）。

### 方向二：正文分发自主化（解决最大痛点）★ 二开前应优先决策

- **问题**：运行时正文走 `raw.githubusercontent.com`，境内直连不稳定，直接决定可用性。
- **方案**：① 保持 GitHub 但前置 CDN/代理（Cloudflare Workers 反代）；② 构建期把正文也打进静态产物（228 个文件约 60MB，Pages 可承受，但会偏离"不打包正文"的上游设计原则）；③ 正文挪对象存储（R2/OSS），`metadata` 增加 CDN 字段，改 `lib/metadata.ts` 一处。
- **可行性**：高，方案③改动最小且不破坏现有 schema（新增可选字段）。

### 方向三：内容库扩充与版本管理（持续性工作）

- `workId` 多语言/多译本机制已就绪：补英文原版、增补译本只需加文件 + 登记 JSON，前端自动出现版本切换。
- 可建 EPUB 化流水线：在 `convert.sh` 的清洗/校验思路上加 calibre/pandoc，把 TXT 批量转 EPUB（注意版权边界，仅自用）。
- 期刊线（《科幻世界》）刚起步（2 期），`issues/<slug>.json` 按年分期结构已能承载归档。

### 方向四：功能增强（中等工作量，前端为主）

- **全文搜索**：静态站用 Pagefind/lunr 建索引（注意：正文不在构建期可得的现状下，需在 CI 增加一个拉取正文的步骤，或只索引元数据）。
- **阅读体验**：字体/排版已有设置面板（reader-settings），可加 TTS 朗读（Web Speech API）、行间距主题、双页模式。
- **数据同步**：阅读进度目前仅 localStorage（按文件 ID，机制稳定），接 Supabase/Firebase 即可得跨设备进度，无需自建后端。

### 方向五：元数据治理自动化（防御性投资）

- 把 `sync.py --check` + JSON Schema 校验 + 图片规格检查（160px/600×900/WebP）接入两个仓库的 CI，挡住手工登记出错。
- 主仓库 README/CATALOG 已自动生成，可再让 `script.py` 输出 OpenGraph 卡片和 OPDS 目录（把文库变成 Kobo/Calibre 可订阅的书源）——改动集中在 `script.py`，风险低。

### 方向六：上游策略与合规（决策项，不是编码项）

- **上游同步**：Web 仓库 fork 自 `BiscuitCoder/github_novel_read`，但已深度改造（上游时代提交仅 15 个）。建议**放弃常规同步**，仅定期择取上游的安全修复（cherry-pick），避免 rebase 冲突成本。
- **版权边界**：正文与封面均为受保护内容，`LICENSE-SCOPE.md` 明确"书目或封面来源链接不代表开放许可"。二次开发若公开部署，需重新评估收录范围（公版书 vs 在版书）；自用/内网部署风险可控。二开发布时建议保留 LICENSE-SCOPE 的双层授权结构。
- **已知待办**：主仓库 4 个 open issues；《创世纪》正文缺失（已登记来源待核）；个别文件为 GBK 转换残留检查可用 `convert.sh` 全量跑一遍。

### 风险清单

| 风险 | 影响 | 缓解 |
| --- | --- | --- |
| `raw.githubusercontent.com` 境内不可达 | 站点核心功能不可用 | 见方向二 |
| Node 24 / pnpm 9.15.2 钉死 | 环境搭建受限 | 遵循 `.nvmrc`；升级需同步 5 处声明并重跑测试 |
| 主仓库 `.git` 60MB（历史含大文件） | clone 慢 | 评估时可用 `--filter=blob:limit=1m` 浅取内容 |
| 中文路径 + `core.quotePath` | 脚本统计/遍历易踩坑 | 脚本统一加 `-c core.quotePath=false` |
| Next 16 / React 19 较新 | 生态插件兼容 | 锁定 lockfile；升级前跑 `pnpm test` + `check:export` |

---

## 8. 附录：评估方法与证据

| 步骤 | 方法 |
| --- | --- |
| 元数据探针 | GitHub API（认证）获取两仓库可见性、fork 关系、体积、贡献度；输出脚本 `scripts/gh_probe.sh` |
| 完整克隆 | `git clone` 两仓库至沙箱 `/home/z/my-project/`（124MB + 15MB，含全部历史） |
| 结构统计 | `git ls-files` 按扩展名/目录聚合（`core.quotePath=false` 修正中文路径转义）；`du` 体积分布 |
| 历史分析 | `shortlog -sne`、按月提交分布、commit message 风格抽样 |
| 深读文件 | 主仓库 `README.md`、`docs/PROJECT.md`、`convert.sh`、`script.py`、`metadata/authors.json`、`metadata/books/*.json`；Web 仓库 `package.json`、`.env.example`、`scripts/prepare-collection.mjs`、`lib/site.ts`、`lib/metadata.ts`、`.github/workflows/pages.yml`、`docs/PROJECT.md` |
| 交叉验证 | 官方登记数字（228 阅读文件等）与实际文件清单一一核对；正文加载 URL 在源码中定位（`lib/metadata.ts:144`） |

> 本报告基于 2026-10-31 的 `master@faf3afc`（主）与 `master@c9340a0`（Web）快照，后续上游演进可能使个别结论过时。
