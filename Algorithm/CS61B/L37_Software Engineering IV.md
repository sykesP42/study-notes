# 🎮 软件工程 IV：游戏开发从入门到实践——设计哲学、代码架构与独立开发

> 本文是 CS61B Spring 2024 Lecture 37 的超详细笔记，由助教 Circle 主讲。内容涵盖游戏设计核心原则（易学难精、反馈机制、随机化）、游戏开发流程（从纸面原型到发布）、代码架构设计（有限状态机解决场景管理）、游戏引擎（Unity）基础、模组开发入门以及独立开发者生态。通过大量案例（Flappy Bird、Minecraft、以撒的结合）和代码示例，帮助你理解游戏开发的全貌，并为 Project 3C 提供实用指导。

---

## 🧭 一、课程导言

### 1.1 课程背景

本讲是软件工程系列的最后一课，也是本学期最后一节正式授课（后续课程均为选修）。由于临近假期、天气晴好，现场出席人数较少，但内容依然干货满满——聚焦于**电子游戏设计**，既是对 Project 3 的延伸指导，也为有志于独立开发的同学们打开一扇窗。

### 1.2 为什么 CS 课程总爱讲游戏？

从 CS61A 的 Hog、Ants，到 CS61B 的 2048、BYOW，几乎每门入门课都有游戏项目。原因很简单：**游戏是学习编程的最佳实践场**——它融合了算法、数据结构、用户交互、实时系统、UI 设计，且能带来即时成就感。如果你喜欢这个过程，**独立游戏开发（indie game development）** 可能会成为你终身的爱好甚至职业。

---

## 🎯 二、游戏设计核心原则

### 2.1 易学难精（Easy to Learn, Hard to Master）

这是游戏设计的黄金法则。玩家应该能快速上手，但想精通需要大量练习和技巧。

**案例：Flappy Bird**

- **操作**：只需点击屏幕，小鸟就会短暂上升。
    
- **物理**：点击后上升一段，随后受重力下落。
    
- **精髓**：看似简单，但管道间隙的时机把握极难，导致玩家反复尝试、欲罢不能。
    

**启示**：好的核心机制往往简单，但通过参数调整（重力系数、管道间距）和关卡设计，可以产生深度。

### 2.2 渐进学习（Gradual Learning）

如果游戏机制本身复杂，应让玩家**逐步接触**，而非一次性灌输。

**反面案例**：冗长的文字教程——玩家不乐意读文档，他们想玩！  
**正面案例**：**Antepiece（前置组件）**——在真正挑战前设置一个简单的任务，暗示解法，让玩家自己领悟。

**《传送门2》经典设计**：

- 场景：一个无法直接跳上的高台。
    
- 暗示：对面墙上已有一个橙色传送门。
    
- 解法：玩家意识到需要用手中的蓝色传送门创造路径，利用抛射的抛物线到达高处。
    
- 效果：玩家通过**行动**学会机制，而非阅读说明。
    

**应用到 BYOW**：如果你想加入“踩机关触发隐藏门”的机制，可以在第一个房间设置一个显眼的机关和半开的门，让玩家尝试互动，而不是弹出一个文本框说“按 E 可开门”。

### 2.3 反馈机制（Feedback）与玩家动机

玩家为什么愿意学习机制？因为他们需要**动机**。动机来自反馈：做得好有奖励，做差了有惩罚（但惩罚不能太狠）。

#### 2.3.1 正反馈（Positive Feedback）

- 定义：表现好时获得奖励（金币、装备、能力），使游戏变得更简单。
    
- 例子：MOBA 游戏中，击杀敌人获得金币，购买更强装备，滚起雪球。
    
- **过度风险**：正反馈太强会导致“胜者通吃”，落后玩家毫无体验，游戏变得无聊（领先者只需平A就能赢）。
    

#### 2.3.2 负反馈（Negative Feedback）

- 定义：表现差时获得补偿，让落后玩家有机会追赶。
    
- 例子：马里奥赛车中，落后的玩家更容易获得强力道具（加速蘑菇、蓝龟壳）。
    
- **适用场景**：派对游戏、休闲游戏，强调欢乐而非竞技。
    
- **过度风险**：负反馈太强会削弱努力的价值，高手觉得“被系统制裁”，低手觉得“反正躺赢”。
    

**设计平衡**：好的游戏会混合使用两种反馈，根据目标调整权重。例如《英雄联盟》的赏金机制（正反馈的修正）就是一种温和的负反馈。

### 2.4 体验多样性（Variability）

玩家通关一次后掌握了所有内容，游戏就容易变得无聊。解决方案有二：

#### 2.4.1 随机化（Randomization）

- 每局游戏随机生成地图、敌人、道具，增加重玩性。
    
- **案例**：《以撒的结合》的房间布局、道具掉落、Boss 出现位置都是随机生成，确保每次体验不同。
    
- **BYOW 应用**：你的世界生成算法已经是随机化的第一步。可以进一步随机化敌人的位置、宝箱内容、NPC 对话。
    

#### 2.4.2 持续更新（New Content）

- 定期添加新角色、新关卡、新机制，保持游戏生命力。
    
- **案例**：《以撒的结合》在 1.6 版本更新中添加了角色“Chino”，后续还有大型 DLC “Repentance”。
    
- **独立开发者策略**：可以先发布基础版，然后根据玩家反馈逐步添加内容。
    

---

## 🛠️ 三、游戏开发流程（从想法到发布）

### 3.1 阶段概览

1. **创意阶段（Idea）**：用几句话描述游戏核心。例如 Minecraft：“方块世界，随机生成，自由建造和破坏”。
    
2. **纸面原型（Paper Prototype）**：用纸笔模拟游戏玩法，测试核心机制是否有趣，避免过早陷入编码细节。
    
3. **最小可行产品（MVP）**：实现最简可玩版本，证明核心玩法可行。例如 Minecraft 最早只有草方块和圆石，WASD 移动，左右键放置/破坏。
    
4. **Alpha 版**：功能完整，但可能有 bug，仅供内部测试。
    
5. **Beta 版**：对外测试，收集反馈，修复问题。
    
6. **正式发布（Release）**：上线商店，持续更新。
    

### 3.2 BYOW 项目定位

你的 BYOW 项目（Project 3A/B）通常停留在 MVP 阶段：随机生成世界、玩家移动、简单交互。Project 3C 则鼓励你**添加新元素**，让它更像一个真正的游戏。

---

## 💻 四、代码架构设计：有限状态机（FSM）

### 4.1 问题：场景管理的混乱

随着游戏功能增加，你会有多个场景：主菜单、世界探索、遭遇战、游戏结束等。每个场景需要不同的输入处理和渲染逻辑。如果用一个大的 `while` 循环和嵌套 `if` 来管理，代码会迅速腐化：


```java

// 糟糕的设计：嵌套循环 + 标志位
while (true) {
    if (currentScene.equals("menu")) {
        // 处理菜单输入
        if (key == ENTER) currentScene = "world";
    } else if (currentScene.equals("world")) {
        while (inWorld) {
            // 处理世界输入
            if (encounterTriggered) {
                currentScene = "encounter";
                break; // 跳出世界循环？
            }
        }
    } else if (currentScene.equals("encounter")) {
        // 处理战斗输入...
    }
}
```
**问题**：

- 缩进爆炸，可读性差。
    
- 场景跳转复杂（例如从战斗返回世界需要多层 break）。
    
- 新增场景需修改主循环，容易引入 bug。
    

### 4.2 解决方案：有限状态机

**有限状态机（FSM）** 是一种数学模型，包含：

- 一组**状态**（State）
    
- 一组**转移**（Transition），定义状态间的切换条件
    

在代码中实现 FSM 的典型模式：

#### 4.2.1 定义状态接口

```java

public interface GameState {
    void update();   // 处理输入、更新逻辑
    void render();   // 绘制画面
    void onEnter();  // 进入状态时的初始化
    void onExit();   // 退出状态时的清理
}```
#### 4.2.2 实现具体状态

```java

public class WorldState implements GameState {
    private Player player;
    private World world;
    @Override
    public void onEnter() {
        // 加载世界、初始化玩家
    }
    @Override
    public void update() {
        if (StdDraw.hasNextKeyTyped()) {
            char key = StdDraw.nextKeyTyped();
            if (key == 'w') player.moveUp();
            // ... 其他移动
            if (encounterTriggered()) {
                StateMachine.getInstance().changeState("encounter");
            }
        }
    }
    @Override
    public void render() {
        world.draw();
        player.draw();
    }
    @Override
    public void onExit() {
        // 保存状态？清理临时数据？
    }
}
```
#### 4.2.3 状态机管理类

```java

public class StateMachine {
    private static StateMachine instance;
    private Map<String, GameState> states = new HashMap<>();
    private GameState currentState;
    private StateMachine() {} // 单例
    public static StateMachine getInstance() { ... }
    public void registerState(String name, GameState state) {
        states.put(name, state);
    }
    public void changeState(String name) {
        if (currentState != null) currentState.onExit();
        currentState = states.get(name);
        currentState.onEnter();
    }
    public void update() {
        if (currentState != null) currentState.update();
    }
    public void render() {
        if (currentState != null) currentState.render();
    }
}
```
#### 4.2.4 主循环简化

```java

public static void main(String[] args) {
    StateMachine fsm = StateMachine.getInstance();
    fsm.registerState("menu", new MenuState());
    fsm.registerState("world", new WorldState());
    fsm.registerState("encounter", new EncounterState());
    fsm.changeState("menu");
    while (true) {
        fsm.update();
        fsm.render();
        StdDraw.pause(16); // ~60 FPS
    }
}
```
**优势**：

- 每个状态独立成类，逻辑清晰。
    
- 新增状态只需实现接口并注册，主循环不变。
    
- 状态切换通过 `changeState` 统一管理，安全可靠。
    

**BYOW 应用**：你可以为主菜单、游戏世界、战斗/遭遇战、游戏结束分别创建状态类，让代码结构瞬间升级。

---

## 🎮 五、游戏引擎：为什么需要引擎？

从零开始写游戏非常困难，因为你需要实现几乎所有游戏都需要的通用功能：

- 窗口管理、输入处理
    
- 渲染管线、精灵动画
    
- 物理碰撞、音频播放
    
- 场景管理（就是上面讲的 FSM）
    

**游戏引擎**（如 Unity、Unreal）内置了这些样板代码，让你专注于游戏逻辑。

### 5.1 Unity 基础概念

Unity 使用**组件化架构**：

- **GameObject**：场景中的每个实体（玩家、敌人、灯光、摄像机）都是一个 GameObject。
    
- **Component**：附加在 GameObject 上的功能模块。例如 `Transform`（位置旋转）、`SpriteRenderer`（显示图片）、`Rigidbody2D`（物理模拟）。
    
- **Script**：你写的 C# 脚本，继承自 `MonoBehaviour`，可以重写特定方法：
    
    - `void Start()`：在第一帧更新前调用，用于初始化。
        
    - `void Update()`：每帧调用，用于处理输入、更新逻辑。
        

### 5.2 Unity 版 FSM 实现

Unity 的场景管理 API 已经实现了状态机思想：

- **SceneManager.LoadScene("sceneName")**：切换到另一个场景。
    
- 每个场景可以包含不同的 GameObject 和初始逻辑。
    

例如，主菜单场景有 UI 按钮，点击“开始游戏”时调用 `SceneManager.LoadScene("World")`，切换到底层世界场景。世界场景中的玩家脚本在 `Start()` 中初始化生命值、位置等。

### 5.3 为什么引擎让你少写“脏活”？

- 你不需要自己写 FSM 框架（引擎内置）。
    
- 你不需要处理 OpenGL/DirectX 渲染。
    
- 你不需要实现物理引擎（Unity 的 PhysX 已集成）。
    
- 你只需要关心：**我的游戏对象应该有什么行为？**
    

**代价**：你失去了底层控制权，必须遵循引擎的规则。但对于独立开发者来说，收益远大于代价。

---

## 🧩 六、模组开发（Modding）

### 6.1 什么是模组？

**Mod = Modification**，即对现有游戏的修改或扩展，添加新内容或改变玩法，而不需要从零开发。

### 6.2 为什么要做模组？

- **门槛低**：不用处理底层引擎，专注于创意。
    
- **社区基础**：现有游戏已有玩家基础，你的模组能快速获得受众。
    
- **学习价值**：阅读和修改成熟代码，是极好的学习方式。
    

### 6.3 如何开发一个模组？

以《泰拉瑞亚》的 tModLoader 为例：

1. 安装 tModLoader（Steam 版需先拥有 Terraria）。
    
2. 阅读官方 Wiki：区分玩家指南和开发者指南。
    
3. 查看 Example Mod 项目，了解 API 用法。
    
4. 继承游戏中的基类（如 `ModItem`），重写方法：
    

```csharp

public class Sword61B : ModItem
{
    public override void SetDefaults()
    {
        item.damage = 61;       // 基础伤害
        item.useStyle = 1;      // 挥舞风格
        // ...
    }
    public override void OnHitNPC(Player player, NPC target, int damage, float knockback, bool crit)
    {
        Main.NewText("Take that! The power of Data Structures!", 255, 100, 100);
    }
}
```
### 6.4 模组的合法性

- 大多数单机游戏允许玩家自制模组，但需遵守游戏开发者的规定（通常禁止商业化）。
    
- 优秀的模组甚至会被官方吸纳，成为正式内容。例如：
    
    - 《以撒的结合》DLC “Repentance” 整合了大型模组 “Antibirth”。
        
    - 《超级马里奥制造》的灵感来源于玩家自制关卡的风潮。
        

---

## 🚀 七、独立开发者生态

### 7.1 自由与责任

作为独立开发者：

- **自由**：不受公司目标和截止日期限制，可以自由选择开发层级（纯代码、用引擎、做模组）。
    
- **责任**：所有事情自己扛——设计、编程、美术、音乐、营销、社区维护。成功时没人抢功，失败时没人背锅。
    

### 7.2 现实挑战

- **时间管理**：就像创业，需要高度自律。讲者本学期因课业和申请，减少了游戏开发时间。
    
- **收入不稳定**：可能很长时间没有收入，需要副业或积蓄支撑。
    
- **孤独感**：一个人面对所有问题，需要强大的内心。
    

### 7.3 发布平台

- **Steam**：主流平台，支持付费销售和创意工坊（Workshop），适合成熟作品。
    
- **[itch.io](https://itch.io/)**：独立开发者乐园，操作简单，适合免费或小额付费的小品级游戏。
    

### 7.4 学习资源

- **官方文档**：Unity Learn、tModLoader Wiki。
    
- **社区**：Reddit 的 r/gamedev、Discord 频道、B站独立游戏开发 UP 主。
    
- **示例项目**：从克隆经典游戏开始（如 Flappy Bird、俄罗斯方块），逐步增加自己的创意。
    

---

## 📚 八、课程总结

### 8.1 游戏设计核心回顾

- **易学难精**：机制简单但深度无限。
    
- **Antepiece**：让玩家通过行动学习。
    
- **反馈**：平衡正负反馈，维持玩家动机。
    
- **可变性**：随机化和持续更新，延长游戏寿命。
    

### 8.2 开发流程回顾

- 创意 → 纸面原型 → MVP → Alpha → Beta → 发布
    
- BYOW 项目已走到 MVP 阶段，Project 3C 是向更完整游戏迈进的尝试。
    

### 8.3 技术选择回顾

- **纯 Java**：适合学习数据结构，可用 FSM 管理状态。
    
- **Unity**：适合快速开发，减少底层工作。
    
- **模组**：基于现有游戏，专注创意。
    

### 8.4 独立开发寄语

> “在每个开发层级，你都会失去一些自由度，但也会减少一些‘脏活’。作为独立开发者，你可以选择自己最享受的那个层级——不受公司目标限制，只为自己的热爱。”

无论你是想用纯 Java 挑战自我，还是用 Unity 快速实现想法，或是通过模组参与现有游戏社区，游戏开发都能成为伴随一生的有趣爱好。希望这节课能为你打开一扇门，看到更广阔的世界。

---

## ❓ 九、问答精选

### Q1: 游戏公司如何招聘？需要什么技能？

A1: 不同公司侧重点不同。大型公司（如 Riot、Blizzard）通常看重计算机基础（算法、数据结构）、游戏引擎经验（Unity/Unreal）、团队协作能力。建议咨询行业内的 HR 或开发者获取最新信息。

### Q2: 独立开发者的一天是怎样的？

A2: 高度灵活。可能上午写代码，下午画图，晚上回复社区评论。需要极强的自我驱动力和时间管理能力，否则容易陷入拖延。

### Q3: 模组开发有哪些工具？

A3: 每个游戏不同。例如《泰拉瑞亚》用 tModLoader，《我的世界》用 Forge/Fabric，《星露谷物语》用 SMAPI。通常官方社区会提供 Wiki 和示例代码。

### Q4: 如何开始学习游戏开发？

A4:

1. 选择一个方向：纯代码（如用 Java 写 2D 小游戏）、用引擎（Unity 推荐）、做模组（选一个你喜欢的游戏）。
    
2. 从小项目开始，克隆一个经典游戏（Flappy Bird、打砖块）。
    
3. 加入社区，阅读文档，多看多问。
    
4. 坚持完成项目，不要半途而废。
    

---

## 📊 十、知识小结（彩色标记）

|主题|核心观点|关键案例/代码|应用提示|
|---|---|---|---|
|游戏设计原则|易学难精、Antepiece、反馈平衡、随机化|Flappy Bird（易学难精）、传送门2（Antepiece）、马里奥赛车（负反馈）|<span style="color:red">BYOW 可加入随机宝箱、隐藏房间，增加重玩性</span>|
|代码架构|用有限状态机（FSM）管理场景|`GameState` 接口、`StateMachine` 单例|<span style="color:red">主菜单、世界、战斗、结束场景各一状态类</span>|
|游戏引擎|引擎提供通用功能，让你专注游戏逻辑|Unity 的 `MonoBehaviour`、`SceneManager`|<span style="color:blue">学习 Unity 官方教程，从 2D 项目开始</span>|
|模组开发|基于现有游戏修改，门槛低，社区活跃|tModLoader、`ModItem` 重写|<span style="color:green">从 Example Mod 开始，理解 API</span>|
|独立开发生活|自由但需自律，成功全归自己，失败也无怨|时间管理、多平台发布（Steam、[itch.io](https://itch.io/)）|<span style="color:red">建议作为爱好，成熟后再考虑商业化</span>|
