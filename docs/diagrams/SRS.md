# Software Requirements Specification (SRS)
## Project: Blue Yard Logistics — Intelligent Maritime Logistics Platform

---

## 1. Introduction
### 1.1 Purpose
Blue Yard Logistics is an integrated maritime cargo flow and terminal operating optimization platform designed to manage cargo manifest intake, perform AI-driven risk scoring, optimize physical 2D yard storage, dynamically match destination warehouses and transport vehicles, and compute shortest delivery routes via Dijkstra's algorithm.

### 1.2 System Scope
The system integrates four core computing disciplines:
- **Software Engineering:** Monolithic modular architecture with ASP.NET Core Web API, SQL Server ORM, and high-performance glass UI.
- **Introduction to Data Science:** Python FastAPI microservice utilizing historical cargo profiles, priority scoring formulas, and linear regression dwell-time estimation.
- **Data Structures & Algorithms:** Custom Binary Max-Heap Priority Queue, 2D Matrix Yard Slot Optimizer, and Graph Dijkstra Shortest Path Router.
- **Computer Networks:** 4-VLAN network segmentation in Cisco Packet Tracer with Router-on-a-Stick Inter-VLAN Routing.

---

## 2. Overall Description
### 2.1 System Architecture
[Client / UI Layer]
│ (HTTP Fetch)
▼
[ASP.NET Core Web API] ──► [C# DSA Engine: Heap, 2D Grid, Dijkstra]
│
├──► [Python FastAPI Microservice: Dwell Regression & Scoring]
└──► [SQL Server: BlueYardDB]

### 2.2 User Characteristics
- **Port Operations Master:** Observes real-time vessel berthing and triggers manifest analysis.
- **Yard Manager:** Inspects 2D grid slot allocations across Terminals Alpha, Beta, and Gamma.
- **Dispatch Coordinator:** Reviews vehicle payload allocation and computed road routes.

---

## 3. Specific Functional Requirements

### FR-01: Vessel & Manifest Intake
- **FR-01.1:** System shall record vessel berthing events with timestamp, origin, and status.
- **FR-01.2:** System shall ingest container manifest metadata (ID, Cargo Type, Weight in Kg, Declared Urgency, Destination).

### FR-02: AI Cargo Risk & Dwell-Time Intelligence
- **FR-02.1:** Backend shall invoke the Python analytics microservice (`POST /analyze`).
- **FR-02.2:** Python engine shall calculate a normalized Priority Score (1–100) based on cargo sensitivity, declared urgency, and weight.
- **FR-02.3:** System shall predict expected dwell time in hours via trained linear regression.

### FR-03: Multi-Criteria Terminal Allocation
- **FR-03.1:** C# DSA scoring engine shall evaluate terminal suitability:
  - Pharma / Food ➔ Terminal Beta (Priority/Cold-Chain)
  - Electronics ➔ Terminal Gamma (Special Cargo)
  - General Goods ➔ Terminal Alpha (General Cargo)
- **FR-03.2:** System shall verify available capacity before incrementing load counter.

### FR-04: 2D Yard Grid Storage Optimization
- **FR-04.1:** Yard slots shall be modeled as a 2D matrix (`[Row, Column]`).
- **FR-04.2:** High-priority cargo shall be assigned to Row 1 (Gate-proximity fast-track slots).
- **FR-04.3:** Normal/Low priority cargo shall be assigned to Row 2 (Buffer storage slots).

### FR-05: Capacity-Constrained Fleet & Warehouse Matching
- **FR-05.1:** System shall allocate specialized vehicles:
  - Perishable / Medicine ➔ Reefer Truck
  - Heavy / General ➔ Flatbed / Heavy Hauler
- **FR-05.2:** Vehicle allocation shall enforce `CapacityKg >= WeightKg` and availability constraints.
- **FR-05.3:** Specialized warehouse matching shall balance facility capacity and destination proximity.

### FR-06: Graph Construction & Dijkstra Route Optimization
- **FR-06.1:** Road network shall be constructed dynamically as a weighted directional/bi-directional graph.
- **FR-06.2:** System shall execute Dijkstra's algorithm from origin `Port` to assigned target warehouse.
- **FR-06.3:** System shall persist optimal node path, cumulative distance in kilometers, and estimated travel time in minutes.

### FR-07: Full Audit Traceability & Dispatch
- **FR-07.1:** Every state transition (`Manifested` ➔ `Analyzed` ➔ `TerminalAssigned` ➔ `InYard` ➔ `WarehouseAssigned` ➔ `VehicleAssigned` ➔ `RouteOptimized` ➔ `Dispatched`) shall log an immutable timestamped event to `ContainerHistory`.

---

## 4. Non-Functional Requirements (NFR)
- **NFR-01 (Performance):** Complete automated pipeline execution (`POST /auto-process`) shall complete within < 300ms.
- **NFR-02 (Reliability):** System shall prevent race conditions and over-allocation of yard slots or fleet capacity.
- **NFR-03 (Security & Network Isolation):** Port zones shall be segregated across distinct broadcast domains (VLANs 10, 20, 30, 40) using Cisco 802.1Q encapsulation.
- **NFR-04 (Maintainability):** Loose coupling between ASP.NET Core orchestrator, Python analytics microservice, and pure C# DSA algorithms.