graph TD
    %% Sources
    subgraph HelioCRM
        H1[crm_people] --> LZ
        H2[crm_orgs] --> LZ
        H3[crm_events] --> LZ
    end

    subgraph CascadeERP
        E1[erp_items] --> LZ
        E2[erp_orders] --> LZ
        E3[erp_order_lines] --> LZ
        E4[erp_vendors] --> LZ
    end

    %% Landing Zone and ETL
    LZ[Landing Zone (Raw CSV/JSON)] --> ETL[Transform & Validate (ETL)]

    %% Data Warehouse
    ETL --> BR[Bronze Layer\n(raw_* tables)]
    BR --> SV[Silver Layer\n(dim_*, fact_* tables)]
    SV --> GD[Gold Layer\n(Curated Marts)]

    %% Bronze Tables
    BR --> BR1[raw_crm_people]
    BR --> BR2[raw_crm_orgs]
    BR --> BR3[raw_crm_events]
    BR --> BR4[raw_erp_items]
    BR --> BR5[raw_erp_orders]
    BR --> BR6[raw_erp_order_lines]
    BR --> BR7[raw_erp_vendors]

    %% Silver Tables
    SV --> SV1[dim_org]
    SV --> SV2[dim_person]
    SV --> SV3[dim_item]
    SV --> SV4[dim_date]
    SV --> SV5[fact_order]
    SV --> SV6[fact_order_line]

    %% Gold Tables
    GD --> GD1[Sales Insights]
    GD --> GD2[CRM Activity]
