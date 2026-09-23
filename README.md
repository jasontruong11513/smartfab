# 🏭 SmartFab

**Multi-Factory Manufacturing Planning & Scheduling Platform**

SmartFab is a manufacturing planning prototype that transforms production demand data into constraint-aware production schedules through an automated planning pipeline.

The system generates production jobs, creates process routings, assigns products to compatible factories, schedules operations across production lines, accounts for changeovers and downtime, validates delivery deadlines, and visualizes the resulting production plan using interactive Gantt charts.

> **Project Status:** Portfolio and educational prototype. The application is designed and tested for local execution and is not currently deployed as a production service.

---

## 🚀 Overview

Manufacturing planning requires coordinating demand, product routings, factory capabilities, production lines, changeovers, downtime, and delivery requirements.

SmartFab integrates these planning steps into a single automated workflow.

Given production demand data, the platform:

* Generates manufacturing jobs based on demand and lot sizes
* Builds process routings for individual jobs
* Assigns products to factories based on manufacturing requirements
* Validates factory and process compatibility
* Schedules operations across eligible production lines
* Accounts for processing times and product changeovers
* Incorporates planned and operational downtime
* Checks production schedules against job deadlines
* Generates interactive Gantt charts for schedule visualization

The project explores manufacturing planning, production scheduling, and decision support in a multi-factory production environment.

---

## 🎯 Key Features

### 📁 Demand Input

Accept production demand data in CSV format and use it as the starting point for the planning workflow.

### ⚙️ Automated Job Generation

Convert product demand into individual manufacturing jobs based on product-specific lot sizes and required delivery dates.

### 🔄 Process Routing

Generate the required manufacturing process sequence for each production job.

### 🏭 Factory Assignment

Assign products to factories according to manufacturing requirements and available factory capabilities.

The current implementation uses deterministic rule-based assignment logic.

### 📅 Production Scheduling

Generate production schedules while considering:

* Factory assignment
* Process sequence
* Production-line availability
* Processing time
* Product changeover time
* Planned downtime
* Fixed blocked periods
* Process compatibility
* Job deadlines

For each operation, the scheduler evaluates eligible production lines within the assigned factory and selects a feasible line based on completion time.

### 🛠 Planned Downtime Scheduling

Generate planned downtime windows for production lines while considering process-specific scheduling preferences and minimum spacing between downtime periods.

### ⚠️ Deadline Validation

Compare the final completion time of each job against its required deadline and identify deadline violations.

### 📊 Interactive Gantt Visualization

Visualize production schedules, line utilization, and operation timelines through interactive Plotly Gantt charts.

### 🌐 REST API

Use a FastAPI backend to trigger the planning pipeline and interact with the application locally.

---

## 🏗 System Architecture

```text
Production Demand
       │
       ▼
┌──────────────────┐
│  Job Generator   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Routing Generator│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│Factory Assignment│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│Feasibility Check │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Production       │
│ Scheduler        │
└────────┬─────────┘
         │
         ├──────────────► Deadline Validation
         │
         ▼
┌──────────────────┐
│ Gantt Generator  │
└────────┬─────────┘
         │
         ▼
   Production Plan

🧠 Scheduling Logic

SmartFab currently uses deterministic planning rules and scheduling heuristics.

Factory Assignment

Products are assigned to factories according to their required manufacturing processes and factory capabilities.

The assignment logic ensures that production requirements are compatible with the selected facility.

Factory Feasibility Validation

Before scheduling begins, SmartFab verifies that all processes required by each job can be performed within the job's assigned factory.

If a required process is unavailable, the planning workflow stops and reports the feasibility issue instead of allowing the job to switch factories.

Production-Line Selection

For each production operation, the scheduler:

Identifies production lines capable of performing the required process
Restricts candidates to the job's assigned factory
Calculates applicable product changeover time
Determines the earliest possible start time
Accounts for blocked periods and planned downtime
Calculates the resulting completion time
Selects the feasible line with the earliest completion time
Schedule Constraints

The scheduling workflow considers:

Process precedence
Factory compatibility
Production-line availability
Processing duration
Product-family changeovers
Fixed blocked time windows
Planned downtime
Delivery deadlines

Additional validation ensures that each job remains within its assigned factory throughout its production routing.

⏱ Changeover Handling

SmartFab incorporates changeover time when consecutive products are processed on the same production line.

Changeover requirements can vary depending on whether the next job belongs to:

The same product
The same product family
A different product family

These transition times are incorporated before the next production operation begins.

🛠 Downtime Handling

The scheduling engine accounts for multiple forms of production downtime.

Fixed Blocked Windows

Predefined time periods can be blocked from production scheduling.

Planned Downtime

Production lines can receive planned downtime windows based on process-specific time preferences and minimum spacing requirements.

Operational Downtime

Additional downtime can be incorporated into the generated schedule based on expected operating intervals and repair durations.

⚡ Technology Stack
Backend
Python
FastAPI
Uvicorn
Data Processing
Pandas
Visualization
Plotly
Data Storage
CSV-based input, intermediate, and output datasets
API
REST API
File upload support
Automated planning workflow
📂 Project Structure
smartfab/
│
├── app.py
├── requirements.txt
├── index.html
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
│   └── downtime.csv
│
├── database/
│   ├── generated_jobs.csv
│   ├── job_process_flow.csv
│   ├── product_factory_assignment.csv
│   ├── schedule_baseline.csv
│   ├── schedule_final.csv
│   ├── planned_downtime_schedule.csv
│   └── deadline_violations.csv
│
└── gantt_chart.html
🔄 Planning Workflow
1. Load Production Demand

The workflow begins with product demand quantities and required delivery dates.

2. Generate Production Jobs

Demand quantities are divided into individual manufacturing jobs according to product-specific lot sizes.

3. Generate Process Routings

Each job receives the manufacturing process sequence required for its product.

4. Assign Factories

Products are assigned to compatible factories based on their manufacturing requirements.

5. Validate Factory Feasibility

The system verifies that every required process can be performed within the assigned factory before scheduling begins.

6. Schedule Production

Operations are assigned to eligible production lines while accounting for process precedence, availability, changeovers, blocked periods, and planned downtime.

7. Apply Operational Downtime

Additional expected downtime is incorporated into the baseline production schedule.

8. Validate Deadlines

The final completion time of each job is compared with its required delivery deadline.

9. Visualize the Schedule

The completed production plan is displayed through an interactive Gantt chart.

📡 API Endpoints
Health Check
GET /

Example response:

{
  "message": "SmartFab API is running"
}
Run Planning Pipeline
POST /run

Input:

multipart/form-data
demand.csv

Example response:

{
  "status": "success",
  "message": "Pipeline completed"
}
Web Interface
GET /ui

Opens the local SmartFab interface.

🛠 Running Locally
Clone the Repository
git clone <your-repository-url>
cd smartfab
Create a Virtual Environment
python -m venv venv

Activate the environment on Windows:

venv\Scripts\activate

On macOS/Linux:

source venv/bin/activate
Install Dependencies
pip install -r requirements.txt
Start the Application
uvicorn app:app --reload

Local application:

http://127.0.0.1:8000

Swagger API documentation:

http://127.0.0.1:8000/docs

SmartFab is currently designed and tested for local execution. Public cloud deployment is not part of the current implementation.

📈 Generated Outputs

The planning pipeline generates intermediate and final datasets such as:

generated_jobs.csv
job_process_flow.csv
product_factory_assignment.csv
schedule_baseline.csv
schedule_final.csv
planned_downtime_schedule.csv
deadline_violations.csv
gantt_chart.html

These outputs make the planning workflow traceable from production demand through factory assignment, scheduling, and deadline validation.

🎓 Applications

SmartFab explores concepts related to:

Manufacturing Planning
Production Scheduling
Multi-Factory Production Planning
Factory Allocation
Capacity Planning
Manufacturing Analytics
Operations Management
Supply Chain Decision Support
Smart Manufacturing
⚠️ Current Limitations

SmartFab is a portfolio prototype and does not represent a production-grade manufacturing planning system.

The current implementation:

Uses rule-based factory assignment
Uses heuristic production scheduling
Uses CSV files rather than a production database
Does not guarantee a globally optimal production schedule
Uses simplified representations of manufacturing capacity and operating conditions
Does not currently integrate with ERP or MES systems
Is designed and tested primarily for local execution

These limitations provide opportunities for future development.

🔮 Future Improvements

Potential extensions include:

Linear and mixed-integer programming for production optimization
Dynamic multi-factory allocation
More detailed production capacity constraints
Labor and workforce constraints
Inventory availability constraints
Material availability constraints
Scenario and what-if analysis
Production cost modeling
ERP and MES integration
Real-time shop-floor data integration
Predictive maintenance
Cloud deployment
Advanced production planning dashboards
👨‍💻 Author

Jason Truong

Master of Science in Information Systems Candidate
California State University, Long Beach

📄 License

This project is provided for educational and portfolio purposes
