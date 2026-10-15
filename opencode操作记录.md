# VSCode 拼音输入问题排查与修复记录

**日期**: 2026-09-29

## 问题

VSCode 中无法切换/输入中文拼音，其余应用正常。

## 排查过程

1. 确认系统为 Manjaro Linux + KDE Plasma Wayland 会话
2. 输入法为 fcitx 4.2.9.9，环境变量 GTK_IM_MODULE/QT_IM_MODULE/XMODIFIERS 均正确设为 fcitx
3. VSCode 启动参数含 `--ozone-platform=wayland --enable-wayland-ime`，以 Wayland 原生模式运行
4. fcitx4 不支持 Wayland text-input 协议，导致 VSCode 无法接收输入法输入
5. 其他应用走 XWayland 或 GTK immodule，不受影响

## 最终改动

仅修改一个文件：`~/.config/code-flags.conf`

**改前**:
```
--enable-features=UseOzonePlatform
--ozone-platform=wayland
--enable-wayland-ime
--gtk-version=4
```

**改后**:
```
--enable-features=UseOzonePlatform
--ozone-platform=x11
```

将 VSCode 从 Wayland 原生模式切换为 Ozone X11 后端（XWayland），fcitx4 输入法集成恢复正常。

## 效果

VSCode 中 `Ctrl+Space` 可正常切换拼音输入，代码补全已改绑 `Ctrl+Shift+Space`，无冲突。

## 性能影响分析

- **输入延迟**: XWayland 多一层 X11→Wayland 转换，日常编码不可感知
- **显示**: 整数缩放（如 200%）无影响；分数缩放（如 150%）下画面可能略糊
- **资源占用**: XWayland 守护进程额外几 MB 内存，可忽略
- **总结**: 对日常写代码无实际影响。若未来升级到 fcitx5 可切回 Wayland 原生模式

---

# Manjaro 滚动更新（保留 NVIDIA 驱动）

**日期**: 2026-10-08

## 任务

对 Manjaro 做全面滚动更新（`pacman -Syu`），NVIDIA 显卡驱动暂时不更新。内核 `linux70 7.0.10`，驱动 `nvidia-580xx 580.159.04`。

## 排查/执行过程

1. 直接 `--ignore` nvidia 驱动后报依赖冲突：仓库用 `linux-meta` 替换 `linux70`，而保留的 `linux70-nvidia-580xx` 依赖 `linux70` → `linux-meta` 与 `linux70` 冲突。
2. 确认 `linux-meta` 不提供 `linux`、无包依赖它，仅靠 `Replaces: linux70` 被拉入 → 扩大忽略集，连 `linux-meta`/`linux-headers-meta`/`linux71` 一起 ignore，旧内核+旧驱动成对冻结。
3. 镜像超时：原镜像 `mirrors.pku.edu.cn`、`mirrors.zju.edu.cn` 均 <1B/s。`pacman-mirrors -f` 因连通性检查访问 wikipedia/archlinux.org（国内被墙）失败。
4. 手动测速国内镜像：NJU 0.135s / TUNA 0.190s / BFSU 0.312s / SJTU 0.321s / USTC 0.549s，写入 `mirrorlist`（TUNA 优先）。
5. 下载仍中途掐断（"Operation too slow"）→ 加 `--disable-download-timeout` 解决。
6. 签名验证失败：`arm-none-eabi-*` "勉强信任(marginally trusted)" → 先 `pacman -Sy archlinux-keyring manjaro-keyring` + `pacman-key --populate archlinux manjaro` 重建信任。

## 最终改动

**镜像** `/etc/pacman.d/mirrorlist`（原文件备份为 `mirrorlist.bak`）：
```
Server = https://mirrors.tuna.tsinghua.edu.cn/manjaro/stable/$repo/$arch
Server = https://mirrors.bfsu.edu.cn/manjaro/stable/$repo/$arch
Server = https://mirrors.sjtug.sjtu.edu.cn/manjaro/stable/$repo/$arch
Server = https://mirrors.ustc.edu.cn/manjaro/stable/$repo/$arch
Server = https://mirror.nju.edu.cn/manjaro/stable/$repo/$arch
```

**更新命令**：
```bash
pacman -Sy archlinux-keyring manjaro-keyring && pacman-key --populate archlinux manjaro
pacman --disable-download-timeout -Syu \
  --ignore linux70,linux70-headers,linux70-nvidia-580xx,linux-meta,linux-headers-meta,linux71,linux71-headers,nvidia-580xx-utils,lib32-nvidia-580xx-utils,nvidia-580xx-settings,libxnvctrl,libxnvctrl-580xx \
  --noconfirm
```

## 效果

- ✅ 约 961 个包升级完成（glibc 2.44、systemd 261.2、mesa 26.2.2、vulkan 栈、Plasma 6.7.4、KDE Gear 26.08、Qt 6.11.2、Firefox 157、Python 3.14、archlinux-keyring 20260902 等），42 个事务后钩子跑完。
- 🟡 NVIDIA 580xx 驱动整套 + 匹配内核 linux70 按要求保留未动，显卡功能正常。
- ❌ 两处问题（均与额外内核 linux618 有关，详见下节）。

## 影响与风险分析

- **linux618 被顺带安装**：作为 wine/ntsync 的 `NTSYNC-MODULE` 提供者（默认选项 1）被拉入。系统现有两内核：linux70 + linux618。
- **linux618 initramfs 构建失败**：`mkinitcpio.conf` 硬编码 `MODULES=(nvidia ...)`，而 linux618 无对应 nvidia 模块，报 `module not found: nvidia`，镜像"不完整"。
- **影响范围**：仅 linux618 无 NVIDIA 驱动；linux70 的 initramfs 正常，显卡照常。事务本身已完成，后续钩子正常。
- **重启安全性**：`GRUB_DEFAULT=saved` + `GRUB_SAVEDEFAULT=true`，`grubenv` 的 `saved_entry` 未设置 → 启动首个菜单项；`grub-mkconfig` 把 **linux70 列首位** → 默认进 linux70（正常）。
- **风险点**：`SAVEDEFAULT=true` 会记住上次启动项；一旦手动选过 linux618，以后将一直默认进 linux618（无驱动）。

## 待办 / 建议（未执行，需确认）

- 删除 `linux618`（消除隐患，只留 linux70；代价：wine 的 ntsync 同步特性失效，不影响显卡）；或
- 锁定 GRUB 默认项为 linux70（保留 linux618 但保证不误进）。
- 下次升级 NVIDIA 驱动时需连内核一起升，跟上 linux-meta/linux71 迁移。

---

# 微信打不开修复（回退 portable + bubblewrap）

**日期**: 2026-10-08（紧接系统更新后）

## 问题

系统滚动更新后，微信（桌面图标/菜单）无法启动。

## 排查过程

1. 微信为本地包 `wechat 4.0.0.30-2`（2024-12-09 安装，已不在任何仓库，疑似 AUR 旧包）。
2. 启动脚本 `/usr/bin/wechat.sh` 实际执行 `portable $@`（`portable` 是沙箱守护进程）。
3. 命令行运行 `wechat.sh` 报：
   ```
   [Fatal]: ConfigError( NoAvailableConfig(
       [ "InvalidTomlVar(NotPresent)", "InvalidBashVar(NotPresent)" ] ) )
   ```
   → portable 读不到配置变量。
4. 查 `pacman.log`：本次更新把 `portable` 从 `1:16.0.2-1` 升到 `1:20.2-2`（跨度很大）。旧版微信的 `portable-config` 不满足新版 portable 的配置要求。
5. 缓存有旧版 `portable-1:16.0.2-1`，且无任何包依赖 portable → 降级安全。降级后配置报错消失，但出现新错误：
   ```
   bwrap: Can't mkdir parents for /host: Function not implemented
   ```
6. `bwrap` 自测正常，说明非环境问题。本次更新同时升了 `bubblewrap 0.11.2 → 0.12.0`；portable 16 调 bwrap 的方式被新版 bwrap 拒。
7. 缓存有 `bubblewrap-0.11.2-1`，且无包硬依赖 → 再降级 bwrap。

## 最终改动

回退两个包到更新前的版本（均从 `/var/cache/pacman/pkg/` 安装，无包依赖、无连累）：

```bash
sudo pacman -U /var/cache/pacman/pkg/portable-1:16.0.2-1-x86_64.pkg.tar.zst
sudo pacman -U /var/cache/pacman/pkg/bubblewrap-0.11.2-1-x86_64.pkg.tar.zst
```

还原出的工作组合：`wechat 4.0.0.30` + `portable 16.0.2` + `bubblewrap 0.11.2`（三者均为更新前版本）。

## 效果

- 微信正常启动，沙箱内主程序 `/opt/wechat/wechat` 跑起。
- 残留无害警告：fontconfig 对 `/etc/fonts/conf.d/48-guessfamily.conf` 报 "invalid attribute 'xsi:nil'"（字体配置格式问题，不影响运行，暂不处理）。

## 影响与风险

- **风险**：下次 `pacman -Syu` 会再次升级 `portable` 和 `bubblewrap`，微信会再次打不开。
- **本次未做 IgnorePkg 拦截**（用户选择先不挡住）。

## 待办 / 建议（未执行）

- 治本：从 AUR 重装一个适配新版 portable/bwrap 的微信（当前 `wechat 4.0.0.30` 为 2024-12 旧包，已不在仓库）。
- 或临时：在 `/etc/pacman.conf` 的 `IgnorePkg` 加入 `portable bubblewrap`，更新时跳过（代价：bubblewrap 安全更新也暂停）。

---

# 系统更新后健康体检

**日期**: 2026-10-08（更新完成后、未重启时检测）

## 检测项与结果

| 项目 | 结果 | 状态 |
|---|---|---|
| 依赖一致性 `pacman -Dk` | `lib32-pam` 缺少 `lib32-audit` 依赖 | ⚠️ 轻微 |
| 包文件完整性 `pacman -Qk` | 0 个包有缺失文件 | ✅ |
| 失败 systemd 单元 | `wiki.service` failed | ⚠️ 用户应用 |
| 孤儿包 `pacman -Qdt` | 约 110 个残留（含 wechat，勿删） | 🟡 可清理 |
| `/etc` 下 `.pacnew` | 11 个待合并 | 🟡 需处理 |
| 磁盘 | `/` 用 752G/916G（87%），缓存 22GB | 🟡 偏满 |
| 启动内核+initramfs | linux70(7.0) 完整含 nvidia；linux618(6.18) initramfs 仅 30MB 不完整 | ✅ 默认安全 |
| GRUB 默认项 | `GRUB_DEFAULT=saved`，saved_entry 未设 → 启动首个菜单项=linux70 | ✅ |
| 数据库服务 | mariadb、postgresql 均 inactive | ✅ 当前未用 |
| portable/bubblewrap | 已回退 16.0.2 / 0.11.2 | ✅ |

## 关键文件实测值

```
/boot/vmlinuz-7.0-x86_64            16.4M  (2026-06-02)
/boot/initramfs-7.0-x86_64.img     201.6M  (含 nvidia 模块，正常)
/boot/vmlinuz-6.18-x86_64          15.6M  (2026-10-08 新)
/boot/initramfs-6.18-x86_64.img      30.3M  (无 nvidia，不完整，仅手动选它时才受影响)
```

`.pacnew` 列表：
```
/etc/passwd.pacnew                 /etc/shells.pacnew
/etc/hosts.pacnew                  /etc/locale.gen.pacnew
/etc/pacman.conf.pacnew            /etc/mkinitcpio.conf.pacnew
/etc/plymouth/plymouthd.conf.pacnew
/etc/tpm2-tss/fapi-profiles/P_RSA3072SHA384.json.pacnew
/etc/tpm2-tss/fapi-profiles/P_ECCP384SHA384.json.pacnew
/etc/makepkg.conf.d/fortran.conf.pacnew
/etc/makepkg.conf.d/rust.conf.pacnew
```

## 结论

更新整体**安全，可重启**：默认会进 linux70（有 nvidia，显卡正常），无包文件缺失，关键回退已完成。

## 待办 / 行动计划

**A. 现在可安全做**（待用户确认后执行）
- 修依赖：`pacman -S lib32-audit`
- 合并低风险 `.pacnew`（locale.gen/hosts/passwd/shells/plymouth/tpm2/makepkg）；`mkinitcpio.conf` 与 `pacman.conf` 逐行核对（保留 nvidia MODULES 行、保留 archlinuxcn 仓库）
- 清缓存：`paccache -r`（留最近 1 版，省十几 GB）

**B. 需判断**
- `wiki.service` + postgres：若仍用 Wiki.js，需查（可能要做 postgres 数据格式升级 + 确认 nodejs 26 兼容）
- 孤儿包清理：`pacman -Rns $(pacman -Qdtq)`（先排除 wechat 等 AUR 包再删）
- linux618：删除 or 保留（之前已记）

**C. 收尾**
- 重启一次，让 glibc/systemd/mesa 生效
- 重启后复查：`systemctl --failed`、`journalctl -p err -b`

---

# opencode 通过 GitHub MCP 服务器操作 GitHub

**日期**: 2026-10-15

## 背景

opencode 集成了 GitHub MCP（Model Context Protocol）服务器，可在对话中直接调用 GitHub API，无需手动切换浏览器或命令行。

## 本次使用的功能

### 1. 获取当前认证用户信息

- **工具**: `github_get_me`
- **用途**: 查看当前登录的 GitHub 账户
- **示例返回**: 用户名 `wkkkk1`、主页 URL、公开仓库数等

### 2. 搜索用户的仓库

- **工具**: `github_search_repositories`
- **参数**: `query = "user:wkkkk"`（或 `user:wkkkk1`）
- **用途**: 列出指定用户的所有公开仓库
- **返回**: 仓库名、描述、语言、Stars、创建/更新时间等

### 3. 读取仓库文件内容

- **工具**: `github_get_file_contents`
- **参数**: `owner`、`repo`、`path`（如 `README.md` 或 `/` 列目录）
- **用途**: 获取仓库中指定文件的内容或目录结构
- **示例**: 读取 `wkkkk/NC-PTM` 的 README 和根目录列表，发现仓库为空

### 4. 查看用户收藏（Starred）仓库

- **说明**: GitHub MCP 服务器无直接的"列出 starred 仓库"工具
- **替代方案**: 使用 `webfetch` 抓取 `https://github.com/<用户名>?tab=stars` 页面
- **结果**: 获取到 wkkkk 收藏的 18 个仓库列表（涵盖零样本学习、图像翻译、虚拟试衣等方向）

## 常用 GitHub MCP 工具速查

| 工具 | 用途 |
|------|------|
| `github_get_me` | 获取当前认证用户信息 |
| `github_search_repositories` | 按条件搜索仓库（`user:`、`org:`、`language:` 等） |
| `github_get_file_contents` | 读取仓库文件/目录内容 |
| `github_list_issues` | 列出仓库的 Issues |
| `github_search_issues` | 语义搜索 Issues |
| `github_list_pull_requests` | 列出仓库的 PR |
| `github_search_code` | 跨仓库搜索代码 |
| `github_list_commits` | 列出分支提交记录 |
| `github_list_branches` | 列出仓库分支 |
| `github_list_tags` | 列出仓库标签 |
| `github_list_releases` | 列出仓库发布版本 |
| `github_create_pull_request` | 创建 PR |
| `github_merge_pull_request` | 合并 PR |
| `github_issue_write` | 创建/更新 Issue |
| `github_add_issue_comment` | 给 Issue/PR 添加评论 |
| `github_create_or_update_file` | 远程创建/更新仓库文件 |
| `github_push_files` | 单次提交推送多个文件 |
| `github_create_branch` | 创建新分支 |
| `github_fork_repository` | Fork 仓库 |

## 总结

GitHub MCP 服务器让 opencode 能在对话中无缝操作 GitHub：查仓库、读文件、管 Issue/PR、搜索代码等。对于没有直接工具的操作（如列出 starred 仓库），可搭配 `webfetch` 抓取网页作为补充。
