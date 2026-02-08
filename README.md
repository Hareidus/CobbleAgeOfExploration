# **宝可梦航海系统 - 核心架构设计（完整版）**

## **一、核心数据模型**

### **1. 船只系统**
```
ShipTemplate (船只模板)
├── 基础属性
│   ├── ship_id: String                    # 船只唯一标识
│   ├── ship_name: String                  # 显示名称
│   ├── ship_rarity: RarityLevel           # 品质等级 (白/绿/蓝/紫/金)
│   ├── base_speed: Int                    # 基础航行速度
│   ├── base_capacity: Int                 # 基础载重
│   ├── base_durability: Int               # 基础耐久度
│   ├── fuel_consumption: Int              # 每次航行消耗燃油瓶数
│   ├── race_value_limit: IntRange         # 可使用的宝可梦种族值范围
│   └── initial_fuel_tank: Int             # 初始油箱容量(L)
│
├── 技能系统
│   ├── passive_skills: List<SkillEffect>  # 被动技能列表
│   │   ├── type: SkillType                # 技能类型(减损耗/加属性/特殊效果)
│   │   ├── condition: TriggerCondition    # 触发条件(特定海域/天气)
│   │   └── effect_data: Map<String, Any>  # 效果数据(数值/概率等)
│   │
│   └── active_skills: List<SkillEffect>   # 主动技能(需手动使用)
│
├── 解锁条件
│   ├── unlock_type: UnlockType            # 解锁方式(声望/图纸/道具)
│   ├── required_items: Map<Item, Int>     # 所需材料
│   ├── required_currency: Int             # 所需货币/声望
│   └── prerequisite_ships: List<String>   # 前置船只要求
│
└── 进阶系统
    ├── upgrade_paths: List<UpgradePath>   # 升级路径
    ├── max_upgrade_level: Int             # 最大升级等级
    └── special_unlocks: Map<Int, String>  # 特定等级解锁的功能
```

### **2. 宝可梦加成系统**
```
PokemonBonus (宝可梦加成)
├── 属性加成
│   ├── pokemon_type: PokemonType          # 宝可梦属性(水系/飞行系等)
│   ├── bonus_type: BonusType              # 加成类型(速度/载重/概率等)
│   ├── bonus_value: Double                # 加成数值
│   └── max_stack: Int = 2                 # 同属性最大携带数量
│
├── 羁绊系统
│   ├── bond_id: String                    # 羁绊组合ID
│   ├── required_pokemons: Set<String>     # 所需宝可梦组合
│   ├── bond_effect: BondEffect            # 羁绊效果
│   │   ├── effect_type: EffectType        # 效果类型(减损耗/加概率)
│   │   ├── value: Double                  # 效果数值
│   │   └── conditions: List<Condition>    # 生效条件(特定船/海域)
│   │
│   └── bond_level: Int                    # 羁绊等级(可升级)
│
└── 特性联动
    ├── trait_combinations: Set<String>    # 特性组合列表
    ├── combo_effect: TraitEffect          # 组合效果
    └── priority: Int                      # 效果优先级
```

### **3. 海域探索系统**
```
SeaArea (海域)
├── 基础配置
│   ├── area_id: String                    # 海域唯一ID
│   ├── area_name: String                  # 海域名称
│   ├── unlock_requirement: Requirement    # 解锁要求(声望/前置海域)
│   ├── base_reputation: Int               # 基础声望奖励
│   └── is_infinite: Boolean = true        # 是否为无限海域
│
├── 航行参数
│   ├── total_distance: IntRange?          # 固定总里程(随机时为范围)
│   ├── base_fuel_per_nautical: Double     # 每海里基础油耗
│   ├── base_durability_cost: Int          # 每次航行基础耐久损耗
│   ├── cargo_weight_cost: Int             # 货物装载耐久惩罚
│   └── recommended_speed: Int             # 推荐速度(3天返回)
│
├── 专属配置
│   ├── allowed_ships: Set<String>         # 可进入的船只类型
│   ├── ship_bonus_map: Map<String, Double> # 特定船只加成
│   ├── pokemon_bonus_map: Map<String, Double> # 特定属性加成
│   └── special_restrictions: List<String> # 特殊限制条件
│
└── 产出系统
    ├── loot_tables: Map<LootLevel, LootTable> # 不同等级掉落表
    ├── special_drops: List<SpecialDrop>   # 专属物品掉落
    └── reputation_calculation: ReputationFormula # 声望计算公式
```

### **4. 货物与载重系统**
```
CargoSystem (货物系统)
├── 货物类型
│   ├── cargo_type: CargoType              # 货物等级(普通/中级/高级/稀有/特殊)
│   ├── unit_weight: Int                   # 单件重量
│   ├── base_value: Double                 # 基础价值
│   ├── stack_limit: Int                   # 堆叠上限
│   └── special_attributes: Map<String, Any> # 特殊属性
│
├── 载重计算
│   ├── base_capacity: Int                 # 基础载重(船只)
│   ├── bonus_capacity: Int                # 加成载重(地面系+货箱)
│   ├── current_load: Int                  # 当前装载量
│   └── overload_penalty: PenaltyEffect    # 超载惩罚
│
└── 货箱系统
    ├── cargo_box_type: BoxType            # 货箱类型(初级/中级/高级)
    ├── capacity_bonus: Int                # 容量加成
    ├── fuel_penalty: Double               # 燃油惩罚比例(>2个时每个+5%)
    └── durability_cost: Int               # 耐久成本
```

### **5. 事件与天气系统**
```
EventSystem (事件系统)
├── 事件模板
│   ├── event_id: String                   # 事件唯一ID
│   ├── event_name: String                 # 事件名称
│   ├── event_type: EventType              # 类型(普通/稀有/史诗)
│   ├── trigger_type: TriggerType          # 触发方式(距离/时间/概率)
│   ├── trigger_conditions: List<Condition> # 触发条件
│   │   ├── depth_condition: IntRange      # 深度范围
│   │   ├── ship_condition: Set<String>    # 船只要求
│   │   ├── pokemon_condition: Set<String> # 宝可梦要求
│   │   ├── weather_condition: Set<String> # 天气要求
│   │   └── time_condition: TimeRange?     # 时间要求
│   │
│   ├── event_effects: List<EventEffect>   # 事件效果
│   │   ├── effect_type: EffectType        # 效果类型
│   │   │   ├── DURABILITY_CHANGE         # 耐久变化
│   │   │   ├── CARGO_LOSS                # 货物损失
│   │   │   ├── ITEM_REWARD               # 物品奖励
│   │   │   ├── STAT_MODIFIER             # 属性修改
│   │   │   ├── FUEL_CONSUMPTION          # 燃油消耗
│   │   │   └── SPECIAL_ACTION            # 特殊动作
│   │   │
│   │   └── effect_data: Map<String, Any>  # 效果数据
│   │
│   └── cooldown_config: CooldownConfig   # 冷却配置(如每周触发)
│
├── 天气系统
│   ├── weather_id: String                 # 天气ID
│   ├── weather_name: String               # 天气名称
│   ├── spawn_weight: Int                  # 出现权重
│   ├── duration: IntRange                 # 持续时间范围
│   ├── area_restrictions: Set<String>     # 海域限制
│   └── weather_effects: List<WeatherEffect> # 天气效果
│
└── 随机任务系统
    ├── task_type: TaskType                # 任务类型(收集/击败/探索)
    ├── task_objective: Objective          # 任务目标
    ├── task_reward: Reward                # 任务奖励
    └── fail_conditions: List<Condition>   # 失败条件
```

### **6. 养成与成长系统**
```
ProgressionSystem (成长系统)
├── 油箱升级
│   ├── fuel_tank_level: Int               # 当前油箱等级
│   ├── max_fuel_tank_level: Int = 5       # 最大等级
│   ├── tank_capacity_formula: Formula     # 容量计算公式
│   └── upgrade_requirements: Map<Int, UpgradeReq> # 每级升级要求
│       ├── required_reputation: Int       # 所需声望
│       ├── required_items: Map<Item, Int> # 所需材料
│       └── prerequisite_level: Int?       # 前置等级要求
│
├── 船只进阶
│   ├── ship_tier: ShipTier                # 当前船只品阶
│   ├── tier_upgrade_path: UpgradePath     # 进阶路径
│   ├── inherit_fuel_tank: Boolean = true  # 是否继承油箱等级
│   └── tier_bonuses: Map<ShipTier, TierBonus> # 品阶加成
│
├── 声望系统
│   ├── player_reputation: Int             # 当前声望
│   ├── reputation_sources: Map<String, Int> # 声望来源记录
│   ├── reputation_multiplier: Formula     # 声望倍数公式(基于深度)
│   └── unlocked_areas: Set<String>        # 已解锁海域
│
└── 船长等级
    ├── captain_level: Int                 # 船长等级
    ├── captain_exp: Int                   # 船长经验
    ├── level_rewards: Map<Int, LevelReward> # 等级奖励
    └── global_bonuses: Map<Int, GlobalBonus> # 全局加成
```

### **7. 物品与道具系统**
```
ItemSystem (物品系统)
├── 消耗品类
│   ├── FuelItem (燃油)
│   │   ├── fuel_type: FuelType            # 燃油类型(普通/高级)
│   │   ├── fuel_amount: Int               # 燃油量(L)
│   │   ├── efficiency_bonus: Double       # 效率加成
│   │   └── special_effects: List<Effect>  # 特殊效果
│   │
│   └── ConsumableItem (消耗品)
│       ├── usage_type: UsageType          # 使用类型(单次/永久)
│       ├── effect_duration: Duration      # 效果持续时间
│       ├── stackable: Boolean             # 是否可堆叠
│       └── cooldown: Duration?            # 使用冷却
│
├── 材料类
│   ├── UpgradeMaterial (升级材料)
│   │   ├── material_type: MaterialType    # 材料类型(基础/中级/高级/史诗)
│   │   ├── used_for: Set<UpgradeType>     # 可用于的升级类型
│   │   └── rarity: RarityLevel            # 材料稀有度
│   │
│   └── EvolutionMaterial (进化材料)
│       ├── target_pokemon: String         # 目标宝可梦
│       ├── evolution_stage: Int           # 进化阶段
│       └── additional_requirements: List<Requirement> # 额外要求
│
├── 功能道具
│   ├── CargoBoxItem (货箱)
│   │   ├── box_level: BoxLevel            # 货箱等级
│   │   ├── capacity_increase: Int         # 容量增加
│   │   └── installation_cost: Cost        # 安装成本
│   │
│   ├── ShipUpgradeItem (船只升级道具)
│   │   ├── upgrade_type: ShipUpgradeType  # 升级类型(图纸/部件)
│   │   ├── target_ship: String            # 目标船只
│   │   └── unlock_requirement: Requirement # 解锁要求
│   │
│   └── ProtectionItem (防护道具)
│       ├── protection_type: ProtectionType # 防护类型(事件/天气/袭击)
│       ├── protected_distance: Int        # 保护距离
│       └── immunity_list: Set<String>     # 免疫事件列表
│
└── 宝可梦道具
    ├── PokemonEnhancementItem (增强道具)
    │   ├── target_attribute: AttributeType # 目标属性
    │   ├── enhancement_value: Double      # 增强数值
    │   └── duration_type: DurationType    # 持续时间类型
    │
    └── BondItem (羁绊道具)
        ├── bond_combination: Set<String>  # 羁绊组合
        ├── bond_strength_increase: Double # 羁绊强度增加
        └── is_permanent: Boolean          # 是否为永久加成
```

### **8. 战斗与遭遇系统**
```
CombatSystem (战斗系统)
├── 遭遇类型
│   ├── encounter_id: String               # 遭遇ID
│   ├── encounter_type: EncounterType      # 类型(海盗/野生宝可梦/海怪)
│   ├── spawn_conditions: SpawnCondition   # 生成条件
│   └── combat_ai: CombatAI                # 战斗AI配置
│
├── 战斗属性
│   ├── enemy_stats: CombatStats          # 敌方属性
│   │   ├── health: Int                    # 生命值
│   │   ├── attack: Int                    # 攻击力
│   │   ├── defense: Int                   # 防御力
│   │   └── special_abilities: List<Ability> # 特殊能力
│   │
│   ├── player_stats: CombatStats         # 玩家属性(基于宝可梦)
│   └── environmental_factors: Map<String, Double> # 环境因素加成
│
├── 战斗流程
│   ├── battle_phases: List<BattlePhase>  # 战斗阶段
│   ├── turn_system: TurnConfig           # 回合制配置
│   ├── escape_chance: Double             # 逃跑概率
│   └── surrender_conditions: List<Condition> # 投降条件
│
└── 战斗结果
    ├── victory_rewards: Reward           # 胜利奖励
    ├── defeat_penalties: Penalty         # 失败惩罚
    ├── draw_conditions: Condition        # 平局条件
    └── reputation_impact: Int            # 声望影响
```

### **9. 经济与商店系统**
```
EconomySystem (经济系统)
├── 货币体系
│   ├── primary_currency: Currency        # 主要货币(声望)
│   ├── secondary_currency: Currency?     # 次要货币(游戏币)
│   ├── exchange_rate: Double?            # 兑换比例
│   └── currency_sources: Map<String, Int> # 货币来源
│
├── 商店系统
│   ├── shop_id: String                   # 商店ID
│   ├── shop_type: ShopType               # 商店类型(港口/神秘商人)
│   ├── unlock_requirements: Requirement  # 解锁要求
│   ├── stock_items: List<ShopItem>       # 库存商品
│   │   ├── item: Item                    # 商品物品
│   │   ├── price: Price                  # 价格
│   │   ├── stock_limit: Int              # 库存限制
│   │   ├── refresh_interval: Duration    # 刷新间隔
│   │   └── purchase_restrictions: List<Restriction> # 购买限制
│   │
│   └── special_offers: List<SpecialOffer> # 特价商品
│
├── 交易系统
│   ├── trading_formulas: Map<String, Formula> # 交易计算公式
│   ├── price_fluctuation: PriceFluctuation # 价格波动
│   └── black_market: BlackMarketConfig   # 黑市配置
│
└── 租赁系统
    ├── rental_items: List<RentalItem>    # 可租赁物品
    ├── rental_duration: Duration         # 租赁时长
    ├── rental_cost: Cost                 # 租赁费用
    └── overdue_penalty: Penalty          # 超时惩罚
```

### **10. 航行模拟器**
```
VoyageSimulator (航行模拟器)
├── 航行状态
│   ├── voyage_id: String                 # 航行唯一ID
│   ├── player_uuid: UUID                 # 玩家UUID
│   ├── start_time: Long                  # 开始时间戳
│   ├── estimated_end_time: Long          # 预计结束时间
│   ├── current_status: VoyageStatus      # 当前状态(进行中/已完成/失败)
│   └── voyage_log: List<VoyageLogEntry>  # 航行日志
│
├── 实时数据
│   ├── current_distance: Double          # 当前航行距离
│   ├── remaining_distance: Double        # 剩余距离
│   ├── current_speed: Double             # 当前速度(受天气影响)
│   ├── fuel_remaining: Double            # 剩余燃油
│   ├── durability_remaining: Int         # 剩余耐久
│   └── current_weather: String?          # 当前天气
│
├── 事件调度
│   ├── event_scheduler: EventScheduler   # 事件调度器
│   ├── scheduled_events: Queue<ScheduledEvent> # 已调度事件
│   ├── triggered_events: Set<String>     # 已触发事件(防重复)
│   └── cooldown_tracker: CooldownTracker # 冷却跟踪器
│
└── 结算系统
    ├── result_calculator: ResultCalculator # 结果计算器
    ├── reward_distributor: RewardDistributor # 奖励分发器
    ├── penalty_applier: PenaltyApplier   # 惩罚应用器
    └── reputation_calculator: ReputationCalculator # 声望计算器
```

## **二、核心流程架构**

### **1. 航行启动流程**
```
1. 玩家配置航行
   ├── 选择船只(检查种族值限制)
   ├── 选择宝可梦编队(检查属性上限)
   ├── 选择目标海域(检查解锁状态)
   ├── 设置航行距离(计算燃油需求)
   ├── 装载货物(检查载重上限)
   └── 使用道具(祝福类)

2. 预检系统
   ├── 燃油充足检查
   ├── 耐久度检查
   ├── 船只状态检查
   ├── 宝可梦状态检查
   ├── 货物合法性检查
   └── 海域准入检查

3. 成本扣除
   ├── 扣除燃油消耗
   ├── 应用耐久基础损耗
   ├── 应用货物装载损耗
   └── 消耗使用的道具

4. 创建航行任务
   └── 初始化VoyageSimulator实例
```

### **2. 航行模拟流程**
```
1. 时间推进循环(每tick/异步)
   ├── 更新航行进度
   ├── 检查天气变化
   ├── 触发距离事件
   ├── 触发时间事件
   └── 更新实时数据

2. 事件处理流程
   ├── 检查触发条件
   ├── 应用天气影响
   ├── 应用宝可梦加成
   ├── 应用船只技能
   ├── 执行事件效果
   └── 记录航行日志

3. 状态监控
   ├── 燃油耗尽检测
   ├── 耐久耗尽检测
   ├── 突发事件处理
   └── 玩家离线处理
```

### **3. 航行结算流程**
```
1. 航行结束判定
   ├── 正常完成(到达距离)
   ├── 中途返回(玩家取消)
   ├── 失败返回(燃油/耐久耗尽)
   └── 紧急救援(特殊道具)

2. 结果计算
   ├── 计算总耗时
   ├── 计算总消耗
   ├── 汇总获得物品
   ├── 计算货物损失
   └── 计算声望奖励

3. 奖励发放
   ├── 物品入库(检查仓库空间)
   ├── 声望更新
   ├── 经验分配
   ├── 成就解锁
   └── 统计数据更新

4. 船只维护
   ├── 耐久度修复
   ├── 技能冷却重置
   ├── 宝可梦疲劳恢复
   └── 道具使用记录
```

## **三、核心管理器架构**

### **1. 管理器类结构**
```
├── VoyageManager (航行管理器)
│   ├── 管理所有进行中的航行
│   ├── 处理航行创建/取消/查询
│   └── 协调其他管理器工作
│
├── ShipManager (船只管理器)
│   ├── 玩家船只数据管理
│   ├── 船只升级/进阶处理
│   └── 船只状态维护
│
├── PokemonManager (宝可梦管理器)
│   ├── 编队配置管理
│   ├── 加成计算服务
│   └── 羁绊效果管理
│
├── AreaManager (海域管理器)
│   ├── 海域数据管理
│   ├── 解锁状态跟踪
│   └── 海域事件池管理
│
├── EventManager (事件管理器)
│   ├── 事件调度与触发
│   ├── 事件效果执行
│   └── 事件冷却管理
│
├── WeatherManager (天气管理器)
│   ├── 天气生成与变化
│   ├── 天气效果应用
│   └── 天气持续时间管理
│
├── InventoryManager (库存管理器)
│   ├── 货物存储管理
│   ├── 载重计算服务
│   └── 物品存取处理
│
├── EconomyManager (经济管理器)
│   ├── 货币管理
│   ├── 商店交易处理
│   └── 价格系统管理
│
├── ReputationManager (声望管理器)
│   ├── 声望计算与发放
│   ├── 成就系统管理
│   └── 解锁条件检查
│
└── DataManager (数据管理器)
    ├── 玩家数据持久化
    ├── 航行数据保存
    └── 系统配置管理
```

### **2. 扩展点设计**
```
├── 插件扩展点
│   ├── CustomShip (自定义船只)
│   ├── CustomArea (自定义海域)
│   ├── CustomEvent (自定义事件)
│   ├── CustomItem (自定义物品)
│   └── CustomBonus (自定义加成)
│
├── 条件系统
│   ├── ConditionParser (条件解析器)
│   ├── RequirementChecker (要求检查器)
│   └── ValidatorChain (验证器链)
│
├── 效果系统
│   ├── EffectExecutor (效果执行器)
│   ├── ModifierStack (修改器堆栈)
│   └── PriorityQueue (优先级队列)
│
└── 公式系统
    ├── FormulaParser (公式解析器)
    ├── VariableResolver (变量解析器)
    └── ExpressionEvaluator (表达式求值器)
```

## **四、数据持久化架构**

### **1. 数据存储结构**
```
├── 玩家数据表
│   ├── player_uuid
│   ├── reputation
│   ├── captain_level
│   ├── captain_exp
│   └── unlocked_areas (JSON数组)
│
├── 船只数据表
│   ├── ship_instance_id
│   ├── player_uuid
│   ├── ship_template_id
│   ├── current_durability
│   ├── fuel_tank_level
│   ├── fuel_tank_capacity
│   ├── installed_boxes (JSON数组)
│   └── skill_cooldowns (JSON对象)
│
├── 航行数据表
│   ├── voyage_id
│   ├── player_uuid
│   ├── voyage_config (JSON对象)
│   ├── start_time
│   ├── estimated_end_time
│   ├── current_progress (JSON对象)
│   └── voyage_status
│
├── 库存数据表
│   ├── player_uuid
│   ├── item_type
│   ├── item_id
│   ├── item_data (JSON对象)
│   ├── quantity
│   └── expiration_time
│
└── 统计数据表
    ├── player_uuid
    ├── total_voyages
    ├── total_distance
    ├── total_reputation
    └── achievement_data (JSON对象)
```

### **2. 配置存储结构**
```
├── 静态配置(JSON/YAML)
│   ├── ships/ (船只配置)
│   ├── areas/ (海域配置)
│   ├── events/ (事件配置)
│   ├── items/ (物品配置)
│   ├── pokemon_bonuses/ (宝可梦加成)
│   ├── formulas/ (公式配置)
│   └── localization/ (本地化文本)
│
├── 动态配置(数据库)
│   ├── shop_stock (商店库存)
│   ├── global_weather (全局天气)
│   ├── event_cooldowns (事件冷却)
│   └── system_variables (系统变量)
│
└── 运行时缓存
    ├── template_cache (模板缓存)
    ├── player_cache (玩家缓存)
    ├── voyage_cache (航行缓存)
    └── calculation_cache (计算缓存)
```

## **五、界面系统架构**

### **1. GUI界面结构**
```
├── 主菜单界面
│   ├── 航行控制面板
│   ├── 船只管理界面
│   ├── 宝可梦编队界面
│   ├── 货物仓库界面
│   ├── 商店交易界面
│   └── 成就统计界面
│
├── 航行配置界面
│   ├── 海域选择面板
│   ├── 距离设置滑块
│   ├── 货物装载面板
│   ├── 道具使用面板
│   ├── 成本预览面板
│   └── 确认启动按钮
│
├── 航行监控界面
│   ├── 实时进度显示
│   ├── 当前状态信息
│   ├── 事件日志查看
│   ├── 紧急操作按钮
│   └── 预估返回时间
│
└── 结算界面
    ├── 航行结果汇总
    ├── 获得物品展示
    ├── 消耗统计显示
    ├── 声望获得显示
    └── 继续航行按钮
```

### **2. 信息显示系统**
```
├── HUD显示组件
│   ├── 航行进度条
│   ├── 燃油剩余条
│   ├── 耐久度条
│   ├── 当前速度显示
│   └── 天气图标显示
│
├── 通知系统
│   ├── 事件触发通知
│   ├── 状态变化通知
│   ├── 危险预警通知
│   └── 完成提醒通知
│
└── 数据展示组件
    ├── 属性对比面板
    ├── 概率计算器
    ├── 收益模拟器
    └── 历史记录查看
```

这个架构完整覆盖了策划案中的所有系统，包括：
- 完整的船只养成体系(升级/进阶)
- 宝可梦编队与羁绊系统
- 海域探索与事件系统
- 货物与载重管理系统
- 天气与随机事件系统
- 声望与成就系统
- 商店与经济系统
- 航行模拟与结算系统

所有组件都设计为可配置、可扩展的，便于后续添加新的船只、海域、事件和功能。



# **冒险故事生成系统 - 深度集成架构**

## **一、故事生成核心系统**

### **1. 故事数据模型**
```
StoryGenerationSystem (故事生成系统)
├── 输入数据模型
│   ├── StoryInputData (故事输入数据)
│   │   ├── voyage_context: VoyageContext      # 航行上下文
│   │   ├── team_composition: TeamInfo         # 队伍构成
│   │   ├── event_sequence: List<EventRecord>  # 事件序列
│   │   ├── current_status: VoyageStatus       # 当前状态
│   │   └── additional_context: Map<String, Any> # 额外上下文
│   │
│   ├── VoyageContext (航行上下文)
│   │   ├── sea_area: AreaInfo                 # 海域信息
│   │   ├── distance_traveled: Double          # 已航行距离
│   │   ├── time_elapsed: Duration             # 已用时间
│   │   ├── weather_history: List<WeatherRecord> # 天气历史
│   │   └── destination: String?               # 目标/终点
│   │
│   └── TeamInfo (队伍信息)
│       ├── captain_pokemon: PokemonProfile    # 队长宝可梦
│       ├── crew_members: List<PokemonProfile> # 船员宝可梦
│       ├── team_dynamics: TeamDynamics        # 队伍互动关系
│       └── special_roles: Map<String, String> # 特殊角色分配
│
├── 宝可梦角色模型
│   ├── PokemonProfile (宝可梦档案)
│   │   ├── pokemon_id: String                 # 宝可梦ID
│   │   ├── nickname: String?                  # 昵称
│   │   ├── level: Int                         # 等级
│   │   ├── nature: String                     # 性格
│   │   ├── ability: String                    # 特性
│   │   ├── moves: List<String>                # 技能列表
│   │   ├── friendship: Int                    # 亲密度
│   │   ├── personality_traits: List<String>   # 性格特征
│   │   └── background_story: String?          # 背景故事
│   │
│   └── TeamDynamics (队伍动态)
│       ├── relationships: Map<Pair<String, String>, Relationship> # 关系网
│       ├── team_mood: TeamMood                # 队伍情绪
│       ├── inside_jokes: List<String>         # 内部梗/笑话
│       └── past_adventures: List<String>      # 过去冒险回忆
│
├── 事件记录模型
│   ├── EventRecord (事件记录)
│   │   ├── event_id: String                   # 事件ID
│   │   ├── event_type: EventType              # 事件类型
│   │   ├── timestamp: Long                    # 发生时间
│   │   ├── location: String                   # 发生位置
│   │   ├── participants: List<String>         # 参与者(宝可梦ID)
│   │   ├── outcome: EventOutcome              # 事件结果
│   │   ├── detailed_log: String               # 详细日志
│   │   └── emotional_impact: EmotionalImpact  # 情感影响
│   │
│   └── EventOutcome (事件结果)
│       ├── success: Boolean                   # 是否成功
│       ├── casualties: List<Casualty>         # 伤亡情况
│       ├── items_gained: List<ItemGain>       # 获得物品
│       ├── items_lost: List<ItemLoss>         # 损失物品
│       └── lessons_learned: List<String>      # 学到的教训
│
└── 情感系统
    ├── EmotionalImpact (情感影响)
    │   ├── excitement_level: Int              # 兴奋度(0-100)
    │   ├── fear_level: Int                    # 恐惧度(0-100)
    │   ├── camaraderie_change: Int            # 团队情谊变化(-100 to 100)
    │   └── morale_change: Int                 # 士气变化(-100 to 100)
    │
    └── TeamMood (队伍情绪)
        ├── overall_mood: MoodType             # 总体情绪
        ├── stress_level: Int                  # 压力水平
        ├── fatigue_level: Int                 # 疲劳度
        └── special_conditions: List<String>   # 特殊状态
```

### **2. 故事生成器配置**
```
StoryGeneratorConfig (故事生成器配置)
├── AI接口配置
│   ├── ai_provider: AIProvider                # AI提供商(DeepSeek)
│   ├── api_endpoint: String                   # API端点
│   ├── api_key: String                        # API密钥(加密存储)
│   ├── model_name: String                     # 模型名称
│   ├── temperature: Double                    # 温度参数
│   ├── max_tokens: Int                        # 最大token数
│   └── timeout_seconds: Int                   # 超时时间
│
├── 提示词模板系统
│   ├── prompt_templates: Map<StoryType, PromptTemplate> # 提示词模板
│   │   ├── template_id: String                # 模板ID
│   │   ├── template_type: StoryType           # 故事类型
│   │   ├── system_prompt: String              # 系统提示词
│   │   ├── user_prompt_template: String       # 用户提示词模板
│   │   ├── variables: Set<String>             # 模板变量
│   │   ├── format_specification: String       # 输出格式规范
│   │   └── example_output: String             # 示例输出
│   │
│   └── variable_handlers: Map<String, VariableHandler> # 变量处理器
│       ├── extractor: (Context) -> String     # 变量提取函数
│       ├── formatter: (Any) -> String         # 格式化函数
│       └── fallback_value: String             # 回退值
│
├── 风格控制系统
│   ├── writing_styles: Map<String, WritingStyle> # 写作风格
│   │   ├── style_name: String                 # 风格名称
│   │   ├── tone: ToneType                     # 语气(幽默/严肃/史诗)
│   │   ├── perspective: PerspectiveType       # 视角(第一人称/第三人称)
│   │   ├── detail_level: DetailLevel          # 细节程度
│   │   ├── dialogue_ratio: Double             # 对话比例(0-1)
│   │   └── special_instructions: List<String> # 特殊指令
│   │
│   └── character_voices: Map<String, CharacterVoice> # 角色语音风格
│       ├── character_id: String               # 角色ID
│       ├── speaking_style: String             # 说话风格
│       ├── catchphrases: List<String>         # 口头禅
│       ├── speech_patterns: List<String>      # 说话模式
│       └── emotional_responses: Map<Emotion, String> # 情绪反应
│
└── 质量控制
    ├── validators: List<StoryValidator>       # 故事验证器
    │   ├── validator_type: ValidatorType      # 验证类型
    │   ├── validation_rules: List<Rule>       # 验证规则
    │   └── correction_prompt: String?         # 修正提示词
    │
    ├── content_filters: List<ContentFilter>   # 内容过滤器
    ├── length_constraints: LengthConstraints  # 长度限制
    ├── retry_policy: RetryPolicy              # 重试策略
    └── fallback_generators: List<FallbackGenerator> # 备用生成器
```

### **3. 故事类型与模板**
```
StoryTypeSystem (故事类型系统)
├── 航行阶段故事
│   ├── DEPARTURE_STORY (启航故事)            # 航行开始时生成
│   ├── DAILY_LOG_STORY (每日日志)            # 每日航行总结
│   ├── MILESTONE_STORY (里程碑故事)          # 达到重要里程碑
│   ├── EVENT_ENCOUNTER_STORY (事件遭遇故事)   # 遭遇特殊事件时
│   └── ARRIVAL_STORY (抵达故事)              # 成功抵达目的地
│
├── 事件类型故事
│   ├── COMBAT_STORY (战斗故事)               # 战斗遭遇
│   ├── TREASURE_STORY (寻宝故事)             # 发现宝藏
│   ├── STORM_SURVIVAL_STORY (风暴求生故事)   # 度过风暴
│   ├── MYSTERY_ENCOUNTER_STORY (神秘遭遇故事) # 遇到神秘生物/事件
│   └── CHARACTER_DEVELOPMENT_STORY (角色成长故事) # 宝可梦成长
│
├── 结局类型故事
│   ├── SUCCESSFUL_RETURN_STORY (成功返航故事) # 成功返回
│   ├── HARROWING_RETURN_STORY (惊险返航故事)  # 险象环生返回
│   ├── FAILED_VOYAGE_STORY (失败航行故事)     # 航行失败
│   ├── SHIPWRECK_STORY (沉船故事)            # 船只沉没
│   └── HEROIC_SACRIFICE_STORY (英勇牺牲故事)  # 有宝可梦牺牲
│
└── 特殊类型故事
    ├── CHARACTER_BONDING_STORY (角色羁绊故事) # 宝可梦建立羁绊
    ├── COMIC_RELIEF_STORY (喜剧 relief 故事)  # 轻松幽默时刻
    ├── FLASHBACK_STORY (闪回故事)            # 回忆过去冒险
    ├── PROPHECY_STORY (预言故事)             # 预示未来事件
    └── LEGEND_UNCOVERED_STORY (传说揭示故事)  # 揭示海域传说
```

### **4. 纪念物与纪念品系统**
```
MementoSystem (纪念物系统)
├── 纪念物类型
│   ├── MementoType (纪念物类型)
│   │   ├── PHYSICAL_MEMENTO (实体纪念物)     # 可收集的物品
│   │   ├── DOCUMENTARY_MEMENTO (文件纪念物)   # 照片/日志/地图
│   │   ├── AUDIO_MEMENTO (音频纪念物)        # 录音/歌声
│   │   ├── SKETCH_MEMENTO (素描纪念物)       # 手绘素描
│   │   └── METAPHORICAL_MEMENTO (象征纪念物)  # 教训/回忆/情感
│   │
│   └── MementoItem (纪念物物品)
│       ├── memento_id: String                # 纪念物ID
│       ├── name: String                      # 名称
│       ├── description: String               # 描述
│       ├── story_context: String             # 故事上下文
│       ├── associated_pokemon: List<String>  # 关联的宝可梦
│       ├── rarity: RarityLevel               # 稀有度
│       ├── display_properties: DisplayProps  # 显示属性
│       └── interactive_actions: List<Action> # 可交互动作
│
├── 纪念物生成规则
│   ├── GenerationTrigger (生成触发器)
│   │   ├── trigger_type: TriggerType         # 触发类型
│   │   ├── required_conditions: List<Condition> # 触发条件
│   │   ├── probability: Double               # 触发概率
│   │   └── cooldown_period: Duration         # 冷却时间
│   │
│   ├── MementoGenerationRule (生成规则)
│   │   ├── rule_id: String                   # 规则ID
│   │   ├── applicable_story_types: Set<StoryType> # 适用故事类型
│   │   ├── memento_template: MementoTemplate # 纪念物模板
│   │   ├── quantity_range: IntRange          # 数量范围
│   │   └── quality_factors: List<QualityFactor> # 质量影响因素
│   │
│   └── MementoTemplate (纪念物模板)
│       ├── name_template: String             # 名称模板
│       ├── description_template: String      # 描述模板
│       ├── variable_slots: Map<String, String> # 变量槽位
│       └── validation_rules: List<Rule>      # 验证规则
│
└── 纪念品收集系统
    ├── MementoCollection (纪念品收藏)
    │   ├── collection_id: String             # 收藏ID
    │   ├── player_uuid: UUID                 # 玩家UUID
    │   ├── collected_mementos: List<CollectedMemento> # 已收集纪念物
    │   ├── display_album: AlbumConfig        # 展示相册配置
    │   └── statistics: CollectionStats       # 收集统计
    │
    ├── CollectedMemento (已收集纪念物)
    │   ├── collection_date: Long             # 收集日期
    │   ├── voyage_id: String                 # 关联的航行ID
    │   ├── memento_item: MementoItem         # 纪念物物品
    │   ├── display_location: DisplayLocation # 展示位置
    │   └── personal_note: String?            # 个人备注
    │
    └── MementoGallery (纪念物画廊)
        ├── gallery_layout: GalleryLayout     # 画廊布局
        ├── categorization: Categorization    # 分类方式
        ├── sharing_features: SharingConfig   # 分享功能
        └── interactive_features: InteractiveConfig # 交互功能
```

### **5. 邮件集成系统**
```
EmailIntegrationSystem (邮件集成系统)
├── 邮件内容模型
│   ├── VoyageReportEmail (航行报告邮件)
│   │   ├── email_id: String                  # 邮件ID
│   │   ├── player_uuid: UUID                 # 收件人
│   │   ├── voyage_id: String                 # 关联航行ID
│   │   ├── subject: String                   # 邮件主题
│   │   ├── greeting: String                  # 问候语
│   │   ├── adventure_story: AdventureStory   # 冒险故事
│   │   ├── mementos_list: List<MementoItem>  # 纪念物列表
│   │   ├── voyage_statistics: VoyageStats    # 航行统计
│   │   ├── attached_items: List<Item>        # 附件物品
│   │   ├── next_steps: List<String>          # 下一步建议
│   │   ├── signature: String                 # 签名
│   │   └── postscript: String?               # 附言
│   │
│   ├── AdventureStory (冒险故事)
│   │   ├── title: String                     # 故事标题
│   │   ├── author: String                    # 作者(宝可梦名字)
│   │   ├── date_written: Long                # 写作日期
│   │   ├── story_content: String             # 故事内容
│   │   ├── story_type: StoryType             # 故事类型
│   │   ├── emotional_tone: EmotionalTone     # 情感基调
│   │   ├── featured_characters: List<String> # 主要角色
│   │   ├── key_moments: List<String>         # 关键时刻
│   │   └── moral_lesson: String?             # 道德/教训
│   │
│   └── SpecialEmailTypes (特殊邮件类型)
│       ├── URGENT_MESSAGE (紧急消息)         # 航行中出现危机
│       ├── CHARACTER_LETTER (角色信件)       # 宝可梦写的信
│       ├── MYSTERY_MAIL (神秘邮件)          # 未知发件人
│       ├── TREASURE_MAP (藏宝图)            # 附带地图
│       └── INVITATION (邀请函)              # 活动邀请
│
├── 邮件生成流程
│   ├── EmailGenerationPipeline (邮件生成流水线)
│   │   ├── 1. 收集航行数据
│   │   ├── 2. 确定邮件类型
│   │   ├── 3. 生成故事内容
│   │   ├── 4. 生成纪念物
│   │   ├── 5. 组装邮件内容
│   │   ├── 6. 应用邮件模板
│   │   ├── 7. 添加附件
│   │   └── 8. 发送到邮件系统
│   │
│   └── EmailTemplateSystem (邮件模板系统)
│       ├── template_repository: TemplateRepo # 模板仓库
│       ├── template_variables: VariableSet   # 模板变量集
│       ├── conditional_sections: List<ConditionalSection> # 条件区块
│       └── style_cascading: StyleCascade     # 样式层叠
│
├── 邮件插件接口
│   ├── EmailPluginInterface (邮件插件接口)
│   │   ├── send_email(email: Email): Boolean # 发送邮件
│   │   ├── check_inbox(player: UUID): List<Email> # 检查收件箱
│   │   ├── mark_as_read(email_id: String)    # 标记为已读
│   │   ├── delete_email(email_id: String)    # 删除邮件
│   │   └── get_attachment(email_id: String, attachment_id: String): Item? # 获取附件
│   │
│   └── EmailPluginConfig (邮件插件配置)
│       ├── plugin_name: String               # 插件名称
│       ├── api_version: String               # API版本
│       ├── required_features: Set<String>    # 所需功能
│       ├── callback_urls: Map<String, String> # 回调URL
│       └── rate_limits: RateLimitConfig      # 频率限制
│
└── 邮件交互系统
    ├── EmailInteraction (邮件交互)
    │   ├── reply_options: List<ReplyOption>  # 回复选项
    │   ├── quick_actions: List<QuickAction>  # 快速操作
    │   ├── embedded_forms: List<EmbeddedForm> # 内嵌表单
    │   └── tracking_pixels: List<TrackingPixel> # 跟踪像素
    │
    ├── EmailTracking (邮件跟踪)
    │   ├── open_tracking: Boolean            # 打开跟踪
    │   ├── click_tracking: Boolean           # 点击跟踪
    │   ├── read_duration_tracking: Boolean   # 阅读时长跟踪
    │   └── analytics_collection: AnalyticsConfig # 分析收集
    │
    └── EmailScheduling (邮件调度)
        ├── delivery_schedule: DeliverySchedule # 投递计划
        ├── timezone_handling: TimezoneConfig # 时区处理
        ├── priority_levels: PriorityLevels   # 优先级等级
        └── retry_mechanism: RetryMechanism   # 重试机制
```

### **6. 故事生成管理器**
```
StoryGenerationManager (故事生成管理器)
├── 管理器核心
│   ├── StoryGenerator (故事生成器)
│   │   ├── generate_story(input: StoryInputData): StoryResult # 生成故事
│   │   ├── generate_mementos(story: AdventureStory, context: VoyageContext): List<MementoItem> # 生成纪念物
│   │   ├── validate_story(story: String): ValidationResult   # 验证故事
│   │   └── fallback_generation(input: StoryInputData): StoryResult # 备用生成
│   │
│   ├── PromptEngine (提示词引擎)
│   │   ├── build_prompt(input: StoryInputData, template: PromptTemplate): String # 构建提示词
│   │   ├── extract_variables(template: String, context: Map<String, Any>): Map<String, String> # 提取变量
│   │   ├── apply_formatting(template: String, variables: Map<String, String>): String # 应用格式化
│   │   └── sanitize_input(input: String): String             # 清理输入
│   │
│   └── AIClient (AI客户端)
│       ├── send_request(prompt: String, config: GenerationConfig): AIResponse # 发送请求
│       ├── parse_response(response: AIResponse): ParsedResponse # 解析响应
│       ├── handle_errors(error: AIError): ErrorHandlingResult # 处理错误
│       └── manage_rate_limits(): RateLimitStatus             # 管理频率限制
│
├── 上下文构建器
│   ├── ContextBuilder (上下文构建器)
│   │   ├── build_voyage_context(voyage_id: String): VoyageContext # 构建航行上下文
│   │   ├── build_team_context(team: List[Pokemon]): TeamInfo # 构建队伍上下文
│   │   ├── build_event_sequence(voyage_log: List[VoyageLogEntry]): List<EventRecord> # 构建事件序列
│   │   └── enrich_context(base_context: StoryInputData, enrichment_rules: List[EnrichmentRule]): StoryInputData # 丰富上下文
│   │
│   └── CharacterBuilder (角色构建器)
│       ├── build_pokemon_profile(pokemon: Pokemon): PokemonProfile # 构建宝可梦档案
│       ├── infer_personality(pokemon: Pokemon): List<String> # 推断性格特征
│       ├── generate_background_story(pokemon: Pokemon): String # 生成背景故事
│       └── update_relationships(team: List[Pokemon], events: List[EventRecord]): TeamDynamics # 更新关系
│
├── 故事后处理器
│   ├── StoryPostProcessor (故事后处理器)
│   │   ├── format_story(raw_story: String, format_spec: FormatSpec): String # 格式化故事
│   │   ├── add_illustrations(story: String, context: VoyageContext): IllustratedStory # 添加插图描述
│   │   ├── localize_story(story: String, target_locale: String): String # 本地化故事
│   │   └── personalize_story(story: String, player_preferences: PlayerPreferences): String # 个性化故事
│   │
│   └── QualityAssurance (质量保证)
│       ├── check_content_quality(story: String): QualityScore # 检查内容质量
│       ├── filter_inappropriate_content(story: String): String # 过滤不当内容
│       ├── ensure_consistency(story: String, lore: LoreDatabase): ConsistencyCheck # 确保一致性
│       └── apply_corrections(story: String, corrections: List[Correction]): String # 应用修正
│
└── 集成协调器
    ├── IntegrationCoordinator (集成协调器)
    │   ├── orchestrate_story_generation(voyage_id: String): GeneratedContent # 协调故事生成
    │   ├── trigger_email_generation(story_content: GeneratedContent): EmailContent # 触发邮件生成
    │   ├── handle_failure_scenarios(failure: GenerationFailure): RecoveryPlan # 处理失败场景
    │   └── update_player_journal(player: UUID, content: GeneratedContent) # 更新玩家日志
    │
    └── EventListener (事件监听器)
        ├── on_voyage_start(voyage_id: String)               # 航行开始时
        ├── on_voyage_milestone(voyage_id: String, milestone: Milestone) # 达到里程碑时
        ├── on_special_event(voyage_id: String, event: SpecialEvent) # 特殊事件发生时
        ├── on_voyage_end(voyage_id: String, result: VoyageResult) # 航行结束时
        └── on_emergency_situation(voyage_id: String, emergency: Emergency) # 紧急情况时
```

## **二、集成流程设计**

### **1. 完整故事生成流程**
```
1. 触发阶段
   ├── 航行状态变化触发故事生成
   │   ├── 航行开始 → 生成启航故事
   │   ├── 达到里程碑 → 生成里程碑故事
   │   ├── 遭遇特殊事件 → 生成事件故事
   │   └── 航行结束 → 生成结局故事
   │
   └── 条件检查
       ├── 检查是否满足生成条件
       ├── 检查冷却时间
       ├── 检查玩家设置(是否启用AI故事)
       └── 检查API可用性

2. 数据收集阶段
   ├── 收集航行数据
   │   ├── 从VoyageManager获取航行详情
   │   ├── 从ShipManager获取船只状态
   │   ├── 从PokemonManager获取队伍信息
   │   └── 从EventManager获取事件日志
   │
   ├── 丰富上下文数据
   │   ├── 添加海域背景描述
   │   ├── 添加天气影响描述
   │   ├── 添加宝可梦性格特征
   │   └── 添加历史互动记录
   │
   └── 构建故事输入
       ├── 选择合适的故事类型
       ├── 选择合适的写作风格
       ├── 设置情感基调
       └── 准备提示词变量

3. AI生成阶段
   ├── 调用提示词引擎
   │   ├── 加载对应模板
   │   ├── 填充模板变量
   │   ├── 添加风格指令
   │   └── 构建最终提示词
   │
   ├── 调用AI API
   │   ├── 发送生成请求
   │   ├── 处理流式响应
   │   ├── 监控生成质量
   │   └── 处理超时/错误
   │
   └── 解析AI响应
       ├── 提取故事内容
       ├── 提取纪念物描述
       ├── 验证格式正确性
       └── 保存原始响应

4. 后处理阶段
   ├── 故事格式化
   │   ├── 分段和标点修正
   │   ├── 添加标题和元数据
   │   ├── 突出关键情节
   │   └── 确保长度合适
   │
   ├── 质量检查
   │   ├── 内容安全检查
   │   ├── 逻辑一致性检查
   │   ├── 与世界观一致性检查
   │   └── 情感基调检查
   │
   └── 个性化处理
       ├── 插入玩家姓名
       ├── 引用玩家历史
       ├── 符合玩家偏好
       └── 添加个性化元素

5. 纪念物生成阶段
   ├── 分析故事内容
   │   ├── 识别关键时刻
   │   ├── 识别重要物品
   │   ├── 识别情感高潮
   │   └── 识别角色成长
   │
   ├── 生成纪念物描述
   │   ├── 根据规则生成纪念物
   │   ├── 确保与故事关联
   │   ├── 设置稀有度等级
   │   └── 生成物品属性
   │
   └── 创建纪念物物品
       ├── 生成唯一ID
       ├── 设置显示属性
       ├── 添加交互功能
       └── 关联到航行记录

6. 邮件集成阶段
   ├── 构建邮件内容
   │   ├── 生成邮件主题
   │   ├── 组装故事和纪念物
   │   ├── 添加航行统计
   │   └── 个性化问候和签名
   │
   ├── 调用邮件插件
   │   ├── 创建邮件对象
   │   ├── 添加附件(纪念物)
   │   ├── 设置发送时间
   │   └── 调用发送接口
   │
   └── 处理发送结果
       ├── 记录发送状态
       ├── 处理发送失败
       ├── 更新玩家通知
       └── 触发相关事件
```

### **2. 特殊场景处理**
```
1. 全军覆没场景
   ├── 条件检测
   │   ├── 船只耐久度归零
   │   ├── 所有宝可梦无法战斗
   │   └── 无幸存者
   │
   ├── 故事生成
   │   ├── 生成悲壮结局故事
   │   ├── 无纪念物生成
   │   ├── 生成最后日志/遗言
   │   └── 可能生成"漂流瓶"消息
   │
   └── 邮件处理
       ├── 发送沉船通知
       ├── 附带最后记录
       ├── 可能的救援线索
       └── 保险/赔偿信息

2. 紧急情况场景
   ├── 实时故事生成
   │   ├── 风暴中实时更新
   │   ├── 战斗中的紧张时刻
   │   ├── 危机决策时刻
   │   └── 生成"求救信号"
   │
   ├── 实时邮件通知
   │   ├── 紧急情况通报
   │   ├── 实时状态更新
   │   ├── 请求玩家决策
   │   └── 倒计时警告
   │
   └── 玩家干预
       ├── 远程发送指令
       ├── 使用特殊道具
       ├── 呼叫救援
       └── 改变航行计划

3. 多结局分支
   ├── 根据结果生成不同故事
   │   ├── 完美结局(无损失)
   │   ├── 成功但有代价
   │   ├── 险胜结局
   │   ├── 失败但幸存
   │   └── 悲剧结局
   │
   ├── 纪念物变化
   │   ├── 结局影响纪念物类型
   │   ├── 结局影响纪念物数量
   │   ├── 结局影响纪念物质量
   │   └── 隐藏结局解锁特殊纪念物
   │
   └── 邮件内容变化
       ├── 语气随结局变化
       ├── 附件内容不同
       ├── 后续建议不同
       └── 解锁新内容提示
```

## **三、提示词模板示例**

### **1. 基础提示词结构**
```yaml
系统提示词: |
  你是一个宝可梦世界中的航海故事讲述者。请根据以下航行经历，创作一段生动有趣的冒险故事。
  故事需要：
  1. 以{主角宝可梦}的视角叙述
  2. 体现队伍成员的性格特点
  3. 包含{海域名称}的环境描写
  4. 突出{关键事件}的紧张感
  5. 结尾要有教训或感悟
  6. 字数在300-500字之间
  
  格式要求：
  [故事标题]
  {空行}
  故事正文...
  {空行}
  纪念物:
  - "纪念物1名称: 描述"
  - "纪念物2名称: 描述"

用户提示词: |
  海域背景: {海域背景描述}
  航行队伍: {队伍成员详情}
  航行事件: {事件序列描述}
  当前状态: {航行状态信息}
  特别要求: {玩家个性化要求}
```

### **2. 具体模板示例**
```yaml
启航故事模板:
  系统提示词: |
    创作一段充满希望的启航故事，描述宝可梦们对这次航行的期待和准备。
    重点描写：
    - 船只出发时的场景
    - 每个宝可梦的角色和任务
    - 对未知海域的期待
    - 队伍之间的互动
  
  用户提示词: |
    船长宝可梦: {皮卡丘}，性格: {乐观勇敢}
    船员: {杰尼龟(导航员)、妙蛙种子(植物学家)、小火龙(瞭望员)}
    目标海域: {珊瑚秘境海域}，特点: {充满神秘珊瑚和友善海洋宝可梦}
    航行目标: {探索未知的珊瑚礁，寻找稀有贝壳}
    天气状况: {晴朗，微风}
    特殊装备: {新型声纳探测器、水下相机}
  
  输出格式: |
    《启航：向着珊瑚秘境》
    {故事内容}
    
    纪念物:
    - "启航合影: 全体成员在甲板上的第一张合影"
    - "船长日志·第一页: 皮卡丘写下的航行目标"

战斗故事模板:
  系统提示词: |
    创作一场紧张刺激的海上战斗故事，描述宝可梦们如何协作对抗敌人。
    包含：
    - 敌人的突然出现
    - 战斗策略的制定
    - 每个宝可梦的贡献
    - 关键时刻的转折
    - 战斗后的反思
  
  用户提示词: |
    遭遇敌人: {狂暴的巨牙鲨群}
    战斗背景: {在{风暴海域}遭遇突然袭击}
    我方队伍: {暴鲤龙(主力)、电击魔兽(控制)、水箭龟(防御)}
    战斗过程: {巨牙鲨从海底突袭，暴鲤龙正面迎击，电击魔兽用电网控制，水箭龟保护船只}
    战斗结果: {成功击退，但船只轻微受损}
    受伤情况: {电击魔兽轻伤}
  
  输出格式: |
    《风暴中的獠牙》
    {故事内容}
    
    纪念物:
    - "巨牙鲨的断牙: 战斗中击落的敌人牙齿"
    - "战斗伤痕: 电击魔兽手臂上的光荣伤痕照片"

结局故事模板:
  系统提示词: |
    根据航行结果创作一个合适的结局故事。
    成功结局: 重点描写成就感和成长
    失败结局: 重点描写教训和新的决心
    悲剧结局: 庄严而充满敬意
  
  用户提示词: |
    航行结果: {成功返回，发现稀有宝藏}
    航行成就: {探索了{深渊回廊}最深处的古代遗迹}
    队伍变化: {妙蛙种子进化成妙蛙花}
    获得物品: {古代宝可梦雕像、神秘石板}
    损失情况: {无人员损失，船只耐久度剩余30%}
    总航行时间: {7天6小时}
  
  输出格式: |
    《归航：深渊的馈赠》
    {故事内容}
    
    纪念物:
    - "古代雕像: 从遗迹中带回的神秘雕像"
    - "进化纪念照: 妙蛙种子进化瞬间的照片"
    - "船长日志·完结篇: 完整的航行记录"
```

## **四、数据表扩展设计**

### **1. 新数据表设计**
```sql
-- 故事生成记录表
CREATE TABLE story_generation_records (
    record_id VARCHAR(36) PRIMARY KEY,
    voyage_id VARCHAR(36) NOT NULL,
    story_type VARCHAR(50) NOT NULL,
    trigger_time TIMESTAMP NOT NULL,
    input_data JSON NOT NULL,           -- 原始输入数据
    raw_response TEXT,                  -- AI原始响应
    generated_story JSON NOT NULL,      -- 生成的故事数据
    mementos_generated JSON,           -- 生成的纪念物
    quality_score INT,                  -- 质量评分
    generation_cost DECIMAL(10,4),     -- API调用成本
    status VARCHAR(20) DEFAULT 'success',
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (voyage_id) REFERENCES voyage_data(voyage_id)
);

-- 纪念物收藏表
CREATE TABLE memento_collections (
    collection_id VARCHAR(36) PRIMARY KEY,
    player_uuid VARCHAR(36) NOT NULL,
    memento_id VARCHAR(36) NOT NULL,
    voyage_id VARCHAR(36) NOT NULL,
    memento_type VARCHAR(50) NOT NULL,
    name VARCHAR(100) NOT NULL,
    description TEXT NOT NULL,
    story_context TEXT,                 -- 关联的故事上下文
    rarity VARCHAR(20) NOT NULL,
    display_config JSON,                -- 展示配置
    interactive_actions JSON,          -- 交互动作配置
    acquired_date TIMESTAMP NOT NULL,
    display_order INT DEFAULT 0,
    is_favorite BOOLEAN DEFAULT FALSE,
    notes TEXT,                         -- 玩家备注
    metadata JSON,                      -- 元数据
    FOREIGN KEY (player_uuid) REFERENCES player_data(player_uuid),
    FOREIGN KEY (voyage_id) REFERENCES voyage_data(voyage_id)
);

-- AI生成配置表
CREATE TABLE ai_generation_configs (
    config_id VARCHAR(36) PRIMARY KEY,
    player_uuid VARCHAR(36) NOT NULL,
    story_preferences JSON,            -- 故事偏好设置
    content_filters JSON,              -- 内容过滤器
    generation_frequency VARCHAR(20),  -- 生成频率
    preferred_style VARCHAR(50),       -- 偏好风格
    auto_generate BOOLEAN DEFAULT TRUE, -- 是否自动生成
    email_notifications BOOLEAN DEFAULT TRUE, -- 邮件通知
    max_stories_per_day INT DEFAULT 5, -- 每日最大故事数
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (player_uuid) REFERENCES player_data(player_uuid)
);

-- 邮件发送记录表
CREATE TABLE email_delivery_records (
    email_id VARCHAR(36) PRIMARY KEY,
    player_uuid VARCHAR(36) NOT NULL,
    voyage_id VARCHAR(36) NOT NULL,
    story_record_id VARCHAR(36),       -- 关联的故事记录
    email_type VARCHAR(50) NOT NULL,
    subject VARCHAR(200) NOT NULL,
    content_hash VARCHAR(64) NOT NULL, -- 内容哈希(去重)
    sent_time TIMESTAMP NOT NULL,
    delivery_status VARCHAR(20) NOT NULL,
    open_count INT DEFAULT 0,
    last_opened TIMESTAMP,
    attachment_ids JSON,               -- 附件ID列表
    metadata JSON,
    FOREIGN KEY (player_uuid) REFERENCES player_data(player_uuid),
    FOREIGN KEY (voyage_id) REFERENCES voyage_data(voyage_id),
    FOREIGN KEY (story_record_id) REFERENCES story_generation_records(record_id)
);

-- 宝可梦角色档案表
CREATE TABLE pokemon_profiles (
    profile_id VARCHAR(36) PRIMARY KEY,
    pokemon_uuid VARCHAR(36) NOT NULL,  -- 宝可梦实例ID
    player_uuid VARCHAR(36) NOT NULL,
    personality_traits JSON,           -- 性格特征
    background_story TEXT,             -- AI生成的背景故事
    relationship_map JSON,             -- 与其他宝可梦的关系
    story_contributions INT DEFAULT 0, -- 在故事中的贡献次数
    preferred_roles JSON,              -- 偏好角色
    character_development JSON,        -- 角色发展记录
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (player_uuid) REFERENCES player_data(player_uuid)
);
```

### **2. 现有表扩展字段**
```sql
-- 在voyage_data表中添加故事相关字段
ALTER TABLE voyage_data ADD COLUMN story_generated BOOLEAN DEFAULT FALSE;
ALTER TABLE voyage_data ADD COLUMN story_record_ids JSON;  -- 关联的故事记录ID列表
ALTER TABLE voyage_data ADD COLUMN mementos_collected JSON; -- 收集的纪念物ID列表

-- 在player_data表中添加故事偏好字段
ALTER TABLE player_data ADD COLUMN story_settings JSON;
ALTER TABLE player_data ADD COLUMN email_preferences JSON;
ALTER TABLE player_data ADD COLUMN collected_memento_count INT DEFAULT 0;

-- 在area_data表中添加背景描述字段
ALTER TABLE area_data ADD COLUMN background_description TEXT;
ALTER TABLE area_data ADD COLUMN lore_entries JSON;  -- 海域传说/背景故事
ALTER TABLE area_data ADD COLUMN story_triggers JSON; -- 故事触发点配置
```

## **五、API接口设计**

### **1. 故事生成API**
```kotlin
interface StoryGenerationAPI {
    // 生成航行故事
    @POST("/api/v1/story/generate")
    suspend fun generateVoyageStory(
        @Body request: GenerateStoryRequest
    ): GenerateStoryResponse
    
    // 手动触发故事生成
    @POST("/api/v1/story/trigger/{voyageId}")
    suspend fun triggerStoryGeneration(
        @Path("voyageId") voyageId: String,
        @Body trigger: StoryTrigger
    ): TriggerResponse
    
    // 获取生成的故事
    @GET("/api/v1/story/{recordId}")
    suspend fun getGeneratedStory(
        @Path("recordId") recordId: String
    ): StoryDetailResponse
    
    // 重新生成故事（不满意时）
    @POST("/api/v1/story/{recordId}/regenerate")
    suspend fun regenerateStory(
        @Path("recordId") recordId: String,
        @Body options: RegenerateOptions
    ): RegenerateResponse
    
    // 获取玩家的故事历史
    @GET("/api/v1/player/{playerId}/stories")
    suspend fun getPlayerStories(
        @Path("playerId") playerId: String,
        @Query("page") page: Int = 0,
        @Query("size") size: Int = 20
    ): PaginatedStoryResponse
}

data class GenerateStoryRequest(
    val voyageId: String,
    val storyType: StoryType,
    val stylePreferences: StylePreferences? = null,
    val additionalContext: Map<String, Any> = emptyMap(),
    val forceGenerate: Boolean = false
)

data class GenerateStoryResponse(
    val success: Boolean,
    val storyRecord: StoryRecord? = null,
    val mementos: List<MementoItem> = emptyList(),
    val emailSent: Boolean = false,
    val warnings: List<String> = emptyList()
)
```

### **2. 纪念物管理API**
```kotlin
interface MementoAPI {
    // 获取玩家的纪念物收藏
    @GET("/api/v1/mementos/collection")
    suspend fun getMementoCollection(
        @Query("playerId") playerId: String,
        @Query("filter") filter: MementoFilter? = null
    ): MementoCollectionResponse
    
    // 查看特定纪念物详情
    @GET("/api/v1/mementos/{mementoId}")
    suspend fun getMementoDetails(
        @Path("mementoId") mementoId: String
    ): MementoDetailResponse
    
    // 更新纪念物展示设置
    @PUT("/api/v1/mementos/{mementoId}/display")
    suspend fun updateDisplaySettings(
        @Path("mementoId") mementoId: String,
        @Body settings: DisplaySettings
    ): UpdateResponse
    
    // 分享纪念物
    @POST("/api/v1/mementos/{mementoId}/share")
    suspend fun shareMemento(
        @Path("mementoId") mementoId: String,
        @Body shareRequest: ShareRequest
    ): ShareResponse
    
    // 导出纪念物相册
    @GET("/api/v1/mementos/export")
    suspend fun exportCollection(
        @Query("playerId") playerId: String,
        @Query("format") format: ExportFormat = ExportFormat.PDF
    ): ExportResponse
}
```

### **3. 邮件集成API**
```kotlin
interface EmailIntegrationAPI {
    // 发送航行报告邮件
    @POST("/api/v1/email/send/voyage-report")
    suspend fun sendVoyageReport(
        @Body request: VoyageReportRequest
    ): EmailSendResponse
    
    // 获取玩家的未读邮件
    @GET("/api/v1/email/inbox")
    suspend fun getInbox(
        @Query("playerId") playerId: String,
        @Query("unreadOnly") unreadOnly: Boolean = true
    ): InboxResponse
    
    // 标记邮件为已读
    @PUT("/api/v1/email/{emailId}/read")
    suspend fun markAsRead(
        @Path("emailId") emailId: String
    ): MarkReadResponse
    
    // 领取邮件附件
    @POST("/api/v1/email/{emailId}/claim-attachment")
    suspend fun claimAttachment(
        @Path("emailId") emailId: String,
        @Query("attachmentId") attachmentId: String
    ): ClaimResponse
    
    // 配置邮件偏好
    @PUT("/api/v1/email/preferences")
    suspend fun updateEmailPreferences(
        @Body preferences: EmailPreferences
    ): PreferencesResponse
}
```

## **六、配置管理**

### **1. 配置文件结构**
```yaml
# story-generation-config.yml
story-generation:
  enabled: true
  ai-provider: "deepseek"
  
  deepseek-config:
    api-key: "${DEEPSEEK_API_KEY}"
    endpoint: "https://api.deepseek.com/v1/chat/completions"
    model: "deepseek-chat"
    temperature: 0.7
    max-tokens: 2000
    timeout-seconds: 30
  
  generation-policies:
    max-stories-per-day: 10
    max-tokens-per-month: 100000
    cost-limit-per-month: 50.0
    quality-threshold: 0.7
  
  content-filters:
    enabled: true
    blocked-topics:
      - "violence"
      - "political"
      - "religious"
    language-filter-level: "moderate"
  
  email-integration:
    enabled: true
    plugin-name: "MailBox"
    default-template: "voyage-report"
    attachment-limit: 5
    auto-send: true
  
  memento-generation:
    enabled: true
    max-per-story: 3
    rarity-distribution:
      common: 0.60
      uncommon: 0.25
      rare: 0.10
      epic: 0.04
      legendary: 0.01
  
  player-preferences:
    default-style: "adventurous"
    allow-personalization: true
    include-statistics: true
    show-hints: false
```

### **2. 模板配置文件**
```yaml
# prompt-templates.yml
templates:
  departure-story:
    system-prompt: |
      你是一个宝可梦航海故事的讲述者。请创作一段启航故事，描述宝可梦们开始新冒险的兴奋和期待。
      要求：
      1. 以{船长宝可梦}的视角叙述
      2. 包含每个船员的特写
      3. 描写海域的初始印象
      4. 字数在200-300字
      5. 结尾表达对未来的期待
      
      格式：
      《{自定义标题}》
      {故事内容}
      
      纪念物:
      - "{纪念物1名称}: {描述}"
    variables:
      - "船长宝可梦"
      - "船员列表"
      - "目标海域"
      - "航行目标"
      - "天气状况"
  
  battle-story:
    system-prompt: |
      创作一场紧张刺激的海上战斗故事。包含：
      1. 敌人的突然出现
      2. 战斗策略和配合
      3. 关键时刻的转折
      4. 战斗结果和影响
      5. 字数在300-400字
      
      格式：
      《{战斗名称}》
      {故事内容}
      
      纪念物:
      - "{纪念物1名称}: {描述}"
      - "{纪念物2名称}: {描述}"
    variables:
      - "敌人类型"
      - "战斗地点"
      - "我方队伍"
      - "战斗过程"
      - "战斗结果"
      - "受伤情况"
  
  success-return-story:
    system-prompt: |
      创作一段成功的归航故事，重点描写：
      1. 归航时的喜悦
      2. 航行中的收获和成长
      3. 队伍成员的变化
      4. 对未来的展望
      5. 字数在250-350字
      
      格式：
      《归航：{副标题}》
      {故事内容}
      
      纪念物:
      - "{纪念物1名称}: {描述}"
      - "{纪念物2名称}: {描述}"
      - "{纪念物3名称}: {描述}"
    variables:
      - "航行成就"
      - "获得物品"
      - "队伍变化"
      - "总航行时间"
      - "损失情况"
```

## **七、错误处理与监控**

### **1. 错误处理策略**
```kotlin
enum class StoryGenerationError {
    // API错误
    API_UNAVAILABLE,           // AI服务不可用
    RATE_LIMIT_EXCEEDED,       // 频率限制
    INVALID_API_KEY,           // API密钥无效
    NETWORK_ERROR,             // 网络错误
    
    // 内容错误
    CONTENT_FILTERED,          // 内容被过滤
    INVALID_FORMAT,            // 格式不符合要求
    QUALITY_TOO_LOW,           // 质量评分过低
    INCONSISTENT_WITH_LORE,    // 与世界观不一致
    
    // 系统错误
    INSUFFICIENT_DATA,         // 数据不足
    TEMPLATE_NOT_FOUND,        // 模板不存在
    CONTEXT_BUILD_FAILED,      // 上下文构建失败
    EMAIL_SEND_FAILED,         // 邮件发送失败
    
    // 玩家限制
    DAILY_LIMIT_EXCEEDED,      // 达到每日限制
    COST_LIMIT_EXCEEDED,       // 达到成本限制
    PLAYER_OPTED_OUT,          // 玩家已禁用
}

class StoryGenerationException(
    val errorType: StoryGenerationError,
    val details: String,
    val recoverable: Boolean = true,
    val retryAfter: Duration? = null
) : RuntimeException("Story generation failed: $errorType - $details")

// 错误处理策略
object ErrorHandler {
    fun handleError(error: StoryGenerationError, context: ErrorContext): RecoveryAction {
        return when (error) {
            StoryGenerationError.API_UNAVAILABLE -> {
                // 降级到备用生成器
                RecoveryAction.UseFallbackGenerator(
                    fallbackType = FallbackType.TEMPLATE_BASED,
                    retryLater = true
                )
            }
            StoryGenerationError.RATE_LIMIT_EXCEEDED -> {
                // 延迟重试
                RecoveryAction.DelayAndRetry(
                    delay = Duration.ofMinutes(5),
                    maxRetries = 3
                )
            }
            StoryGenerationError.CONTENT_FILTERED -> {
                // 使用更严格的提示词重试
                RecoveryAction.RetryWithStrictPrompt(
                    additionalFilters = listOf("safety_filter_high"),
                    maxAttempts = 2
                )
            }
            StoryGenerationError.INSUFFICIENT_DATA -> {
                // 生成简化版故事
                RecoveryAction.GenerateSimpleVersion(
                    template = "minimal-story",
                    notifyPlayer = true
                )
            }
            else -> {
                // 记录错误并通知管理员
                RecoveryAction.LogAndNotify(
                    severity = if (error.recoverable) "WARN" else "ERROR",
                    notifyAdmin = true
                )
            }
        }
    }
}
```

### **2. 监控指标**
```kotlin
data class StoryGenerationMetrics {
    // 性能指标
    val apiLatency: Histogram              // API响应时间
    val generationTime: Histogram          // 总生成时间
    val tokensUsed: Counter                // 使用的token数
    val costIncurred: Gauge<Double>       // 产生的成本
    
    // 质量指标
    val qualityScores: Histogram           // 质量评分分布
    val playerRatings: Histogram           // 玩家评分
    val contentFilterRate: Meter           // 内容过滤率
    
    // 使用指标
    val storiesGenerated: Meter            // 生成的故事数
    val mementosGenerated: Meter           // 生成的纪念物数
    val emailsSent: Meter                  // 发送的邮件数
    
    // 错误指标
    val errorRates: Map<StoryGenerationError, Meter>  // 各类错误率
    val retryCounts: Histogram             // 重试次数分布
    val fallbackUsage: Meter               # 备用生成器使用次数
}
```

这个完整的故事生成系统架构深度集成了AI故事生成、纪念物系统和邮件插件，能够实现策划案中描述的所有功能：

1. **智能故事生成**：根据航行数据、队伍信息和事件序列生成个性化的冒险故事
2. **动态纪念物系统**：根据故事内容生成有意义的纪念物品
3. **无缝邮件集成**：通过邮件插件将故事和纪念物发送给玩家
4. **多类型故事支持**：支持启航、事件、战斗、结局等多种故事类型
5. **情感与角色发展**：跟踪宝可梦的性格发展和队伍关系变化
6. **质量控制与过滤**：确保生成内容的质量和安全性
7. **个性化体验**：根据玩家偏好定制故事风格和内容
8. **失败场景处理**：正确处理沉船、全军覆没等特殊情况

系统设计为高度可配置和可扩展，可以方便地添加新的故事类型、纪念物类型和集成不同的AI提供商。
