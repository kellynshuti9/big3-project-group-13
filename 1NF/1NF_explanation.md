# 1NF Transformation Explanation

## 1. Which columns violated 1NF?

These columns had multi-valued fields (values separated by `|`):

 **WorkerSkills**: e.g., `"Carpentry|Framing"`
 **WorkerCertifications**: e.g., `"OSHA|First Aid"`
 **SupplierPhones**: e.g., `"617-555-9000|617-555-9001"`
 **MaterialSupplied**: e.g., `"Concrete|Steel"`
 **MaterialUnitCost**: e.g., `"120|300"`
 **EquipmentUsed**: e.g., `"Crane|Bulldozer"`
 **EquipmentRentalCost**: e.g., `"5000|3000"`

Also, the raw file had repeating rows (same ProjectID multiple times) and mixed entities all in one table.


## 2. How did I make the values atomic?

I:

1. **Separated entities** into standalone tables:
   - `projects_1NF.csv`, `clients_1NF.csv`, `supervisors_1NF.csv`, `workers_1NF.csv`, `suppliers_1NF.csv`

2. **Created new tables** for multi-valued attributes:
    `workers_skills_1NF.csv` – one row per (worker, skill)
    `workers_certifications_1NF.csv` – one row per (worker, certification)
    `suppliers_phones_1NF.csv` – one row per (supplier, phone)
    `project_materials_1NF.csv` – one row per (project, supplier, material)
    `project_equipment_1NF.csv` – one row per (project, equipment)
    `project_workers_1NF.csv` – one row per (project, worker)

3. **Split cells with `|`** into multiple rows:
   - `"Carpentry|Framing"` -- 2 rows
   - `"617-555-9000|617-555-9001"` -- 2 rows

No cell now contains `|`.


## 3. Did I create new rows, new tables, or both?

**Both.**

 **11 new tables total**
 Every multi-valued cell became **multiple rows** in new tables


## 4. What key identifies each row?

### Primary Keys

| Table                      | Primary Key                                 |
|----------------------------|---------------------------------------------|
| `projects_1NF.csv`         | `ProjectID`                                 |
| `clients_1NF.csv`          | `ClientName`                                |
| `supervisors_1NF.csv`      | `SupervisorName`                            |
| `workers_1NF.csv`          | `WorkerName`                                |
| `suppliers_1NF.csv`        | `SupplierName`                                |
| `workers_skills_1NF.csv`   | (`WorkerName`, `Skill`)                     |
| `workers_certifications_1NF.csv` | (`WorkerName`, `Certification`)         |
| `suppliers_phones_1NF.csv` | (`SupplierName`, `Phone`)                   |
| `project_materials_1NF.csv` | (`ProjectID`, `SupplierName`, `Material`) |
| `project_equipment_1NF.csv` | (`ProjectID`, `Equipment`)                |
| `project_workers_1NF.csv`   | (`ProjectID`, `WorkerName`)               |

## Foreign Keys

 `workers_skills_1NF.WorkerName` → `workers_1NF.WorkerName`
 `workers_certifications_1NF.WorkerName` → `workers_1NF.WorkerName`
 `suppliers_phones_1NF.SupplierName` → `suppliers_1NF.SupplierName`
 `project_materials_1NF.ProjectID` → `projects_1NF.ProjectID`
 `project_equipment_1NF.ProjectID` → `projects_1NF.ProjectID`
 `project_workers_1NF.ProjectID` → `projects_1NF.ProjectID`
 `project_workers_1NF.WorkerName` → `workers_1NF.WorkerName`
 `project_workers_1NF.SupervisorName` → `supervisors_1NF.SupervisorName`


## Correction: duplicate rows removed

`project_equipment_1NF.csv` originally had two exact duplicate rows under the key `(ProjectID, Equipment)`: `P004,Lift,2000` and `P007,Excavator,4200` each appeared twice. This broke the 1NF requirement that each row be uniquely identified, so the duplicates were removed (no data lost — the repeated rows held identical values). Propagated through `2NF/project_equipment_2NF.csv` and `3NF/project_equipment_3NF.csv` as well.

`workers_skills_1NF.csv` and `workers_certifications_1NF.csv` also had many exact duplicate `(WorkerName, Skill)` / `(WorkerName, Certification)` rows (e.g. `Tom Hardy,Welding` appeared 3 times, `Mike Ross,Framing` appeared twice). These also broke the 1NF uniqueness requirement and were deduplicated down to one row per distinct pair, with no data lost. Propagated through the corresponding `2NF` and `3NF` files as well.

## Notes for 2NF Teammate

Tables with **composite keys** (check for partial dependencies):

 `workers_skills_1NF`
 `workers_certifications_1NF`
 `suppliers_phones_1NF`
 `project_materials_1NF`
 `project_equipment_1NF`
 `project_workers_1NF`

Single-key tables are already in 2NF:

 `projects_1NF`, `clients_1NF`, `supervisors_1NF`, `workers_1NF`, `suppliers_1NF`


## Notes for 3NF Teammate

Look for **transitive dependencies** (non-key → non-key) in all tables.