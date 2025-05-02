# L2 Game Prototype Documentation

## Before You Start: Documentation Convention

Please consider this document as **a cookbook that guides readers to reproduce the game prototype**. The document should be structured to provide a clear and concise overview of each stage of development, including tasks, changes, and code implementations.

The first few steps are pre-filled with simple examples of how to document the game prototype.

The **yaml format is used to represent the structure of the game scenes and nodes**, making it easier to understand the hierarchy, properties and changes of each component.

For example:

```yaml
NodeName:
    type: Node3D
    remarks: "May add notes of special handling about the node for the reader"
    properties:
        Group: "Example Group Name"
        Position: Vector3(0, 0, 0)
        PropertyName: SomeValues
    children:
        - 🟢NewNodeName:
            type: CharacterBody3D
            properties:
                PropertyName: SomeValues
        - ⭕️RemovedNodeName:
            type: CharacterBody3D
            properties:
                PropertyName: SomeValues
```

- The `NodeName` represents the name of the node in the project hierarchy tree.
- Use **snake_case** (e.g., "*type*", "*remarks*", "*properties*", "*children*") to indicate properties that are **NOT** specific to the Godot engine but are used to describe the structure.
- Use **PascalCase** (e.g., "*Position*", "*PropertyName*") to indicate the actual Godot properties.
- **Use `🟢` to indicate a new node** that is being added to the scene at the current stage.
- **Use `⭕️` to indicate a node** that is being removed from the scene at the current stage.
- If a node is a custom scene, may update the "*type*" to "*SceneName*".

**You don't have to include entire tree every time, just the relevant parts that are being changed or added in each stage.**

---

- [ ] **📋 Please complete the documenting the following stages.**

## Stage 1. Environment Setup

1. Add the root node with camera/light/floor.

    ```yaml
    🟢Root:
        type: Node3D
        children:
            - MainCamera:
                type: Camera3D
                properties:
                    Position: "Anywhere that fits"
                    ...
            - WorldEnvironment:
                type: WorldEnvironment
                properties:
                    Environment: "WorldEnvironment"
                    Camera Attributes: CameraAttributesPractical
            - SunLight:
                type: DirectionalLight3D
                properties:
                    Position: "Anywhere that fits"
                    ...
            - Floor:
                type: CSGBox3D
                properties:
                    Size: Vector3(200, 1, 200)
                    UseCollision: true
                    Material:
                        type: StandardMaterial3D
                        properties:
                            Albedo: 
                                Color: <any color you like>
    ```

## Stage 2. Character Creation

1. Download the models from the following link:
   1. <https://kaylousberg.itch.io/kaykit-adventurers>
   2. just some other reference: <https://www.mixamo.com/>

2. Unzip the downloaded file.
3. Drag the unzipped folder into the Godot window (File System panel).

    > Windows users may see dialog that prompts us to download extension to use .FBX files, just click "Disable FBX and restart

4. Briefly go through the imported files and explain we will be using the "gltf"/"glb" files.

5. Create the player node and save as scene.

    ```yaml
    Root:
      type: Node3D
      children:
        - ...
        - 🟢Player:
            type: Player
            properties:
              Position: "Anywhere that fits"
              Rotation: 0, 0, 0
              Scale: 1, 1, 1
    ```

    > Right click on the Player node and select "Save Branch As Scene" to save the player node as a separate scene.

6. Click the "movie" icon of the Player node to open the player scene.

7. Construct the player scene as follows:

    ```yaml
    Player:
      type: CharacterBody3D
      children:
        - 🟢Mage:
          type: Node3D
          remarks:
            - "1. Drag the mage.glb file under this scene"
            - "2. Right-click this and select [Make Local]"
          children:
            - AnimationPlayer:
                type: AnimationPlayer
                remarks: "Comes with the mage.glb file"
            - Rig:
                type: Node3D
                remarks: "Comes with the mage.glb file"
        - 🟢CollisionShape3D:
          type: CollisionShape3D
          properties:
            Shape:
              type: CapsuleShape3D
              properties:
                Radius: 1
            Position: 0, 1, 0
    ```

8. Expand the `Rig` node and the inner `Skeleton3D` node. Use the eye icons to hide un-wanted parts.

9. Ask students to repeat steps 5-8 to create the enemy scene:

    ```yaml
    Root:
      type: Node3D
      children:
        - ...
        - Enemy:
            type: Enemy
    Enemy:
      type: CharacterBody3D
      children:
        - Barbarian:
          type: Node3D
          remarks:
            - "1. Drag the barbarian.glb file under this scene"
            - "2. Right-click this and select [Make Local]"
          children:
            - AnimationPlayer:
                type: AnimationPlayer
                remarks: "Comes with the barbarian.glb file"
            - Rig:
                type: Node3D
                remarks: "Comes with the barbarian.glb file"
        - CollisionShape3D:
          type: CollisionShape3D
          properties:
            Shape:
              type: CapsuleShape3D
            Position: 0, 1, 0

    ```

10. Adds multiple enemy nodes to Root node and place them away from the player.

    ```yaml
    Root:
      type: Node3D
      children:
        - ...
        - Enemy1:
            type: Enemy
            properties:
              Position: ...
              Rotation: ...
        - Enemy2:
            type: Enemy
            properties:
              Position: ...
              Rotation: ...
        - Enemy3:
            type: Enemy
            properties:
              Position: ...
              Rotation: ...
    ```

11. Introduce the idea of groups in Godot.
    > Imagine group as tags, where we can label nodes with specific tags to identify them later. There are scene group and global group.
    >
    > Scene group is used in current scene, while global group is used within the entire project.

12. Create 2 groups: "player" and "enemy", and assign both scenes to corresponding group.

    ```yaml
    Player:
      type: CharacterBody3D
      group: player
      ...

    Enemy:
      type: CharacterBody3D
      group: enemy
      ...
    ```

## Stage 3. Character Animation Management

1. Select the `AnimationPlayer` of `Enemy` and mark the following animations as looping animation:
   1. "Idle"
   2. "Running_A"
   3. "Running_B"

2. Attach a script for enemy and play the idle animation:

    ```yaml
    Enemy:
      type: CharacterBody3D
      group: enemy
      script: enemy.gd
      children:
        - ...
    ```

    ```gdscript
    # enemy.gd
    extends CharacterBody3D

    var anim: AnimationPlayer

    func _ready() -> void:
      anim = $Barbarian/AnimationPlayer
      anim.play("Idle")
    ```

    > 1. May also simplify it with `@onready`
    > 2. Briefly explain the `$` which allows us to reference nodes in the node tree.
    > 3. Can experiment with other animation keys.

3. Ask students to repeat steps 1~2 play "Idle" animation for `Player`.

    ```gdscript
    # player.gd
    extends CharacterBody3D
    
    var anim: AnimationPlayer
    
    func _ready() -> void:
      anim = $Mage/AnimationPlayer
      anim.play("Idle")
    ```

4. Make enemy nodes look/move toward the player.

    ```gdscript
    # Enemy.gd
    extends CharacterBody3D

    const SPEED = 5.0

    var anim: AnimationPlayer
    var player: CharacterBody3D
    var has_reached_player = false

    func _ready() -> void:
      anim = $Barbarian/AnimationPlayer
      anim.play("Idle")
      player = get_tree().get_first_node_in_group('player')

    func _physics_process(delta: float) -> void:
      look_at(player.global_position, Vector3.UP, true)

      if not has_reached_player:
        move_to_player()
    
    func move_to_player():
      var direction = (player.global_position - global_position).normalized()
      velocity.x = direction.x * SPEED
      velocity.z = direction.z * SPEED
      move_and_slide()
    ```

5. Change enemy animation (idle -> run):

    ```gdscript
    # Enemy.gd
    ...

    func move_to_player():
      if (anim.current_animation != "Running_B"):
        anim.play("Running_B")
      
      var direction = (player.global_position - global_position).normalized()
      velocity.x = direction.x * SPEED
      velocity.z = direction.z * SPEED
      move_and_slide()
    ```

6. How to detect player is in the enemy's attack range.
   1. Add `Area3D` node in the enemy scene:

      ```yaml
      Enemy:
        type: CharacterBody3D
        group: enemy
        children:
          - ...
          - AttackRange:
            type: Area3D
            children:
              - CollisionShape3D:
                type: CollisionShape3D
                properties:
                  Shape: CylinderShape3D
                  Radius: 2
      ```

   2. Introduce the idea of "Signal":
      > They are messages that nodes emit when something happens, other nodes can connect to that signal and make reaction.

   3. Briefly go through the signals `Area3D` provides.

   4. Double-click on `body_entered` signal and connect to `Enemy` node.

   5. Update the states with:

      ```gdscript
      # enemy.gd
      func _on_attack_range_body_entered(body: Node3D) -> void:
        if body.is_in_group("player"):
          has_reached_player = true
          anim.play("1H_Melee_Attack_Chop")
      ```

7. Ask students to observe what happens next after attack animation finished.
    > The enemy halts.

8. Connect the signal from `AnimationPlayer` to `Enemy` scene:
    signal name: `animation_finished`

    ```gdscript
    # enemy.gd
    func _on_animation_player_animation_finished(anim_name: StringName) -> void:
      pass
    ```

9. Switch between idle and attack animations with attack cool-down:

    ```gdscript
    #enemy.gd
    const ATTACK_ANIM = "1H_Melee_Attack_Chop"
    const ATTACK_CD = 1.0

    var attack_timer = 0.0

    func _physics_process(delta: float) -> void:
      look_at(player.global_position, Vector3.UP, true)
      
      var is_attacking = anim.current_animation == ATTACK_ANIM
      if (attack_timer > 0 && !is_attacking):
        attack_timer -= delta

      if not has_reached_player:
        move_to_player()
      elif attack_timer <= 0:
        hit_player()
    
    func hit_player() -> void:
      anim.play(ATTACK_ANIM)
    
    func _on_attack_range_body_entered(body: Node3D) -> void:
      if body.is_in_group("player"):
        has_reached_player = true
        # ❌ anim.play("1H_Melee_Attack_Chop")
    
    func _on_animation_player_animation_finished(anim_name: StringName) -> void:
      # ❌ pass
      if (anim_name == ATTACK_ANIM):
        anim.play("Idle")
        attack_timer = ATTACK_CD
    ```

10. Discuss with students about how to make the player look toward the nearest enemy when they are in attack range.
    1. Add `Area3D` node in the player scene:
    2. Connect the `body_entered` signal to `Player` scene.
    3. Create a list `enemies` and add the enemies to the list when they enter the area.
    4. Use `look_at` to make the player look toward the nearest enemy.

11. Impl. of step 10:
    1. Add `Area3D` node in the player scene:

        ```yaml
        Player:
          type: CharacterBody3D
          group: player
          children:
            - ...
            - AttackRange:
                type: Area3D
                properties:
                  Position: 0, 1, 0
                children:
                  - CollisionShape3D:
                      type: CollisionShape3D
                      properties:
                        Shape: CylinderShape3D
                        Radius: 15
        ```

    2. Connect the `body_entered` signal to `Player` scene.

        ```gdscript
        # player.gd
        func _on_attack_range_body_entered(body: Node3D) -> void:
          pass
        ```

    3. Create a list `enemies` and add the enemies to the list when they enter the area.

        ```gdscript
        extends CharacterBody3D

        const ATTACK_ANIM = "1H_Melee_Attack_Chop"
        const ATTACK_CD = 1.0

        var anim: AnimationPlayer
        var enemies: Array[CharacterBody3D] = []
        var attack_timer = 0.0

        func _ready() -> void:
          anim = $Mage/AnimationPlayer
          anim.play("Idle")

        func _physics_process(delta: float) -> void:
          var is_attacking = anim.current_animation == ATTACK_ANIM
          if attack_timer > 0 and not is_attacking:
            attack_timer -= delta
          
          if not enemies.is_empty() and attack_timer <= 0:
            look_at(enemies[0].global_position, Vector3.UP, true)
            hit_enemy()

        func hit_enemy() -> void:
          anim.play(ATTACK_ANIM)

        func _on_attack_range_body_entered(body: Node3D) -> void:
          if body.is_in_group("enemy") and not enemies.has(body):
            enemies.append(body)
        ```

<!-- 12. How to update the player animation in runtime.

- switch between idle and attack animations.
- add a cooldown to the attack animation. -->

## Stage 4. Bullet Instantiation & Removal

1. Create a ManaBall node and save as scene.

    ```yaml
    ManaBall:
      type: Area3D
      children:
        - CollisionShape3D:
            type: CollisionShape3D
            properties:
              Shape: SphereShape3D
              Radius: 0.5
        - MeshInstance3D:
            type: CSGSphere3D
            properties:
              Material:
                type: StandardMaterial3D
                properties:
                  Albedo:
                    Color: any
    ```

2. Instantiate/spawn a `ManaBall` node by the player.

    ```gdscript
    # player.gd
    const MANA_BALL_SCENE = preload("res://mana_ball.tscn")

    func hit_enemy() -> void:
      anim.play(ATTACK_ANIM)
      
      var mana_ball = MANA_BALL_SCENE.instantiate()
      get_parent().add_child(mana_ball)
      
      attack_timer = ATTACK_CD
    ```

3. Make the `ManaBall` move toward the target enemy.
    (Create a new script for `ManaBall`)

    ```gdscript
    # mana_ball.gd
    extends Area3D

    var target: CharacterBody3D
    var speed = 20
    var attack_power = 50

    func _ready() -> void:
      # Lift the ball up by 1 unit in y-direction
      global_position += Vector3.UP


    func _physics_process(delta: float) -> void:
      var distance = target.global_position - global_position
      # `distance` represents the vector from the current position to the target's position.
      # For example, it could be something like (x: 10, y: 0, z: 50).

      var direction = distance.normalized()
      # `direction` is the normalized vector of `distance`, representing the direction
      # to the target as a unit vector. 
      # For example, it could be something like (x: 0.2, y: 0, z: 1.0).
      # Ultimately, we can calculate the angle to the target with trigonometry formula
      
      global_position += direction * speed * delta
    ```

    ```gdscript
    # player.gd
    func hit_enemy() -> void:
      anim.play(ATTACK_ANIM)
      
      var mana_ball = MANA_BALL_SCENE.instantiate()
      mana_ball.target = enemies[0]
      get_parent().add_child(mana_ball)
      
      attack_timer = ATTACK_CD
    ```

4. Remove the `ManaBall` after hit.
   (Connect the `body_entered` signal to `ManaBall`)

    ```gdscript
    func _on_body_entered(body: Node3D) -> void:
      if body.is_in_group("enemy"):
        queue_free()
    ```

## Stage 5. Enemy Death

1. Add a health property to the enemy node.

    ```gdscript
    # enemy.gd
    var hp = 100.0
    var is_dead = false

    func take_hit(damage: float):
      if is_dead: return
      
      hp -= damage
      is_dead = hp <= 0
    ```

    ```gdscript
    # mana_ball.gd

    func _on_body_entered(body: Node3D) -> void:
      if body.is_in_group("enemy") and not body.is_dead:
        body.take_hit(attack_power)
        queue_free()
    ```

2. Make enemy "dead" when hp <= 0.

    ```gdscript
    # enemy.gd
    func _physics_process(delta: float) -> void:
      if is_dead: return
      ...

    func take_hit(damage: float):
      if is_dead: return
      
      hp -= damage
      is_dead = hp <= 0
      
      if (is_dead):
        anim.play("Death_A")

      func _on_animation_player_animation_finished(anim_name: StringName) -> void:
        if anim_name == ATTACK_ANIM:
          anim.play("Idle")
          attack_timer = ATTACK_CD
        
        if anim_name == "Death_A":
          await get_tree().create_timer(2.0).timeout
          queue_free()
    ```

3. Make `Player` forgets the dead enemies.

    ```gdscript
    # player.gd

    func _physics_process(delta: float) -> void:
      
      check_dead_enemies()
      
      ...

    func check_dead_enemies() -> void:
      var i = enemies.size() - 1;
      while i >= 0:
        if enemies[i].is_dead:
          enemies.remove_at(i)
        i -= 1
    ```

4. (Extra) Sort the enemies by distance to the player.
    (Can provide the code snippet to students directly)

    ```gdscript
    # player.gd
    func check_dead_enemies() -> void:
      var i = enemies.size() - 1;
      while i >= 0:
        if enemies[i].is_dead:
          enemies.remove_at(i)
        i -= 1
      sort_enemies()

    func sort_enemies() -> void:
      enemies.sort_custom(func(a, b):
        return (a.global_position.distance_squared_to(global_position)
          < b.global_position.distance_squared_to(global_position))
        )
    ```

## Stage 6. Spawning Enemies

1. Create a new scene called `EnemySpawner` and add to `Root` node.

    ```yaml
    Root:
      type: Node3D
      children:
        - ...
        - EnemySpawner:
            type: EnemySpawner

    EnemySpawner:
      type: Node3D
      script: enemy_spawner.gd
    ```

    ```gdscript
    extends Node3D

    @export var enemy_scene: PackedScene
    @export var floor_node: CSGBox3D
    @export var spawn_interval = 3.0
    @export var spawn_distance_from_edge = 1.0
    ```

2. Assign the required properties in the inspector.

    ```yaml
    EnemySpawner:
      type: Node3D
      properties:
        Enemy Scene: Enemy.tscn
        Floor Node: Floor
        Spawn Interval: 3.0
        Spawn Distance From Edge: 1.0
    ```

3. Add a `Timer` node to the `EnemySpawner` scene.

    ```yaml
    EnemySpawner:
      type: Node3D
      children:
        - Timer:
            type: Timer
            properties:
              Wait Time: 2.0
              Autostart: true
    ```

4. Connect the `timeout` signal of the `Timer` node to the `EnemySpawner` scene.

    ```gdscript
    # enemy_spawner.gd
    extends Node3D

    @export var enemy_scene: PackedScene
    @export var floor_node: CSGBox3D
    @export var spawn_interval = 3.0
    @export var spawn_distance_from_edge = 1.0

    func _on_timer_timeout() -> void:
      pass
    ```

5. Update the `EnemySpawner` script to spawn enemies at random positions on the floor.
    (Can provide the code snippet to students directly)

    ```gdscript
    # enemy_spawner.gd
    extends Node3D

    @export var enemy_scene: PackedScene
    @export var floor_node: CSGBox3D
    @export var spawn_interval = 3.0
    @export var spawn_distance_from_edge = 1.0


    func _on_timer_timeout() -> void:
      print('hi from enemy spawnner')
      spawn_enemy_at_edge()


    func spawn_enemy_at_edge():
      if not enemy_scene or not floor_node:
        return
      
      # Get floor dimensions
      var floor_size = floor_node.size
      var half_width = floor_size.x / 2
      var half_depth = floor_size.z / 2
      
      # Randomly select which edge to spawn on (0=top, 1=right, 2=bottom, 3=left)
      var edge = randi() % 4
      var spawn_pos = Vector3()
      
      match edge:
        0: # Top edge (negative Z)
          spawn_pos = Vector3(
            randf_range(-half_width + spawn_distance_from_edge, half_width - spawn_distance_from_edge),
            0,
            -half_depth + spawn_distance_from_edge
          )
        1: # Right edge (positive X)
          spawn_pos = Vector3(
            half_width - spawn_distance_from_edge,
            0,
            randf_range(-half_depth + spawn_distance_from_edge, half_depth - spawn_distance_from_edge)
          )
        2: # Bottom edge (positive Z)
          spawn_pos = Vector3(
            randf_range(-half_width + spawn_distance_from_edge, half_width - spawn_distance_from_edge),
            0,
            half_depth - spawn_distance_from_edge
          )
        3: # Left edge (negative X)
          spawn_pos = Vector3(
            -half_width + spawn_distance_from_edge,
            0,
            randf_range(-half_depth + spawn_distance_from_edge, half_depth - spawn_distance_from_edge)
          )
      
      # Adjust for floor's global position
      spawn_pos += floor_node.global_position
      
      # Create and position enemy
      var enemy = enemy_scene.instantiate()
      get_parent().add_child(enemy)
      enemy.global_position = spawn_pos
    ```

## Stage 7. Camera Control

1. Update the `Player` scene with Camera and SpringArm nodes.

    ```yaml
    Player:
      type: CharacterBody3D
      group: player
      children:
        - ...
        - SpringArm3D:
            type: SpringArm3D
            properties:
              SprintLength: 3.5
              Position: 0, 3, -2.8
              Rotation: -30, 180, 0
            children:
              - Camera3D:
                  type: Camera3D
    ```

    > The `SpringArm3D` node is used to position the camera at a distance from the player while avoiding collisions with the environment.

2. Update the `Player` script to control the camera rotation.

    ```gdscript
    # player.gd
    @export var mouse_sensitivity = 0.02
    @export var camera_vertical_limit = deg_to_rad(70)
    var camera_rotation = Vector2.ZERO

    func _ready() -> void:
      anim = $Mage/AnimationPlayer
      anim.play("Idle")
      
      # Use the player camera
      $SpringArm3D/Camera3D.current = true

    func _input(event: InputEvent) -> void:
      if event is InputEventMouseMotion:
        # Horizontal rotation (player turns left/right):
        rotate_y(-event.relative.x * mouse_sensitivity)

    func _physics_process(delta: float) -> void:
      
      check_dead_enemies()
      
      var is_attacking = anim.current_animation == ATTACK_ANIM
      if attack_timer > 0 and not is_attacking:
        attack_timer -= delta
      
      if not enemies.is_empty() and attack_timer <= 0:
        #❌look_at(enemies[0].global_position, Vector3.UP, true)
        hit_enemy()
    ```

3. Make the camera follow the mouse movement.

    ```gdscript
    # player.gd
    func _input(event: InputEvent) -> void:
      if event is InputEventMouseMotion:
        # Horizontal rotation (player turns left/right):
        rotate_y(-event.relative.x * mouse_sensitivity)
        
        # Vertical rotation (camera tilts up/down):
        camera_rotation.x -= event.relative.y * mouse_sensitivity
        camera_rotation.x = clamp(camera_rotation.x, -camera_vertical_limit, camera_vertical_limit)
        $SpringArm3D.rotation_degrees.x = camera_rotation.x
    ```

## Stage 8. Player Movement

1. Update the `Player` script to control the player movement.

    ```gdscript
    # player.gd
    const SPEED = 5.0

    var gravity = ProjectSettings.get_setting("physics/3d/default_gravity")

    func _physics_process(delta: float) -> void:
      if not is_on_floor():
        velocity.y -= gravity * delta
      
      var input_dir = Input.get_vector("ui_right", "ui_left", "ui_down", "ui_up")
      var direction = (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()
      if direction:
        velocity.x = direction.x * SPEED
        velocity.z = direction.z * SPEED
      else:
        velocity.x = move_toward(velocity.x, 0, SPEED)
        velocity.z = move_toward(velocity.z, 0, SPEED)
      
      move_and_slide()
      ...
    ```

2. Adds W/A/S/D keys to "ui_up"/"ui_down"/"ui_left"/"ui_right".
   (Project Settings > Input Map)

3. Make sure the `ManaBall` is spawned in front of the player.

    ```gdscript
    # player.gd
    func hit_enemy() -> void:
      anim.play(ATTACK_ANIM)
      
      var mana_ball = MANA_BALL_SCENE.instantiate()
      mana_ball.target = enemies[0]
      mana_ball.global_position = global_position + Vector3(0, 1, -2.8)
      get_parent().add_child(mana_ball)
      
      attack_timer = ATTACK_CD
    ```

4. Make `Enemy` able to follow the player when player left the attack range.
   (Connect the `body_exited` signal to `Enemy` scene)

    ```gdscript
    # enemy.gd
    func _on_attack_range_body_exited(body: Node3D) -> void:
      if body.is_in_group("player"):
        has_reached_player = false
    ```

5. Adds Run/Idle animation to the player.

    ```gdscript
    # player.gd
    func _physics_process(delta: float) -> void:
      ...
      move_and_slide()
      handle_movement_animation(input_dir)
      ...
    
    func handle_movement_animation(input_dir: Vector2) -> void:
      # Only change animation if we are not attacking
      if anim.current_animation == ATTACK_ANIM: return
      
      if input_dir.length() > 0.1: # if there is significant input
        if anim.current_animation != "Running_B":
          anim.play("Running_B")
      else:
        if anim.current_animation != "Idle":
          anim.play("Idle")
    ```

6. Disable the attack animation.

    ```gdscript
    # player.gd
    func hit_enemy() -> void:
      #❌anim.play(ATTACK_ANIM)
      ...
    ```

7. Smoothen the transition between idle and run animations.
   1. Select the `AnimationPlayer` node in the player scene.
   2. Select the `Idle` animation and click on the "Animation" button.
   3. Select "Edit Transition"
   4. Click on the "Running_B" animation and change the value to 0.5.
   5. Repeat (i ~ iv) for the `Running_B` animation and change the value of `Idle` to 0.5.

8. Handle `ManaBall` that lost its target.

    ```gdscript
    # mana_ball.gd    
    func _physics_process(delta: float) -> void:
      if not is_instance_valid(target):
        queue_free()
        return
    ```

---
<!-- TODO: below -->

## Stage 5. UI Components

1. How to adds states to the player/enemy nodes.
   - damage
   - health
   - isDead

2. How to add health bar to the enemy nodes.

3. How to update the enemy health bar.
   - Update the health bar when the enemy is damaged.
   - Remove the health bar when the enemy is dead.

4. How to create a floating text scene.

5. How to instantiate the floating text scene when the enemy is damaged.

6. How to remove the floating text scene after a few seconds.

7. How to create the HUD scene.
   - Player health bar
   - Player score label

8. How to update the HUD scene.
   - Update the player health bar when the player is damaged.
   - Update the player score label when the player kills an enemy.

9. How to add a game over screen.
   - Show the game over screen when the player is dead.
   - Restart the game when the player clicks the restart button.

10. How to add a win screen.

- Show the win screen when the player kills all enemies.
- Restart the game when the player clicks the restart button.

## Stage 6. Map Decoration

1. How to add a skybox to the scene.
2. How to add a skybox material to the sc
3. Where to find the skybox material.
4. What is the skybox material.
5. How to add a skybox to the scene.
