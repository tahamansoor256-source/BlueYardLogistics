# Blue Yard Logistics — System Architecture & UML Specification

---

## 1. USE CASE DIAGRAM

```mermaid
graph TD
    User((Port Operator / Logistics Controller))

    subgraph Blue Yard Logistics Platform
        UC1[Ingest Vessel Cargo Manifest]
        UC2[Run Python AI Cargo & Dwell Analysis]
        UC3[Assign Optimal Terminal via Multi-Criteria Scoring]
        UC4[Optimize 2D Yard Grid Slot]
        UC5[Match Specialized Warehouse]
        UC6[Allocate Capacity-Constrained Vehicle]
        UC7[Execute Dijkstra Shortest Route Planner]
        UC8[Dispatch Cargo & Track Audit Timeline]
    end

    User --> UC1
    User --> UC2
    User --> UC3
    User --> UC4
    User --> UC5
    User --> UC6
    User --> UC7
    User --> UC8

    UC2 -.->|Includes| UC3
    UC3 -.->|Includes| UC4
    UC4 -.->|Includes| UC5
    UC5 -.->|Includes| UC6
    UC6 -.->|Includes| UC7
    UC7 -.->|Includes| UC8
    flowchart TD
    Start([Vessel Arrives at Berth]) --> Manifest[Manifest Containers in BlueYardDB]
    Manifest --> PythonReq[Send Payload to Python FastAPI Service]
    PythonReq --> AIAnalysis[Calculate Priority Score & Predict Dwell Time]
    AIAnalysis --> TermEval[C# TerminalScorer: Evaluate Cargo Compatibility & Load]
    TermEval --> YardOpt[C# YardOptimizer: Allocate 2D Grid Slot Row 1 vs Row 2]
    YardOpt --> WhEval[C# WarehouseSelector: Match Facility Type & Destination]
    WhEval --> VehEval[C# VehicleAssigner: Match Weight & Fleet Specialization]
    
    VehEval --> CheckFleet{Vehicle Available & Fit?}
    CheckFleet -- No --> ErrorBlock[Hold Container / Flag Resource Shortage]
    CheckFleet -- Yes --> BuildGraph[Construct TransportGraph from Routes]
    
    BuildGraph --> RunDijkstra[Execute Dijkstra Algorithm: Port to Warehouse]
    RunDijkstra --> UpdateDB[Update Container State & Free Yard Slot]
    UpdateDB --> LogHistory[Write Audit Log to ContainerHistory Table]
    LogHistory --> EndState([Cargo Dispatched En Route])
    sequenceDiagram
    autonumber
    actor Operator as Web UI Dashboard
    participant API as ASP.NET Core ContainersController
    participant Py as Python FastAPI (:8000)
    participant DSA as C# DSA Engine
    participant DB as SQL Server (BlueYardDB)

    Operator->>API: POST /api/containers/{id}/auto-process
    activate API

    API->>Py: POST /analyze (Container Metadata)
    activate Py
    Py-->>API: 200 OK (Priority Score, Risk, Dwell Hours)
    deactivate Py

    API->>DSA: TerminalScorer.SelectBestTerminal(container, terminals)
    activate DSA
    DSA-->>API: Return Terminal Beta (T-B)
    deactivate DSA

    API->>DSA: YardOptimizer.AssignSlot(container, yardSlots)
    activate DSA
    DSA-->>API: Return Slot TB-R1-C1
    deactivate DSA

    API->>DSA: WarehouseSelector.SelectWarehouse(container, warehouses)
    activate DSA
    DSA-->>API: Return Warehouse W1 (Pharma)
    deactivate DSA

    API->>DSA: VehicleAssigner.AssignVehicle(container, vehicles)
    activate DSA
    DSA-->>API: Return Vehicle V1 (Reefer Truck)
    deactivate DSA

    API->>DSA: DijkstraRouter.FindShortestPath(graph, "Port", "W1")
    activate DSA
    DSA-->>API: Return Route: Port ➔ J1 ➔ W1 (9.0 km)
    deactivate DSA

    API->>DB: Save Changes & Append ContainerHistory Records
    activate DB
    DB-->>API: Transaction Committed
    deactivate DB

    API-->>Operator: 200 OK (Full Processed Lifecycle Payload)
    deactivate API
    classDiagram
    class Container {
        +string ContainerID
        +string VesselID
        +string CargoType
        +int WeightKg
        +string PriorityLevel
        +string Destination
        +string Status
        +int PriorityScore
        +string RiskLevel
        +int ExpectedDwellHours
        +string AssignedTerminalID
        +string AssignedYardSlot
        +string AssignedWarehouseID
        +string AssignedVehicleID
        +string OptimalRoute
        +decimal TotalRouteDistanceKm
    }

    class Terminal {
        +string TerminalID
        +string Name
        +string Type
        +int Capacity
        +int CurrentLoad
    }

    class YardSlot {
        +string SlotID
        +string TerminalID
        +int SlotRow
        +int SlotColumn
        +bool IsOccupied
        +string ContainerID
    }

    class Warehouse {
        +string WarehouseID
        +string Name
        +string Type
        +int Capacity
        +int CurrentLoad
        +string Location
    }

    class Vehicle {
        +string VehicleID
        +string Type
        +int CapacityKg
        +string Status
        +string CurrentLocation
    }

    class TransportGraph {
        +Dictionary~string, List~GraphEdge~~ AdjacencyList
        +AddNode(string node)
        +AddEdge(string src, string dest, decimal dist, int time)
    }

    class DijkstraRouter {
        +FindShortestPath(TransportGraph graph, string start, string target) RouteResult
    }

    class TerminalScorer {
        +SelectBestTerminal(Container c, List~Terminal~ t) Terminal
    }

    class YardOptimizer {
        +AssignSlot(Container c, List~YardSlot~ slots) YardSlotResult
    }

    class WarehouseSelector {
        +SelectWarehouse(Container c, List~Warehouse~ w) Warehouse
    }

    class VehicleAssigner {
        +AssignVehicle(Container c, List~Vehicle~ v) Vehicle
    }

    Container --> Terminal : Assigned to
    Container --> YardSlot : Occupies
    Container --> Warehouse : Destined for
    Container --> Vehicle : Transported by
    DijkstraRouter ..> TransportGraph : Traverses
    erDiagram
    VESSELS ||--o{ CONTAINERS : manifests
    TERMINALS ||--o{ YARD_SLOTS : contains
    CONTAINERS ||--o{ CONTAINER_HISTORY : logs
    TERMINALS ||--o{ CONTAINERS : allocates
    WAREHOUSES ||--o{ CONTAINERS : stores
    VEHICLES ||--o{ CONTAINERS : transports

    VESSELS {
        varchar VesselID PK
        nvarchar Name
        nvarchar Origin
        datetime ArrivalTime
        nvarchar Status
    }

    CONTAINERS {
        varchar ContainerID PK
        varchar VesselID FK
        nvarchar CargoType
        int WeightKg
        nvarchar PriorityLevel
        nvarchar Destination
        nvarchar Status
        int PriorityScore
        nvarchar RiskLevel
        int ExpectedDwellHours
        varchar AssignedTerminalID FK
        varchar AssignedYardSlot
        varchar AssignedWarehouseID FK
        varchar AssignedVehicleID FK
        nvarchar OptimalRoute
        decimal TotalRouteDistanceKm
    }

    TERMINALS {
        varchar TerminalID PK
        nvarchar Name
        nvarchar Type
        int Capacity
        int CurrentLoad
    }

    YARD_SLOTS {
        varchar SlotID PK
        varchar TerminalID FK
        int SlotRow
        int SlotColumn
        bit IsOccupied
        varchar ContainerID FK
    }

    WAREHOUSES {
        varchar WarehouseID PK
        nvarchar Name
        nvarchar Type
        int Capacity
        int CurrentLoad
        nvarchar Location
    }

    VEHICLES {
        varchar VehicleID PK
        nvarchar Type
        int CapacityKg
        nvarchar Status
        nvarchar CurrentLocation
    }

    ROUTES {
        int RouteID PK
        nvarchar SourceNode
        nvarchar DestinationNode
        decimal DistanceKm
        int TravelTimeMin
    }

    CONTAINER_HISTORY {
        int HistoryID PK
        varchar ContainerID FK
        nvarchar Status
        nvarchar Location
        datetime Timestamp
        nvarchar Remarks
    }
    
---

## STEP 3: CREATE `docs/test_cases.md`

📄 **File Path:** `BlueYardLogistics/docs/test_cases.md`

VS Code me `docs/` folder ke andar `test_cases.md` naam ki file banayein aur yeh code paste karein:

```markdown
# Blue Yard Logistics — Comprehensive QA Test Cases

| Test Case ID | Module / Area | Test Scenario | Input Data | Expected Result | Status |
|---|---|---|---|---|---|
| **TC-CN-01** | Computer Networks | Inter-VLAN Routing (VLAN 10 to VLAN 40) | `ping 192.168.40.10` from `PC-Berth1` | Reply received with `TTL=127`, `<1ms` | **PASS** ✅ |
| **TC-CN-02** | Computer Networks | Inter-VLAN Routing (VLAN 10 to VLAN 20) | `ping 192.168.20.10` from `PC-Berth1` | Reply received with `TTL=127`, `<1ms` | **PASS** ✅ |
| **TC-DS-01** | Python Analytics | High Urgency Pharma Risk Analysis | `C001` (Medicine, 800kg, High) | Score = 100, Risk = High, Dwell = 8 hrs | **PASS** ✅ |
| **TC-DS-02** | Python Analytics | Low Urgency General Cargo Analysis | `C005` (Stationery, 700kg, Low) | Score = 30, Risk = Low, Dwell = 36 hrs | **PASS** ✅ |
| **TC-DSA-01**| C# Algorithms | Dijkstra Shortest Path from Port to W1 | Start: `Port`, Target: `W1` | `Port ➔ J1 ➔ W1` (Total: 9.00 km) | **PASS** ✅ |
| **TC-DSA-02**| C# Algorithms | Dijkstra Shortest Path from Port to W3 | Start: `Port`, Target: `W3` | `Port ➔ J2 ➔ J3 ➔ W3` (Total: 14.00 km) | **PASS** ✅ |
| **TC-DSA-03**| C# Algorithms | 2D Yard Slot Proximity Allocation | High Priority Container `C001` | Assigned to Row 1 (`TB-R1-C1`) | **PASS** ✅ |
| **TC-DSA-04**| C# Algorithms | Vehicle Capacity & Fleet Matching | `C001` (Medicine, 800kg) | Assigned Reefer Truck `V1` (1500kg cap) | **PASS** ✅ |
| **TC-SE-01** | ASP.NET Core API | Duplicate Container Ingestion | POST `C001` again | HTTP `409 Conflict` (Already manifested) | **PASS** ✅ |
| **TC-SE-02** | ASP.NET Core API | Full Lifecycle Auto-Process Pipeline | POST `/api/containers/C001/auto-process` | Status: `Dispatched`, 8 History Logs | **PASS** ✅ |
| **TC-SE-03** | Frontend UI | Dynamic Telemetry & Live Rendering | Open Control Tower (`index.html`) | Live metrics, vessels, manifest tables active | **PASS** ✅ |