# ShooterSam

Learning project for Unreal Engine 5.6 — a third-person shooter: run around the level
with a rifle and shoot enemy soldiers that patrol, chase and fire back. Built to practice
C++ in UE: `ACharacter`, spawning and attaching actors to sockets, line traces on a
custom trace channel, damage, Enhanced Input, AI with Behavior Trees (custom tasks and
services), UMG HUD, Niagara effects and sounds.

Based on the stock UE5 Third Person template (`Variant_Combat`, `Variant_Platforming`,
`Variant_SideScrolling` in `Source/ShooterSam/`), with custom classes and levels added
on top (`Content/MyAssets/`). Character model — Paragon Wraith (`Content/ParagonWraith/`).

## Custom code

`Source/ShooterSam/ShooterSamCharacter.h/.cpp` — on top of the template character
(used by both the player and the enemies):

- `GunClass` — spawned in `BeginPlay()` and attached to `WeaponSocket` on the mesh;
  the mesh's own `weapon_r` bone is hidden (`HideBoneByName`)
- Enhanced Input: `ShootAction` → `Shoot()`, which calls `Gun->PullTrigger()`
- health: `MaxHealth`/`Health`, `IsAlive` (`BlueprintReadOnly`, drives the death
  animation); `OnDamageTaken()` is bound to `OnTakeAnyDamage` — on death disables the
  capsule collision and detaches the controller
- `UpdateHUD()` — pushes the health percentage to the player controller's HUD widget

`Source/ShooterSam/Gun.h/.cpp` — a gun (`AActor`): `SceneRoot` → `Mesh` →
`MuzzleFlashParticleSystem` (Niagara):

- `PullTrigger()` — plays the muzzle flash and `ShootSound`, then line-traces from the
  owner controller's view point (`GetPlayerViewPoint`) up to `MaxRange` on the custom
  `Bullet` trace channel (`ECC_GameTraceChannel2`), ignoring the gun and its owner
- on hit: spawns `ImpactParticleSystem`, plays `ImpactSound` and applies `BulletDamage`
  via `UGameplayStatics::ApplyDamage`
- `OwnerController` — set by the character on spawn; for enemies the AI controller's
  view point is used for aiming

`Source/ShooterSam/ShooterAI.h/.cpp` — the enemy controller (`AAIController`):
`StartBehaviorTree(Player)` stores `MyCharacter`/`PlayerCharacter`, runs
`EnemyAIBehaviorTree` and writes the enemy's `StartLocation` to the blackboard.

Behavior Tree nodes:

- `BTService_PlayerLocation` — writes the player's location to the selected blackboard
  key every tick
- `BTService_PlayerLocationIfSeen` — if the enemy has line of sight to the player,
  writes their location and focuses on them; otherwise clears the key and the focus
- `BTTask_ClearBlackboardValue` — clears the selected blackboard key
- `BTTaskNode_Shoot` — makes the enemy shoot while the player is alive

`Source/ShooterSam/ShooterSamGameMode.h/.cpp` — `BeginPlay()` finds all `AShooterAI`
actors on the level and starts their behavior trees with a reference to the player.

`Source/ShooterSam/ShooterSamPlayerController.h/.cpp` — on top of the template
controller, creates the `HUDWidgetClass` widget and adds it to the viewport.

`Source/ShooterSam/HUDWidget.h/.cpp` — a `UUserWidget` base with a bound `HealthBar`
(`BindWidget`) and `SetHealthBarPercent()`.

## Content

- `Content/MyAssets/Blueprints/` — Blueprint wrappers on top of the C++ classes:
  `BP_ShooterCharacter`, `BP_Player`, `BP_Rifle`, `BP_ShooterSamGameMode`,
  `BP_ShooterSamPlayerController`, `BP_HUDWidget`
- `Content/MyAssets/AI/` — `BP_ShooterAI`, `BT_EnemyAI`, `BB_EnemyAi`
- `Content/MyAssets/Animation/` — `ABP_ShooterCharacter`, `BS_Locomation` (locomotion
  blend space)
- `Content/MyAssets/SFX/` — `SC_RifleShot`, `SC_RifleShotImpact`
- `Content/MyAssets/Maps/` — `MainLevel` (default map), `TestLevel`
- `Content/Input/` — Enhanced Input assets: `IMC_Default`, `IMC_MouseLook`, `IA_Move`,
  `IA_Look`, `IA_MouseLook`, `IA_Jump`, `IA_Shoot`
- Third-party/template assets: `ParagonWraith`, `ShooterGame`, `StarterBundle`,
  `LevelPrototyping`, `Characters`, `ThirdPerson`

## Controls

- Move / look — `IA_Move` / `IA_Look`, `IA_MouseLook` (bound in `IMC_Default`,
  `IMC_MouseLook`)
- Jump — `IA_Jump`
- Shoot — `IA_Shoot`

## Tech notes

- Engine: UE 5.6 (engine installed at `D:\UE_5.6`)
- Module dependencies: `EnhancedInput`, `AIModule`, `UMG`, `Niagara`
- Custom collision trace channel `Bullet` (`ECC_GameTraceChannel2`, default Block) in
  `Config/DefaultEngine.ini`
- Git LFS for binary assets (`.uasset`, `.umap`, textures, audio, `.fbx`)
- Generated folders (`Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/`) are
  gitignored

## Run

Open `ShooterSam.uproject` in UE 5.6 (regenerate VS project files if needed).
