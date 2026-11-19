# HBNS-AUE-V3 整合包工作区

HiBer 北冰洋服务器 V3 的 NSUM 整合包仓库。

## 分支（逻辑分支，定义在 `workspace.json`）

| 分支 | 父分支 | 说明 |
| --- | --- | --- |
| `root` | — | 根分支，共享基础模组与配置 |
| `server-base` | root | 服务端基础 |
| `server-HBNS` | server-base | HBNS 使用的服务器版本 |
| `client-base` | root | 客户端基础 |
| `client-HBNS` | client-base | 标准客户端（推荐） |
| `client-HBNS-DH` | server-HBNS | 用 Distant Horizons 替代 Voxy，适合无法忍受 Voxy 内存占用的玩家 |
| `client-lowmem` | client-HBNS | 低配置优化，降低内存占用 |
| `client-debug` | client-base | 调试用 |

其中 `client-HBNS-DH` 与 `client-lowmem` 是对外可见（`hidden: false`）的两个可选分支。

## mclhr 集成

本工作区启用了 **mclhr**（`K:\mclhr`，LGPL-2.1）作为库解析与启动计划层。
NSUM 侧的接入点是 `applyMclhrStore()`（`modules/NeoCLI/src/cli_dispatcher.cpp`）：

```
exec build --format hmcl
  └─ 构建成功后，把 mods/**/*.jar 收进 <runtime>/stores/mods/<stem>_<sha16>.jar
      └─ 用硬链接回指到实例 mods/（跨卷回退为复制）
          └─ 写 .mclhr/mods.json 声明 store 来源与链接模式
```

实测输出：

```
[mclhr] mods store: .../stores/mods
[mclhr] jars linked=46 copied-to-store=0 skipped=0
```

实例产物：

```
.mclhr/mods.json                      schema 1，source mode=link
mods/*.jar                            LinkType=HardLink
tools/mclhr/mclhr.jar                 供 bridge 调用
```

bridge（`NSumMclhr.afterSync`）在同步成功后会执行：

```bash
java -jar <runtime>/tools/mclhr/mclhr.jar mc mods sync --instance <dir> --mode link --prune
```

因此 `tools/mclhr/mclhr.jar` 必须随包下发——已放入 `branches/root/tools/mclhr/`，
并在 `workspace.json` 的 `sync_policies.folders` 里加了 `"tools": "incremental_overwrite"`。

## 构建

```powershell
NeoServerUpdateModpack.exe exec build `
  --repo <本仓库> --modpack client-HBNS --git-branch master `
  --format hmcl --export <输出目录>
```

> 逻辑分支是目录（`branches/<name>/`），git 侧只有 `master` 一条线，
> 所以必须显式传 `--git-branch master`。
