# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Godot 4.5** game project titled "sandwich_20s" (三明治大师). The project is a mobile-friendly sandwich making game with comprehensive ingredient system and nutrition tracking.

## Game Concept

A sandwich creation game where players can:
- Choose from various bread types, vegetables, meats, and condiments
- Build custom sandwiches with detailed nutritional information
- Track calories, protein, carbs, and fat content
- Experience different taste effects and bonuses

## Common Development Commands

Since this is a Godot project, development is primarily done through the Godot Editor:

- **Open Project**: Launch Godot Editor and open `project.godot`
- **Run Project**: Press F5 in Godot Editor or use "Play" button
- **Run Specific Scene**: Press F6 in Godot Editor with a scene selected
- **Export Project**: Use Godot's Export dialog (Project -> Export)

## Project Structure

### Core Files
- `project.godot` - Main Godot project configuration file
- `.godot/` - Godot's metadata directory (editor settings, import cache, etc.)
- `icon.svg` - Project icon file with imported texture
- `.editorconfig` - Basic editor configuration (UTF-8 encoding)
- `.gitignore` - Git ignore rules specific to Godot projects

### Game Data
- `ingredients_database.json` - Comprehensive ingredient database with nutritional information
  - **Categories**: Bread, Vegetables, Meats, Condiments, Cheeses
  - **Nutrition Data**: Calories, protein, carbohydrates, fat
  - **Effects**: Game bonuses (健康+1, 口感+1, 饱腹感+2, etc.)
  - **Localization**: Chinese names with English equivalents

### Documentation
- `sandwich_game_design.md` - Core game design document
- `technical_implementation_guide.md` - Technical implementation details
- `nutrition_system_example.md` - Nutrition system examples and formulas
- `CLAUDE.md` - This development guide

## Godot Configuration

- **Godot Version**: 4.5.1 (stable)
- **Target Platforms**: Windows, Mobile (Android/iOS configured)
- **Rendering Method**: Mobile optimization enabled
- **Features**: PackedStringArray("4.5", "Mobile")

## Game Data Structure

### Ingredient Categories
1. **面包类 (Bread)**: White, whole wheat, sourdough, rye bread
2. **蔬菜类 (Vegetables)**: Lettuce, tomato, onion, cucumber, etc.
3. **肉类 (Meats)**: Ham, chicken breast, beef, bacon, tuna
4. **调味品类 (Condiments)**: Mayonnaise, mustard, ketchup, etc.
5. **奶酪类 (Cheeses)**: Cheddar, mozzarella, swiss, etc.

### Nutrition System
- **Base Formula**: Base calories + (Ingredient calories × quantity)
- **Quality Multipliers**: Freshness bonus, combination effects
- **Special Effects**: Health, taste, satiety bonuses

## Development Status

### Current State
- ✅ Project configuration completed
- ✅ Comprehensive ingredient database created (50+ ingredients)
- ✅ Game design documentation finalized
- ✅ Technical architecture planned
- ❌ Game scenes not yet created
- ❌ Scripts not yet implemented
- ❌ Visual assets not yet imported

### Next Development Steps
1. Create main game scene structure
2. Implement ingredient selection system
3. Develop nutrition calculation logic
4. Design UI/UX for mobile interface
5. Add visual assets and animations

## File Organization Guidelines

### Scene Structure (To Be Created)
- `scenes/main/` - Main game scenes
- `scenes/ui/` - User interface components
- `scenes/game/` - Gameplay scenes

### Script Organization (To Be Created)
- `scripts/` - Main game logic scripts
- `scripts/ui/` - UI controller scripts
- `scripts/data/` - Data management scripts

### Asset Structure (To Be Created)
- `assets/textures/` - Ingredient and UI textures
- `assets/sounds/` - Game audio
- `assets/fonts/` - Localized fonts for Chinese/English

## Development Notes

- The project uses Godot's node hierarchy and scene organization best practices
- All ingredients support Chinese localization with English fallbacks
- Mobile optimization is enabled for touch interface development
- Nutrition system supports complex calculations with multiple modifiers
- When creating new content, follow the established ingredient data structure format
- 如果涉及节点操作，请你告诉我如何做，不要直接修改。