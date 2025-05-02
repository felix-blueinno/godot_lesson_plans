# 🕹️ L2 Game Prototype Documentation

## 📘 Introduction

This document is a **step-by-step guide** to help you recreate a Godot 4 game prototype.  
It follows a clear and consistent format to outline tasks, scene structure changes, and scripts using examples and YAML formatting.

---

## 🧱 Documentation Format Convention

We use **YAML format** to describe Godot node hierarchies and structure changes:

```yaml
NodeName:
    type: Node3D
    remarks: "Optional notes for clarity."
    properties:
        Group: "Optional group name"
        Position: Vector3(0, 0, 0)
    children:
        - 🟢NewNode:
            type: CharacterBody3D
        - ⭕️RemovedNode:
            type: CharacterBody3D
```

---

## 🌐 Stage 1: Environment Setup

**Tasks**

- Add a root node called `Level`
- Add `Camera3D`, `DirectionalLight3D`, and `CSGBox3D` as ground
- Add a `WorldEnvironment` for sky

**Scene Structure**

```yaml
🟢Level:
  type: Node3D
  groups: ["level"]
  children:
    - MainCamera:
        type: Camera3D
    - SunLight:
        type: DirectionalLight3D
    - Environment:
        type: WorldEnvironment
    - Floor:
        type: CSGBox3D
        properties:
          Size: (200, 1, 200)
    - CanvasLayer:
        type: CanvasLayer
        children:
          - Control:
              type: Control
              children:
                - LoseBlock (Control)
                - WinBlock (Control)
                - ProgressBar
                - ScoreLabel (Label)
```

---

## 🧍 Stage 2: Player Creation

**Tasks**

- Import model (e.g., `Mage.glb`)
- Create a new scene:
  - Root node `CharacterBody3D` named `Player`
  - Add `Mage` (Node3D) and drag model inside
  - Right click to the model node and select `Make Local`
  - This will add a bunch of new nodes and an `AnimationPlayer`
  - Add `CollisionShape3D` and `Timer`
- Modify imported animations on `AnimationPlayer` node:  
  ➔ Set **Loop** ON for "Idle", "Running", "Attack" animations inside AnimationPlayer.
  - Select the animation name and find the correct animation frm the drop down list
  - On the right side of the Animation Length box, select the loop arrows to make animation loop
- Add `ReachArea` (Area3D) for enemy detection.

**Scene Structure**

```yaml
🟢Player:
  type: CharacterBody3D
  groups: ["player"]
  signals:
    - animation_finished: _on_animation_player_animation_finished()
  children:
    - Mage:
        type: Node3D
        children:
          - AnimationPlayer
    - CollisionShape3D:
        type: CollisionShape3D
        properties:
          Shape: CapsuleShape3D
          Size: (0.5, 2.0)
          Position: (0, 1, 0)
    - ReachArea:
        type: Area3D
        children:
          - CollisionShape3D:
              type: CollisionShape3D
              shape: CylinderShape3D
    - Timer:
        type: Timer
```

---

## 🎮 Stage 3: Character Animation Management

### 3.1 Make Enemies Move Toward the Player

```gdscript
extends CharacterBody3D

var player: CharacterBody3D
var velocity: Vector3

const SPEED = 5.0

func _ready() -> void:
    player = get_tree().get_first_node_in_group("player")
    look_at(player.global_position)

func _physics_process(delta: float) -> void:
    if is_dead: return

    if not has_reached_player:
    #Makes the enemy go toward the player
        var direction = (player.global_position - global_position).normalized()
        velocity.x = direction.x * SPEED
        velocity.z = direction.z * SPEED
        move_and_slide()
```

---

### 3.2 Enemy Idle and Run Animation

```gdscript
@onready var anim: AnimationPlayer = $Barbarian/AnimationPlayer

func _ready() -> void:
    anim.play("Idle")
    await get_tree().create_timer(0.5).timeout
    anim.play("Running_B")
```

---

### 3.3 Detect Player in Reach Range & Switch Animations

```gdscript
func _on_reach_area_body_entered(body: Node3D) -> void:
    if body.is_in_group("enemy"):
        body.has_reached_player = true
        body.anim.play("1H_Melee_Attack_Chop")
```

**Switch Animation on Attack Hit**\
(Makes monster damage the player at the end of the animation)

```gdscript
func _on_animation_player_animation_finished(anim_name: StringName) -> void:
    if anim_name == "1H_Melee_Attack_Chop" and !is_dead:
        hit_player()
        anim.play("1H_Melee_Attack_Chop")
```

**Enemy Attack Code**

```gdscript
func hit_player() -> void:
    player.change_hp(attack_power)
```

---

### 3.4 Player Shooting Animation

```gdscript
func _on_animation_player_animation_finished(anim_name: StringName) -> void:
    if anim_name == "1H_Ranged_Shoot":
        if monsters[0].hp <= 0:
            monsters.pop_at(0)

        if not monsters.is_empty():
            var bullet = bullet_scene.instantiate()
            add_child(bullet)
            bullet.global_position = global_position + Vector3.UP
            $Mage.look_at(monsters[0].global_position)
            anim.play("1H_Ranged_Shoot")
            bullet.direction = (monsters[0].global_position - global_position).normalized()
        else:
            level.win_block.show()
    else:
        anim.play("Idle")
```

---

### 3.5 Player Node Tree and Monster Handling

**Scene Structure**

```yaml
🟢Player:
  groups: ["player"]
  signals:
    - animation_finished: _on_animation_player_animation_finished()
  children:
    - ReachArea:
        type: Area3D
        children:
          - CollisionShape3D:
              type: CollisionShape3D
    - Mage:
        type: Node3D
          children:
            - AnimationPlayer
    - CollisionShape3D
    - Timer
```

**How Monsters Are Populated**\
This line gets all the monsters on the scene so player can attack the first monster on the list

```gdscript
monsters = get_tree().get_nodes_in_group("enemy")
```

---

## 💥 Stage 4: Bullet Instantiation & Removal

### 4.1 Create Bullet Scene

- Root node: `Area3D`
- Child: `CollisionShape3D`
- Attach script for movement

**Scene Structure**

```yaml
🟢ManaBall:
  type: Area3D
  groups: ["bullet"]
  children:
    - CollisionShape3D:
        type: CollisionShape3D
        shape: SphereShape3D
  signals:
    - body_entered: _on_body_entered()
```

---

### 4.2 Instantiate Bullet

```gdscript
var bullet_scene: PackedScene = preload("res://mana_ball.tscn")
```

---

### 4.3 Move Bullet Toward Target

```gdscript
bullet.global_position = global_position + Vector3.UP
bullet.direction = (monsters[0].global_position - global_position).normalized()
```

---

### 4.4 Remove Bullet On Hit

```gdscript
func _on_body_entered(body: Node3D) -> void:
    if body.is_in_group("enemy"):
        body.change_hp(attack_power)
        queue_free()
```

---

## 🧾 Stage 5: UI Components

### 5.1 Add Health and Damage States

```gdscript
var hp: int = 100
var max_hp: int = 100
var is_dead: bool = false
var attack_power: int = 16
```

---

### 5.2 Create Floating Text Scene

- Create new scene `indicator_label.tscn`
- Root: `Node3D`
- Add a `SubViewport`
- Add a `Label` under it

**Scene Structure**

```yaml
🟢IndicatorLabel:
  type: Node3D
  children:
    - SubViewport:
        type: SubViewport
        children:
          - Label:
              type: Label
```

---

### 5.3 Create Damage Indicator Node

- Create new scene `damage_indicator.tscn`
- Root: `Node3D`
- Attach script

**Scene Structure**

```yaml
🟢DamageIndicator:
  type: Node3D
  children: []
```

**Damage Indicator Script**\
The first function creates a label one unit above the monster, the second one uses a tween to make it float then disappear

```gdscript
var indicator_label: PackedScene = preload("res://indicator_label.tscn")

func create_indicator_label(value):
    var indicatorLabel = indicator_label.instantiate()
    indicatorLabel.set_value(value)
    get_tree().current_scene.add_child(indicatorLabel)
    indicatorLabel.global_position = global_position + Vector3.UP
    _tween_indicator(indicatorLabel)

func _tween_indicator(label):
    var tween = create_tween()
    tween.tween_property(label, "position", label.global_position + Vector3(randf_range(-2, 2), randf_range(2, 4), randf_range(-2, 2)), 2)
    tween.tween_callback(label.queue_free)
```

---

## 🌄 Stage 6: Map Decoration

### Tasks

- Add a skybox material in `WorldEnvironment`
- Add props like trees, rocks, fences using `MeshInstance3D`
- Use Godot primitives or free assets
- Focus on creativity — no code needed

---

🎉 **End of Guide** — Congratulations! You've created a working 3D action prototype with movement, combat, UI, and decoration!

---
