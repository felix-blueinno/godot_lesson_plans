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
                  Scale: 2, 2, 2
                children:
                  - CollisionShape3D:
                      type: CollisionShape3D
                      properties:
                        Shape: CylinderShape3D
                        Radius: 2
        ```

    2. Connect the `body_entered` signal to `Player` scene.

        ```gdscript
        # player.gd
        func _on_attack_range_body_entered(body: Node3D) -> void:
          pass
        ```

    3. Create a list `enemies` and add the enemies to the list when they enter the area.

        ```gdscript
        # player.gd
        extends CharacterBody3D
        
        var enemies: Array[CharacterBody3D] = []
        var attack_timer = 0.0

        func _physics_process(delta: float) -> void:
          var is_attacking = anim.current_animation == ATTACK_ANIM
          if (attack_timer > 0 && !is_attacking):
            attack_timer -= delta

          if (not enemies.is_empty()):
            look_at(enemies[0].global_position, Vector3.UP, true)
              
            if (attack_timer <= 0):
              hit_enemy()

        func _on_attack_range_body_entered(body: Node3D) -> void:
          if (body.is_in_group("enemy")):
            if (not enemies.has(body)):
              enemies.append(body);
              sort_enemies()
        
        func sort_enemies() -> void:
          enemies.sort_custom(func(a, b):
            return (a.global_position.distance_squared_to(global_position) 
            < b.global_position.distance_squared_to(global_position))
          )
        ```

    ```gdscript
    # player.gd
    extends CharacterBody3D

    var anim: AnimationPlayer
    var enemies: Array = []

    func _ready() -> void:
      anim = $Mage/AnimationPlayer
      anim.play("Idle")
    
    func _on_attack_range_body_entered(body: Node3D) -> void:
      if body.is_in_group("enemy"):
        enemies.append(body)
        look_at(body.global_position, Vector3.UP, true)
    
    func _on_attack_range_body_exited(body: Node3D) -> void:
      if body.is_in_group("enemy"):
        enemies.erase(body)
        if enemies.size() > 0:
          look_at(enemies[0].global_position, Vector3.UP, true)
    ```

12. How to update the player animation in runtime.

- switch between idle and attack animations.
- add a cooldown to the attack animation.

## Stage 4. Bullet Instantiation & Removal

1. How to create a bullet scene.

2. How to instantiate a bullet node to the player.

3. How to make the bullet move toward the target enemy.

4. How to remove the bullet after hit.

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
