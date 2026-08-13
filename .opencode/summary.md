## Objective
- 让 BedWars1058 fork 在用户的 Paper 1.21.11 服务端上真正跑起来（已完成全部阻塞点：R7 versionsupport 陈旧产物、NamespacedKey 大写、sidebar v1_21_R7 缺失）
- 用户指令：**构建产物留在 `F:\BedWars1058_Reloaded\bedwars-plugin\target\bedwars-plugin-25.2.jar`，不复制到测试服务器，用户自己测**

## Important Details
- **现代 Paper（1.20.5+）**：craftbukkit 无版本包；NMS 用 Mojang 名；Paper 首载自动 remap；spigot 1.21.11 运行时才有 `v1_21_R7` 包 + Spigot NMS 名
- **m2 陈旧陷阱**：`-rf :bedwars-plugin` 从 m2 解析 versionsupport（可能旧）→ **全量 `mvn clean package`（reactor）**
- **m2 中毒源**：`repo.andrei1058.dev/snapshots/` JWT 代理返回 HTML(200) 被当 jar 缓存；已修 guava-33.5.0-jre、failureaccess-1.0.3、j2objc-annotations-3.1、commons-codec-1.16.0、bungeecord-chat-1.21-R0.4、gson-2.13.2
- **sidebar 模块 pom 必须保留 andrei1058-snapshots 仓库**：m2 base/cmn1 26.6 的 `_remote.repositories` 归属该 repo ID
- **sidebar 装配**：`SidebarService.init()` `Class.forName("com.andrei1058.bedwars.libs.sidebar."+serverVersion+".ProviderImpl")`；shade 把 `com.andrei1058.spigot.sidebar`→`com.andrei1058.bedwars.libs.sidebar`；`serverVersion`="v1_21_R7" 已验证
- **1.21.4→1.21.11 NMS API 变化（sidebar 模块已按此改造）**：
  - `EntityPlayer.g` = PlayerConnection；发包用 `sendPacket(Packet<?>)`（继承自 ServerCommonPacketListenerImpl）
  - `ScoreboardTeam`：`e()`→IChatMutableComponent(prefix)、`f()`→IChatBaseComponent(suffix)、`h()`→Collection<String>(玩家集合，R3 是 g())；name getter 是 `c()`
  - `ScoreboardObjective`：name=`c()`、displayName getter=`e()`(R3 是 d())、formattedName=`h()`(R3 是 g())、displayName setter=`a(IChatBaseComponent)`、displayName 是公有字段 `d`；7 参构造匹配
  - `IScoreboardCriteria`：`b` 是 Codec；criteria 常量 c..m；**用 byname `IScoreboardCriteria.a("dummy")/a("health")` 替代 R3 字段**
  - `PacketPlayOutScoreboardScore(String,String,int,Optional<IChatBaseComponent>,Optional<NumberFormat>)`；`ClientboundResetScorePacket(String,String)`
  - 26.6 base `PlayerTab` 抽象方法含 `add/remove(Entity)` 与 `add/remove(Player)`（R7 模块补了 Entity 两个）
- **NamespacedKey 1.21 严格小写**；tag key 只写不读
- **`aH` TNT whitelist NoSuchFieldException**（`v1_21_R7.java:351`）：Paper 反射重写器无法映射混淆字段 → WARN 非致命，TNT 白名单在 Paper 不生效（可后修）
- **PowerShell 5.1 `Set-Content -Encoding UTF8` 写 BOM** → javac 报 `'\ufeff'`；用 `UTF8Encoding($false)`
- 编译/API 验证构件：`~/.m2/repository/org/spigotmc/spigot/1.21.11-R0.1-SNAPSHOT/spigot-1.21.11-R0.1-20260115.203122-4.jar`
- SidebarLib 源：`https://github.com/andrei1058/SidebarLib`（master=25.2.2，base PlayerTab 是 `remove(Player)` 版本）

## Work State
### Completed
- **R7 陈旧产物根因修复**：m2 安装的 R7 jar 是重定向前旧版（v1_21_R5 + `animal/EntityIronGolem`）→ 删 m2 → 全量重建 → 字节码验证
- **NamespacedKey 小写**：`VersionSupport.java:49` "bedwars1058"、`BuyItem.java:106/139/149`、`ContentTier.java:108/112`、`ShopCategory.java:95/99`
- **sidebar v1_21_R7 模块自建（完成）**：按 API 变化改造 ProviderImpl/SidebarImpl/PlayerListImpl（`EntityPlayer.g`/`sendPacket`、`ScoreboardTeam.e()/h()/c()`、`ScoreboardObjective.e()/h()`、criteria byname、补 `add/remove(Entity)`）；**源码已提取到 `F:\BedWars1058_Reloaded\sidebar-v1_21_R7\`（独立模块，加入根 reactor pom），不再依赖 m2 手工安装的 jar**
- **bedwars-plugin pom**：加 `sidebar-v1_21_R7:26.6-SNAPSHOT` compile 依赖（shade 自动重定位）
- **全量 `mvn clean package` BUILD SUCCESS**；shaded jar 验证含 `com/andrei1058/bedwars/libs/sidebar/v1_21_R7/ProviderImpl.class`，无 stale 内容
- 已提交：`17ce52ab`（sidebar 构建修复）、`673d3311`（Paper 检测）

### Active
- 无（待用户实测最新 jar）
- 未提交改动：BedWars.java 版本检测 + SidebarService.java + 8 处小写 key（本次已构建进 jar，未 commit）

### Blocked
- 待用户测试：**`bedwars-plugin\target\bedwars-plugin-25.2.jar`（2026-08-13 19:05:16, 2.8MB）** 是否能在 Paper 1.21.11 正常启动（sidebar 应能加载）

## Next Move
1. 用户测试最新 jar；若 sidebar 仍有问题，需在临时 SidebarLib 克隆调试（模块已 m2 安装，改完需重新 `mvn install` + 全量重建）
2. 测试通过后提交未提交改动（问用户）

## Relevant Files
- `F:\BedWars1058_Reloaded\bedwars-plugin\target\bedwars-plugin-25.2.jar`：交付产物（2026-08-13 19:12, 2.8MB）
- `F:\BedWars1058_Reloaded\sidebar-v1_21_R7\`：自建 sidebar 模块（pom.xml + src，已入根 reactor；bedwars-plugin 从 reactor 解析）
- `F:\BedWars1058_Reloaded\pom.xml`（:129 加了 `<module>sidebar-v1_21_R7</module>`）
- `F:\BedWars1058_Reloaded\bedwars-plugin\src\main\java\com\andrei1058\bedwars\BedWars.java`（:511-517 sidebar 自禁用、版本检测未提交）
- `F:\BedWars1058_Reloaded\bedwars-plugin\src\main\java\com\andrei1058\bedwars\sidebar\SidebarService.java`（:122 Class.forName）
- `F:\BedWars1058_Reloaded\bedwars-api\src\main\java\com\andrei1058\bedwars\api\server\VersionSupport.java`（:49）
- `F:\BedWars1058_Reloaded\bedwars-plugin\src\main\java\com\andrei1058\bedwars\shop\main\`：BuyItem.java / ContentTier.java / ShopCategory.java
- 测试服 `C:\Users\Max\AppData\Local\Temp\opencode\paperrun\`：不再使用
