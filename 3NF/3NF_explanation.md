# 3NF Transformation Explanation

## 1. Which transitive dependencies did I find?

I checked every 2NF table for non-key attributes that determine other non-key attributes.

| Table | Transitive Dependency | Explanation |
|-------|------------------------|--------------|
| `projects_2NF` | `ProjectID -> SiteCity -> SiteState` | `SiteState` does not depend directly on `ProjectID`. It depends on `SiteCity` (every project located in Boston is in MA, every project in Chicago is in IL, etc.). Since `SiteCity` is a non-key attribute of `projects_2NF` that determines another non-key attribute (`SiteState`), this is a transitive dependency. |

**All other 2NF tables had NO transitive dependencies:**
- `clients_2NF`: `ClientPhone`, `ClientEmail`, `ClientCity` are all independent facts about the client — none of them determines another.
- `supervisors_2NF`: `SupervisorPhone` and `SupervisorCity` are independent; neither determines the other.
- `suppliers_2NF`: only one non-key attribute (`SupplierCity`), so no transitive dependency is possible.
- `workers_2NF`: `WorkerPhone` and `WorkerHourlyRate` are independent.
- `project_workers_2NF`, `project_equipment_2NF`, `project_materials_2NF`, `workers_skills_2NF`, `workers_certifications_2NF`, `suppliers_phones_2NF`: each has at most one non-key attribute depending on the full composite key, so no transitive dependency exists.

## 2. Which non-key attributes depended on other non-key attributes?

In `projects_2NF`, the non-key attribute `SiteCity` determines the non-key attribute `SiteState`. (`SiteAddress` is project-specific and does not repeat across projects, so it stays in `projects_3NF` — it is not the source of a transitive dependency by itself, but the City -> State link is.)

## 3. Which new tables did I create?

| New Table | Purpose |
|-----------|---------|
| `site_locations_3NF.csv` | Holds the `SiteCity -> SiteState` mapping once per city, instead of repeating `SiteState` on every project row. |

`projects_2NF` was changed to `projects_3NF` by **removing the `SiteState` column**. `SiteCity` stays in `projects_3NF` as a foreign key into `site_locations_3NF`.

## 4. How did I use foreign keys to preserve relationships?

| Table | Primary Key | Foreign Key(s) |
|-------|-------------|-----------------|
| `projects_3NF` | ProjectID | SiteCity -> site_locations_3NF.SiteCity, SupervisorName -> supervisors_3NF.SupervisorName |
| `site_locations_3NF` | SiteCity | - |
| `clients_3NF` | ClientName | - |
| `supervisors_3NF` | SupervisorName | - |
| `suppliers_3NF` | SupplierName | - |
| `suppliers_phones_3NF` | (SupplierName, Phone) | SupplierName -> suppliers_3NF.SupplierName |
| `workers_3NF` | WorkerName | - |
| `workers_skills_3NF` | (WorkerName, Skill) | WorkerName -> workers_3NF.WorkerName |
| `workers_certifications_3NF` | (WorkerName, Certification) | WorkerName -> workers_3NF.WorkerName |
| `project_workers_3NF` | (ProjectID, WorkerName) | ProjectID -> projects_3NF.ProjectID, WorkerName -> workers_3NF.WorkerName |
| `project_equipment_3NF` | (ProjectID, Equipment) | ProjectID -> projects_3NF.ProjectID |
| `project_materials_3NF` | (ProjectID, SupplierName, Material) | ProjectID -> projects_3NF.ProjectID, SupplierName -> suppliers_3NF.SupplierName |

No data was lost: joining `projects_3NF` to `site_locations_3NF` on `SiteCity` reconstructs the original `(ProjectID, SiteState)` pairs exactly as they appeared in 2NF.

## 5. Why is your design now in 3NF?

The design is now in **Third Normal Form (3NF)** because:

1. All tables are already in 2NF (no partial dependencies — verified in the 2NF stage).
2. The only transitive dependency found, `ProjectID -> SiteCity -> SiteState` in `projects_2NF`, was removed by extracting `SiteCity -> SiteState` into its own `site_locations_3NF` table.
3. Every remaining non-key attribute in every table depends **only** on its table's primary key, and not on any other non-key attribute.

Therefore every non-key column is now directly (and only) dependent on the whole key of its own table, satisfying the requirements of 3NF.

## Note on data quality (carried forward from earlier stages)

`clients_3NF.csv` has no foreign key linking it to `projects_3NF`. This was already the case in 1NF/2NF — the dataset, as split by the team, never recorded which client owns which project. We also noticed `Pacific Materials` and `Southern Concrete` appear with identical phone/city data in both `clients_3NF.csv` and `suppliers_3NF.csv`, suggesting these two records may have been suppliers misclassified as clients in the original raw file. We did not alter this data since it was inherited from the 1NF/2NF stages already submitted by the group; this is flagged here for the group's review before final BCNF/4NF/PDF write-up.
