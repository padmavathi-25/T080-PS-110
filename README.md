PS-110: Cloud Kitchen Multi-brand Order Routing and Kitchen Capacity Engine
Project Overview
PS-110 is a command-file-driven backend engine for a multi-brand cloud-kitchen platform. It models how customer orders from different virtual brands are assigned to physical kitchens, scheduled across kitchen stations, and accounted for financially.
The project is developed in Python 3.10+. It does not require a graphical user interface.
Main Features
Register kitchens, stations, station capacities, and operating hours.
Register brands, menus, recipes, preparation times, and kitchen associations.
Read customer orders from CSV files.
Route orders to eligible kitchens using projected ready time, current load, and kitchen ID.
Schedule station work with capacity limits, batching, and item dependencies.
Handle RUSH orders, stock-outs, substitutions, cancellations, and hand-off events.
Calculate ready times, promised times, channel commissions, packaging charges, penalties, and brand-kitchen rent.
Generate routing decisions, station timelines, and end-of-day brand P&L reports.
Technology
Language: Python 3.10+
Input format: CSV command/data files
Testing: Python `unittest` (recommended for implementation)
Interface: Command line; no UI
Repository Structure
```text
PS-110_Cloud_Kitchen_Project/
├── README.md
├── sample_data/
│   ├── kitchens.csv
│   ├── brands.csv
│   ├── brand_kitchens.csv
│   ├── zones.csv
│   ├── menus.csv
│   ├── substitutions.csv
│   ├── orders.csv
│   └── events.csv
└── tests/
    ├── unit_test_cases.csv
    └── boundary_test_cases.csv
```
Sample Data Files
File	Purpose
`kitchens.csv`	Kitchen details, operating hours, stations, and station capacity
`brands.csv`	Registered virtual brands
`brand_kitchens.csv`	Kitchens permitted to prepare each brand's orders
`zones.csv`	Delivery-zone kitchen list and delivery duration
`menus.csv`	Menu items, required stations, prep time, batch size, and price
`substitutions.csv`	Brand-level item substitution mapping
`orders.csv`	Sample customer orders
`events.csv`	Stock-out, cancellation, and hand-off events
Order item format
The `items` column in `orders.csv` uses this format:
```text
item_id:quantity
```
For example, `I01:2` means two units of menu item `I01`.
Business Rules
The implementation must follow the 12 business rules in the project specification:
BR-01 – Routing eligibility: Check brand-kitchen association, delivery-zone eligibility, operating hours, and required stations. Select by lowest projected ready time, then lowest current load, then lowest kitchen ID.
BR-02 – Station capacity and batching: Respect station slots. Identical recipes placed within 3 minutes may share a slot up to the recipe batch size.
BR-03 – Order sequencing: Use FIFO except for RUSH orders. A RUSH order may jump ahead of up to three orders, and an order cannot be delayed by more than two jumps.
BR-04 – Ready time: Use the latest item completion time, respecting dependencies, then add 3 minutes for packaging.
BR-05 – Promised time: Add zone delivery minutes and a 5-minute buffer to ready time, then round up to the next 5-minute boundary. Apply the 45-minute late-risk and alternate-kitchen rule.
BR-06 – Load shedding: Mark a kitchen THROTTLED when projected load exceeds 60 minutes; clear throttling when it falls below 40 minutes.
BR-07 – Stock-outs: Use an allowed brand substitution when its price is within ₹20 of the original; otherwise remove the item and reprice the order. Cancel if no items remain.
BR-08 – Cancellation: Before preparation starts, cancellation is free. After preparation starts, charge for started items, applying the channel-cancellation rule where relevant.
BR-09 – Channel economics: Aggregator A commission is 22%, Aggregator B is 18%, and the own app has 0% commission plus a ₹15 payment fee. Packaging is ₹10 per item, capped at ₹40. Food GST is 5%.
BR-10 – Penalties: Charge ₹50 for every started 10 minutes late, capped at ₹200. If lateness is caused by THROTTLED routing, charge the kitchen cost centre instead of the brand.
BR-11 – Brand-kitchen rent: Charge ₹2 per order plus 3% of net food value. Apply the ₹500 minimum when the brand has fewer than 30 orders at that kitchen in a day.
BR-12 – Processing precedence: Eligibility → throttling → routing → sequencing → batching → dependency scheduling → promised time. Apply substitution before scheduling.
Unit and Boundary Test Cases
The `tests/` directory contains test-case specifications for the main business rules and the boundary conditions listed in the project statement. These CSV files describe the input scenario and expected result. They are test specifications, not executable Python test modules.
When implementing the engine, convert each case into a Python `unittest.TestCase` test and assert the expected result against the relevant function or module.
Running the Project
The sample CSV files and test specifications are provided as project input/reference data. Once the Python engine is implemented, a typical command-line execution can be exposed as:
```bash
python main.py --data-dir sample_data
```
Run executable unit tests with:
```bash
python -m unittest discover -s tests -v
```
The command above will run Python test modules added to `tests/`. The CSV test specifications are not run directly by `unittest`.
Expected Outputs
The completed engine should generate:
Routing decisions for each order.
Per-order ready time and promised time.
Station-wise preparation timelines.
Order status and flags such as `RUSH`, `LATE_RISK`, and `THROTTLED`.
Per-order economics and end-of-day brand P&L.
Notes
All sample records are illustrative and are intended for development and testing.
The final implementation must use the official sample data and visible expected outputs supplied for the project evaluation.
Boundary conditions must be handled consistently, especially exact time limits and cancellation at preparation start.
