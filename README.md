# 🚀 High-Performance Invoice Generator (M365)

> **Automated mass invoicing system capable of processing 800+ invoices/hour using a serverless architecture within the Microsoft 365 ecosystem.**

![Power Automate](https://img.shields.io/badge/Power_Automate-Flow-3659e3?style=flat&logo=powerautomate)
![Excel Online](https://img.shields.io/badge/Excel-Database-217346?style=flat&logo=microsoft-excel)
![SharePoint](https://img.shields.io/badge/SharePoint-CMS-0078d4?style=flat&logo=microsoft-sharepoint)
![Status](https://img.shields.io/badge/Status-Production-success)

## 📋 Overview
This project was born from the need to scale an invoicing department without increasing headcount or investing in expensive ERP software. A manual, decentralized process was replaced by an automated cloud solution.

The system orchestrates **Excel Online** (as a relational database), **Power Automate** (business logic), and **SharePoint** (storage) to generate complex PDF invoices, distribute them, and log accounting entries in real-time.

---

## ⚙️ Technical Architecture

The system utilizes high-level concurrency to process multiple sales orders simultaneously rather than linearly.

![Invoice Generator Architecture](INVOICE_GENERATOR_ARCHITECTURE.png)

### Key Technical Features
* **Mass Concurrency:** Configured to run up to 50 parallel threads. Batch processing time (50 invoices): **< 2 minutes**.
* **Relational Data in Excel:** Utilized OData Queries to cross-reference `Clients`, `Products` (thousands of SKUs), and `Orders` tables without SQL overhead.
* **Dynamic Business Logic:** The system automatically evaluates:
    * **Tax Regimes:** Standard VAT, Exempt, or Intra-community.
    * **Localization:** Currency and regional formatting based on client profile.
    * **Translation:** Automatic template selection based on the client's nationality.
* **Accounting Integrity:** Automatic ledger entry only triggers upon successful PDF generation, ensuring 100% data consistency.

---

## 📸 Project Gallery

### 1. Workflow Orchestration (Power Automate)
*A bird's-eye view of the automation logic, featuring error handling blocks and parallel branches.*

![Workflow Overview](assets/flow-overview.png)

### 2. Data Structure (Input)
*Excel structured as a relational database with data validation to prevent entry errors.*

![Excel Structure](assets/excel-db.png)

### 3. Final Output (Generated Invoice)
*Sample of a generated invoice, converted to PDF and archived automatically.*

![Invoice Demo](assets/invoice-demo.png)

---

## 📊 Impact & ROI

| Metric | Previous (Manual) | Automated Solution | Improvement |
| :--- | :--- | :--- | :--- |
| **Speed (50 invoices)** | > 4 Hours | < 2 Minutes | **120x Faster** |
| **Software Cost** | N/A (Manual labor cost) | ~€60 / Year (M365 License) | **Drastic Reduction** |
| **Error Rate** | High (Manual Copy/Paste) | 0% (Logic Validation) | **Total Integrity** |
| **Availability** | Local / WhatsApp | Centralized Cloud (SharePoint) | **Global Access** |

---

## 🚀 Deployment Concept
While tailored to specific business rules, the core engine can be replicated by importing the flow package:

1. Set up the **SharePoint** document library structure.
2. Configure **Excel Tables** (Schema included in `/docs`).
3. Import the **Power Automate** solution (.zip).
4. Map **Office 365 connectors** to your environment.

---

## 📬 Contact
Interested in optimizing business processes with Low-Code tools? Feel free to reach out via [GitHub](https://github.com/pabloballestin1)

---
## Workflow 
graph TD
    %% Estilos
    classDef trigger fill:#0066cc,stroke:#004d99,stroke-width:2px,color:white;
    classDef excel fill:#217346,stroke:#1e6b41,stroke-width:2px,color:white;
    classDef sharepoint fill:#0078d4,stroke:#005a9e,stroke-width:2px,color:white;
    classDef logic fill:#e1e1e1,stroke:#333,stroke-width:2px,color:black;
    classDef condition fill:#ffcc00,stroke:#e6b800,stroke-width:2px,color:black;

    subgraph "Trigger & Inicialización"
        Start(("🚀 Botón Manual<br/>(Inputs: Código, FC, FV, Prueba)")):::trigger
        InitVars[Inicializar Variables<br/>Contadores y Filas]:::logic
    end

    subgraph "Proceso Facturas Compra"
        CheckFC{"¿Generar Compras?"}:::condition
        GetOrders[📊 Excel: Obtener Pedidos<br/>(Filtro: Semana)]:::excel
        LoopFC("🔄 Loop: Por cada Factura<br/>(Concurrencia Activada)"):::logic
        
        CheckFacturar{"¿FACTURAR == SI?"}:::condition
        PopulateHeader[📝 Excel: Rellenar Cabecera<br/>(Plantilla Compra Trufa)]:::excel
        GetProducts[🔍 Excel: Obtener Productos]:::excel
        
        subgraph "Detalle Productos"
            LoopProd("🔄 Loop: Por cada Producto"):::logic
            CalcImp{¿Calcular Impuestos?}:::condition
            AddLine[➕ Excel: Añadir Línea Contable]:::excel
        end
        
        CreateXLSX[💾 SharePoint: Crear Excel Temporal]:::sharepoint
        ConvertToPDF[📄 OneDrive: Convertir a PDF]:::sharepoint
        SavePDF[archivado SharePoint: Guardar PDF Final]:::sharepoint
        UpdateAcct[💰 Excel: Actualizar Balance Final]:::excel
    end

    %% Conexiones
    Start --> InitVars
    InitVars --> CheckFC
    
    CheckFC -- Sí --> GetOrders
    GetOrders --> LoopFC
    
    LoopFC --> CheckFacturar
    CheckFacturar -- Sí --> PopulateHeader
    PopulateHeader --> GetProducts
    GetProducts --> LoopProd
    
    LoopProd --> CalcImp
    CalcImp --> AddLine
    
    AddLine --> CreateXLSX
    CreateXLSX --> ConvertToPDF
    ConvertToPDF --> SavePDF
    SavePDF --> UpdateAcct
