# 2NF Transformation Explanation

## 1. Which tables had composite keys?

The following tables from 1NF had composite primary keys:

| Table | Composite Key |
|-------|---------------|
| `project_workers_1NF` | (ProjectID, WorkerName) |
| `project_equipment_1NF` | (ProjectID, Equipment) |
| `project_materials_1NF` | (ProjectID, SupplierName, Material) |
| `workers_skills_1NF` | (WorkerName, Skill) |
| `workers_certifications_1NF` | (WorkerName, Certification) |
| `suppliers_phones_1NF` | (SupplierName, Phone) |

## 2. Which columns depended only on part of a composite key?

| Table | Partial Dependency | Why it's a partial dependency |
|-------|-------------------|-------------------------------|
| `project_workers_1NF` | `SupervisorName` depends on `ProjectID` | Each project has only ONE supervisor. SupervisorName does NOT depend on WorkerName. So SupervisorName depends on only PART of the composite key (ProjectID, WorkerName). |

**All other composite key tables had NO partial dependencies:**
- `project_equipment_1NF`: RentalCost depends on BOTH ProjectID AND Equipment (same equipment can have different rental costs for different projects)
- `project_materials_1NF`: UnitCost depends on ALL THREE (ProjectID, SupplierName, Material)
- `workers_skills_1NF`: No non-key attributes
- `workers_certifications_1NF`: No non-key attributes
- `suppliers_phones_1NF`: No non-key attributes

## 3. Which data did I move into separate tables?

| Original Table | Change Made |
|----------------|-------------|
| `project_workers_1NF` | Removed `SupervisorName` column |
| `projects_1NF` | Added `SupervisorName` column |

## 4. What primary keys and foreign keys did I introduce?

| Table | Primary Key | Foreign Key(s) |
|-------|-------------|----------------|
| `projects_2NF` | ProjectID | SupervisorName → supervisors_2NF.SupervisorName |
| `project_workers_2NF` | (ProjectID, WorkerName) | ProjectID → projects_2NF.ProjectID, WorkerName → workers_2NF.WorkerName |
| `project_equipment_2NF` | (ProjectID, Equipment) | ProjectID → projects_2NF.ProjectID |
| `project_materials_2NF` | (ProjectID, SupplierName, Material) | ProjectID → projects_2NF.ProjectID, SupplierName → suppliers_2NF.SupplierName |
| `workers_skills_2NF` | (WorkerName, Skill) | WorkerName → workers_2NF.WorkerName |
| `workers_certifications_2NF` | (WorkerName, Certification) | WorkerName → workers_2NF.WorkerName |
| `suppliers_phones_2NF` | (SupplierName, Phone) | SupplierName → suppliers_2NF.SupplierName |
| `workers_2NF` | WorkerName | - |
| `suppliers_2NF` | SupplierName | - |
| `supervisors_2NF` | SupervisorName | - |
| `clients_2NF` | ClientName | - |

## 5. Why is your design now in 2NF?

The design is now in **Second Normal Form (2NF)** because:

1. ✅ All tables are already in 1NF (Mathew ensured atomic values and no repeating groups).
2. ✅ There are **no partial dependencies**:
   - I identified that `SupervisorName` depended only on `ProjectID` in the `project_workers_1NF` table.
   - I removed `SupervisorName` from `project_workers_1NF` and added it to `projects_1NF`.
   - All other composite key tables either had no non-key attributes or their non-key attributes depended on the **entire** composite key.

Therefore, every non-key attribute now depends on the **whole** primary key (or composite key), satisfying the requirements of 2NF.