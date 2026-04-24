# A Dark Room 项目功能解析文档

## 项目概述

**A Dark Room** 是一款经典的极简主义文字冒险游戏，最初由 Michael Townsend 开发。这是一款增量式(incremental)文本冒险游戏，玩家在一个黑暗的房间中开始，通过一系列的选择和探索，逐步揭开世界的秘密。

**官方网站**: http://adarkroom.doublespeakgames.com

**支持平台**:
- 浏览器版 (Web)
- iOS
- Android
- Steam

---

## 核心游戏架构

### 1. 技术栈
```
前端:     HTML5 + JavaScript + jQuery
样式:     纯 CSS
音频:     Web Audio API
本地化:   gettext 风格翻译系统
存储:     localStorage + Dropbox 云同步
```

### 2. 核心模块结构

| 模块文件 | 功能描述 |
|---------|---------|
| [engine.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/engine.js) | 游戏核心引擎，全局状态管理，游戏循环 |
| [state_manager.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/state_manager.js) | 存档/读档系统，状态持久化 |
| [room.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/room.js) | 房间场景，建造系统，资源生产 |
| [world.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/world.js) | 大地图探索，战斗系统，随机事件 |
| [outside.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/outside.js) | 外出探索，采集系统 |
| [ship.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/ship.js) | 飞船建造与管理 |
| [space.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/space.js) | 太空场景，弹幕射击小游戏 |
| [fabricator.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/fabricator.js) | 物品制作系统 |
| [prestige.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/prestige.js) | 周目继承系统，多周目 |
| [scoring.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/scoring.js) | 成就与评分系统 |
| [localization.js](file:///Users/chengshang/Code/trae_projects/DarkRoom/script/localization.js) | 多语言支持 (27种语言) |

---

## 游戏阶段与核心玩法

### 🎮 第一阶段：黑暗房间 (The Room)

**起始状态**：玩家在黑暗中醒来，只有一堆余烬。

**核心玩法**：
1. **生火机制**
   - 添柴维持火堆温度
   - 火堆有冷却计时器 (5分钟不添柴就熄灭)
   - 温度影响后续NPC出现

2. **神秘访客事件**
   - 生火后触发：一名陌生女人（建造者）出现
   - 触发建造系统解锁

3. **建造系统** - 16种建筑可建造：
   ```
   基础建筑: 陷阱 → 推车 → 小屋
   生产建筑: 狩猎小屋 → 制革厂 → 熏肉房
   进阶建筑: 工坊 → 炼铁厂 → 交易所
   ```

4. **村民与资源自动生产**：
   - 每间小屋可容纳4名村民
   - 村民自动采集：木材、毛皮、肉类、煤炭
   - 最大人口：20小屋 × 4 = 80村民

---

### 🎮 第二阶段：荒野探索 (Outside)

**解锁条件**：建造完成一定建筑后解锁出门。

**核心系统**：

1. **探索机制**
   - 60×60 的程序化生成地图
   - 可见范围：以玩家为中心 5×5 区域
   - 地形类型：森林、田野、荒地、沼泽

2. **地标系统 (14种地标)**：
   ```
   资源类: 铁矿 → 煤矿 → 硫磺矿
   探索类: 废弃房屋 → 洞穴 → 废弃小镇 → 废墟城市
   剧情类: 战场 → 钻孔 → 沼泽 → 坠毁的外星飞船
   ```

3. **战斗系统**
   - 武器类型：徒手 → 近战 → 远程 → 科幻武器
   - 11种武器从骨矛到等离子步枪梯度升级
   - Perk技能系统：拳击手、武术家、野蛮人、神枪手等12种天赋

4. **生存属性**：
   - 生命值 (Health)
   - 饱食度 (Food) - 每2步消耗1
   - 饮水度 (Water) - 每步消耗1

---

### 🎮 第三阶段：太空 (Space)

**解锁条件**：修复坠毁的外星飞船。

**核心玩法**：
1. **飞船系统**
   - 船体值 (Hull)
   - 武器系统升级
   - 推进器升级

2. **弹幕射击小游戏**：
   - WASD 或 方向键控制
   - 躲避陨石
   - 突破大气层
   - 5层大气区：对流层 → 平流层 → 中间层 → 热层 → 外逸层

---

### 🎮 第四阶段：新游戏+ (Prestige)

**多周目继承系统**：
- 通关后保留特殊技能点
- 解锁隐藏天赋
- 加快游戏节奏
- 挑战更高难度

---

## 技术设计亮点

### 1. 事件驱动架构 (Pub/Sub)
```javascript
// 状态更新事件订阅
$.Dispatch('stateUpdate').subscribe(handler);

// 所有模块通过事件通信，解耦合
```

### 2. 程序化地图生成
```javascript
// 使用斑块算法生成自然地形边界
STICKINESS = 0.5  // 地形粘连度
// 根据地标半径放置关键剧情点
minRadius / maxRadius 控制探索节奏
```

### 3. 音频情绪系统
- 不同场景对应不同BGM
- 关键事件触发音效
- 支持动态音量控制

### 4. 云存档系统
- localStorage 本地自动存档
- Dropbox OAuth 云同步
- 存档间隔：30秒

---

## 游戏数值设计

| 资源类型 | 用途 | 获取途径 |
|---------|------|---------|
| **木材** | 所有建筑基础材料 | 收集、村民生产 |
| **毛皮** | 制革、交易 | 陷阱、狩猎 |
| **肉类** | 回血、熏制 | 狩猎 |
| **皮革** | 高级装备 | 制革厂 |
| **铁/煤** | 钢铁生产 | 矿洞探索 |
| **硫磺** | 弹药制作 | 硫磺矿 |
| **外星科技** | 能量武器 | 飞船残骸 |

---

## 开发与扩展

### 启动开发服务器：
```bash
npm install
npm start
```

### 添加新语言：
```
lang/ 目录下添加对应语言文件夹
  - strings.js 翻译文件
  - main.css 字体适配
lang/langs.js 中注册语言代码
```

---

## 总结

**A Dark Room** 是文字冒险游戏中的经典之作，其成功之处在于：

1. ✅ **极简主义美学** - 纯文字营造沉浸感
2. ✅ **节奏把控精妙** - 从黑暗小屋到太空史诗的层层递进
3. ✅ **系统深度足够** - 建造→探索→战斗→太空 多系统融合
4. ✅ **叙事手法高明** - 碎片化信息拼凑世界观
5. ✅ **重玩价值高** - 多周目系统支持

这是一个设计非常成熟的HTML5游戏架构，代码结构清晰，模块划分合理，值得作为HTML5游戏开发的参考范例。
