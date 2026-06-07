# African Bird Migration — Flyway Threat Intersection (Phase 3)

> **Phase 3 of the corridor analysis workflow.** Spatially intersects the 30
> threatened species priority points with the Phase 1 flyway corridor polygons
> to compute a weighted threat score per corridor, and identifies critical
> stopover nodes where 2 or more threatened species converge geographically.

---

## Project Overview

| Property | Value |
|---|---|
| **Project file** | `African_Bird_Flyway_Threat_Intersection.qgz` |
| **CRS** | EPSG:4326 — WGS 84 |
| **Threat points** | 30 (3 threatened species × 10 priority points) |
| **Stopover sites tested** | 96 |
| **Flyway corridors scored** | 6 |
| **Critical convergence nodes** | 19 of 31 threatened-species stopovers |
| **Convergence radius** | 1.5° (~165 km) |
| **Priority weights** | High = 1.0 · Medium = 0.5 |

This is **Phase 3** built on top of Phase 1 corridor zones. It introduces the
**threat dimension** — answering both *which corridors carry the most
threatened species pressure* and *where multiple threatened species converge*.

## Folder Structure

```
African_Bird_Flyway_Threat_Intersection/
├── African_Bird_Flyway_Threat_Intersection.qgz   (27.6 KB)
├── README.md
├── Input_layers/
│   ├── flyway_corridor_zones.gpkg          (Phase 1 output, used as input)
│   ├── migration_routes.gpkg               (27 lines · context)
│   ├── stopover_sites.gpkg                 (96 points · proximity input)
│   └── threatened_species_priority.gpkg    (30 points · primary input)
└── Output_layer/
    ├── corridor_threat_scores.gpkg         (6 polygons · 14 attributes)
    └── critical_threat_nodes.gpkg          (31 points · 11 attributes)
```

## Methodology

### Part A — Per-Corridor Weighted Threat Score

For each Phase 1 corridor polygon, count the threatened species priority
points contained within it, weighted by priority level:

```
weighted_score = (n_high   × 1.0) + (n_medium × 0.5)
threat_density = weighted_score / area_deg2
```

Then classify each corridor by its weighted score normalised against the
network-wide maximum:

```
normalised = weighted_score / max_score_across_all_corridors

Class:
  Critical  if normalised >= 0.70
  High      if normalised >= 0.45
  Moderate  if normalised >= 0.20
  Low       if normalised >  0.00
  None      if no threatened points present
```

### Part B — Critical Stopover Nodes (Multi-Species Convergence)

Filter the 96 stopovers to only those used by threatened species (31 stopovers).
For each, find every other threatened-species stopover within 1.5° (~165 km),
count the **distinct threatened species** in the resulting cluster:

```
For each threatened stopover S:
    nearby_spp = {S.species}
    For each other threatened stopover T:
        if euclidean_distance(S, T) <= 1.5°:
            nearby_spp.add(T.species)
    is_critical = (len(nearby_spp) >= 2)
    threat_score = len(nearby_spp) * 0.50 + n_neighbours * 0.05
```

A stopover is flagged **critical** when 2+ distinct threatened species
converge there — these are the conservation high-priority sites where
protection benefits multiple species simultaneously.

### Output Attribute Schemas

**corridor_threat_scores.gpkg:**

| Field | Type | Description |
|---|---|---|
| `flyway` | String | Flyway name |
| `route_count` | Integer | Routes in corridor (Phase 1 carry-over) |
| `length_km` | Real | Dissolved-route length km (Phase 1) |
| `area_deg2` | Real | Buffer polygon area in deg² (Phase 1) |
| `n_threat_pts` | Integer | Threatened priority points contained |
| `n_threat_species` | Integer | Distinct threatened species present |
| `n_high_priority` | Integer | High-priority points |
| `n_medium_priority` | Integer | Medium-priority points |
| `n_breeding` | Integer | Breeding-type threat points |
| `n_wintering` | Integer | Wintering-type threat points |
| `weighted_threat_score` | Real | (n_high × 1.0) + (n_medium × 0.5) |
| `threat_density` | Real | Score per square degree |
| `threat_class` | String | Critical / High / Moderate / Low / None |
| `species_present` | String | Comma-separated threatened species in corridor |

**critical_threat_nodes.gpkg:**

| Field | Type | Description |
|---|---|---|
| `fid_orig` | Integer | Original FID from stopover layer |
| `species` | String | Threatened species using this stopover |
| `location` | String | Site location name |
| `flyway` | String | Flyway affiliation |
| `habitat` | String | Habitat type |
| `duration_d` | Real | Stopover duration (days) |
| `is_critical` | Integer | 1 if 2+ threatened species converge within 1.5° |
| `n_overlap_spp` | Integer | Distinct threatened species in 1.5° cluster |
| `overlap_spp` | String | Comma-separated converging species |
| `n_neighbours` | Integer | Other threatened stopovers within 1.5° |
| `threat_score` | Real | (n_overlap × 0.50) + (n_neighbours × 0.05) |

## Results

### Per-Corridor Threat Intersection

| Flyway | Pts | Spp | High | Med | WScore | Class |
|---|---|---|---|---|---|---|
| **Mediterranean Flyway** | 13 | 3 | 5 | 8 | **9.00** | 🔴 Critical |
| **East African Flyway** | 8 | 3 | 5 | 3 | **6.50** | 🔴 Critical |
| Central African Flyway | 3 | 1 | 0 | 3 | 1.50 | 🟢 Low |
| Atlantic Flyway | 0 | 0 | 0 | 0 | 0.00 | ⚪ None |
| Sahara Flyway | 0 | 0 | 0 | 0 | 0.00 | ⚪ None |
| West African Flyway | 0 | 0 | 0 | 0 | 0.00 | ⚪ None |

### Critical Convergence Nodes (Top by Score)

| Location | Species | Convergence | Score |
|---|---|---|---|
| **Sudan** | All 3 | 3 species | 1.65 |
| **Tanzania** | All 3 | 3 species | 1.60 |
| **South Africa** | All 3 | 3 species | 1.60 |
| Egypt | 2 species | 2 species | 1.10 |
| Uganda | 2 species | 2 species | 1.10 |
| Morocco | 2 species | 2 species | 1.10 |

### Key Findings

- **Mediterranean Flyway carries the highest threat load** (weighted 9.00) —
  13 priority points spread across all 3 threatened species, with 5 High-
  priority and 8 Medium-priority. This makes Mediterranean the apex threat
  corridor in the network.

- **East African Flyway is also Critical** (weighted 6.50) — 8 points across
  all 3 threatened species. Combined with Mediterranean, these two corridors
  account for 21 of the 30 threatened priority points (70%).

- **Three corridors carry zero threatened priority points** — Atlantic,
  Sahara, and West African. From a threat-intersection perspective these
  are conservation-resilient corridors. Whether this reflects genuine low
  threat or absence of monitoring data is worth flagging.

- **Sudan, Tanzania, and South Africa are 3-species convergence hotspots** —
  these locations host stopovers for ALL three threatened species within
  165 km of each other. Protecting these regions delivers conservation
  benefit to every threatened species in the dataset simultaneously —
  the highest-leverage sites in the network.

- **19 of 31 threatened-species stopovers are critical convergence nodes**
  (61%). The convergence is geographically clustered — the same 3 East
  African (Sudan, Tanzania, South Africa) plus Egypt, Uganda, and Morocco
  in the north account for almost all critical nodes.

- **High-priority weighting matters.** Mediterranean and East African both
  carry 5 High-priority points each — this drives most of their threat
  score. If priority weights were equal (1.0 for both), corridor rankings
  would be the same but absolute scores would compress.

## How to Reproduce

### PyQGIS Pseudocode

```python
PRIORITY_WEIGHT = {'High': 1.0, 'Medium': 0.5}

# Part A: Per-corridor weighted threat score
corridor_threat = {fw: {'pts':[], 'spp':set(), 'weighted_sum':0.0}
                   for fw in zones}
for p in threat_pts:
    for fw, polygon in zones.items():
        if polygon.contains(p.geometry()):
            corridor_threat[fw]['pts'].append(p)
            corridor_threat[fw]['spp'].add(p['species'])
            corridor_threat[fw]['weighted_sum'] += PRIORITY_WEIGHT[p['priority']]
            break

# Part B: Multi-species convergence nodes
PROX_DEG = 1.5
threat_stops = [s for s in stops if s['species'] in threatened_species]
for s in threat_stops:
    nearby_spp = {s['species']}
    for t in threat_stops:
        if t.fid != s.fid and dist(s, t) <= PROX_DEG:
            nearby_spp.add(t['species'])
    s.is_critical    = (len(nearby_spp) >= 2)
    s.n_overlap_spp  = len(nearby_spp)
    s.threat_score   = len(nearby_spp) * 0.50 + n_neighbours * 0.05
```

### Critical Replication Notes

- **Use polygon.contains(point), break on first match** — corridor polygons
  can overlap slightly at flyway transitions; without the early break a
  threat point near a boundary would be counted twice.
- **Convergence is computed over threatened stopovers only**, not all 96.
  The semantic question is *do multiple threatened species share this
  region?* — non-threatened stopovers don't contribute to that signal.
- **1.5° proximity is intentionally generous** — matches the DBSCAN cluster
  radius used in the Community Hotspots analysis. Tightening to 1.0° (~110
  km) would reduce convergence counts but produce more spatially precise
  hotspots; 2.0° would broaden them into regional zones.
- **Priority weights are a policy choice.** High = 1.0, Medium = 0.5 follows
  the conventional 2:1 ratio. Field assessments may justify alternative
  weights — change `PRIORITY_WEIGHT` and rerun.
- **Class thresholds are normalised against this dataset's max** (Mediterranean,
  weighted 9.00). To compare across studies, switch to absolute thresholds
  (e.g., score ≥ 8 = Critical).

## File Inventory

| File | Folder | Size | Description |
|---|---|---|---|
| `African_Bird_Flyway_Threat_Intersection.qgz` | Root | 27.6 KB | QGIS project |
| `README.md` | Root | — | This file |
| `flyway_corridor_zones.gpkg` | `Input_layers/` | 144.0 KB | Phase 1 output (input here) |
| `migration_routes.gpkg` | `Input_layers/` | 104.0 KB | 27 route lines (context) |
| `stopover_sites.gpkg` | `Input_layers/` | 116.0 KB | 96 stopover sites |
| `threatened_species_priority.gpkg` | `Input_layers/` | 96.0 KB | 30 priority points (primary input) |
| `corridor_threat_scores.gpkg` | `Output_layer/` | 144.0 KB | 6 corridors with weighted threat |
| `critical_threat_nodes.gpkg` | `Output_layer/` | 104.0 KB | 31 stopovers with convergence flag |

---

*African Bird Migration — Flyway Threat Intersection (Phase 3)*
*Priority weights: High=1.0, Medium=0.5  ·  Convergence radius: 1.5° (~165 km)*
*Built on Phase 1 corridor zones (50 km buffered, dissolved by flyway)*
*QGIS 3.40.14-Bratislava  ·  PyQGIS spatial-intersection + convergence pipeline*