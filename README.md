# Pixel Adventure 像素冒险游戏 - 面向对象设计与实现

## 一、项目概述

Pixel Adventure 是一款基于控制台的2D像素风格冒险RPG游戏，采用C++面向对象编程实现。游戏核心玩法包括地图探索、战斗系统、道具收集和成长系统。

## 二、关键类与属性设计说明

### 1. **基础结构类**

#### `Position` 结构体（坐标表示）
- **属性**：
  - `x: int` - X坐标（水平位置）
  - `y: int` - Y坐标（垂直位置）
- **设计思路**：封装坐标信息，提供位置比较和距离计算功能
- **面向对象特性**：
  - 构造函数重载
  - 运算符重载（==）
  - 成员函数封装

### 2. **消息系统类**

#### `GameMessage` 类（游戏消息）
- **属性**：
  - `text: string` - 消息显示的文本内容
  - `color: Color` - 消息颜色（使用颜色枚举）
  - `lifetime: int` - 消息显示的生命周期（帧数）
- **设计思路**：实现消息生命周期管理，支持颜色显示
- **面向对象特性**：
  - 私有数据封装
  - Getter方法提供只读访问
  - 状态管理（生命周期）

## 三、继承体系设计

### 1. **道具继承体系**

#### `Item` 抽象基类（道具基类）
- **属性**：
  - `pos: Position` - 道具在地图中的位置
  - `symbol: char` - 控制台中显示的字符符号
  - `name: string` - 道具名称（如"生命药水"）
  - `color: Color` - 道具在控制台中的显示颜色
- **设计思路**：定义道具通用接口，支持多态使用

#### `HealthPotion` 派生类（生命药水）
- **继承关系**：`Item` → `HealthPotion`
- **属性**：
  - `healAmount: int` - 治疗的生命值量（固定为30）
- **设计思路**：消耗性道具，使用后恢复玩家生命值

#### `WeaponUpgrade` 派生类（武器升级）
- **继承关系**：`Item` → `WeaponUpgrade`
- **属性**：
  - `attackBonus: int` - 攻击力提升值（固定为5）
- **设计思路**：永久性提升道具，增加玩家攻击力

#### `Key` 派生类（钥匙）
- **继承关系**：`Item` → `Key`
- **属性**：无额外属性，完全继承`Item`类的属性
- **设计思路**：功能性道具，用于解锁通关的门

### 2. **角色继承体系**

#### `Character` 基类（角色基类）
- **属性**：
  - `pos: Position` - 角色当前位置
  - `symbol: char` - 控制台显示字符（如'@'、'E'）
  - `name: string` - 角色名称（如"Player"、"Goblin"）
  - `health: int` - 当前生命值
  - `maxHealth: int` - 最大生命值
  - `attack: int` - 攻击力
  - `defense: int` - 防御力
  - `color: Color` - 控制台显示颜色
- **设计思路**：统一管理战斗属性和位置信息

#### `Player` 派生类（玩家）
- **继承关系**：`Character` → `Player`
- **属性**：
  - `treasures: int` - 已收集的宝藏数量
  - `keys: int` - 拥有的钥匙数量
  - `experience: int` - 当前经验值
  - `level: int` - 玩家等级
  - `nextLevelExp: int` - 升级所需经验值
- **设计思路**：扩展玩家特有功能，实现成长系统

#### `Enemy` 派生类（敌人基类）
- **继承关系**：`Character` → `Enemy`
- **属性**：
  - `experienceReward: int` - 击败后奖励给玩家的经验值
- **设计思路**：提供敌人移动的AI逻辑基础

#### 具体敌人类（三级继承）
- `Goblin` 类（地精）：`Character` → `Enemy` → `Goblin`
  - 属性：无额外属性，通过构造函数初始化
  - 参数：生命30、攻击8、防御2、经验奖励20
- `Orc` 类（兽人）：`Character` → `Enemy` → `Orc`
  - 属性：无额外属性，通过构造函数初始化
  - 参数：生命50、攻击12、防御5、经验奖励40
- `Boss` 类（首领）：`Character` → `Enemy` → `Boss`
  - 属性：无额外属性，通过构造函数初始化
  - 参数：生命100、攻击20、防御10、经验奖励100

### 3. **环境类**

#### `Trap` 类（陷阱）
- **属性**：
  - `pos: Position` - 陷阱在地图中的位置
  - `active: bool` - 陷阱是否处于激活状态
  - `damage: int` - 陷阱造成的伤害值（固定为15）
- **设计思路**：一次性环境危险元素，触发后造成伤害

## 四、核心游戏类

### `Game` 类（主控类）
- **属性分类说明**：

#### **地图与布局相关**
- `map: vector<vector<char>>` - 游戏地图（二维字符数组）
- `explored: vector<vector<bool>>` - 已探索区域标记（战争迷雾）

#### **游戏实体管理**
- `player: Player` - 玩家对象
- `enemies: vector<shared_ptr<Enemy>>` - 敌人列表（智能指针）
- `items: vector<shared_ptr<Item>>` - 道具列表（智能指针）
- `traps: vector<Trap>` - 陷阱列表
- `door: Position` - 出口门的位置

#### **游戏状态控制**
- `state: GameState` - 当前游戏状态
- `level: int` - 当前关卡编号
- `doorLocked: bool` - 门是否被锁定

#### **视图与交互**
- `viewport: Position` - 视口（显示窗口）的左上角位置
- `messages: deque<GameMessage>` - 消息队列（双端队列）

#### **关键方法说明**：

1. **地图生成系统** `initializeMap()`
```cpp
void Game::initializeMap() {
    map.resize(MAP_HEIGHT, vector<char>(MAP_WIDTH, PATH));
    generateSimpleDungeon();  // 随机生成算法
    placeTreasures(5);        // 放置宝物
    spawnEnemies(3);          // 生成敌人
    spawnItems();             // 生成道具
    spawnTraps(3);            // 放置陷阱
}
```

2. **渲染系统** `draw()`
```cpp
void Game::draw() {
    system("cls");
    // 1. 绘制地图视口
    for (int i = 0; i < VIEWPORT_HEIGHT; i++) {
        for (int j = 0; j < VIEWPORT_WIDTH; j++) {
            // 分层绘制：地形→道具→敌人→玩家
            if (!explored[mapY][mapX]) cout << ' ';  // 战争迷雾
            else if (玩家位置) 绘制玩家;
            else if (敌人位置) 绘制敌人;
            else if (道具位置) 绘制道具;
            else 绘制地形;
        }
    }
    // 2. 显示状态信息
    player.displayStatus();
    // 3. 显示消息队列
    for (const auto& msg : messages) cout << msg.getText();
}
```

3. **游戏逻辑更新** `update()`
```cpp
void Game::update() {
    if (state != PLAYING) return;
    
    // 敌人AI移动（多态调用）
    for (auto& enemy : enemies) {
        if (enemy && enemy->isAlive()) {
            enemy->move(player.getPosition(), map);
            if (enemy->getPosition() == player.getPosition()) {
                combat(player, *enemy);  // 触发战斗
            }
        }
    }
    
    // 碰撞检测与交互
    检测陷阱触发();
    检测道具拾取();
    检测宝物收集();
    检测门解锁();
}
```

4. **战斗系统** `combat()`
```cpp
void Game::combat(Player& player, Enemy& enemy) {
    // 伤害计算公式
    int playerDamage = max(1, player.getAttack() - enemy.getDefense() / 2);
    enemy.takeDamage(playerDamage);
    
    if (!enemy.isAlive()) {
        player.addExperience(enemy.getExperienceReward());  // 经验获取
        return;
    }
    
    // 敌人反击
    int enemyDamage = max(1, enemy.getAttack() - player.getDefense() / 2);
    player.takeDamage(enemyDamage);
}
```

## 五、设计模式应用

### 1. **工厂模式**
```cpp
// 在Game::spawnEnemies中根据条件创建不同类型敌人
int enemyType = rand() % 100;
if (level >= 3 && enemyType < 10) {
    enemies.push_back(make_shared<Boss>(x, y));  // 创建Boss
} else if (level >= 2 && enemyType < 30) {
    enemies.push_back(make_shared<Orc>(x, y));   // 创建Orc
} else {
    enemies.push_back(make_shared<Goblin>(x, y)); // 创建Goblin
}
```

### 2. **观察者模式**
消息系统实现了简单的观察者模式，当游戏事件发生时通知消息队列：
```cpp
void Game::addMessage(const string& text, Color color) {
    messages.push_back(GameMessage(text, color));
    if (messages.size() > MAX_MESSAGES) messages.pop_front();
}

// 事件触发时添加消息
addMessage("You triggered a trap!", BRIGHT_RED);
addMessage("You found a treasure!", BRIGHT_YELLOW);
```

### 3. **策略模式**
道具系统使用策略模式，每种道具实现不同的`use`策略：
```cpp
class HealthPotion : public Item {
    void use(Player& player) override { player.heal(healAmount); }
};

class WeaponUpgrade : public Item {
    void use(Player& player) override { player.setAttack(player.getAttack() + attackBonus); }
};
```

## 六、内存管理

### 智能指针应用
```cpp
vector<shared_ptr<Enemy>> enemies;  // 使用shared_ptr自动管理内存
vector<shared_ptr<Item>> items;

// 创建敌人
enemies.push_back(make_shared<Goblin>(x, y));

// 自动清理死亡敌人
enemies.erase(remove_if(enemies.begin(), enemies.end(),
    [](const shared_ptr<Enemy>& enemy) { return !enemy || !enemy->isAlive(); }),
    enemies.end());
```

## 七、面向对象设计原则应用

### 1. **单一职责原则**
- `Character`：只处理角色基本属性和行为
- `Player`：扩展玩家特有功能（经验、等级）
- `Game`：负责游戏流程控制和协调各组件

### 2. **开放-封闭原则**
- 添加新敌人类型只需继承`Enemy`类
- 添加新道具只需继承`Item`类并实现`use`方法

### 3. **里氏替换原则**
- 所有派生类（Goblin、Orc、Boss）都可以替换基类`Enemy`
- 所有道具类都可以替换基类`Item`

### 4. **依赖倒置原则**
- 高层模块`Game`依赖抽象类`Character`和`Item`
- 具体实现细节在派生类中完成

### 5. **接口隔离原则**
- `Character`提供角色通用接口
- `Item`提供道具使用接口
- 每个接口都专注于特定功能

## 八、枚举类型说明

### **`GameState` 游戏状态**
- `PLAYING`：游戏进行中
- `WIN`：玩家获胜
- `LOSE`：玩家失败
- `PAUSED`：游戏暂停

### **`Color` 颜色枚举**
- 基础颜色：`BLACK`(0)、`BLUE`(1)、`GREEN`(2)、`CYAN`(3)、`RED`(4)等
- 亮色变体：`BRIGHT_BLUE`(9)、`BRIGHT_GREEN`(10)、`BRIGHT_RED`(12)等
- 共16种颜色，支持丰富的控制台显示效果


## 九、总结

### 面向对象设计的优点体现：

1. **封装性良好**：每个类都有清晰的职责边界，属性私有化
2. **继承层次合理**：建立清晰的类层次结构，支持代码复用
3. **多态应用恰当**：虚函数实现不同类型实体的差异化行为
4. **组合优于继承**：`Game`类通过组合方式管理所有游戏组件
5. **智能内存管理**：使用智能指针避免内存泄漏


### 类关系图总结：
```
Game (主控类)

├── 管理 Player (玩家)

├── 管理 vector<shared_ptr<Enemy>> (敌人集合)

├── 管理 vector<shared_ptr<Item>> (道具集合)

├── 管理 vector<Trap> (陷阱集合)

├── 管理 deque<GameMessage> (消息队列)

└── 管理 vector<vector<char>> (游戏地图)

继承体系：
Character (角色基类)

├── Player (玩家) - 添加成长属性

└── Enemy (敌人基类) - 添加经验奖励

    ├── Goblin (地精)

    ├── Orc (兽人)

    └── Boss (首领)

Item (道具抽象基类)

├── HealthPotion (生命药水) - 添加治疗量

├── WeaponUpgrade (武器升级) - 添加攻击加成

└── Key (钥匙) - 无额外属性

辅助类：

Position (坐标) - 所有位置相关的基础

GameMessage (消息) - 游戏事件反馈

Trap (陷阱) - 环境危险元素
``

辅助类：
Position (坐标) - 所有位置相关的基础
GameMessage (消息) - 游戏事件反馈
Trap (陷阱) - 环境危险元素
``
