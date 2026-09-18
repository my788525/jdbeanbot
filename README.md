# 京东轻量京豆 · JDBeanBot

不依赖青龙面板的京东京豆自动化工具。**一个进程**完成全部工作：添加账号 → 轮换多账号运行京东脚本
→ 实时日志 → 按号统计京豆增量 → 控制台一键重启 / 在线更新脚本库 / 自动评价。

提供**两个平台的成品包**，都是解压/安装即用，不需要自己装 Node、不需要命令行：

| 平台 | 包 | 形态 | 说明 |
|---|---|---|---|
| Windows | `jdbeanbot-win-<版本>.zip` | 便携版 | 解压后双击 `start.bat`，**内置 Node 运行时**，离线可用 |
| 飞牛 fnOS | `jdbeanbot-<版本>.fpk` | 应用中心安装 | 内置 Linux Node 运行时，装完从飞牛桌面图标进入 |
| 飞牛 fnOS | `jdbeanbot-kernel-<版本>.tar.gz` | 内核升级包 | 只含代码，供**应用内自升级**用，一般不必手动下载 |

> 应用内自带**自动升级**（默认开启）：检测 GitHub Release 新版本 → 校验 → 覆盖升级 → 自动重启，
> 起不来还能一键回滚。详见[六、自动升级](#六自动升级)。

---

## 一、下载

到本仓库的 **Releases** 页面下载对应平台的文件：

| 文件 | 用途 |
|---|---|
| `jdbeanbot-win-1.0.5.zip` | Windows 便携版（约 54 MB） |
| `jdbeanbot-1.0.5.fpk` | 飞牛 fnOS 安装包（约 60 MB） |
| `jdbeanbot-kernel-1.0.5.tar.gz` | 飞牛内核升级包（自升级用，一般不必手动下载） |
| `*.sha256` | 每个包的校验和，**建议一起下载** |
| `SHA256SUMS.txt` | 三个包的校验和汇总 |

校验（可选但推荐）：

```bash
# Linux / macOS
sha256sum -c jdbeanbot-win-1.0.5.zip.sha256
# Windows PowerShell
(Get-FileHash .\jdbeanbot-win-1.0.5.zip -Algorithm SHA256).Hash
# 与 .sha256 文件里的值比对
```

---

## 二、Windows 部署（详细）

### 2.1 三步跑起来

1. 把 `jdbeanbot-win-1.0.5.zip` 解压到任意目录。解压出来是一个 **`JDBeanBot` 文件夹**，
   整个文件夹就是应用本体，可以随便挪、随便拷贝。
   - **别放在需要管理员权限才能写入的位置**（如 `C:\Program Files`）——升级要覆盖 `app\`；
   - **路径别太长**，且**中间别有怪字符**（全角引号、emoji 之类会让 .bat 出问题）；
   - 推荐位置：`D:\JDBeanBot`。
2. 进入文件夹，双击 **`start.bat`**。黑窗口就是运行日志，**关掉窗口 = 停止服务**。
3. 浏览器自动打开 <http://127.0.0.1:3100>。**首次进入请先设置一个控制台密码**（见 2.5）。

**不需要安装 Node、不需要联网** —— 包里的 `runtime\node.exe` 就是完整运行时。
万一它缺失，`start.bat` 会自动从 nodejs.org 下载安装（约 30 MB）。

停止：双击 **`stop.bat`**，或直接关掉那个黑窗口。

### 2.2 启动前自检（可选）

端口被占是最常见的起不来的原因。cmd 里查一下 3100 有没有被人用：

```bat
netstat -ano | findstr ":3100"
```

有输出且状态是 `LISTENING` → 换个端口（见 2.4）。

### 2.3 想让局域网其它设备访问（防火墙放行）

服务默认监听 `0.0.0.0`（局域网可访问），但 Windows 防火墙可能拦截入站。
以**管理员身份**运行一次：

```bat
netsh advfirewall firewall add rule name="JDBeanBot" dir=in action=allow protocol=TCP localport=3100
```

然后其它设备访问 `http://<你这台电脑的IP>:3100`。
⚠️ 一旦开放局域网访问，**控制台密码必须设置**（见 2.5）。

### 2.4 常用配置与环境变量

三种设法，任选其一：

```bat
rem ① 只对当前窗口生效（最简单，关掉就没了）
set PORT=3100
start.bat

rem ② 永久写入用户环境变量（setx 之后要**开新窗口**才生效，对双击启动同样有效）
setx PORT 3100

rem ③ 只想本机访问（改 HOST）
setx HOST 127.0.0.1
```

| 变量 | 默认 | 说明 |
|---|---|---|
| `PORT` | `3100` | 控制台端口 |
| `HOST` | `0.0.0.0` | 改 `127.0.0.1` = 只允许本机访问 |
| `NO_BROWSER` | 空 | 设 `1` = 启动时不自动弹浏览器（**开机自启时务必设**，否则每次开机弹一个网页） |
| `CONSOLE_AUTH` | 开启 | 设 `0` 关闭登录门禁（仅在完全可信的内网用） |
| `DATA_DIR` | `<应用目录>\data` | 数据目录，可重定向到别的盘 |

### 2.5 安全：务必先做这两件事之一

- **设置控制台密码**（打开页面后按引导设置），或
- 把 `HOST` 设成 `127.0.0.1` 只允许本机访问。

原因很直接：`data\accounts.json` 里存着京东 Cookie **明文**，服务又默认对局域网开放。
忘了密码：删掉 `data\console-auth.json` 再打开页面即可重新设置。

### 2.6 开机自启（推荐：任务计划程序）

**方式 A：启动文件夹（简单）**

1. `Win+R` 输入 `shell:startup` 回车；
2. 把 `start.bat` 的**快捷方式**放进去；
3. 右键快捷方式 → 属性 →「目标」末尾无需改动；但为了让它不弹浏览器，建议先执行一次
   `setx NO_BROWSER 1`。

**方式 B：任务计划程序（更稳，可后台运行）**

以当前用户运行一次即可：

```bat
schtasks /Create /F /TN "JDBeanBot" /SC ONLOGON /TR "cmd /c set NO_BROWSER=1 && D:\JDBeanBot\start.bat"
```

删除自启：`schtasks /Delete /TN "JDBeanBot" /F`。

### 2.7 数据备份与迁移

**你的全部数据都在 `JDBeanBot\data\` 一个目录里**。备份 = 拷走它；迁移 = 新机器解压包后把旧 `data\` 整个放回去。

```
JDBeanBot\
  start.bat / stop.bat    启动与停止
  runtime\node.exe        内置 Node 运行时（升级时会被覆盖，别往这里放东西）
  app\                    程序代码（升级时整体覆盖，别往这里放东西）
  data\                   ★ 你的数据：账号 / 运行记录 / 日志 / 脚本库 / 评价图池
  version.txt             当前版本号
```

**升级只覆盖 `app\` 与 `runtime\`，`data\` 永远不会被清掉。**

### 2.8 手动升级（不想用自动升级时）

1. 下载新版本的 zip，解压到一个**临时目录**；
2. 把解压出来的 `app\` 与 `runtime\` **覆盖**到现有 `JDBeanBot\` 里（`data\` 不要动）；
3. 重新运行 `start.bat`，到控制台「⬆️ 应用更新」里确认版本号已变。

### 2.9 排障：双击 start.bat 一闪就没了

黑窗口一闪而过时看不到报错。在文件夹地址栏输入 `cmd` 回车，然后手动运行：

```bat
start.bat
```

报错会停在窗口里。最常见的三种：端口被占（2.2）、路径里有特殊字符（2.1）、被杀毒软件拦截（把 `JDBeanBot` 目录加入白名单）。

---

## 三、飞牛 fnOS 部署（详细）

### 3.1 应用中心安装（推荐，全程图形界面）

1. 飞牛桌面 → **应用中心** → 右上角 **「手动安装」** → 选择 `jdbeanbot-1.0.5.fpk`；
2. 按向导填写：**控制台登录密码**（其它项保持默认即可）；
3. 等待安装完成，桌面会出现「京东轻量京豆」图标；
4. 点图标进入控制台，或直接访问 `http://<NAS的IP>:3100`。

### 3.2 SSH 命令行安装（适合远程/自动化）

```bash
ssh admin@<NAS的IP>
sudo -i   # 输入 admin 的密码

# 方式一：直接装（安装过程中会走向导问密码）
sudo appcenter-cli install-fpk /path/to/jdbeanbot-1.0.5.fpk -v 1

# 方式二：先准备向导参数文件，跳过交互（适合脚本化）
cat > /tmp/app-env <<'EOF'
wizard_console_password=你的控制台密码
EOF
sudo appcenter-cli install-fpk /path/to/jdbeanbot-1.0.5.fpk -e /tmp/app-env -v 1
rm -f /tmp/app-env    # 装完删掉，别把密码留在 /tmp
```

### 3.3 安装后验证

```bash
# 应用状态（running = 正常）
sudo appcenter-cli status jdbeanbot

# 健康检查（返回 JSON，ok:true 即正常）
curl -s http://127.0.0.1:3100/api/health
```

### 3.4 数据在哪、怎么备份

```
/vol1/@appcenter/jdbeanbot/        应用本体（代码与运行时）
  node/bin/node                    内置 Node 运行时（内核升级时不动）
  src/                             程序代码（自升级原地覆盖这一层）
  ui/                              桌面图标与入口
/vol1/@appdata/jdbeanbot/data/     ★ 你的数据（与代码完全分离）
  accounts.json                    账号与 Cookie
  runs.json                        运行记录
  app.log                          运行日志
  scripts/                         脚本库（在线更新写这里）
  review-photos/                   评价图池
  BeanCache/                       京豆余额快照
  .update/                         升级暂存与备份（backup-<版本> 可一键回滚）
```

- **数据全在 `@appdata` 下：升级、卸载、重装都不会清掉。**
- 备份：直接拷 `/vol1/@appdata/jdbeanbot/data/`，或用内置导出：

```bash
# 导出成一个归档（存到当前目录）
sudo /var/apps/jdbeanbot/target/cmd/main export
# 导入
sudo /var/apps/jdbeanbot/target/cmd/main import <归档.tar.gz>
```

- 卸载时还会额外把数据复制一份到 `@appdata/jdbeanbot-keep`，重装后自动回填（双保险）。

### 3.5 升级的三种途径

| 途径 | 操作 | 适用 |
|---|---|---|
| ① 自动（默认） | 什么都不用做。应用自己检测新版本，**只升级内核**（`src/`），不动 Node 运行时与数据 | 日常 |
| ② 控制台手动 | 打开控制台 →「⬆️ 应用更新」→「检查更新」→「立即升级」 | 想立刻升 |
| ③ fpk 重装 | 下载新 fpk → 应用中心手动安装，或 SSH 执行下面这段 | 大版本 / 升级失败兜底 |

⚠️ **途径③的坑（实测踩过）**：对**已安装**的应用直接再跑 `install-fpk` 会报成功但**什么都不更新**。
正确的重装顺序：

```bash
sudo appcenter-cli stop jdbeanbot
sudo appcenter-cli uninstall jdbeanbot
sudo appcenter-cli install-fpk /path/to/jdbeanbot-1.0.5.fpk -e /tmp/app-env -v 1
sudo appcenter-cli start jdbeanbot
```

（`@appdata` 下的数据在卸载时不会丢，放心执行。）

### 3.6 日志与排障

```bash
# 实时看运行日志
tail -f /vol1/@appdata/jdbeanbot/data/app.log

# 应用起不来时，看 fnOS 的启动日志
sudo appcenter-cli check jdbeanbot

# 进程在不在
ps -ef | grep -i jdbeanbot
```

常见问题：升级后版本号没变 → 八成是**旧进程还活着**，看 `/api/health` 里的 `pid` 变没变；
没变就 `stop` 后再 `start`。

---

## 四、控制台功能

| 面板 | 作用 |
|---|---|
| 添加账号 | 粘贴 **App 端 Cookie**（形如 `pt_key=app_open...`）或 App 扫码登录 |
| 账号列表 | 昵称 / 状态 / 当前京豆 / 最近运行 / **可编辑备注**；操作收成一个下拉菜单 |
| 自动运行 | 轮换多账号跑全部脚本；间隔、随机抖动可调 |
| 脚本库仓库 | 脚本来源（默认 `6dylan6/jdpro` 为主、`shufflewzc/faker2` 补齐）；在线更新、失效剔除 |
| 自动评价 | 独立模块，默认只生成不提交；**评价图池支持在控制台批量上传** |
| 应用更新 | 检测 GitHub Release 新版本；默认自动覆盖升级，也可改为一键手动 |
| 脚本健康 | 长期零收益或连续被风控的脚本自动停跑（可一键恢复） |
| 实时运行日志 | 刷新后仍回填最近 100 条；自动评价有独立持久化日志 |

其它：控制台登录密码（含「记住登录」）、一键重启服务、按号统计京豆增量。

### 京豆收益是怎么算的

统计口径是**账号京豆余额的前后差**，不是抓脚本输出里的数字。
原因很实际：脚本会打印资产总额、累计值等一堆数字，按文本匹配会把「+2」记成「1050」。
所以每个脚本执行前后各取一次真实余额，脚本之间互不污染。

---

## 五、账号与凭证（重要）

**2026 年起京东只认 App 端 Cookie**，形如：

```
pt_key=app_openXXXXXXXXXXXXXXXX; pt_pin=xxxxx;
```

- **不要把 `app_open` 前缀删掉** —— 删了就失效；
- 推荐直接粘贴整串（含分号也行，会自动解析）；
- 备选是 App 扫码登录，但受京东 App 版本门禁影响。

> ⚠️ 已知失效通道：`wskey` 免浏览器换票**已被京东封禁**（2026-09 实测）。
> 用 wskey 换出来的 `pt_key` 一律是 `fake_` 开头的诱饵，重抓 wskey 没有意义。

### 账号失效会自动停用

Cookie 真的失效后不再每轮全量空跑（白耗时间、还持续叠风控分）。

判定**不看脚本打印的文本** —— 脚本里大量「CK已失效 / 请登录」其实来自已死的接口，
同一 Cookie 换个脚本照样能领豆。真正的依据是权威校验接口的三态结果：

- `valid` → 清零
- `invalid` → 计数；**连续 3 次**、且两次计数至少隔 **30 分钟**，才自动停用
- `unknown` → **绝不计数**（接口挂了/被限流 ≠ 凭证失效）

自动恢复只作用于「自动停用」的账号，**你手动停用的绝不被自动启用**。

---

## 六、自动升级

### 默认行为

启动 **45 秒**后做第一次检查，之后**每 12 小时**一次：

```
读 https://api.github.com/repos/my788525/jdbeanbot/releases/latest
  → 版本号比较（按数字比，1.10 > 1.9）
  → 有新版 → 下载资产 → 校 sha256 → 解包（逐文件校 CRC 与长度）
  → 结构检查：新代码里必须有 server.js / config.js / package.json
  → 全过 → 备份旧代码 → 原地覆盖 → 自动重启
```

**检测到新版本就自动覆盖升级（默认开启，无需任何配置）。**
任何一步校验不过，就一个字都不写。升级前旧代码备份到 `data/.update/backup-<旧版本>/`，
升级后起不来，在控制台点「回滚」即可（同样需要一次重启）。

> **国内网络增强（v1.0.5 起）**：实测经常出现「api.github.com 可达（检查正常）、
> 但 releases 下载域不通」——症状是检查说有新版本、下载永远失败。
> 现在下载会**先直连、失败自动走镜像**（默认 `ghfast.top`，可用 `UPDATE_MIRROR` 换或设空禁用）。
> 镜像只负责搬运字节：下载结果仍要过 **sha256 校验和** + **GitHub API 声明的文件大小**两道关，
> 镜像本身换不掉内容。

### 双端两种形态

| 平台 | 方式 | 覆盖范围 |
|---|---|---|
| 飞牛 fnOS | **只升级内核** | 只替换 `src/` 代码。内置 Node 运行时、`@appdata` 数据、`BeanCache` 都不动 |
| Windows | 整包覆盖 | 覆盖 `app\` 与 `runtime\`，**跳过 `data\`**（账号、日志、脚本库全在里面） |

Windows 端为什么要多一步：`runtime\node.exe` 正在运行，**自己覆盖不了自己**。
所以升级会派一个**分离的批处理**：等服务进程树退出 → 覆盖文件 → 重新拉起。
页面会断开十几秒，属正常现象。

### 关掉自动 / 改成手动

```bash
UPDATE_AUTO_INSTALL=0    # 只提示，需在控制台点「立即升级」
UPDATE_ENABLED=0         # 完全关闭检查（连提示都没有）
UPDATE_INTERVAL_HOURS=6  # 改检查频率
```

> 开发者注意：在**源码目录**里跑会自动降级为「只提示」——因为自动覆盖会把不在发布包里的
> 测试件（`_t-*.js`）清掉。控制台会明确显示「已降级（开发目录）」及原因；
> 要强测自动流程可设 `JD_BEAN_ALLOW_DEV_AUTO=1`。

### 保护清单（升级永远不会碰的东西）

账号 Cookie、运行记录、日志、脚本库、评价图池、京豆余额快照（`BeanCache/`）、
`.update/` 里的历史备份。飞牛端连内置 Node 运行时也不动。

### 发布方需要遵守的约定

打一个 tag（如 `v1.0.5`），并上传这些资产：

| 资产名 | 必需 |
|---|---|
| `jdbeanbot-kernel-<版本>.tar.gz` | 飞牛内核升级需要 |
| `jdbeanbot-win-<版本>.zip` | Windows 升级需要 |
| `jdbeanbot-<版本>.fpk` | 首次安装 / 整包重装 |
| 上述每个包的 `<资产名>.sha256` | **强烈建议**（缺了就只能退化为校验包内 CRC） |

`<版本>` 必须与 `app/VERSION` 一致，格式 `x.y.z`。

---

## 七、环境变量参考

| 变量 | 默认 | 说明 |
|---|---|---|
| `PORT` | `3100` | 控制台端口 |
| `HOST` | `0.0.0.0` | 监听地址；改 `127.0.0.1` 只允许本机访问 |
| `NO_BROWSER` | 空 | Windows 端设 `1` = 启动不弹浏览器（自启场景建议） |
| `DATA_DIR` | `<应用目录>/data` | 数据目录，可重定向到别的盘 |
| `CONSOLE_AUTH` | 开启 | 设 `0` 关闭登录门禁（**仅在完全可信的内网里用**） |
| `CONSOLE_AUTH_DAYS` | `365` | 「记住登录」有效期（天） |
| `SCHEDULE_HOURS` | `6` | 自动运行间隔（小时） |
| `CONCURRENCY` | `1` | 并发账号数（建议保持 1，多开容易触发风控） |
| `RUN_TIMEOUT` | `180` | 单脚本超时（秒） |
| `ACCOUNT_GUARD` | 开启 | 设 `0` 关闭「账号失效自动停用」 |
| `ACCOUNT_GUARD_INVALID` | `3` | 连续几次权威校验失败才停用 |
| `LIB_UPDATE_HOUR` | `4` | 脚本库自动更新时刻（`04:30`） |
| `UPDATE_ENABLED` | 开启 | 设 `0` 关闭自升级检查 |
| `UPDATE_REPO` | `my788525/jdbeanbot` | 发布仓库，`owner/repo` |
| `UPDATE_AUTO_INSTALL` | **自动安装** | 设 `0` = 只提示不自动装 |
| `UPDATE_ON_START` | 开启 | 设 `0` = 启动时不做首次检查 |
| `UPDATE_START_DELAY` | `45` | 启动后首次检查延迟（秒） |
| `UPDATE_INTERVAL_HOURS` | `12` | 检查间隔（小时） |
| `UPDATE_FNOS_KERNEL` | `1` | 飞牛端设 `0` 可改为整包升级 |
| `UPDATE_KEEP_BACKUPS` | `2` | 保留几个历史备份 |
| `UPDATE_TOKEN` | 空 | 私有仓库 / 提高 API 配额用的 GitHub Token |
| `UPDATE_PROXY` | 跟随 `HTTPS_PROXY` | 走代理下载，如 `http://127.0.0.1:3067` |
| `UPDATE_MIRROR` | `https://ghfast.top/` | 下载镜像回退（直连失败才用）；设空禁用 |
| `HTTPS_PROXY` | 空 | 京东 / 脚本库 / 更新下载都会走它 |
| `JD_BEAN_ALLOW_DEV_AUTO` | 空 | 开发目录里强测自动流程用（普通部署不需要） |
| `JD_BEAN_RESTART_EXIT` | `75` | 重启退出码（改它必须同步改启动器） |

> GitHub 匿名 API 有每 IP 限流。**公开仓库不需要 token**；若日志里出现
> `HTTP 403` 与限流提示，给 `UPDATE_TOKEN` 配一个免费 token 即可（read 权限就够）。

---

## 八、安全与隐私

- **服务默认监听 `0.0.0.0`**，也就是**同一局域网内任何设备都能打开这个控制台**；
- `accounts.json` 里存着京东 Cookie **明文**，谁拿到都能顶你的号领豆；
- 所以：**务必设置控制台密码**，或者把 `HOST` 改成 `127.0.0.1`；
- 数据只存在本机（`data/` 目录）。本工具**不上传任何数据到第三方**；
- 升级只从 GitHub 域名下载，并强制校验 sha256 与包内 CRC；
- 忘记密码的自救口子：删除 `data/console-auth.json`（启动日志里会打印它的完整路径）。

---

## 九、常见问题

**Q：控制台打不开 / 一直「加载中…」**
按顺序查：① 进程在不在（Windows 看那个黑窗口；飞牛看应用中心的运行状态）；
② `/api/health` 能不能打开（`http://127.0.0.1:3100/api/health`）；
③ 端口是不是被占了（Windows：`netstat -ano | findstr ":3100"`）。
页面出现红条提示「服务端是旧版本」时，说明前端文件比后端新 —— 重启服务即可。

**Q：升级后提示成功但版本号没变**
多半是旧进程还在或浏览器缓存。先确认 `/api/health` 里的 `pid` 真的变了，
再强制刷新页面（Ctrl+F5）。Windows 端覆盖脚本需要等服务进程树完全退出，多等十几秒。
飞牛端确认方式：`sudo appcenter-cli status jdbeanbot`。

**Q：Windows 双击 start.bat 一闪就没了**
在文件夹地址栏输入 `cmd` 回车后手动运行 `start.bat`，就能看到报错（见 2.9）。

**Q：升级检查一直报 HTTP 403**
GitHub 匿名请求限流（按 IP 计）。给 `UPDATE_TOKEN` 配一个免费的 GitHub token 即可。

**Q：某个脚本一直零收益**
控制台「脚本健康」面板会按运行结果自动停跑长期零收益或连续被风控的脚本，
也可以手动重新启用。注意有些脚本依赖上游未发布的模块，跑通但收益恒为 0。

**Q：脚本库更新失败**
默认主源是 `6dylan6/jdpro`，国内可能拉不动。控制台「脚本库仓库」里可以给每个源填**镜像**地址。

---

## 十、合规声明

- 本工具只操作**你自己的**京东账号，请勿用于他人账号；
- **自动评价默认只生成不提交**，且配图只接受**你自己拍的照片**。
  刻意不实现「抓取他人评价图片」：那涉及著作权与《电子商务法》第十七条，
  而且平台在按图查重，搬运图片本身就会被识别；
- 请控制运行频率。过于频繁的请求会触发京东风控，后果由使用者承担；
- 本项目仅供学习与个人自动化使用，使用产生的一切后果由使用者自行承担。

---

## 十一、给维护者：怎么发一版

```bash
# 1. 改版本号（唯一来源）
echo 1.0.5 > app/VERSION

# 2. 全量测试（独立服务 + 独立数据目录，不碰真实数据；12 套约 1200 断言）
cd app && node _run-all.js

# 3. 一键打包三件套（Windows zip + 飞牛 fpk + 内核包 + 各自 sha256）
cd .. && python release/build.py            # 首次需加 --fetch-node

# 4. 前端内联脚本自检（改过 public/index.html 之后必跑）
node app/tools/check-inline.js

# 5. Windows 便携包真机冒烟（解压→真双击→探活→停机，38 项断言）
python _win-smoke.py                         # 改 ZIP/PORT 常量指向新包

# 6. 发布：打 tag → GitHub 建 Release → 上传 dist/<版本>/ 的三个包 + .sha256

# 7. 发布后必跑真实链路验证（真打 api.github.com，下载真包、覆盖沙箱、回滚）
#    Windows: UPDATE_REAL=1 UPDATE_TOKEN=<token> node app/_t-update-real.js
```

打包脚本会做几件**不能省**的自查：

- 断言产物里**没有用户数据**（`data/`、`accounts.json`、运行日志、京豆余额缓存、测试套件）；
- 断言体积合理（曾经因为目录层级放错，打包器照样报成功、产出却只有 42KB）；
- 把 `app/VERSION` 同步进 fpk 的 `manifest`，避免「代码升了版本没改」。

真实链路验证能抓到本地假 Release 抓不到的问题 —— 曾经靠它抓出过
「sha256 张冠李戴（拿 fpk 的校验和去校内核包）」和「升级误删 BeanCache 用户数据」两个致命 bug。

---

## 许可

MIT。详见 [LICENSE](LICENSE)。
