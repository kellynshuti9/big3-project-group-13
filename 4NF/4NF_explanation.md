# 4NF Transformation Explanation

## 1. Which multi-valued dependencies did I identify?

In the **raw** `big3_construction_raw_data.csv`, several independent multi-valued dependencies (MVDs) existed. A multi-valued dependency **X →→ Y** means: for a given X value, there is a set of Y values that is independent of other multi-valued facts about X.

| Determinant | Multi-valued attribute(s) | Example from raw data |
|-------------|---------------------------|------------------------|
| `WorkerName` | `Skill` | Mike Ross: `Carpentry\|Framing`, later also `Concrete` |
| `WorkerName` | `Certification` | Mike Ross: `OSHA\|First Aid` (independent list from skills) |
| `SupplierName` | `Phone` | BuildPro Supplies: `617-555-9000\|617-555-9001` |
| `ProjectID` (per row context) | `Material` + `UnitCost` | P001: `Concrete\|Steel` with `120\|300` |
| `ProjectID` (per row context) | `Equipment` + `RentalCost` | P001: `Crane\|Bulldozer` with `5000\|3000` |

The most important 4NF violations in the raw file were **pairs of independent MVDs stored in the same row**, for example:

- `WorkerName →→ Skill` **and** `WorkerName →>> Certification` in the same wide table
- `SupplierName →→ Phone` mixed with `SupplierName → SupplierCity` (single-valued)
- `ProjectID →→ Materials` **and** `ProjectID →→ Equipment` in the same row

---

## 2. Which independent lists were mixed together?

### Raw table (violates 4NF)
One row stored all of these at once:
- Project facts (name, dates, site)
- Client facts (name, phone, email)
- Supervisor facts
- One worker + phone + rate
- **Multiple skills** and **multiple certifications** (two independent lists for the same worker)
- Supplier + **multiple phones**
- **Multiple materials** with costs
- **Multiple equipment** items with rental costs

### Specific independent pairs

| Mixed in raw row | Why independent |
|------------------|-----------------|
| `WorkerSkills` and `WorkerCertifications` | A worker's skills and certifications are unrelated lists — knowing "Carpentry" does not determine "OSHA" |
| `MaterialSupplied` and `EquipmentUsed` | Materials supplied and equipment used on a project are separate facts |
| `SupplierPhones` and `MaterialSupplied` | A supplier's phone numbers do not determine which materials they supply on a given project |

---

## 3. How did you separate them?

Most separation was done progressively in **1NF** (atomic values) and carried forward. The BCNF design already places each independent multi-valued fact in its own relationship table:

| Independent MVD | Resolved in table | Since stage |
|-----------------|-------------------|-------------|
| `WorkerName →→ Skill` | `workers_skills_4NF` | 1NF |
| `WorkerName →→ Certification` | `workers_certifications_4NF` | 1NF |
| `SupplierName →→ Phone` | `suppliers_phones_4NF` | 1NF |
| `ProjectID →→ (SupplierName, Material, UnitCost)` | `project_materials_4NF` | 1NF |
| `ProjectID →→ (Equipment, RentalCost)` | `project_equipment_4NF` | 1NF |
| `ProjectID →→ WorkerName` | `project_workers_4NF` | 1NF |

**No additional decomposition was needed at the 4NF stage** because skills and certifications are no longer in the same table, supplier phones are not stored with materials, and project materials are not stored with project equipment.

The `workers_4NF` table holds only single-valued worker facts (`WorkerPhone`, `WorkerHourlyRate`). It does not mix skills and certifications.

---

## 4. What are the primary keys of the new relationship tables?

| Relationship table | Primary key | What it represents |
|--------------------|-------------|-------------------|
| `workers_skills_4NF` | (`WorkerName`, `Skill`) | One row per skill a worker has |
| `workers_certifications_4NF` | (`WorkerName`, `Certification`) | One row per certification a worker holds |
| `suppliers_phones_4NF` | (`SupplierName`, `Phone`) | One row per phone number for a supplier |
| `project_workers_4NF` | (`ProjectID`, `WorkerName`) | One row per worker assigned to a project |
| `project_materials_4NF` | (`ProjectID`, `SupplierName`, `Material`) | One row per material supplied to a project |
| `project_equipment_4NF` | (`ProjectID`, `Equipment`) | One row per equipment item used on a project |

Entity tables (`projects_4NF`, `clients_4NF`, `workers_4NF`, `suppliers_4NF`, `supervisors_4NF`, `site_locations_4NF`) each have a single-attribute primary key and hold only single-valued facts.

---

## 5. 4NF check on each BCNF table

For a table to violate 4NF, there must be a non-trivial MVD **X →→ Y** where X is **not** a superkey, and Y is not a subset of X.

| Table | MVDs checked | Violation? |
|-------|--------------|------------|
| `workers_skills_4NF` | `WorkerName →→ Skill` — but `WorkerName` is not the PK; however this is a **relationship table** where the full key is (`WorkerName`, `Skill`). Skills are not mixed with any second independent list. | No |
| `workers_certifications_4NF` | Same pattern — certifications separated from skills | No |
| `suppliers_phones_4NF` | Phones separated from supplier city and from materials | No |
| `project_materials_4NF` | Only material facts; equipment is in a separate table | No |
| `project_equipment_4NF` | Only equipment facts; materials are in a separate table | No |
| `project_workers_4NF` | Only worker assignments; skills/certs are in separate tables | No |
| All entity tables | No multi-valued columns remain | No |

---

## 6. Why is the final design in 4NF?

The design is in **Fourth Normal Form (4NF)** because:

1. All tables are already in BCNF (verified in the BCNF stage).
2. Every independent multi-valued fact from the raw file has been placed in its **own** relationship table with a composite primary key.
3. No table stores two or more **independent** multi-valued lists for the same determinant (the main 4NF violation pattern in the original data).
4. No further decomposition was required — the 4NF CSV files match the corrected BCNF/3NF structure.

### Example: why splitting skills and certifications matters

If both skills and certifications stayed in one table like:

| WorkerName | Skill | Certification |
|------------|-------|---------------|
| Mike Ross | Carpentry | OSHA |
| Mike Ross | Carpentry | First Aid |
| Mike Ross | Framing | OSHA |
| Mike Ross | Framing | First Aid |

This creates spurious combinations (every skill paired with every certification). Separating into `workers_skills_4NF` and `workers_certifications_4NF` removes that artificial cross-product and correctly represents two independent multi-valued facts about each worker.

---

## 7. Foreign keys in 4NF design

Same as BCNF — relationships are preserved through foreign keys:

- `projects_4NF.ClientName → clients_4NF.ClientName`
- `projects_4NF.SiteCity → site_locations_4NF.SiteCity`
- `projects_4NF.SupervisorName → supervisors_4NF.SupervisorName`
- `workers_skills_4NF.WorkerName → workers_4NF.WorkerName`
- `workers_certifications_4NF.WorkerName → workers_4NF.WorkerName`
- `suppliers_phones_4NF.SupplierName → suppliers_4NF.SupplierName`
- `project_workers_4NF` → `projects_4NF` + `workers_4NF`
- `project_materials_4NF` → `projects_4NF` + `suppliers_4NF`
- `project_equipment_4NF` → `projects_4NF`

No data is lost: joining these tables reconstructs all facts from the original raw CSV.
