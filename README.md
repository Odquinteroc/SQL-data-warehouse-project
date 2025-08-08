flowchart TD
  %% Layout
  classDef header fill:#ffffff,stroke:#888,stroke-width:1px,color:#222,font-weight:bold
  classDef card fill:#f8fbff,stroke:#7ba7d9,stroke-width:2px,color:#111
  classDef lane fill:#f5f5f5,stroke:#c8c8c8,stroke-width:1px,color:#111
  classDef bronze fill:#efe6d6,stroke:#c7b191,stroke-width:1px,color:#111
  classDef silver fill:#e6e8ea,stroke:#a9aeb3,stroke-width:1px,color:#111
  classDef gold fill:#fcf1c8,stroke:#d9c27a,stroke-width:1px,color:#111
  classDef tbl fill:#fff,stroke:#d0d0d0,stroke-width:1px,color:#111

  %% Title (visual only)
  title["SkyLoom — Data Integration"]

  %% Sources
  subgraph S["Sources"]
    direction LR

    subgraph Helio["HelioCRM (Source)"]
      direction TB
      H1["crm_people (person_id PK, full_name, email, phone, city)"]:::tbl
      H2["crm_orgs (org_id PK, org_name, segment, region)"]:::tbl
      H3["crm_events (event_id PK, org_id FK, person_id FK, kind, event_date)"]:::tbl
    end
    class Helio card

    subgraph Casc["CascadeERP (Source)"]
      direction TB
      E1["erp_items (item_id PK, item_name, category, unit_price)"]:::tbl
      E2["erp_orders (order_id PK, org_id FK, order_date, status)"]:::tbl
      E3["erp_order_lines (line_id PK, order_id FK, item_id FK, qty, unit_price)"]:::tbl
      E4["erp_vendors (vendor_id PK, vendor_name, country)"]:::tbl
    end
    class Casc card
  end

  %% Landing & ETL
  LZ["Landing Zone (Raw CSV/JSON)"]:::lane
  ETL["Transform & Validate (ETL)\nParse → Cleanse → Conform → Deduplicate"]:::lane

  %% Data Warehouse
  subgraph DW["Data Warehouse"]
    direction LR

    subgraph BR["Bronze (raw_*) — load"]
      direction TB
      BR1["raw_crm_people"]:::tbl
      BR2["raw_crm_orgs"]:::tbl
      BR3["raw_crm_events"]:::tbl
      BR4["raw_erp_items"]:::tbl
      BR5["raw_erp_orders"]:::tbl
      BR6["raw_erp_order_lines"]:::tbl
      BR7["raw_erp_vendors"]:::tbl
    end
    class BR bronze

    subgraph SV["Silver (dim_*, fact_*) — model & standardize"]
      direction TB
      SV1["dim_org(org_key, org_id, name, segment, region)"]:::tbl
      SV2["dim_person(person_key, person_id, full_name, email, city)"]:::tbl
      SV3["dim_item(item_key, item_id, item_name, category)"]:::tbl
      SV4["dim_date(date_key, date, month, year)"]:::tbl
      SV5["fact_order(order_key, order_id, org_key, date_key, total_amount)"]:::tbl
      SV6["fact_order_line(line_key, order_key, item_key, qty, unit_price)"]:::tbl
    end
    class SV silver

    subgraph GD["Gold (curated marts) — publish"]
      direction TB
      GD1["Sales Insights (views)\n- v_daily_sales\n- v_top_items"]:::tbl
      GD2["CRM Activity (views)\n- v_touches_by_segment\n- v_events_timeline"]:::tbl
    end
    class GD gold
  end

  %% Flows
  H1 -->|ingest| LZ
  H2 -->|ingest| LZ
  H3 -->|ingest| LZ
  E1 -->|ingest| LZ
  E2 -->|ingest| LZ
  E3 -->|ingest| LZ
  E4 -->|ingest| LZ

  LZ -->|batch/stream| ETL
  ETL --> BR

  BR -->|model & standardize| SV
  SV -->|publish| GD

  %% Styling for group labels
  class S header
  class DW header
