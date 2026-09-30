# 💳 Microsoft 365 License Recharge Dashboard

A Power BI dashboard that automates the **quarterly recharge of Microsoft 365 and other software licence costs** to the departments that use them. Built for a **pan-African telecommunications group**, it turns raw Entra ID licence assignments into an auditable cost-allocation model for finance and IT across **20+ subsidiary entities**.

`Power BI` `DAX` `Power Query` `Entra ID` `Data Modelling`

> 🔒 Client work: the client, entities and all figures are confidential. No screenshots or data are published, and the DAX below uses generic table names.

---

## 🎯 The problem

Every quarter, the group has to recharge software licence costs to the departments and subsidiaries that actually use them. This dashboard combines licence assignments, product unit pricing and department mapping into one model that produces:

- A self-service view of **total spend by software product**
- **Departmental cost allocation** across 20+ entities
- A finance-ready **recharge schedule** for sign-off
- A **SKU-level breakdown per department** for audits and billing queries

---

## 📊 Dashboard pages

| Page | What it shows |
|---|---|
| **Quarter Summary** | KPIs (licensed users, total quarterly recharge in ZAR, E5 licence count, line items billed) plus spend by product (donut) and by department (bar) |
| **Recharge Schedule** | Per product: active licences, USD unit price, USD and ZAR totals and % of total. Also spend by category (Productivity & Collaboration, Specialised / Data & Development, Security / Infrastructure & Creative) |
| **SKU Breakdown** | Stacked bars showing which SKUs make up each department's recharge |

---

## 🧩 Data model

| Table | Role |
|---|---|
| **Licence_Assignments** | Fact table: employee ID, department, department group, person type, licence, licence cost (ZAR) |
| **Employee_Department** | Employee → department mapping |
| **Software_Licensing_Costs** | Product → unit cost (USD) |
| **Product_Summary** | Calculated table of standardised product names with active counts |

**Modelling notes**

- Separates **assigned licences** from **licence-group membership**, so users on bundled SKUs aren't counted twice
- A **Department Group** column rolls individual country entities up into consolidated reporting groups
- A configurable USD → ZAR exchange rate

---

## 📈 Key DAX

**Licence cost (ZAR):** maps raw licence names to standard product names, looks up the unit cost and converts it to a quarterly ZAR amount

```dax
License Cost (ZAR) =
VAR _ExchangeRate = 18
RETURN
    SUMX(
        'Licence_Assignments',
        VAR _RawProduct = 'Licence_Assignments'[Licence]
        VAR _MappedProduct =
            SWITCH(
                TRUE(),
                _RawProduct = "GitHub", "GitHub User",
                _RawProduct = "Microsoft 365 E5", "Microsoft 365 E5s",
                _RawProduct = "Microsoft 365 E5 Nopstnconf", "Microsoft 365 E5s",
                _RawProduct = "Office 365 E1", "Microsoft Office 365 E1",
                _RawProduct = "Powerapps Per User", "PowerApps Per User",
                _RawProduct = "Project Online Essentials", "Microsoft Project Essentials",
                _RawProduct = "Project Online Premium", "Microsoft Project Premium",
                _RawProduct = "Project Online Professional", "Microsoft Project Professional",
                _RawProduct
            )
        VAR _UnitCost =
            LOOKUPVALUE(
                'Software_Licensing_Costs'[Unit Cost],
                'Software_Licensing_Costs'[Software Product], _MappedProduct
            )
        RETURN
            _UnitCost * 3 * _ExchangeRate   -- 3 months per quarter
    )
```

**Product summary:** a calculated table built with `UNION` and `ROW` that standardises product names and counts active licences (shortened here)

```dax
Product Summary = UNION(
    ROW("Software Product", "Microsoft 365 E5s",
        "Active Count", COUNTROWS(FILTER('Licence_Assignments',
            'Licence_Assignments'[Licence] IN { "Microsoft 365 E5", "Microsoft 365 E5 Nopstnconf" })) + 0),
    ROW("Software Product", "Microsoft 365 Copilot",
        "Active Count", COUNTROWS(FILTER('Licence_Assignments',
            'Licence_Assignments'[Licence] = "Microsoft 365 Copilot")) + 0),
    ROW("Software Product", "Microsoft Visio",
        "Active Count", COUNTROWS(FILTER('Licence_Assignments',
            'Licence_Assignments'[Licence] = "Microsoft Visio")) + 0)
    -- ...one ROW per product
)
```

---

## 🛠️ Tech stack

- **Power BI:** data model, DAX measures and report pages
- **Entra ID:** source of licence assignment data
- **Power Query:** transformation and department mapping
- **HTML / Chart.js:** a lightweight single-page version for easy sharing

---

## 🚀 Roadmap

- [ ] Azure Function to automate the quarterly refresh from Entra ID
- [ ] Blob Storage snapshots for each recharge period
- [ ] Row-level security, so each department head sees only their own recharge
- [ ] Historical trends across quarters

---

## 👩🏾‍💻 Author

**Sharon Galela** · [LinkedIn](https://www.linkedin.com/in/sharon-galela-6998bb265) · [GitHub](https://github.com/ShariieG)
