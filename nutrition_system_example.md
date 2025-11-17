# 营养系统和结算界面设计示例

## 🎯 营养计算系统逻辑

### 核心计算公式

```gdscript
# 营养成分累加
class NutritionCalculator:
    var total_calories: int = 0
    var total_protein: float = 0.0
    var total_carbs: float = 0.0
    var total_fat: float = 0.0
    var special_effects: Array[String] = []
    var ingredient_categories: Array[String] = []

    func add_ingredient(ingredient: Ingredient):
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

    func calculate_health_rating() -> String:
        # 基于总卡路里和蛋白质含量计算健康等级
        if total_calories >= 200 and total_calories <= 500 and total_protein >= 15:
            return "S"
        elif total_calories >= 300 and total_calories <= 600 and total_protein >= 10:
            return "A"
        elif total_calories >= 400 and total_calories <= 700 and total_protein >= 8:
            return "B"
        elif total_calories >= 500 and total_calories <= 800 and total_protein >= 5:
            return "C"
        else:
            return "D"

    func calculate_balance_score() -> float:
        # 计算营养成分平衡度
        protein_ratio = total_protein / max(1, total_protein + total_carbs + total_fat) * 100
        balance_score = abs(25 - protein_ratio) # 蛋白质占比越接近25%越好
        return max(0, 100 - balance_score)

    func calculate_final_score(time_taken: float) -> int:
        # 基础分数
        var base_score = int(total_protein * 20) + ingredient_categories.size() * 50

        # 时间奖励
        if time_taken <= 15:
            base_score = int(base_score * 2.0)
        elif time_taken <= 18:
            base_score = int(base_score * 1.5)

        # 健康评级加成
        var rating = calculate_health_rating()
        match rating:
            "S": base_score += 1000
            "A": base_score += 500
            "B": base_score += 200
            "C": base_score += 100
            "D": base_score += 0

        return base_score
```

## 🎨 结算界面设计

### 基础结算界面
```
┌─────────────────────────────────────────────────────────┐
│                   🥪 制作完成！🥪                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ⏱️ 用时：18.5秒      🏆 总分：2850                    │
│                                                         │
│  📊 营养成分分析                                        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   │
│  总卡路里：     485 kcal    ████████▒▒ 60%            │
│  蛋白质：       28g  ⭐⭐⭐⭐ ██████▒▒▒▒ 50%          │
│  碳水化合物：   45g  ⭐⭐⭐   ████████▒▒ 70%            │
│  脂肪：         22g  ⭐⭐⭐⭐ █████▒▒▒▒▒ 40%           │
│                                                         │
│  🌟 健康评级：A级 美味佳肴                              │
│  💡 特殊效果：                                           │
│     • 浓郁 +3   • 健康 +2   • 能量 +1                   │
│                                                         │
│  🏅 成就解锁：                                           │
│     ✅ 速度达人 - 20秒内完成                           │
│     ✅ 营养专家 - 蛋白质超过25g                         │
│     ✅ 创意大师 - 使用了5种不同类别食材                 │
│                                                         │
│           [🔄 重新开始]    [📤 分享成绩]                │
└─────────────────────────────────────────────────────────┘
```

### 详细营养分析界面
```
┌─────────────────────────────────────────────────────────┐
│                   📋 详细营养报告 📋                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🥪 你的三明治配方：                                    │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   │
│  上层：全麦面包                                          │
│  酱料：蛋黄酱 + 蜜芥酱                                  │
│  食材：烤鸡肉 + 生菜 + 番茄 + 牛油果                     │
│  奶酪：莫扎里拉                                          │
│  下层：全麦面包                                          │
│                                                         │
│  📊 各类营养占比（参考每日推荐值）                      │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   │
│  卡路里：     485/2000 kcal  ██████▒▒▒▒▒▒▒▒ 24%      │
│  蛋白质：     28g/50g       █████████▒▒▒▒ 56%        │
│  碳水化合物： 45g/300g      ██▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ 15%    │
│  脂肪：       22g/65g       █████▒▒▒▒▒▒▒▒▒▒▒ 34%      │
│                                                         │
│  🎯 营养建议：                                           │
│     ✅ 蛋白质含量优秀，有助于肌肉恢复                    │
│     ✅ 健康脂肪来源：牛油果提供优质脂肪                  │
│     ✅ 复合碳水化合物：全麦面包提供持续能量              │
│     ⚠️ 可以考虑增加更多蔬菜以补充维生素                  │
│                                                         │
│           [📊 统计数据]    [🏠 返回主页]                │
└─────────────────────────────────────────────────────────┘
```

## 🏆 成就系统设计

### 成就展示效果
```gdscript
# 成就解锁动画
func show_achievement(achievement_name: String, description: String):
    var achievement_popup = preload("res://scenes/AchievementPopup.tscn").instantiate()
    achievement_popup.setup(achievement_name, description)
    add_child(achievement_popup)

    # 播放解锁音效
    $SoundPlayer.play("achievement_unlock")

# 成就进度追踪
class AchievementManager:
    var achievements: Dictionary = {}

    func check_achievements(game_data: Dictionary):
        # 检查各种成就条件
        if game_data.time_taken <= 20 and game_data.completed:
            unlock_achievement("speed_master", "速度达人", "20秒内完成三明治")

        if game_data.total_protein >= 25:
            unlock_achievement("nutrition_expert", "营养专家", "蛋白质超过25g")

        if game_data.category_count >= 5:
            unlock_achievement("creative_master", "创意大师", "使用了5种不同类别食材")

        if game_data.nutrition_rating == "S":
            unlock_achievement("health_star", "健康明星", "获得S级营养评价")
```

### 成就分类和奖励
| 成就类型 | 成就名称 | 解锁条件 | 分数奖励 | 难度 |
|---------|---------|---------|---------|------|
| 速度 | 闪电手 | 15秒内完成 | 800分 | ⭐⭐⭐ |
| 速度 | 速度达人 | 20秒内完成 | 500分 | ⭐⭐ |
| 营养 | 蛋白质专家 | 蛋白质30g+ | 600分 | ⭐⭐ |
| 营养 | 健康明星 | S级评价 | 1000分 | ⭐⭐⭐ |
| 创意 | 食材收集家 | 使用所有类别食材 | 700分 | ⭐⭐⭐ |
| 创意 | 奇奇怪怪 | 使用3种特殊食材 | 400分 | ⭐⭐ |

## 📈 数据追踪和分析

### 玩家统计数据
```json
{
  "player_stats": {
    "total_games_played": 15,
    "best_time": 16.2,
    "best_score": 3250,
    "total_calories_consumed": 7280,
    "favorite_ingredients": {
      "chicken": 12,
      "avocado": 8,
      "bacon": 7
    },
    "achievements_unlocked": [
      "speed_master",
      "nutrition_expert",
      "creative_master"
    ],
    "nutrition_ratings_distribution": {
      "S": 3,
      "A": 7,
      "B": 4,
      "C": 1,
      "D": 0
    }
  }
}
```

### 游戏平衡性数据
```json
{
  "game_balance": {
    "average_completion_time": 18.5,
    "completion_rate": 0.85,
    "most_used_ingredients": [
      {"id": "white_bread", "usage_rate": 0.92},
      {"id": "ham", "usage_rate": 0.78},
      {"id": "lettuce", "usage_rate": 0.65}
    ],
    "least_used_ingredients": [
      {"id": "blue_cheese", "usage_rate": 0.08},
      {"id": "fermented_tofu", "usage_rate": 0.12},
      {"id": "chocolate_sauce", "usage_rate": 0.15}
    ]
  }
}
```

## 🎯 个性化推荐系统

### 基于游戏历史的建议
```gdscript
class PersonalizedRecommender:
    func analyze_player_habits(stats: Dictionary) -> Array[String]:
        var recommendations: Array[String] = []

        # 分析营养偏好
        if stats.average_protein < 10:
            recommendations.append("尝试添加更多高蛋白食材如金枪鱼或烤牛肉")

        if stats.average_time > 19:
            recommendations.append("试试选择更容易处理的食材来提高速度")

        # 分析食材多样性
        if stats.ingredient_variety < 4:
            recommendations.append("尝试新的食材组合，发现新口味！")

        return recommendations
```

## 🔤 多语言支持

### 营养信息本地化
```json
{
  "localization": {
    "zh": {
      "calories": "卡路里",
      "protein": "蛋白质",
      "carbs": "碳水化合物",
      "fat": "脂肪",
      "health_rating_s": "健康之星",
      "health_rating_a": "美味佳肴"
    },
    "en": {
      "calories": "Calories",
      "protein": "Protein",
      "carbs": "Carbohydrates",
      "fat": "Fat",
      "health_rating_s": "Health Star",
      "health_rating_a": "Delicious Meal"
    }
  }
}
```

## 📱 移动端适配

### 触摸优化设计
```css
/* 移动端营养显示样式 */
.nutrition-mobile {
  font-size: 18px;
  padding: 20px;
  touch-action: manipulation;
}

.nutrition-bar {
  height: 30px;
  margin: 10px 0;
  border-radius: 15px;
}

.achievement-mobile {
  animation: bounce 0.5s ease-out;
  transform: scale(1.1);
}
```

## 🎨 视觉效果增强

### 营养成分动画
```gdscript
# 营养条形图动画
func animate_nutrition_bars():
    var tween = create_tween()

    # 卡路里条形图
    tween.parallel().tween_property($CalorieBar, "value", 0.6, 1.0)

    # 蛋白质条形图
    tween.parallel().tween_property($ProteinBar, "value", 0.5, 1.2)

    # 碳水化合物条形图
    tween.parallel().tween_property($CarbsBar, "value", 0.7, 0.8)

    # 脂肪条形图
    tween.parallel().tween_property($FatBar, "value", 0.4, 1.0)

    # 数字增长动画
    tween.parallel().tween_method(_update_score_display, 0, final_score, 2.0)
```

这个营养系统设计提供了完整的计算逻辑、界面展示、成就追踪和个性化推荐，让20秒三明治游戏不仅有趣，还具有教育价值和长期的游戏动力。