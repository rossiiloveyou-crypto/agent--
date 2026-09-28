# 蓝图经验库 · 具体坑（L）

> **文档定位**：本库**只放 L（绑定具体技术栈）**——核心是某条具体命令 / 参数 / API / 库 / 配置项的行为细节，或**具体项目专有**的坑。
>
> **用法：按当前技术栈「检索」，不要通读本文件。**
> **写蓝图前的必读集不含本文件**：H/M 条目在 [BLUEPRINT-经验库-通用.md](BLUEPRINT-经验库-通用.md)；
> 全量档案（186 R + 10 X，逐条带 `文件:行号`）在 [BLUEPRINT-经验库-全量台账.md](BLUEPRINT-经验库-全量台账.md)。
>
> **三档分级定义（与通用库同一套判据）**：
>
> | 等级 | 判据（自评问句） | 读法 |
> |---|---|---|
> | **H** 跨环境通用 | 「换一种语言/框架，这条坑还成立吗？」→ 是 | 写蓝图前**必读**（在通用库） |
> | **M** 同类系统通用 | 「换一个同类系统/项目还成立吗？」→ 是，否 → L | 写蓝图前**必读**（在通用库） |
> | **L** 绑定具体技术栈 | 「核心是某条命令 / 某个 API / 某个库的行为吗？」→ 是 | **按当前技术栈检索，不通读**（在本文件） |
>
> **自检一句**：**换掉这一章的技术栈，这个坑就不成立了 → 它是 L，只留在本章**。
>
> **本库构成**：**54** 条 = 原 100 条取样里判为 L 的 **43** 条 + 台账附录 B 的 **10** 条 X 系列 + **1** 条原误判为 H 的（A15）。按技术栈分章：
>
> | 章 | 主题 | 条数 | 什么时候来查这一章 |
> |---|---|---|---|
> | §1 | git 与仓库操作 | 3 | 写回滚 / 合并 / 分支方案、要预估冲突数、或读冲突块方向时 |
> | §2 | Windows / PowerShell 环境 | 4 | 在 Windows 上写脚本、跑命令、处理编码 / 管道变量 / 退出码 / 后台任务时 |
> | §3 | Electron / Node 运行时 | 7 | 改桌面壳、动主进程/渲染进程边界、依赖异步上下文或落盘时机时 |
> | §4 | TypeScript / 构建与测试工具链 | 5 | 改类型声明、写/跑测试、动 mock 与测试夹具时 |
> | §5 | 本项目专有（Electron + TypeScript，本项目 cyrene-agent：渠道 / 记忆 / 群聊） | 32（5.1 渠道·群聊 **8** / 5.2 记忆·擦除 **18** / 5.3 桌面外壳·配置 **6**） | 在 cyrene-agent 里动渠道、群聊 transcript、记忆与区块、擦除与删除时 |
> | §6 | 其他具体库与 API（正则引擎） | 3 | 写正则、做字符串剥离/解析时 |
>
> **每条反例是五元组**：错误做法 → 症状 → 根因 → 正确做法 → 出处（`文件:行号`）。
> 键号 `类型字母+序号` 沿用原编号池，稳定不变，可被蓝图与报告交叉引用。

---

## §1 git 与仓库操作

**什么时候来查这一章**：写回滚方案、跑 `git merge` / 分支操作、处理冲突与提交状态、要预估冲突数时。

| # | 级 | 错误做法 | 症状 | 根因 | 正确做法 | 出处 |
|---|---|---|---|---|---|---|
| **X4** | L | 执行 `git merge-tree --write-tree --name-only refs/remotes/official/master HEAD`，把输出里的 `CONFLICT` 行数当作冲突总数 | **实测 = 0**（只输出一个树哈希 `8d3250ec…`，无任何 `CONFLICT` 行）→ 误判为"完美无冲突"。而真实合并有 **46** 个冲突 | `merge-tree A B` 的语义是「以 `merge-base(A,B)` 为基合 A 与 B」，此时 `merge-base = HEAD`，**HEAD 一侧无独有改动 → 退化合并**；更关键的是它**只读已提交的树，看不到未提交工作区** | **必须先物化工作区**：`$st = (git stash create "m" 2>$null | Select-Object -Last 1)`，再 `git merge-tree --write-tree --name-only refs/remotes/official/master $st`（得 46） | `../task-仓库合并/PHASE-2-完成报告.md:44,48,60-64；../task-仓库合并/BLUEPRINT-总蓝图.md:105-122` |
| **X8** | L | 在预估命令里带 `-c merge.renormalize=false` | A2 测出 **50** 而非 **46**（多出 4 个冲突） | 命令行 `-c` 覆盖了仓库里已配好的 `merge.renormalize=true`，于是测的不是真实合并行为 | 确认命令里**没有** `-c` 覆盖，用纯 `git merge-tree` 让仓库配置自然生效 | `../task-仓库合并/PHASE-2-合并配置对齐.md:267` |
| **X6** | L | 凭蓝图/文档描述判断"某行属于 ours 还是 theirs" | P4 实测：原以为 `ChatPage.tsx` 的 `useChannelMirrorEvents` 在 ours 侧，**实际在 THEIRS 侧**（HEAD 侧为空块）→ 对 P6 的指示方向相反；同一 `git status` 码下**两侧内容方向不固定** | 冲突标记的语义方向**不由 status 码唯一决定**，只能由 `<<<<<<< / ======= / >>>>>>>` 三行本身确定 | **必须实际展开三行**判断归属，不能凭文档描述推断；每个冲突块单独读，不要批量套用 | `../task-仓库合并/PHASE-5-移植适配.md:478-479；../task-仓库合并/PHASE-4-完成报告.md:415` |

---

## §2 Windows / PowerShell 环境

**什么时候来查这一章**：在 Windows 上写脚本、跑命令，处理 `&&`/`;` 语法、编码与 BOM、管道变量污染、退出码语义、后台任务时。

| # | 级 | 错误做法 | 症状 | 根因 | 正确做法 | 出处 |
|---|---|---|---|---|---|---|
| **X3** | L | 在脚本里用 `&&` / `||` 连接命令，或在括号内用 `;` 分隔语句，或用 `Set-Content -Encoding UTF8` 写中文再读回 | `Missing closing ')'`（括号内不能用 `;`）；`&&` 不可用直接报错；`-Encoding UTF8` 写出 BOM 而读回按 ANSI → **中文乱码** | 本机只有 PowerShell 5.1（无 `pwsh`），其解析器与编码默认值与 7.x 不同 | 用 `;` 分隔语句（不依赖 `&&`）；读文件一律 `[System.IO.File]::ReadAllText($p, [System.Text.Encoding]::UTF8)`；命令均拆为独立语句 | `../task-仓库合并/PHASE-4-处理删除类冲突.md:392；../task-仓库合并/PHASE-2-合并配置对齐.md:265；../task-仓库合并/BLUEPRINT-总蓝图.md:391` |
| **X2** | L | `git rm ... 2>&1` / `git add -A 2>&1` / `$st = git stash create ... 2>&1` | **stderr 的 warning 被灌进管道变量** —— `$st` 里塞满 `warning: …LF will be replaced…`，后续 `rev-parse` 报 `Filename too long`（已复现） | `2>&1` 把 stderr 合并进 stdout 流，于是"命令的输出"不再是"命令的结果" | 用 `2>$null` 抑制 stderr，或 `| Select-Object -Last 1` 只取最后一行 | `../task-仓库合并/PHASE-4-处理删除类冲突.md:391；../task-仓库合并/PHASE-3-完成报告.md:251` |
| **X1** | L | 用 `@($g | Where-Object Name -eq 'UU').Count` 数冲突条数（`$g` 是 `Group-Object` 的产物） | 结果**恒为 1**，导致冲突构成被误判为 1/1/1（PHASE-3-完成报告记为"**已复现**"） | `@()` 作用在**单个 GroupInfo 对象**上得到的是"含一个元素的数组"，`.Count` 恒为 1；分组结果不是"可计数的行集合" | 直接对**原始码列表**用 `$_ -match '^UU'` / `$_ -eq 'XX'` 计数，**不要** `Group-Object` 后再 `@()` 取 `.Count` | `../task-仓库合并/PHASE-4-处理删除类冲突.md:390；../task-仓库合并/PHASE-3-完成报告.md:260` |
| **X7** | L | 用 `robocopy /LOG:x` 记录日志并**以退出码**判断成功 | 带 `/LOG:` 时**日志恒为 2 字节空白**（仓库根有个名为 `NUL` 的 Windows 保留设备条目，无法枚举/重命名/删除，会吞掉日志写入）；退出码 9 可能是假警报 | Windows 保留设备名 `NUL` 在路径解析层吞掉写入；robocopy 的退出码语义（位标志）本身不表示成败 | 判定成功**看汇总的 `Files FAILED` 与 `Mismatch` 两列是否为 0**，不要只看退出码；要日志就用 `& robocopy.exe ... 2>&1 | Set-Content` 捕获 | `../task-仓库合并/BLUEPRINT-总蓝图.md:93；../task-仓库合并/PHASE-0-完成报告.md:218` |

---

## §3 Electron / Node 运行时

**什么时候来查这一章**：改桌面壳（弹窗 / 设置 / 启动闸门）、动主进程与渲染进程边界、依赖"同步 IO""异步上下文（ALS）""userData 落盘时机"这类运行时行为时。

| # | 级 | 错误做法 | 症状 | 根因 | 正确做法 | 出处 |
|---|---|---|---|---|---|---|
| **C13** | L | `showInputModal` 不传 `icon` 时执行 `iconEl.textContent = "<svg …>"` | 整段标记被当普通文字渲染，从弹窗铺满整个面板（用户看到的"乱码"） | 同一个"设置图标"语义在三个弹窗里实现不一致（`textContent` vs `innerHTML`），而默认值是 SVG 字符串 | 新增 `applyModalIcon(el, icon, fallback)`：以 `<` 开头走 `innerHTML`，否则走 `textContent`；默认图标提成常量 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1806-1817` |
| **C15** | L | 在设置框输入后不回车、不切焦点就去检查落盘 | `app-settings.json` 里**根本没有**该键，用的是默认值 → 误判"设置没生效" | 设置入口是 `change` 事件保存，`change` 只在回车/失焦时触发 | 输入后必须回车或切走焦点；核对该文件的 **mtime** 与键是否存在 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1928-1929` |
| **C17** | L | 用 `KeyedQueue` 串行化两条写路径来防竞态 | 队列拿不到，且两条写路径也不在队列里 → 串行化方案落空 | 真正的保证来自另一处：`history-log` **全部 IO 同步、零 `await`** → 纯同步重写不可能被打断 | 依据真实的同步性设计并发假设；**一旦在重写循环里加 `await`，这个保证立刻失效**（须在代码里留注释锁住） | `../2-证据.zip → SELF-update 260925/phase3-p3-erasure-and-console.md:169-183` |
| **E1** | L | 直接读 `dialog.showMessageBoxSync(...)` 的返回值为 `number` | 新版 Electron 返回 `{ response: number }`，直接比较数字会走错分支 | Electron 各版本该 API 返回类型不统一，类型声明不足以暴露运行期差异 | `typeof result === "number" ? result : result.response` | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:232` |
| **A18** | L | 用 ALS（AsyncLocalStorage）在作用域内取归属信息并透传到调度层 | 透传链的**终点在 ALS 作用域之外**——run 结束回调已离开发起请求时的上下文，取不到 | ALS 的生存期与"一次 run"不重合；把"请求期上下文"当成"run 全生命周期可用" | 显式沿调用链透传，把归属做成**参数**而不是隐式上下文 | `../2-证据.zip → SELF-update 260925/phase3-p2-l2-person-attribution.md:136` |
| **A34** | L | schema 闸门返回"用户主动退出"后，调用方按**普通错误**抛出 | 用户只是想退出应用，却看到"启动失败"红框——把用户的主动选择渲染成故障 | "受控退出"与"启动失败"共用同一个错误通道，UI 无法区分 | 抛专用哨兵错误（带 code），上层识别后**跳过错误框**直接受控退出 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1742` |
| **A35** | L | 假定存储一定能落盘（取不到 userData 时直接写） | 测试环境 / 启动早期写盘失败或抛错 | 把"运行环境必然具备某能力"当成前提，而单测与启动早期恰恰不满足 | 取不到路径时返回 `null`，退化为纯内存；与既有同类模块的约定保持一致 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1743` |

---

## §4 TypeScript / 构建与测试工具链

**什么时候来查这一章**：改类型声明与注入契约、写或跑 vitest 用例、动 mock 与被 `exclude` 的文件、增删落盘目录与测试夹具时。

| # | 级 | 错误做法 | 症状 | 根因 | 正确做法 | 出处 |
|---|---|---|---|---|---|---|
| **C11** | L | 顺手"整理"截断算术，改 `MAX_FILE_LINES + 1` 与 `slice` 写法 | 既有断言（截断后最老 `msg50`、最新 `msg249`）变红 | `+1` 是"实现细节与测试断言"的隐式契约，不属可读性优化范围 | 明确标注"不要动"；要动就先改测试并说明语义变化 | `../2-证据.zip → SELF-update 260925/phase1.5-speaker-attribution-patch.md:624` |
| **A25** | L | 用 `require()` 懒加载被测试的模块 | **该路径完全无法单测**——`require` 在 ESM 打包产物与 vitest 下都不可拦截 | `vi.mock` 只能拦 ESM `import`；`require` 在 ESM 产物里是另一套解析机制 | 改为动态 `import()`（懒加载语义不变，但可 mock）；排查全仓其他 `require()` 懒加载点 | `../2-证据.zip → SELF-update 260925/phase3-p2-l2-person-attribution.md:901,934-935` |
| **F8** | L | 新增了一个落盘目录，但测试夹具的清理清单没同步扩展 | 归档相关用例互相污染（上一用例的归档文件被下一用例读到） | 夹具的生命周期与产物的生命周期脱钩 | 每新增一个落盘目录，同批次把它加进所有相关套件的清理；更稳的是按"整个数据根目录"清理 | `../2-证据.zip → SELF-update 260925/phase1.5-speaker-attribution-patch.md:797` |
| **X5** | L | 跑测试时加 `--reporter=basic`（旧版本常用） | `Failed to load custom Reporter from basic` + `Failed to load url basic`，**整个 vitest 启动失败**（看起来像代码坏了） | reporter 名称随大版本变更（vitest 4.1.9 已删除 `basic`）；启动失败与测试失败在输出上不易区分 | 用默认 reporter 或 `--reporter=default`；**不要把这个启动错误误判为测试失败** | `../task-仓库合并/PHASE-5-移植适配.md:480；../task-仓库合并/PHASE-4-完成报告.md:414` |
| **X9** | L | 用 `npm test > test-result.log 2>&1 &`（cmd 后台）跑测试；随后改用 `--reporter=json --outputFile` | cmd 的 `&` 后台语法导致 **exitCode 语义混乱，无法确认测试完成**；`--reporter=json --outputFile` **未产出文件**（连续弯路） | 宿主 shell 的后台语义与"任务是否完成"的判定耦合；reporter 的输出落盘行为未经验证就依赖 | 用受管后台作业并显式轮询结果文件；**先验证 reporter 真的会写文件再用它做判据** | `../2-证据.zip → internal-issue/2026-09-06-shell-output-truncation-timeout-known-issues.md:36,38` |

---

## §5 本项目专有（Electron + TypeScript，本项目 cyrene-agent：渠道 / 记忆 / 群聊）

**什么时候来查这一章**：在 cyrene-agent 里动渠道与群聊 transcript、动记忆 / 区块（zone）/ 归属与擦除、动桌面外壳与配置、或对照本项目文档写判据时。
下面按三个子域分开列，命中哪个子域就只看那一张表。

### 5.1 渠道与群聊（QQ 渠道 / 群白名单 / transcript 格式）

**什么时候来查**：改渠道适配、群白名单、群上下文与旁听块、transcript 前缀与引用行、造群聊验证数据时。

| # | 级 | 错误做法 | 症状 | 根因 | 正确做法 | 出处 |
|---|---|---|---|---|---|---|
| **E2** | L | transcript 文件找不到时先怀疑代码 | `<userData>/channels/history/` 下没有群对应文件 | 两个前置条件没满足：群不在 `allowedGroupIds` 白名单；或改了配置**没重启**（配置在启动期读取） | ① 查 `channels-settings.json` 的 `qq.allowedGroupIds` ② 重启 ③ 发一条**不 @** 的消息再找最新修改的文件 | `../2-证据.zip → SELF-update 260925/phase1-manual-verification.md:244-252` |
| **E8** | L | 把"@ 昔涟"的消息拿去攒群上下文样本 | 旁听条数攒不起来 | `buildGroupContextBlock` 只统计 `observedOnly`（**没 @ 的旁听消息**），触发轮不进这个块 | 喂数据必须发普通消息（不 @、不命中触发词） | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1926-1927` |
| **F11** | L | 用"@ 昔涟"或"已触发"的消息构造验证数据 | 验证条件从根上不满足（该块只统计**没 @** 的旁听消息） | 构造数据的形态与判据统计口径不一致 | 先读清判据统计的是哪一类消息，再按该类构造数据 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1926-1927` |
| **A5** | L | 群聊写入 `msg.text`（丢引用行）或 `modelText` 整段（双前缀） | 前者丢引用行；后者得到 `[小明]: [群聊发送者：小明 (10001)]\n…` | `modelText` 同时含"发送者前缀"（元数据）与"引用行/触发提示行"（正文语义），一刀切都会错 | 写 `stripSpeakerPrefix(modelText)`——**只砍发送者前缀，保留引用行** | `../2-证据.zip → SELF-update 260925/phase1.5-speaker-attribution-patch.md:414,519-520` |
| **A29** | L | 群白名单并入配置时移除了旧输入框，却没补新入口；成员选择器数据源只含"已产生过对话的会话" | "想给一个从没说过话的新群加白"在 UI 上**无路可走** | 数据源被当成"全集"使用，而它其实是"**历史观测集**" | 区块卡片新增「手动加群」；**sessionId 由主进程现算，不接受渲染进程传入**，否则会出现"加白了但记忆是另一个域" | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1819-1829` |
| **A30** | L | 手动加群对某渠道接受纯数字群号 | 填群号**不报错但永远匹配不上任何群**（该渠道只认 openid）——等于写死配置、无法排查 | 校验只做了"格式合法性"，没做"该渠道的**语义可匹配性**" | 按渠道分别校验并**显式拒绝**不合法形态；规则放共享模块，渲染与主进程共用 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1831-1833` |
| **A36** | L | 成员选择器把"桌面对话"也列为可选项 | 与「桌面只属于 root」的契约冲突：桌面对话若可被选进自定义区块，就等于绕过 root 归属 | UI 清单与"域归属**不变量**"没有对齐；可选项集合必须由不变量约束，而不是"能列就列" | 只列外部会话；桌面对话在 root 卡片**只读**展示 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1779` |
| **A39** | L | 桌面会话为持久化成区成员而写进成员表 | 删除/重建桌面对话后留下**悬空引用** | 桌面隐式属于 root，把它物化成"成员记录"会引入一个本不需要维护的引用关系 | 桌面**直接返回 root**，不做成员存储，从数据模型上消除悬空引用的可能 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1740` |

### 5.2 记忆与区块 / 归属与擦除（scope / 向量 / personKey / 删除）

**什么时候来查**：改记忆写入与召回、区块（zone）与域过滤、归属字段（speakerIds / subjectIds）与按人擦除、向量库与级联删除时。

| # | 级 | 错误做法 | 症状 | 根因 | 正确做法 | 出处 |
|---|---|---|---|---|---|---|
| **C19** | L | 把某个纯函数的两条读入口当"暂时不可用就整体失败" | RAG / 关系日志暂时不可用会把整次擦除打挂 | 辅助读入口的失败面被放大成主流程的失败面 | 两个新读入口都包 `safe*()`——读不到就当空，有用例锁住 | `../2-证据.zip → SELF-update 260925/phase3-p3-erasure-and-console.md`（§9.3e 第 25 条） |
| **E4** | L | 看到每轮打印 `RAG not initialized` / `reconciliation skipped` / `StickerEmbedding Model not found` 就当成回归 | `searchMemoryEntries` 返回 `[]`、L2 全 `sync_failed`、`ragId` 为空、`memory-store.json` 永不生成 | `models` 是空目录（只有 `.gitignore`）→ `getEmbeddingProvider()` 返回 null → `addL2MemoryVector` 第一行就 throw。**是 dev 环境既定状态，不是回归** | 记录为"环境说明（非缺陷）"，并写明它对结论的影响：向量召回路径不可用，隔离只能靠 L2 注入路径 + 单测 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:2011-2042` |
| **D7** | L | 群聊判定写成 `if (sessionId?.includes(":group:"))` | 真实 sessionId 是 `channel:qq:<sha256 前16位>`，**不含 `:group:`** → 群上下文注入永远不触发 | 用"看起来像"的字符串片段当类型判据，而 sessionId 是哈希派生的不透明 id | 由 `IncomingMessage.chatType`（或渠道侧显式字段）判定；需从 id 反推时走权威映射函数 | `../2-证据.zip → SELF-update 260925/phase1-group-context-observe.md:104` |
| **D14** | L | 指望 judge 把"跨批提到的人"也映射成 `personKey` | `subjectNames → subjectIds` 只能依赖"本批说过话的人"名册 → 跨批提到的人无法归属、被丢弃 | 名册构造范围是"本批 turns"（受 prompt 长度与判定窗口限制），而人名可以出现在任何一批 | 把限制**写进 judge prompt**，并明确"这是可接受的降级，不是 bug"；映射不上就丢弃 | `../2-证据.zip → SELF-update 260925/phase3-p2-l2-person-attribution.md:211` |
| **D16** | L | 把"场景全绿"当作某条路径已被验证 | **`【相关记忆】`（L2 按域注入）从未真正触发过**——两条 L2 `weight=0`、未达阈值 | 验证样本的激活强度不足，导致"有内容的路径"始终没有内容 | 覆盖表里**显式登记未覆盖项**并写明替代防线；不要用"总表全绿"掩盖"某路径从未被执行" | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:2274-2279` |
| **F12** | L | 复验时把两个待比较的群放进**同一个区块** | 验证退化成"同区块共享"而不是"跨域隔离"（此前已误判过一次） | 实验分组没有区分"同域内共享"与"跨域不串"这两个待证命题 | 让每个域两两不同（一个 solo、一个区块、一个 root），顺带覆盖两个方向都不串 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:2201-2202` |
| **A9** | L | `compressMemories()` 在**全局 L2** 上聚类；总结**不带 scope** | 群聊记忆与桌面记忆被合并成同一条总结 → 既串味，又从此召回不到 | 聚类候选集没按域分桶；"总结不带 scope"让它在分域读取下成为不可见数据 | 先按 `scope` 分桶再聚类；总结继承 `group[0].l2.scope` | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1752` |
| **A11** | L | 隔离清单按"文件/模块"枚举，漏掉 `tool-registry.ts` 的 `user_memory` / `read_memory` / `write_memory` | 域隔离在这条路径上完全失效——**而这正是后来认定的串记忆主战场** | 真正的隔离面是"**所有读写该数据的入口**"，工具层被当成"上层"漏掉了 | 三者都按 `ctx.conversationId` 解析域：读只读本域、写给候选钉上域 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1756` |
| **A12** | L | 删除后**不删向量**，只删结构化条目 | 被删记忆的向量留在库里，语义召回仍能命中一条**已经不存在的**记忆；只有下次启动对账才回收 | `deleteL2Cascade` 的契约是**有意不删向量**（顺序必须"先 store 后 vector"），调用方要自己拿 `removed[].ragId` 去删 | 调用方补这一刀；且要**逐个调用点核对**——三个调用点接了、控制台那一处没接 | `../2-证据.zip → SELF-update 260925/phase3-p3-erasure-and-console.md:1585-1592` |
| **A13** | L | 只擦"原始载体"，不管**向量/副本载体** | 向量库 19 → 14，消失的全是 A 类；B 类（对话向量）**12 条一条没动**，其中 8 条与该人有关，仍可被按域语义召回到**已删掉的经历** | 痕迹**同时存在于原始载体与向量副本**；而且启动对账的视野里根本没有 B 类，**不会自愈** | 按"痕迹的形态"逐类清点（见 §5 形态矩阵思路）；只要某痕迹同时存在于原始与副本载体，**单删原始载体就等于没删** | `../2-证据.zip → SELF-update 260925/phase3-p3-erasure-and-console.md:1599-1604` |
| **A14** | L | 逐行擦除的判据是 `speakerId === <被擦者>` | 擦除后 7 分钟被问及，她答出了**已删掉的经历**——泄漏来自她自己复述他的行，而**这些行没有 `speakerId`**，一条都没动 | assistant 行不带 speakerId，判据**结构上抓不到它**；而这类行**每轮都进上下文**，比向量召回更直接 | 判据加两条：① 正文含别名 ② **轮次配对**（这一行的上一行就是他的行）。**② 是必须的**——泄漏那行一个字都没提他 | `../2-证据.zip → SELF-update 260925/phase3-p3-erasure-and-console.md:1622-1646` |
| **A17** | L | 只在"自动注入"通道上治串记忆，方案重心放在 `buildMemoryInjection` | 全仓核实后推翻：该函数**只有两个调用方**（语音通话、主动消息），主聊天走 `buildAlwaysOnContext` **不调用它** → 整套 L2 + DMAE 在主聊天路径上**空转** | 真实主战场是**工具路径**：模型调 `user_memory` → 检索池里混着所有人 → 把小红的事当小明的事说出来，**且直接当成"你说过的话"复述** | 承认"串记忆没消失，只是换了扇门，而且更严重"；把本阶段**收窄为纯铺垫**，召回侧独立成下一阶段 | `../2-证据.zip → SELF-update 260925/phase3-p2-l2-person-attribution.md:61-103,519-537` |
| **A20** | L | 让"无归属"与"有字段但为空数组"在磁盘上呈两种形状 | 读取侧要判两种形态；回归比对会误报 | 序列化层把"空"与"缺席"当成不同的东西，语义上它们是同一件事 | **空数组不落字段**；按需展开字段（不写 `undefined` 键），用 `Object.keys` 断言锁住形状 | `../2-证据.zip → SELF-update 260925/phase3-p2-l2-person-attribution.md:911,913` |
| **A21** | L | 只给部分字段定退化规则，`speakerIds` 遇到"来源轮取不到"时留空 | 一批记忆没有说话人指针，"谁说的"不可溯 | 同类字段的降级策略不统一 → 最需要兜底的字段反而没有兜底 | 采用同一规则：缺失时退化为整批——**"一批指向"优于"没有指针"** | `../2-证据.zip → SELF-update 260925/phase3-p2-l2-person-attribution.md:908` |
| **A23** | L | 设计时假定"记忆写入时 assistant 消息的 id 已存在"，让 `sourceMessageIds` 同时指向两侧 | 真实顺序是 user 落盘 → run（期间触发写入）→ assistant 才落盘 → **触发时 assistant 的 id 还不存在** | 把"run 结束后"当成了"消息都写完了" | 记忆只需指向 **user 消息**（判定本就要求"必须是用户主动表达的信息"） | `../2-证据.zip → SELF-update 260925/phase3-p1-message-identity.md:83` |
| **A28** | L | 往 `AgentRunFinishedContext` 加字段，未考虑它与插件 `turn:completed` 事件**同源** | 存在顺带把群成员标识泄进**第三方插件事件流**的风险 | 同一个 context 对象被两条消费者链共享（内部链路 vs 插件契约），加字段等于同时扩了两边接口 | 加用例锁住"该字段只走内部链路、不进插件事件契约"；改共享 context 必须逐个消费者确认边界 | `../2-证据.zip → SELF-update 260925/phase3-p2-l2-person-attribution.md:904` |
| **A32** | L | 把"清空/删除"与"按人删除"当同族功能复用实现 | 两者**语义不同、作用域重叠**：一个是目录级、一个是行级过滤重写；共用实现会让"删一个人"顺手动到别人的记录 | 复用点只能落在"路径清单"，**绝不能落在"删除动作"** | 明确两者不能共用一套实现，并用架构守卫测试断言"擦除链路源码里不许出现整体删除函数" | `../2-证据.zip → SELF-update 260925/phase3-p1-message-identity.md:642-645` |
| **X10** | L | 依赖 `history-log` 的文件名反解（`safeName` / `sessionIdFromFileName`）把文件名还原成 sessionId | 该审计项在 merge-plan 里**反复登记为"未审计"**（"官方 `chats-store.ts` 重写是否改变会话 ID 空间"），而它直接决定反解是否还成立；同文件另指出渠道名捕获组 `[a-z][a-z0-9]*` **不含下划线**，与 `safeName` 把 `:` 换成 `_` 会发生歧义 | 文件名编码是**多对一**的，反解本就不可靠；再叠加会话 id 空间可能被上游重写，风险翻倍 | 用权威名册而非反解；把"id 空间是否变化"作为上游重写的必查项而非待办 | `../task-仓库合并/04-审计-chats-store与会话ID空间.md:52,73,118；../task-仓库合并/01-已拍板决策与技术修正.md:138` |

### 5.3 桌面外壳 / 配置 / 项目文档载体

**什么时候来查**：改窗口与本地化、改配置落盘与启动读取、写"删除类"文案、清理"暂时没人用"的项目代码、动面板之间的引用时。

| # | 级 | 错误做法 | 症状 | 根因 | 正确做法 | 出处 |
|---|---|---|---|---|---|---|
| **C12** | L | 手工验证前不备份，直接改配置/发消息 | 老 transcript 被启动闸门清空 → 验收项失去真实样本 | 兼容性分析只看了"格式解析"，没看"生命周期"——数据可能被别的功能的删除路径带走 | 做 transcript 相关手工验证前先备份 `channels/`；`channels-settings.json` 改完**必须重启**（配置在启动期读取） | `../2-证据.zip → SELF-update 260925/phase1-manual-verification.md:244-252` |
| **E3** | L | 靠 `logs/cyrene.log` 排查 | 该文件 36 行后**静默停止落盘**（此后两次正常启动 + 十余轮渠道运行都没写进去），而 `memory-trace.log` 正常追加 | 主日志在某个时间点静默失效，表现是"看起来一切正常" | 排查改看 dev 终端与 `memory-trace.log`；把主日志降级为"待查项" | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:2044-2049` |
| **E7** | L | 假定 `index.html` 里的 `data-i18n` 短 key 会生效 | **不生效**——界面靠 HTML 中文硬编码兜底，多语言切换在该窗口无效 | `applyTranslations` 全仓无调用点，`data-i18n` 属性没有任何运行时消费者 | 静态骨架直接写中文，动态文本用 TS 侧 `t("settings.…")`；知其当前是死属性 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1780` |
| **D10** | L | 删除类文案按"记忆"这个词的直觉写："旧记忆将被清空（桌面对话记录会保留）" | **没提渠道聊天记录也会一起没** —— 文案与 `MEMORY_TARGETS` 实际范围不一致，用户在被删之前没被告知 | 文案由功能作者凭直觉写，没对着目标清单逐项核对 | 删除类文案必须由目标清单生成或逐项核对；预演报告显式列出保留项与"整份销毁"项 | `../2-证据.zip → SELF-update 260925/phase3-p1-message-identity.md:646-647` |
| **A16** | L | 把"目前没有生产消费者的纯函数"当无用代码删掉 | 下一阶段的擦除流程要重写一遍"从 sessionId 取渠道"的逻辑 | "无消费者"被当成"无用"，但它的消费者在**下一个阶段**（输入清单里已列） | 按完整规格实现并配用例，**不删**；文档注明"先于消费者的纯函数"以防后人清理 | `../2-证据.zip → SELF-update 260925/phase3-p2-l2-person-attribution.md:912` |
| **A33** | L | 为实现跨面板跳转直接 `import` 顶点入口模块 | 循环依赖成环 | 面板之间通过顶层入口互相引用，必然成环 | 新增独立的跳转钩子模块；风险在蓝图里已列出、缓解方式写的是"注入式，不 import 入口" | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:1777` |

---

## §6 其他具体库与 API（正则引擎）

**什么时候来查这一章**：写正则（尤其做前缀剥离、定界、贪婪/惰性选择）或做字符串切片解析时。

| # | 级 | 错误做法 | 症状 | 根因 | 正确做法 | 出处 |
|---|---|---|---|---|---|---|
| **A15** | L | 正则用"第一个 `]`"当分隔符（`[^\]\n]+`）匹配昵称 | 对 `[b°t]BEIKIA` 只剥掉 `[群聊发送者：[b°t]`，剩下 `BEIKIA (2914636187)]\n111`，**且已污染下游**（`memory.json` 里两条记忆的 `triggerText` 就是这个残片） | 昵称本身可以含 `]`；终止符没有锚定到"行尾 / `(数字)` 之前"这一结构性位置 | lookahead 锚定；**两个文件（写入侧与读取侧）都要镜像修改**，且两处 lookahead 故意不同 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:2051-2072` |
| **C8** | L | 剥离前缀写成贪婪 `[^\n]*` | 对**无换行的旧记录**会一路吞到最后一个 `]`，把用户正文吃掉 | 贪婪在"有换行的新格式"上恰好正确、在旧格式上错误；单用例通过就写进文档 | 用 lookahead 把分隔锚定为"行尾 / `(数字)` 之前"：`(?=\n|$\|\(\d{1,32}\))`；文档**显式标注旧提案已失效**并指向新方案 | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:2091-2092` |
| **C9** | L | 剥离前缀依次尝试 `[^\]\n]+` → `[^\n]{0,300}?` → `[^\n]*` | 旧版在昵称内部的 `]` 收尾；**惰性有界照样错**；贪婪版吃掉旧记录正文 | "惰性/贪婪"只调节匹配长度偏好，不改变**候选起点**；本质是终止符需要结构性锚点 | lookahead 锚定 + 11 组真实格式（多括号昵称、无 QQ 号、legacy 标记、带引用、命中触发词）验证 span | `../2-证据.zip → SELF-update 260925/phase2-zones-and-memory-isolation.md:2140-2163` |

---

## 追加模板

发现新的 L 反例时，追加到**对应技术栈章**（键号续编，不要重排已有编号；判不准是 H/M 还是 L 时先看通用库抬头的分级定义）：

```markdown
| **<类型字母><序号>** | L | <错误做法：具体到命令/代码/配置项> | <症状：含原样错误信息或真实数字> | <根因：机制层，指向该技术栈的固有行为> | <正确做法：可执行> | `<文件:行号>` |
```

判为 H/M 的条目请追加到 [BLUEPRINT-经验库-通用.md](BLUEPRINT-经验库-通用.md)，不要留在本库；两处都要回到 `BLUEPRINT-经验库-全量台账.md` 补一行 `- 等级:`。
