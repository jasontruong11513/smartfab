# 🏭 SmartFab

### Multi-Factory Manufacturing Planning & Scheduling Platform

SmartFab is a manufacturing planning and scheduling prototype that transforms production demand into constraint-aware production schedules across multiple factories.

The platform automates the planning workflow from demand input to job generation, routing, factory assignment, production scheduling, downtime handling, deadline validation, and interactive schedule visualization.

<p align="center">
  <strong>Manufacturing Planning • Production Scheduling • Factory Allocation • Operations Analytics</strong>
</p>

---

## 📌 Overview

Manufacturing planning requires coordinating multiple operational factors including:

- Product demand
- Lot sizes
- Manufacturing routings
- Factory capabilities
- Production-line availability
- Processing times
- Changeover requirements
- Planned downtime
- Operational interruptions
- Delivery deadlines

SmartFab combines these elements into a single automated planning workflow.

Given production demand data, the system generates manufacturing jobs, determines their required process flows, assigns them to compatible factories, schedules each operation across eligible production lines, and produces a visual production plan.

The project was developed as a portfolio prototype to explore practical applications of production planning, manufacturing analytics, scheduling heuristics, and multi-factory decision support.

---

## ✨ Key Features

### 📥 Demand Processing

Upload production demand data in CSV format containing product requirements and delivery dates.

The system uses this demand information as the starting point for the planning pipeline.

---

### ⚙️ Automated Job Generation

SmartFab converts product-level demand into individual manufacturing jobs.

Job quantities are generated based on:

- Product demand
- Product-specific lot sizes
- Required delivery dates

This allows high-level demand requirements to be translated into executable manufacturing jobs.

---

### 🔄 Process Routing

Each manufacturing job is assigned a sequence of required production processes.

The routing engine determines which manufacturing steps must be completed and preserves process precedence throughout scheduling.

Example:

```text
Wafer Saw
    ↓
Die Attach
    ↓
Wire Bond
    ↓
Molding
    ↓
Final Inspection
```

---

### 🏭 Factory Assignment

Products are assigned to factories according to manufacturing requirements and available production capabilities.

The current implementation uses deterministic rule-based assignment logic.

Factory assignments are validated before scheduling begins to ensure that all required manufacturing processes are available within the selected facility.

---

### ✅ Factory Feasibility Validation

Before building a production schedule, SmartFab verifies that every required process for a job can be completed within its assigned factory.

If a required process is unavailable, the workflow stops and reports the issue rather than allowing the job to move between factories unexpectedly.

This ensures that each job remains within one assigned manufacturing facility throughout its production routing.

---

### 📅 Production Scheduling

SmartFab generates operation-level production schedules across available production lines.

The scheduling engine considers:

- Assigned factory
- Required manufacturing process
- Process precedence
- Production-line capability
- Production-line availability
- Processing duration
- Product changeover time
- Planned downtime
- Fixed blocked periods
- Job deadlines

For each production operation, the scheduler identifies eligible production lines within the assigned factory and selects a feasible line based on earliest completion time.

---

### 🔁 Changeover Handling

Production lines may require setup or transition time when switching between products.

SmartFab incorporates changeover time according to product relationships.

Supported changeover categories include:

- Same product
- Same product family
- Different product family

The applicable changeover duration is added before the next production operation begins.

---

### 🛠 Planned Downtime

The scheduling engine supports planned downtime windows for manufacturing lines.

Planned downtime can be assigned according to:

- Manufacturing process
- Preferred maintenance periods
- Minimum spacing between downtime events
- Production-line availability

These downtime periods are treated as unavailable windows during scheduling.

---

### ⚠️ Operational Downtime

SmartFab can also simulate additional operational downtime using expected operating intervals and repair durations.

This allows the final schedule to reflect production interruptions beyond the original baseline schedule.

---

### ⏰ Deadline Validation

After scheduling is completed, SmartFab compares the completion time of each manufacturing job against its required delivery deadline.

Jobs that exceed their deadline are recorded along with:

- Lateness in minutes
- Lateness in hours

This provides visibility into potential delivery risks.

---

### 📊 Interactive Gantt Visualization

The final production plan can be visualized using interactive Gantt charts generated with Plotly.

The visualization provides a timeline view of:

- Jobs
- Manufacturing processes
- Production lines
- Start times
- Completion times
- Factory assignments

This makes it easier to understand machine utilization and production flow across the planning horizon.

---

### 🌐 REST API

SmartFab includes a FastAPI backend that provides endpoints for interacting with the planning pipeline.

The API supports:

- Demand file upload
- Automated workflow execution
- Local web interface
- API documentation through Swagger

---

# 🏗 System Architecture

```text
                    Production Demand
                           │
                           ▼
                ┌────────────────────┐
                │   Job Generator    │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ Routing Generator  │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ Factory Assignment │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ Feasibility Check  │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ Production         │
                │ Scheduler          │
                └─────────┬──────────┘
                          │
                 ┌────────┴────────┐
                 │                 │
                 ▼                 ▼
       Planned / Operational     Deadline
             Downtime           Validation
                 │                 │
                 └────────┬────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ Final Schedule     │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ Gantt Visualization│
                └─────────┬──────────┘
                          │
                          ▼
                   Production Plan
```

---

# 🧠 Scheduling Methodology

SmartFab currently uses deterministic planning rules and scheduling heuristics.

It does **not** currently use machine learning or a mathematical optimization solver.

The goal of the scheduling engine is to generate a feasible production plan while respecting manufacturing constraints.

---

## 1. Factory Assignment

Products are assigned to factories according to their manufacturing requirements.

The current implementation uses predefined process requirements to determine the appropriate facility.

For example, products requiring specialized processes may be routed to a factory containing the required manufacturing capability.

---

## 2. Factory Feasibility Check

Before scheduling begins, the system verifies that the assigned factory supports every process required by each job.

Conceptually:

```text
Required Processes
        │
        ▼
Assigned Factory
        │
        ▼
Available Processes
        │
        ▼
Feasible?
   │         │
  Yes        No
   │         │
Schedule    Stop + Report
```

This validation prevents jobs from silently moving between factories during scheduling.

---

## 3. Candidate Production Lines

For each operation, SmartFab identifies production lines satisfying both conditions:

```text
Correct Manufacturing Process
            +
Correct Assigned Factory
            =
Eligible Production Lines
```

Only eligible lines are considered during scheduling.

---

## 4. Changeover Calculation

Before assigning an operation to a production line, the scheduler determines the required changeover time.

The transition depends on the previously processed product.

```text
Previous Product
       │
       ├── Same Product
       │
       ├── Same Product Family
       │
       └── Different Product Family
```

The applicable changeover duration is included before production begins.

---

## 5. Earliest Feasible Start

For each eligible line, the scheduler calculates:

```text
Earliest Start =
max(
    Previous Process Completion,
    Production Line Availability
)
+
Changeover Time
```

The resulting start time is then adjusted to avoid unavailable production windows.

---

## 6. Downtime and Blocked Windows

If a scheduled operation overlaps with a blocked period or planned downtime window, its start time is moved forward.

The production duration itself is preserved.

Example:

```text
Requested Production

────────────████████────────────

Blocked Window

────────████────────────────────

Adjusted Production

──────────────████████──────────
```

---

## 7. Production-Line Selection

The scheduler evaluates all eligible production lines for the operation.

For each candidate:

```text
Start Time
    +
Processing Duration
    =
Completion Time
```

The feasible line with the earliest completion time is selected.

---

## 8. Process Precedence

Manufacturing operations must occur in their required sequence.

For example:

```text
Process 1
   │
   ▼
Process 2
   │
   ▼
Process 3
```

A downstream operation cannot begin before the previous process for the same job has been completed.

---

## 9. Final Validation

After scheduling, SmartFab verifies that:

- Each job remains within one factory
- Processing durations are preserved
- Production-line assignments are valid
- Required processes are supported
- Job completion times can be compared against deadlines

---

# 🔄 Planning Workflow

## Step 1 — Load Demand

Production demand is read from the input CSV file.

Input information includes:

```text
Product
Demand Quantity
Delivery Date
```

---

## Step 2 — Generate Manufacturing Jobs

Product demand is divided into manufacturing jobs based on product lot size.

Example:

```text
Demand: 2,500 units
Lot Size: 1,000 units

↓

Job 1: 1,000
Job 2: 1,000
Job 3: 1,000
```

The current prototype generates jobs according to the configured lot size logic.

---

## Step 3 — Generate Process Routes

Each job receives the production process sequence required for its product.

---

## Step 4 — Assign Factory

The system determines which factory is responsible for manufacturing the product.

---

## Step 5 — Validate Factory Capabilities

The assigned factory is checked against all manufacturing processes required by the job.

---

## Step 6 — Build Baseline Schedule

Production operations are scheduled across eligible lines according to:

- Process requirements
- Factory assignment
- Processing time
- Line availability
- Changeovers
- Blocked windows
- Planned downtime

---

## Step 7 — Apply Operational Downtime

Additional production interruptions can be incorporated into the baseline schedule.

---

## Step 8 — Validate Delivery Deadlines

Final job completion times are compared against delivery requirements.

---

## Step 9 — Generate Outputs

SmartFab creates CSV datasets containing planning and scheduling results.

---

## Step 10 — Visualize Production Schedule

The resulting production schedule can be displayed using an interactive Gantt chart.

---

# ⚡ Technology Stack

| Category | Technology |
|---|---|
| Programming Language | Python |
| Backend | FastAPI |
| Application Server | Uvicorn |
| Data Processing | Pandas |
| Visualization | Plotly |
| Input Data | CSV |
| Intermediate Storage | CSV |
| API | REST |
| Interface | HTML |
| Development Environment | Local Python Environment |

---

# 📂 Project Structure

```text
smartfab-ai/
│
├── app.py
├── index.html
├── requirements.txt
│
├── Engine/
│   ├── job_generator.py
│   ├── routing_generator.py
│   ├── factory_assignment.py
│   ├── scheduler.py
│   └── gantt_chart.py
│
├── data/
│   ├── demand.csv
│   ├── factories_capacities.csv
│   ├── changeover.csv
│   ├── downtime.csv
│   └── Produc_Master_file.csv
│
├── database/
│   ├── generated_jobs.csv
│   ├── job_process_flow.csv
│   ├── product_factory_assignment.csv
│   ├── schedule_baseline.csv
│   ├── schedule_final.csv
│   ├── planned_downtime_schedule.csv
│   ├── debug_schedule_candidates.csv
│   └── deadline_violations.csv
│
└── gantt_chart.html
```

---

# 📊 Data Flow

```text
demand.csv
    │
    ▼
generated_jobs.csv
    │
    ▼
job_process_flow.csv
    │
    ▼
product_factory_assignment.csv
    │
    ▼
schedule_baseline.csv
    │
    ▼
schedule_final.csv
    │
    ├────────► deadline_violations.csv
    │
    └────────► gantt_chart.html
```

---

# 📡 API Endpoints

## Health Check

```http
GET /
```

Example response:

```json
{
  "message": "SmartFab API is running"
}
```

---

## Run Planning Pipeline

```http
POST /run
```

Input format:

```text
multipart/form-data
```

Uploaded input:

```text
demand.csv
```

Example response:

```json
{
  "status": "success",
  "message": "Pipeline completed"
}
```

---

## Web Interface

```http
GET /ui
```

Opens the SmartFab web interface.

---

# 🛠 Running SmartFab Locally

## 1. Clone the Repository

```bash
git clone https://github.com/jasontruong11513/smartfab-ai.git
cd smartfab-ai
```

---

## 2. Create a Virtual Environment

```bash
python -m venv venv
```

### Windows

```bash
venv\Scripts\activate
```

### macOS / Linux

```bash
source venv/bin/activate
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

## 4. Start the FastAPI Application

```bash
uvicorn app:app --reload
```

---

## 5. Open the Application

Local application:

```text
http://127.0.0.1:8000
```

Swagger API documentation:

```text
http://127.0.0.1:8000/docs
```

---

> **Note:** SmartFab is currently designed and tested for local execution. Public cloud deployment is not part of the current working implementation.

---

# 📈 Generated Outputs

The planning pipeline generates several intermediate and final datasets.

### `generated_jobs.csv`

Contains manufacturing jobs generated from production demand.

---

### `job_process_flow.csv`

Contains the process routing required by each manufacturing job.

---

### `product_factory_assignment.csv`

Stores factory assignments for individual products.

---

### `schedule_baseline.csv`

Contains the initial production schedule before additional operational downtime is applied.

---

### `schedule_final.csv`

Contains the final production schedule after downtime adjustments.

---

### `planned_downtime_schedule.csv`

Contains scheduled downtime windows for production lines.

---

### `debug_schedule_candidates.csv`

Stores candidate scheduling information used during schedule generation and validation.

---

### `deadline_violations.csv`

Contains jobs that complete after their required delivery deadline.

---

### `gantt_chart.html`

Provides an interactive visualization of the resulting production schedule.

---

# 🎯 Project Applications

SmartFab explores concepts relevant to:

- Manufacturing Planning
- Production Scheduling
- Multi-Factory Production Planning
- Factory Allocation
- Capacity Planning
- Manufacturing Analytics
- Operations Management
- Production Control
- Supply Chain Decision Support
- Smart Manufacturing
- Digital Manufacturing

---

# 📚 Concepts Demonstrated

This project demonstrates practical implementation of several operations and information systems concepts.

### Manufacturing Systems

- Process routing
- Production capacity
- Factory capability
- Lot-based production
- Production-line assignment

### Operations Planning

- Job scheduling
- Production sequencing
- Changeover management
- Downtime planning
- Deadline management

### Software Development

- Modular Python architecture
- Data processing pipelines
- REST APIs
- Automated workflow execution
- Data validation
- Interactive visualization

### Analytics

- Schedule generation
- Deadline analysis
- Production timeline visualization
- Manufacturing data transformation

---

# ⚠️ Current Limitations

SmartFab is an educational and portfolio prototype rather than a production-grade manufacturing planning system.

The current implementation has several limitations:

- Factory assignment uses predefined deterministic rules
- Production scheduling uses scheduling heuristics
- The scheduler does not guarantee a globally optimal production plan
- Manufacturing capacity is represented using simplified assumptions
- Labor constraints are not currently modeled
- Material availability is not currently modeled
- Inventory constraints are not currently modeled
- Production costs are not currently included in the scheduling objective
- Intermediate data is stored primarily using CSV files
- Real-time shop-floor information is not incorporated
- ERP and MES integrations are not currently implemented
- The application is currently intended for local execution

These limitations define opportunities for future development.

---

# 🚀 Future Improvements

Potential future enhancements include:

### Mathematical Optimization

Introduce optimization models using techniques such as:

- Linear Programming
- Mixed-Integer Linear Programming
- Constraint Programming

These approaches could support explicit production objectives such as minimizing:

- Job lateness
- Changeover time
- Machine idle time
- Production cost
- Overall makespan

---

### Dynamic Factory Allocation

Expand factory assignment logic to consider:

- Available capacity
- Current factory workload
- Delivery deadlines
- Manufacturing costs
- Production lead times

---

### Advanced Capacity Modeling

Add support for:

- Shift schedules
- Workforce capacity
- Machine utilization limits
- Production calendars
- Overtime
- Shared manufacturing resources

---

### Material and Inventory Constraints

Integrate:

- Raw material availability
- Work-in-process inventory
- Component availability
- Safety stock requirements

---

### Scenario Analysis

Allow planners to compare multiple production scenarios.

Examples:

```text
What happens if demand increases by 20%?

What happens if a production line goes offline?

What happens if a delivery deadline changes?

What happens if production is moved to another factory?
```

---

### ERP / MES Integration

Future versions could integrate with enterprise manufacturing systems for:

- Production orders
- Inventory information
- Equipment status
- Factory capacity
- Work-in-process tracking

---

### Real-Time Production Monitoring

Connect scheduling results to live production data to compare:

```text
Planned Schedule
       vs.
Actual Production
```

---

### Cloud Deployment

Future development may include deployment using a cloud platform and persistent database infrastructure.

---

### Advanced Planning Dashboard

Expand the interface to provide:

- Production KPIs
- Line utilization
- Factory utilization
- Late-job alerts
- Capacity summaries
- Bottleneck identification
- Schedule comparison

---

# 🧩 Design Philosophy

SmartFab is built around a simple principle:

> **Convert manufacturing demand into a transparent and traceable production plan.**

Rather than treating scheduling as a black box, the system generates intermediate datasets at each stage of the planning process.

This makes it possible to inspect:

```text
Demand
  ↓
Jobs
  ↓
Routing
  ↓
Factory Assignment
  ↓
Scheduling
  ↓
Deadline Validation
  ↓
Visualization
```

Each stage can be evaluated independently, making the planning workflow easier to understand, debug, and extend.

---

# 📌 Project Status

SmartFab is currently a working local prototype developed for educational and portfolio purposes.

Current functionality includes:

- ✅ Demand processing
- ✅ Manufacturing job generation
- ✅ Process routing
- ✅ Factory assignment
- ✅ Factory feasibility validation
- ✅ Production-line scheduling
- ✅ Changeover handling
- ✅ Planned downtime handling
- ✅ Operational downtime simulation
- ✅ Deadline validation
- ✅ CSV output generation
- ✅ Interactive Gantt visualization
- ✅ FastAPI backend
- ✅ Local web interface

Current development scope:

- ❌ Production cloud deployment
- ❌ ERP integration
- ❌ MES integration
- ❌ Real-time factory data
- ❌ Global mathematical optimization

---

# 👨‍💻 Author

**Jason Truong**

Master of Science in Information Systems Candidate  
California State University, Long Beach

Interests:

- Manufacturing Analytics
- Production Planning
- Supply Chain Analytics
- Data Science
- Operations Research
- Information Systems

---

# 📄 License

This project is provided for educational and portfolio purposes.

---

<p align="center">
  <strong>SmartFab</strong><br>
  Multi-Factory Manufacturing Planning & Scheduling Platform
</p>
