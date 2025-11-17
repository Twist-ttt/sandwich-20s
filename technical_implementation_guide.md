# 技术实现指南 - 20秒三明治游戏

## 🏗️ Godot 4.5 项目结构

```
sandwich-20s/
├── project.godot              # 项目配置文件
├── README.md                  # 项目说明
├── sandwich_game_design.md    # 游戏设计文档
├── ingredients_database.json  # 食材数据库
├── nutrition_system_example.md # 营养系统设计
├── technical_implementation_guide.md # 本技术文档

├── scenes/                    # 场景文件
│   ├── main_game.tscn        # 主游戏场景
│   ├── fridge.tscn           # 冰箱场景
│   ├── assembly_table.tscn   # 组装台场景
│   ├── packaging_station.tscn # 打包站场景
│   ├── nutrition_result.tscn # 营养结算界面
│   └── ui/                   # UI场景
│       ├── main_menu.tscn    # 主菜单
│       ├── countdown_timer.tscn # 倒计时器
│       └── achievement_popup.tscn # 成就弹窗

├── scripts/                   # 脚本文件
│   ├── game_manager.gd       # 游戏管理器
│   ├── nutrition_calculator.gd # 营养计算器
│   ├── ingredient.gd         # 食材脚本
│   ├── countdown_timer.gd    # 倒计时器
│   ├── achievement_manager.gd # 成就管理器
│   └── ui/                   # UI脚本
│       ├── main_menu.gd      # 主菜单逻辑
│       ├── nutrition_display.gd # 营养显示
│       └── draggable_item.gd # 可拖拽物品

├── assets/                    # 资源文件
│   ├── textures/             # 纹理图片
│   │   ├── bread/           # 面包类纹理
│   │   ├── meat/            # 肉类纹理
│   │   ├── vegetables/      # 蔬菜类纹理
│   │   ├── cheese/          # 奶酪类纹理
│   │   ├── sauces/          # 酱料类纹理
│   │   ├── special/         # 特殊食材纹理
│   │   ├── ui/              # UI纹理
│   │   └── backgrounds/     # 背景图片
│   ├── sounds/               # 音频文件
│   │   ├── sfx/             # 音效
│   │   │   ├── select.wav   # 选择音效
│   │   │   ├── assemble.wav # 组装音效
│   │   │   ├── pack.wav     # 打包音效
│   │   │   └── achievement.wav # 成就音效
│   │   ├── music/           # 背景音乐
│   │   │   ├── kitchen_theme.ogg
│   │   │   └── countdown_urgent.ogg
│   │   └── voice/           # 语音（可选）
│   └── fonts/               # 字体文件
│       └── game_font.ttf    # 游戏字体

├── resources/               # 资源配置
│   ├── ingredient_data.tres # 食材数据资源
│   ├── achievement_data.tres # 成就数据资源
│   └── game_config.tres    # 游戏配置

└── export_presets.cfg       # 导出配置
```

## 🎮 核心系统实现

### 1. 游戏管理器 (game_manager.gd)

```gdscript
extends Node

signal game_started()
signal game_completed(nutrition_data: Dictionary)
signal game_time_updated(remaining_time: float)
signal ingredient_selected(ingredient: Ingredient)
signal game_paused()
signal game_resumed()

enum GameState {
    MENU,
    PLAYING,
    PAUSED,
    COMPLETED,
    GAME_OVER
}

var current_state: GameState = GameState.MENU
var game_time: float = 20.0
var max_game_time: float = 20.0
var selected_ingredients: Array[Ingredient] = []
var is_game_active: bool = false
var nutrition_calculator: NutritionCalculator

@onready var countdown_timer: Timer = $CountdownTimer
@onready var fridge_node: Node = $GameWorld/Fridge
@onready var assembly_node: Node = $GameWorld/AssemblyTable
@onready var packaging_node: Node = $GameWorld/PackagingStation
@onready var ui_manager: Node = $UIManager

func _ready():
    nutrition_calculator = NutritionCalculator.new()
    add_child(nutrition_calculator)

    # 连接信号
    countdown_timer.timeout.connect(_on_game_time_up)
    fridge_node.ingredient_selected.connect(_on_ingredient_selected)
    assembly_node.assembly_completed.connect(_on_assembly_completed)

func start_game():
    current_state = GameState.PLAYING
    game_time = max_game_time
    selected_ingredients.clear()
    is_game_active = true

    countdown_timer.wait_time = 0.1  # 100ms更新一次
    countdown_timer.start()

    game_started.emit()
    ui_manager.show_game_ui()

func _process(delta):
    if is_game_active and current_state == GameState.PLAYING:
        game_time -= delta
        game_time_updated.emit(game_time)

        if game_time <= 0:
            _end_game()

func _on_ingredient_selected(ingredient: Ingredient):
    selected_ingredients.append(ingredient)
    nutrition_calculator.add_ingredient(ingredient)
    ingredient_selected.emit(ingredient)

func _on_assembly_completed():
    _end_game()

func _end_game():
    is_game_active = false
    countdown_timer.stop()

    var nutrition_data = nutrition_calculator.calculate_final_result()
    current_state = GameState.COMPLETED

    game_completed.emit(nutrition_data)
    ui_manager.show_nutrition_result(nutrition_data)

func pause_game():
    if current_state == GameState.PLAYING:
        current_state = GameState.PAUSED
        countdown_timer.paused = true
        is_game_active = false
        game_paused.emit()

func resume_game():
    if current_state == GameState.PAUSED:
        current_state = GameState.PLAYING
        countdown_timer.paused = false
        is_game_active = true
        game_resumed.emit()

func restart_game():
    # 重置游戏状态
    selected_ingredients.clear()
    nutrition_calculator.reset()
    fridge_node.reset_ingredients()
    assembly_node.reset_assembly()
    ui_manager.reset_ui()

    start_game()
```

### 2. 食材系统 (ingredient.gd)

```gdscript
extends Node2D
class_name Ingredient

signal ingredient_clicked(ingredient: Ingredient)

@export var ingredient_data: Dictionary

var id: String
var name: String
var name_en: String
var category: String
var calories: int
var protein: float
var carbs: float
var fat: float
var effects: Array[String]
var texture_path: String
var description: String

@onready var sprite: Sprite2D = $Sprite2D
@onready var collision_shape: CollisionShape2D = $Area2D/CollisionShape2D
@onready var area_2d: Area2D = $Area2D

func _ready():
    load_ingredient_data()
    setup_visuals()

func load_ingredient_data():
    if ingredient_data.has("id"):
        id = ingredient_data.id
        name = ingredient_data.name
        name_en = ingredient_data.name_en
        category = ingredient_data.category
        calories = ingredient_data.calories
        protein = ingredient_data.protein
        carbs = ingredient_data.carbs
        fat = ingredient_data.fat
        effects = ingredient_data.effects
        texture_path = ingredient_data.texture_path
        description = ingredient_data.description

func setup_visuals():
    # 加载纹理
    var texture = load(texture_path)
    if texture:
        sprite.texture = texture

    # 设置碰撞区域大小
    if sprite.texture:
        var texture_size = sprite.texture.get_size()
        collision_shape.shape.size = texture_size

func _on_area_2d_input_event(viewport: Viewport, event: InputEvent, shape_idx: int):
    if event is InputEventMouseButton and event.pressed and event.button_index == MOUSE_BUTTON_LEFT:
        ingredient_clicked.emit(self)

        # 播放选择音效
        $AudioStreamPlayer.play()

func get_nutrition_summary() -> String:
    return "%s: %d kcal, P%.1fg C%.1fg F%.1fg" % [name, calories, protein, carbs, fat]

func get_effects_string() -> String:
    if effects.is_empty():
        return ""
    return " • ".join(effects)
```

### 3. 营养计算器 (nutrition_calculator.gd)

```gdscript
extends Node
class_name NutritionCalculator

var total_calories: int = 0
var total_protein: float = 0.0
var total_carbs: float = 0.0
var total_fat: float = 0.0
var special_effects: Array[String] = []
var ingredient_categories: Array[String] = []
var ingredient_history: Array[Dictionary] = []

func reset():
    total_calories = 0
    total_protein = 0.0
    total_carbs = 0.0
    total_fat = 0.0
    special_effects.clear()
    ingredient_categories.clear()
    ingredient_history.clear()

func add_ingredient(ingredient: Ingredient):
    # 记录食材历史
    ingredient_history.append({
        "id": ingredient.id,
        "name": ingredient.name,
        "category": ingredient.category,
        "timestamp": Time.get_time_dict_from_system()
    })

    # 累加营养值
    total_calories += ingredient.calories
    total_protein += ingredient.protein
    total_carbs += ingredient.carbs
    total_fat += ingredient.fat

    # 收集特殊效果
    for effect in ingredient.effects:
        if not special_effects.has(effect):
            special_effects.append(effect)

    # 记录食材类别
    if not ingredient_categories.has(ingredient.category):
        ingredient_categories.append(ingredient.category)

func calculate_health_rating() -> Dictionary:
    var rating: String
    var rating_name: String
    var description: String
    var bonus_score: int

    # 基于卡路里和蛋白质计算评级
    if total_calories >= 200 and total_calories <= 500 and total_protein >= 15:
        rating = "S"
        rating_name = "健康之星"
        description = "营养均衡，卡路里适中"
        bonus_score = 3
    elif total_calories >= 300 and total_calories <= 600 and total_protein >= 10:
        rating = "A"
        rating_name = "美味佳肴"
        description = "口感丰富，营养较好"
        bonus_score = 2
    elif total_calories >= 400 and total_calories <= 700 and total_protein >= 8:
        rating = "B"
        rating_name = "标准餐食"
        description = "基本营养需求满足"
        bonus_score = 1
    elif total_calories >= 500 and total_calories <= 800 and total_protein >= 5:
        rating = "C"
        rating_name = "快餐级别"
        description = "高热量，营养单一"
        bonus_score = 0
    else:
        rating = "D"
        rating_name = "罪恶美食"
        description = "超高热量，需谨慎"
        bonus_score = -1

    return {
        "rating": rating,
        "name": rating_name,
        "description": description,
        "bonus_score": bonus_score
    }

func calculate_final_result(time_taken: float) -> Dictionary:
    # 计算健康评级
    var health_rating = calculate_health_rating()

    # 计算基础分数
    var base_score = 0
    base_score += int(total_protein * 20)  # 蛋白质分数
    base_score += ingredient_categories.size() * 50  # 食材多样性分数
    base_score += special_effects.size() * 30  # 特殊效果分数

    # 时间奖励
    var time_bonus = 1.0
    if time_taken <= 15:
        time_bonus = 2.0
    elif time_taken <= 18:
        time_bonus = 1.5
    elif time_taken <= 20:
        time_bonus = 1.0
    else:
        time_bonus = 0.5

    base_score = int(base_score * time_bonus)

    # 健康评级加成
    match health_rating.rating:
        "S": base_score += 1000
        "A": base_score += 500
        "B": base_score += 200
        "C": base_score += 100
        "D": base_score += 0

    return {
        "nutrition": {
            "calories": total_calories,
            "protein": total_protein,
            "carbs": total_carbs,
            "fat": total_fat,
            "special_effects": special_effects,
            "categories": ingredient_categories
        },
        "health_rating": health_rating,
        "final_score": base_score,
        "time_taken": time_taken,
        "ingredient_count": ingredient_history.size(),
        "ingredient_history": ingredient_history
    }
```

### 4. 可拖拽物品系统 (draggable_item.gd)

```gdscript
extends Node2D
class_name DraggableItem

signal drag_started(item: DraggableItem)
signal drag_ended(item: DraggableItem)
signal item_dropped(item: DraggableItem, target_node: Node)

var is_dragging: bool = false
var drag_offset: Vector2
var original_position: Vector2
var original_parent: Node
var snap_zones: Array[Node] = []

@onready var sprite: Sprite2D = $Sprite2D
@onready var area_2d: Area2D = $Area2D

func _ready():
    original_position = global_position
    original_parent = get_parent()

func _input(event):
    if not is_dragging:
        return

    if event is InputEventMouseMotion:
        global_position = event.position - drag_offset
    elif event is InputEventMouseButton and not event.pressed:
        if event.button_index == MOUSE_BUTTON_LEFT:
            stop_drag()

func start_drag():
    if is_dragging:
        return

    is_dragging = true
    drag_offset = get_global_mouse_position() - global_position
    original_position = global_position

    # 移动到最上层
    original_parent.remove_child(self)
    get_tree().current_scene.add_child(self)

    # 提高Z-index
    z_index = 100

    drag_started.emit(self)

func stop_drag():
    if not is_dragging:
        return

    is_dragging = false
    z_index = 0

    # 检查是否在有效的放置区域
    var nearest_snap_zone = find_nearest_snap_zone()
    if nearest_snap_zone:
        item_dropped.emit(self, nearest_snap_zone)
    else:
        # 返回原位
        return_to_original_position()

    # 移回原父节点
    get_tree().current_scene.remove_child(self)
    original_parent.add_child(self)

    drag_ended.emit(self)

func return_to_original_position():
    var tween = create_tween()
    tween.tween_property(self, "global_position", original_position, 0.2)

func find_nearest_snap_zone() -> Node:
    var nearest_zone: Node = null
    var nearest_distance: float = 100.0  # 最大吸附距离

    for zone in snap_zones:
        var distance = global_position.distance_to(zone.global_position)
        if distance < nearest_distance:
            nearest_distance = distance
            nearest_zone = zone

    return nearest_zone

func register_snap_zone(zone: Node):
    if not snap_zones.has(zone):
        snap_zones.append(zone)

func unregister_snap_zone(zone: Node):
    snap_zones.erase(zone)
```

### 5. 倒计时器 (countdown_timer.gd)

```gdscript
extends Control
class_name CountdownTimer

signal time_up()
signal warning_activated(seconds_left: int)

@onready var time_label: Label = $TimeLabel
@onready var warning_animation: AnimationPlayer = $WarningAnimation
@onready var warning_sound: AudioStreamPlayer = $WarningSound

var max_time: float = 20.0
var current_time: float = 20.0
var is_active: bool = false
var warning_times: Array[int] = [5, 3, 1]

func _ready():
    update_display()

func start_countdown(time_seconds: float):
    max_time = time_seconds
    current_time = time_seconds
    is_active = true
    update_display()

func update_countdown(delta: float):
    if not is_active:
        return

    current_time -= delta
    update_display()

    # 检查警告时间
    for warning_time in warning_times:
        if current_time <= warning_time and current_time + delta > warning_time:
            trigger_warning(warning_time)
            break

    if current_time <= 0:
        current_time = 0
        is_active = false
        time_up.emit()

func update_display():
    var display_time = max(0, current_time)
    var seconds = int(display_time)
    var milliseconds = int((display_time - seconds) * 100)

    time_label.text = "⏱️ %02d.%02d" % [seconds, milliseconds]

    # 根据剩余时间改变颜色
    if current_time <= 3:
        time_label.modulate = Color.RED
    elif current_time <= 5:
        time_label.modulate = Color.YELLOW
    else:
        time_label.modulate = Color.WHITE

func trigger_warning(seconds_left: int):
    warning_animation.play("warning_pulse")
    warning_sound.play()
    warning_activated.emit(seconds_left)

func stop_countdown():
    is_active = false
    time_label.modulate = Color.WHITE

func reset():
    current_time = max_time
    is_active = false
    update_display()
```

## 🎨 UI系统实现

### 1. 营养结果界面 (nutrition_result.tscn)

```gdscript
extends Control
class_name NutritionResult

@onready var score_label: Label = $ScoreLabel
@onready var time_label: Label = $TimeLabel
@onready var calories_bar: ProgressBar = $NutritionBars/CaloriesBar
@onready var protein_bar: ProgressBar = $NutritionBars/ProteinBar
@onready var carbs_bar: ProgressBar = $NutritionBars/CarbsBar
@onready var fat_bar: ProgressBar = $NutritionBars/FatBar
@onready var rating_label: Label = $RatingLabel
@onready var effects_container: HBoxContainer = $EffectsContainer
@onready var achievements_container: VBoxContainer = $AchievementsContainer
@onready var restart_button: Button = $RestartButton
@onready var share_button: Button = $ShareButton

func display_results(nutrition_data: Dictionary):
    var nutrition = nutrition_data.nutrition
    var health_rating = nutrition_data.health_rating

    # 显示基础信息
    score_label.text = "🏆 总分：%d" % nutrition_data.final_score
    time_label.text = "⏱️ 用时：%.1f秒" % nutrition_data.time_taken

    # 更新营养条形图
    update_nutrition_bars(nutrition)

    # 显示健康评级
    rating_label.text = "🌟 %s：%s" % [health_rating.rating, health_rating.name]
    rating_label.modulate = get_rating_color(health_rating.rating)

    # 显示特殊效果
    display_effects(nutrition.special_effects)

    # 显示成就
    display_achievements(nutrition_data)

    # 动画效果
    animate_result_display()

func update_nutrition_bars(nutrition: Dictionary):
    # 营养条形图动画
    var max_nutrition = 1000.0  # 最大显示值

    calories_bar.max_value = max_nutrition
    calories_bar.value = 0
    create_tween().tween_property(calories_bar, "value", nutrition.calories, 1.0)

    protein_bar.max_value = 100.0
    protein_bar.value = 0
    create_tween().tween_property(protein_bar, "value", nutrition.protein, 1.2)

    carbs_bar.max_value = 100.0
    carbs_bar.value = 0
    create_tween().tween_property(carbs_bar, "value", nutrition.carbs, 0.8)

    fat_bar.max_value = 100.0
    fat_bar.value = 0
    create_tween().tween_property(fat_bar, "value", nutrition.fat, 1.0)

func display_effects(effects: Array[String]):
    # 清除旧效果
    for child in effects_container.get_children():
        child.queue_free()

    # 显示新效果
    for effect in effects:
        var effect_label = Label.new()
        effect_label.text = "• %s" % effect
        effect_label.add_theme_color_override("font_color", Color.GOLD)
        effects_container.add_child(effect_label)

func get_rating_color(rating: String) -> Color:
    match rating:
        "S": return Color.GOLD
        "A": return Color.GREEN
        "B": return Color.BLUE
        "C": return Color.ORANGE
        "D": return Color.RED
        _: return Color.WHITE

func animate_result_display():
    # 界面出现动画
    modulate = Color.TRANSPARENT
    var tween = create_tween()
    tween.set_ease(Tween.EASE_OUT)
    tween.tween_property(self, "modulate", Color.WHITE, 0.5)

    # 分数增长动画
    var final_score = int(score_label.text.split("：")[1])
    animate_score_growth(final_score)

func animate_score_growth(target_score: int):
    var current_score = 0
    var score_tween = create_tween()

    score_tween.tween_method(
        func(score: int):
            score_label.text = "🏆 总分：%d" % score,
        current_score,
        target_score,
        2.0
    )
```

## 📱 移动端优化

### 1. 触摸输入处理

```gdscript
# 在draggable_item.gd中添加触摸支持
func _unhandled_input(event):
    if is_dragging and event is InputEventScreenTouch and not event.pressed:
        stop_drag()
```

### 2. 响应式UI

```gdscript
# 在UI管理器中
func adjust_for_mobile():
    if OS.has_feature("mobile"):
        # 增大按钮尺寸
        var mobile_scale = 1.5
        for button in get_tree().get_nodes_in_group("ui_buttons"):
            button.scale = Vector2(mobile_scale, mobile_scale)

        # 调整字体大小
        var font_size = 24
        for label in get_tree().get_nodes_in_group("ui_labels"):
            if label has "add_theme_font_size_override":
                label.add_theme_font_size_override("font_size", font_size)
```

## 🌐 HTML5 导出配置

### export_presets.cfg 示例

```ini
[preset.0]

name="HTML5"
platform="Web"
runnable=true
dedicated_server=false
custom_features=""
export_filter="all_resources"
include_filter=""
exclude_filter=""
export_path="builds/web/index.html"
encryption_include_filters=""
encryption_exclude_filters=""

[preset.0.options]

custom_template/debug=""
custom_template/release=""
variant/export_type="2"
variant/extensions_support=true
vram_texture_compression/for_desktop=true
vram_texture_compression/for_mobile=false
html/export_icon=true
html/custom_html_shell=""
html/head_include=""
html/canvas_resize_policy=2
html/stretch_mode=2
html/use_godot_js_wrapper=false
html/use_remote_inspector=false
html/export_font_names=false
html/dot_project_include=""
html/canvas_resize_policy=0
html/focus_canvas_on_start=true
html/experimental_virtual_keyboard=false
progressive_web_app/enabled=false
progressive_web_app/offline_page=""
progressive_web_app/display=1
progressive_web_app/orientation=0
progressive_web_app/icon_144x144=""
progressive_web_app/icon_180x180=""
progressive_web_app/icon_512x512=""
progressive_web_app/background_color=Color(0, 0, 0, 1)
```

## 🔧 性能优化策略

### 1. 资源管理

```gdscript
# 资源预加载器
class ResourcePreloader:
    static var loaded_textures: Dictionary = {}
    static var loaded_sounds: Dictionary = {}

    static func get_texture(path: String) -> Texture2D:
        if not loaded_textures.has(path):
            loaded_textures[path] = load(path)
        return loaded_textures[path]

    static func preload_ingredients():
        var json_data = JSON.parse_string(FileAccess.open("res://ingredients_database.json", FileAccess.READ).get_as_text())

        for category in json_data.ingredients.values():
            for ingredient in category:
                if ingredient.has("texture_path"):
                    get_texture(ingredient.texture_path)
```

### 2. 对象池

```gdscript
# 音效对象池
class AudioPool extends Node
var available_sounds: Array[AudioStreamPlayer] = []
var active_sounds: Array[AudioStreamPlayer] = []
var sound_scene: PackedScene

func _ready():
    sound_scene = preload("res://scenes/AudioStreamPlayer.tscn")
    _create_initial_pool(10)

func _create_initial_pool(size: int):
    for i in size:
        var sound_player = sound_scene.instantiate()
        add_child(sound_player)
        available_sounds.append(sound_player)

func play_sound(sound_stream: AudioStream):
    var player: AudioStreamPlayer

    if available_sounds.size() > 0:
        player = available_sounds.pop_back()
    else:
        player = sound_scene.instantiate()
        add_child(player)

    player.stream = sound_stream
    player.play()
    active_sounds.append(player)

    player.finished.connect(_on_sound_finished.bind(player))

func _on_sound_finished(player: AudioStreamPlayer):
    active_sounds.erase(player)
    available_sounds.append(player)
```

## 🧪 测试和调试

### 1. 调试工具

```gdscript
# 调试管理器
extends Node
class_name DebugManager

var debug_enabled: bool = false
var debug_overlay: Control

func _ready():
    if OS.is_debug_build():
        debug_enabled = true
        create_debug_overlay()

func create_debug_overlay():
    debug_overlay = preload("res://ui/DebugOverlay.tscn").instantiate()
    add_child(debug_overlay)

func toggle_debug():
    debug_enabled = !debug_enabled
    if debug_overlay:
        debug_overlay.visible = debug_enabled

func log_nutrition(nutrition_data: Dictionary):
    if debug_enabled:
        print("=== 营养数据 ===")
        print("卡路里: %d" % nutrition_data.nutrition.calories)
        print("蛋白质: %.1fg" % nutrition_data.nutrition.protein)
        print("评级: %s" % nutrition_data.health_rating.rating)
        print("==============")
```

这个技术实现指南提供了完整的Godot 4.5项目结构、核心系统实现、UI系统、移动端优化、HTML5导出配置以及性能优化策略，确保20秒三明治游戏能够在3周内高质量完成开发。