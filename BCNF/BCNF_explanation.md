# BCNF Transformation Explanation

## Review of Earlier Stages (1NF → 3NF)

Before BCNF analysis, the 3NF design was checked against `big3_construction_raw_data.csv`. Most of the team's work was correct. Three issues were found and corrected:

| Issue | Problem | Fix applied |
|-------|---------|-------------|
| Missing skill | `Tom Hardy, Concrete` was missing from `workers_skills` (present in raw row P006) | Added to `workers_skills` in 1NF, 2NF, 3NF, BCNF, and 4NF |
| Wrong clients | `Pacific Materials` and `Southern Concrete` were listed in `clients` but they only appear as **suppliers** in the raw file | Removed from `clients` (only 4 real clients remain) |
| Missing project–client link | `ClientName` was never stored on `projects`, so the project–client relationship was lost | Added `ClientName` as a foreign key on `projects` in all stages |

Everything else validated correctly: 15 project–worker rows, 16 material rows, 13 equipment rows, 8 supplier phones, 12 certifications, and all pipe-separated (`|`) values were properly split.

---

## 1. Which functional dependencies did I check?

For **every** 3NF table, I listed all non-trivial functional dependencies (FDs) and asked: *Is the determinant a candidate key (superkey)?*

### `projects_BCNF`
- **PK:** `ProjectID`
- **FDs checked:**
  - `ProjectID → ProjectName, ProjectType, StartDate, EndDate, SiteAddress, SiteCity, ClientName, SupervisorName` — determinant is the PK ✓
  - `SiteCity → SiteState` — already moved to `site_locations_BCNF` in 3NF ✓
  - `ClientName → ClientPhone, ClientEmail, ClientCity` — lives in `clients_BCNF`, not here ✓
  - `SupervisorName → SupervisorPhone` — lives in `supervisors_BCNF` ✓

### `site_locations_BCNF`
- **PK:** `SiteCity`
- **FD:** `SiteCity → SiteState` — determinant is the PK ✓

### `clients_BCNF`
- **PK:** `ClientName`
- **FDs checked:**
  - `ClientName → ClientPhone, ClientEmail, ClientCity` — PK ✓
  - `ClientPhone → ClientName` — alternate candidate key (each phone is unique) ✓
  - `ClientEmail → ClientName` — alternate candidate key ✓

### `supervisors_BCNF`
- **PK:** `SupervisorName`
- **FD:** `SupervisorName → SupervisorPhone` — PK ✓

### `suppliers_BCNF`
- **PK:** `SupplierName`
- **FD:** `SupplierName → SupplierCity` — PK ✓

### `suppliers_phones_BCNF`
- **PK:** (`SupplierName`, `Phone`)
- No non-key attributes — only the composite key ✓

### `workers_BCNF`
- **PK:** `WorkerName`
- **FDs checked:**
  - `WorkerName → WorkerPhone, WorkerHourlyRate` — PK ✓
  - `WorkerPhone → WorkerName, WorkerHourlyRate` — `WorkerPhone` is an **alternate candidate key** (each worker has a unique phone) ✓

### `workers_skills_BCNF` / `workers_certifications_BCNF`
- **PK:** (`WorkerName`, `Skill`) / (`WorkerName`, `Certification`)
- No non-key attributes ✓

### `project_workers_BCNF`
- **PK:** (`ProjectID`, `WorkerName`)
- No non-key attributes ✓

### `project_equipment_BCNF`
- **PK:** (`ProjectID`, `Equipment`)
- **FD:** (`ProjectID`, `Equipment`) → `RentalCost` — full composite key ✓
- **Checked:** `Equipment → RentalCost`? **No** — Crane costs 5000, 5500, 6000, and 6500 on different projects, so equipment alone is not a determinant.

### `project_materials_BCNF`
- **PK:** (`ProjectID`, `SupplierName`, `Material`)
- **FD:** (`ProjectID`, `SupplierName`, `Material`) → `UnitCost` — full composite key ✓
- **Checked:** (`SupplierName`, `Material`) → `UnitCost`? **No** — Steel Beams from SteelWorks cost 500 on P001/P004 but 520 on P007.
- **Checked:** `Material → UnitCost`? **No** — Concrete ranges from 110 to 140 across projects.

---

## 2. Did I find any determinant that was not a candidate key?

**No.** After the 3NF corrections above, every functional dependency in every table has a determinant that is either the primary key or an alternate candidate key.

The closest cases were:
- `WorkerPhone → WorkerHourlyRate` in `workers_BCNF`, but `WorkerPhone` is a valid alternate key.
- Transitive paths like `ProjectID → SiteCity → SiteState` were already broken in 3NF by extracting `site_locations_BCNF`.

---

## 3. Which table or tables violated BCNF?

**None.** No further decomposition was required at the BCNF stage.

---

## 4. How did I decompose the table or tables?

No tables were decomposed. The corrected 3NF schema already satisfies BCNF, so the BCNF CSV files are the same structure as 3NF (with the three data fixes applied).

---

## 5. What candidate keys exist after decomposition?

| Table | Primary Key | Alternate Candidate Keys |
|-------|-------------|--------------------------|
| `projects_BCNF` | `ProjectID` | — |
| `site_locations_BCNF` | `SiteCity` | — |
| `clients_BCNF` | `ClientName` | `ClientPhone`, `ClientEmail` |
| `supervisors_BCNF` | `SupervisorName` | — |
| `suppliers_BCNF` | `SupplierName` | — |
| `suppliers_phones_BCNF` | (`SupplierName`, `Phone`) | — |
| `workers_BCNF` | `WorkerName` | `WorkerPhone` |
| `workers_skills_BCNF` | (`WorkerName`, `Skill`) | — |
| `workers_certifications_BCNF` | (`WorkerName`, `Certification`) | — |
| `project_workers_BCNF` | (`ProjectID`, `WorkerName`) | — |
| `project_equipment_BCNF` | (`ProjectID`, `Equipment`) | — |
| `project_materials_BCNF` | (`ProjectID`, `SupplierName`, `Material`) | — |

---

## 6. Foreign keys in BCNF design

| Table | Foreign Key(s) |
|-------|----------------|
| `projects_BCNF` | `SiteCity → site_locations_BCNF.SiteCity`, `ClientName → clients_BCNF.ClientName`, `SupervisorName → supervisors_BCNF.SupervisorName` |
| `suppliers_phones_BCNF` | `SupplierName → suppliers_BCNF.SupplierName` |
| `workers_skills_BCNF` | `WorkerName → workers_BCNF.WorkerName` |
| `workers_certifications_BCNF` | `WorkerName → workers_BCNF.WorkerName` |
| `project_workers_BCNF` | `ProjectID → projects_BCNF.ProjectID`, `WorkerName → workers_BCNF.WorkerName` |
| `project_equipment_BCNF` | `ProjectID → projects_BCNF.ProjectID` |
| `project_materials_BCNF` | `ProjectID → projects_BCNF.ProjectID`, `SupplierName → suppliers_BCNF.SupplierName` |

---

## 7. Why is the design now in BCNF?

The design is in **Boyce-Codd Normal Form (BCNF)** because:

1. All tables are already in 3NF (no transitive dependencies remain within any table).
2. For every non-trivial functional dependency **X → Y** in every table, **X is a superkey** (either the declared primary key or a proven alternate candidate key).
3. No table required further decomposition.

BCNF is stricter than 3NF. We verified this stricter rule table by table rather than assuming 3NF implies BCNF.
