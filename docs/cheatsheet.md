# FastCast2 CheatSheet v0.0.9

## Caster

```luau
--// Construct & Init

local Caster = FastCast2.new() -- Construct a new Caster

Caster:Init(
    numWorker: number,                  -- Number of worker VMs. Must be > 1.
    newParent: Folder,                  -- Parent Folder for the FastCastVMs Folder.
    newName: string,                    -- Name for the FastCastVMs Folder.
    ContainerParent: Folder,            -- Parent Folder for worker VM Containers.
    VMname: string,                     -- Name given to each worker VM.
    useBulkMoveTo: boolean,             -- Enable BulkMoveTo for CosmeticBulletObjects.
    FastCastEventsModule: ModuleScript, -- ModuleScript returning a FastCastEvents table.
    useObjectCache: boolean,            -- Enable ObjectCache for this Caster.
    Template: BasePart | Model,         -- Template object for ObjectCache (if enabled).
    CacheHolder: Instance               -- Parent Instance for cached objects (if enabled).
)
-- ⚠ Must be called before any Fire methods — nothing happens without Init!


--// Fire Methods

Caster:RaycastFire(
    origin: Vector3,
    direction: Vector3,
    velocity: Vector3 | number,
    BehaviorData: FastCastBehavior?
) → ()  -- Fire a raycast projectile.

Caster:BlockcastFire(
    origin: Vector3,
    Size: Vector3,
    direction: Vector3,
    velocity: Vector3 | number,
    BehaviorData: FastCastBehavior?
) → ()  -- Fire a blockcast projectile.

Caster:SpherecastFire(
    origin: Vector3,
    Radius: number,
    direction: Vector3,
    velocity: Vector3 | number,
    BehaviorData: FastCastBehavior?
) → ()  -- Fire a spherecast projectile.


--// Configuration

Caster:SetFastCastEventsModule(moduleScript: ModuleScript) → ()
-- Set a new FastCastEventsModule for all future BaseCasts from this Caster.

Caster:SetBulkMoveEnabled(enabled: boolean) → ()
-- Toggle BulkMoveTo for cosmetic bullet CFrame updates.

Caster:SetObjectCacheEnabled(
    enabled: boolean,           -- Toggle ObjectCache on/off.
    Template: BasePart | Model, -- Projectile template to cache.
    CacheSize: number,          -- Number of objects to pre-allocate.
    CacheHolder: Instance       -- Where cached objects are stored.
) → ()


--// Cast Manipulation  (use Caster reference, not cast itself)

Caster:GetPositionCast(cast: vaildcast) → Vector3     -- Current position of the cast.
Caster:GetVelocityCast(cast: vaildcast) → Vector3     -- Current velocity of the cast.
Caster:GetAccelerationCast(cast: vaildcast) → Vector3 -- Current acceleration of the cast.

Caster:SetVelocityCast(cast: vaildcast, velocity: Vector3) → ()         -- Override velocity.
Caster:SetAccelerationCast(cast: vaildcast, acceleration: Vector3) → () -- Override acceleration.

Caster:AddPositionCast(cast: vaildcast, position: Vector3) → ()         -- Add to current position.
Caster:AddVelocityCast(cast: vaildcast, velocity: Vector3) → ()         -- Add to current velocity.
Caster:AddAccelerationCast(cast: vaildcast, acceleration: Vector3) → () -- Add to current acceleration.

-- ⚠ After any Set/Add call, sync changes or they won't take effect in the VM:
Caster:SyncChangesToCast(cast: vaildcast) → () -- Push pending state changes to the worker VM.

Caster:PauseCast(cast: vaildcast) → ()  -- Pause simulation for a cast.
Caster:ResumeCast(cast: vaildcast) → () -- Resume a previously paused cast.
Caster:TerminateCast(cast: vaildcast) → () -- Forcefully terminate a cast early.


--// Lifecycle

Caster:Destroy() → () -- Destroy the Caster and clean up all resources.


--// Fields

-- Signals (assign a callback OR connect an RBXScriptConnection)
Caster.LengthChanged: (RBXScriptConnection | OnLengthChangedFunction)?
-- Fired every simulation step with: cast, segmentOrigin, segmentDirection, length, segmentVelocity, cosmeticBullet

Caster.Hit: (RBXScriptConnection | OnHitFunction)?
-- Fired when the cast hits something (non-piercing): cast, raycastResult, segmentVelocity, cosmeticBullet

Caster.Pierced: (RBXScriptConnection | OnPierceFunction)?
-- Fired when the cast pierces through something: cast, raycastResult, segmentVelocity, cosmeticBullet

Caster.CastFire: (RBXScriptConnection | OnCastFireFunction)?
-- Fired when a cast is initially launched: cast, origin, direction, velocity, behavior

Caster.CastTerminating: (RBXScriptConnection | OnCastTerminatingFunction)?
-- Fired just before a cast is destroyed: cast

-- State
Caster.AlreadyInit: boolean          -- True after Init() has been called.
Caster.ObjectCacheEnabled: boolean   -- Whether ObjectCache is currently active.
Caster.BulkMoveEnabled: boolean      -- Whether BulkMoveTo is currently active.

-- References
Caster.WorldRoot: WorldRoot                  -- The WorldRoot this Caster simulates in.
Caster.FastCastEventsModule: ModuleScript    -- The currently assigned FastCastEvents module.
Caster.ObjectCache: ObjectCache              -- The ObjectCache instance (if enabled).
Caster.Dispatcher: Dispatcher               -- Internal worker VM dispatcher.
```

```lua
-- ActiveCastData fields
-- All three cast variants (Raycast, Blockcast, Spherecast) share the same structure.

cast.ID: number                    -- Unique identifier for this ActiveCast instance.
cast.Type: "Raycast" | "Blockcast" | "Spherecast"  -- The cast variant type.
cast.CFrame: CFrame                -- Current CFrame of the cosmetic bullet object.
cast.UserData: { [any]: any }      -- Free-use table for storing custom data on the cast.

cast.Caster: BaseCastData          -- Reference to the parent BaseCastData (internal caster bindings).
cast.RayInfo: CastRayInfo          -- Ray/cast geometry info (params, world root, max distance, cosmetic object, pierce module).
cast.StateInfo: CastStateInfo      -- Runtime state of the cast (see below).

--// CastStateInfo (cast.StateInfo)

cast.StateInfo.UpdateConnection: RBXScriptSignal   -- The heartbeat/stepped connection driving this cast.
cast.StateInfo.Paused: boolean                     -- Whether the cast is currently paused.
cast.StateInfo.TotalRuntime: number                -- Total elapsed simulation time (seconds).
cast.StateInfo.DistanceCovered: number             -- Total distance traveled so far (studs).
cast.StateInfo.IsActivelySimulatingPierce: boolean -- True while the cast is processing a pierce check.
cast.StateInfo.IsActivelyResimulating: boolean     -- True while the cast is doing a high-fidelity resimulation pass.
cast.StateInfo.CancelHighResCast: boolean          -- Set to true to abort the current high-res cast.
cast.StateInfo.HighFidelityBehavior: number        -- Current high-fidelity behavior mode.
cast.StateInfo.HighFidelitySegmentSize: number     -- Segment size used in high-fidelity mode.
cast.StateInfo.VisualizeCasts: boolean             -- Whether debug visualization is active for this cast.
cast.StateInfo.Trajectories: { [number]: CastTrajectory } -- List of trajectory segments for this cast.

cast.StateInfo.VisualizeCastSettings: VisualizeCastSettings     -- Debug visualization config.
cast.StateInfo.FastCastEventsConfig: FastCastEventsConfig       -- Which events are enabled (non-module callbacks).
cast.StateInfo.FastCastEventsModuleConfig: FastCastEventsModuleConfig -- Which module events are enabled.

--// CastTrajectory (entry in cast.StateInfo.Trajectories)

trajectory.StartTime: number        -- Simulation time when this trajectory segment began.
trajectory.EndTime: number          -- Simulation time when this segment ended (0 if still active).
trajectory.Origin: Vector3          -- Origin point of this segment.
trajectory.InitialVelocity: Vector3 -- Velocity at the start of this segment.
trajectory.Acceleration: Vector3    -- Acceleration applied throughout this segment.

--// CastRayInfo (cast.RayInfo)

cast.RayInfo.Parameters: RaycastParams         -- RaycastParams used for this cast.
cast.RayInfo.WorldRoot: WorldRoot              -- The WorldRoot the cast is simulating in.
cast.RayInfo.MaxDistance: number               -- Maximum travel distance before the cast terminates.
cast.RayInfo.CosmeticBulletObject: Instance?   -- The cosmetic bullet object attached to this cast (if any).
cast.RayInfo.CanPierceModule: ModuleScript?    -- Optional pierce-decision module (legacy / manual pierce setup).

--// BaseCastData (cast.Caster)

cast.Caster.Output: BindableEvent              -- Internal event used to relay cast signals back to the Caster.
cast.Caster.ActiveCastCleaner: BindableEvent   -- Fired when an ActiveCast is cleaned up.
cast.Caster.SyncChange: BindableEvent          -- Used to sync manual state changes (velocity, position, etc.) back to the VM.
cast.Caster.ObjectCache: BindableFunction?     -- Reference to the ObjectCache BindableFunction (if ObjectCache is enabled).
cast.Caster.CacheHolder: any?                  -- The Instance holding cached objects (if ObjectCache is enabled).
```