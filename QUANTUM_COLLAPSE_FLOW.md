
# QUANTUM_COLLAPSE_FLOW.md

This document describes the canonical lifecycle of the Quantum Collapse Event within the Rahinii.VoxelNet architecture.

## Mermaid.js Sequence Diagram

```mermaid
sequenceDiagram
    participant MOT as ManualObserverTrigger.cs (MonoBehaviour)
    participant VOTS as VoxelObservationTriggerSystem.cs (ISystem)
    participant ECS_WORLD as Unity DOTS World (EntityManager)
    participant TV_VOXEL as TemporalQuantumVoxel (Entity + Components)
    participant QEES as QuantumEntanglementEffectSystem.cs (ISystem)
    participant QSL as QuantumStateSerializer.cs
    participant TQVLS as TemporalQuantumVoxelLoggerSystem.cs (ISystem)
    participant BSVC as BlockchainService.cs
    participant QL_SC as QuantumLogger.sol (Smart Contract on Testnet)
    participant QCEL as QuantumCollapseEventListener.cs (MonoBehaviour)

    alt Manual Trigger
        MOT ->>+ ECS_WORLD: User Interaction (e.g., Key Press, Inspector Button)
        ECS_WORLD -->> MOT: (Selects TargetVoxelEntity)
        MOT ->>+ ECS_WORLD: AddComponent(TargetVoxelEntity, ObserveVoxelTag)
        Note right of ECS_WORLD: TargetVoxelEntity now has ObserveVoxelTag
        ECS_WORLD -->>- MOT: Confirmation (Implicit)
    end

    alt Automatic Trigger (t == t_now)
        VOTS ->>+ ECS_WORLD: OnUpdate() - Query for eligible TV_VOXEL
        Note right of VOTS: Query: TV_VOXEL where t == t_now AND NOT (ObserveVoxelTag OR ObservedTag)
        ECS_WORLD -->> VOTS: Returns eligible TV_VOXEL entities
        VOTS ->>+ ECS_WORLD: For each eligible entity: AddComponent(entity, ObserveVoxelTag)
        ECS_WORLD -->>- VOTS: Confirmation (Implicit)
    end

    Note over ECS_WORLD, QEES: QuantumEntanglementEffectSystem runs (after VOTS/ManualTrigger)
    QEES ->>+ ECS_WORLD: OnUpdate() - Query for TV_VOXEL with ObserveVoxelTag AND NOT ObservedTag
    ECS_WORLD -->> QEES: Returns entities to process (e.g., Voxel_A)
    QEES ->> QEES: Perform CollapseVoxelState(Voxel_A.QState)
    QEES ->> QEES: Propagate to entangled voxels (e.g., Voxel_B)
    QEES ->> ECS_WORLD: Set Voxel_A.Observed = true
    QEES ->> ECS_WORLD: Set Voxel_B.Observed = true (if entanglement occurred)
    QEES ->> ECS_WORLD: AddComponent(Voxel_A, QuantumCollapseEvent)
    QEES ->> ECS_WORLD: AddComponent(Voxel_B, QuantumCollapseEvent) (if entanglement occurred)

    Note over ECS_WORLD, TQVLS: TemporalQuantumVoxelLoggerSystem runs (after QEES)
    TQVLS ->>+ ECS_WORLD: OnUpdate() - Query for entities with QuantumCollapseEvent
    ECS_WORLD -->> TQVLS: Returns entities with QuantumCollapseEvent (e.g., Voxel_A, Voxel_B)
    TQVLS ->> QSL: HashQuantumState(Voxel_A.QStateReal, Voxel_A.QStateImag)
    QSL -->> TQVLS: Returns bytes32 stateHash_A
    TQVLS ->> BSVC: LogQuantumCollapse(BlockchainQuantumLogEvent_A)
    BSVC ->>+ QL_SC: SendTransaction("logQuantumCollapse", params_A)
    QL_SC -->> BSVC: Returns TransactionReceipt_A
    TQVLS ->> ECS_WORLD: RemoveComponent(Voxel_A, QuantumCollapseEvent)

    Note over ECS_WORLD, QCEL: QuantumCollapseEventListener (Async WebSocket Listener)
    QCEL ->>+ QL_SC: Subscribes to QuantumCollapseLogged events
    QL_SC -->> QCEL: Streams new QuantumCollapseLogged event (e.g., event_A)
    QCEL ->> ECS_WORLD: Trigger in-world visualization updates or further logic
```

## Additional Notes

- This file is stored under `docs/QUANTUM_COLLAPSE_FLOW.md`
- Include both Mermaid.js and PlantUML versions for flexibility in rendering environments
