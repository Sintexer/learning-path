## Star Schema

**Structure:** One large central fact table surrounded by multiple dimension tables. The relationship pattern looks like a star when visualized.

**Key characteristics:**

- **Denormalized** — dimensions contain redundant data to avoid complex joins
- **Simpler queries** — fewer joins required (fact table joins directly to dimensions)
- **Faster query performance** — optimized for read-heavy analytical queries
- **More storage** — dimension tables contain repeated data


```mermaid
graph TB
    subgraph Fact["Sales Fact Table"]
        F["SaleID (PK)<br/>ProductID (FK)<br/>CustomerID (FK)<br/>TimeID (FK)<br/>StoreID (FK)<br/>Amount<br/>Quantity"]
    end
    
    subgraph ProductDim["Product Dimension"]
        P["ProductID (PK)<br/>Name<br/>Category<br/>Brand<br/>Price<br/>Supplier"]
    end
    
    subgraph TimeDim["Time Dimension"]
        T["TimeID (PK)<br/>Date<br/>Month<br/>Quarter<br/>Year<br/>DayOfWeek"]
    end
    
    subgraph CustomerDim["Customer Dimension"]
        C["CustomerID (PK)<br/>Name<br/>Email<br/>City<br/>Country<br/>Segment"]
    end
    
    subgraph StoreDim["Store Dimension"]
        S["StoreID (PK)<br/>Location<br/>Region<br/>Manager<br/>OpenDate"]
    end
    
    F -->|"ProductID"| P
    F -->|"TimeID"| T
    F -->|"CustomerID"| C
    F -->|"StoreID"| S
    
    style Fact fill:#4a90e2,stroke:#2e5c8a,color:#fff
    style ProductDim fill:#7ed321,stroke:#5a9f1a,color:#fff
    style TimeDim fill:#7ed321,stroke:#5a9f1a,color:#fff
    style CustomerDim fill:#7ed321,stroke:#5a9f1a,color:#fff
    style StoreDim fill:#7ed321,stroke:#5a9f1a,color:#fff
```


**Pros:**

- Fast queries (minimal joins)
- Intuitive for analysts
- Good for most OLAP scenarios

**Cons:**

- Data redundancy
- Updates to dimension data can be complex
- Higher storage requirements

## Snowflake Schema

**Structure:** A refinement of the star schema where dimension tables are themselves normalized into sub-dimensions, creating a snowflake pattern.

**Key characteristics:**

- **Normalized** — dimensions broken into hierarchies (e.g., Location → Region → Country)
- **More complex queries** — requires more joins between dimension tables
- **More storage efficient** — eliminates data redundancy
- **Slower queries** — additional joins add processing overhead

```mermaid
graph TB
    subgraph Fact["Sales Fact Table"]
        F["SaleID (PK)<br/>ProductID (FK)<br/>CustomerID (FK)<br/>TimeID (FK)<br/>StoreID (FK)<br/>Amount<br/>Quantity"]
    end
    
    subgraph Product["Product Dimension"]
        P["ProductID (PK)<br/>Name<br/>Price<br/>CategoryID (FK)<br/>SupplierID (FK)"]
    end
    
    subgraph Category["Category"]
        CAT["CategoryID (PK)<br/>CategoryName<br/>Department"]
    end
    
    subgraph Supplier["Supplier"]
        SUP["SupplierID (PK)<br/>SupplierName<br/>Contact<br/>Country"]
    end
    
    subgraph Calendar["Calendar Dimension"]
        T["DateID (PK)<br/>Date<br/>DayOfWeek<br/>MonthID (FK)"]
    end
    
    subgraph Month["Month"]
        M["MonthID (PK)<br/>MonthName<br/>Quarter<br/>Year"]
    end
    
    subgraph Customer["Customer Dimension"]
        C["CustomerID (PK)<br/>Name<br/>Email<br/>CityID (FK)"]
    end
    
    subgraph City["City"]
        CI["CityID (PK)<br/>CityName<br/>StateID (FK)"]
    end
    
    subgraph State["State"]
        ST["StateID (PK)<br/>StateName<br/>CountryID (FK)"]
    end
    
    subgraph Country["Country"]
        CO["CountryID (PK)<br/>CountryName"]
    end
    
    subgraph Store["Store Dimension"]
        S["StoreID (PK)<br/>Location<br/>Region<br/>Manager<br/>OpenDate"]
    end
    
    F -->|"ProductID"| P
    P -->|"CategoryID"| CAT
    P -->|"SupplierID"| SUP
    F -->|"TimeID"| T
    T -->|"MonthID"| M
    F -->|"CustomerID"| C
    C -->|"CityID"| CI
    CI -->|"StateID"| ST
    ST -->|"CountryID"| CO
    F -->|"StoreID"| S
    
    style Fact fill:#4a90e2,stroke:#2e5c8a,color:#fff
    style Product fill:#7ed321,stroke:#5a9f1a,color:#fff
    style Category fill:#f5a623,stroke:#c67f1a,color:#fff
    style Supplier fill:#f5a623,stroke:#c67f1a,color:#fff
    style Calendar fill:#7ed321,stroke:#5a9f1a,color:#fff
    style Month fill:#f5a623,stroke:#c67f1a,color:#fff
    style Customer fill:#7ed321,stroke:#5a9f1a,color:#fff
    style City fill:#f5a623,stroke:#c67f1a,color:#fff
    style State fill:#f5a623,stroke:#c67f1a,color:#fff
    style Country fill:#f5a623,stroke:#c67f1a,color:#fff
    style Store fill:#7ed321,stroke:#5a9f1a,color:#fff
```

**Pros:**

- Reduced storage (normalized)
- Easier data maintenance
- Cleaner data model

**Cons:**

- Slower queries (more joins)
- More complex SQL
- Steeper learning curve for analysts

---

## Quick Comparison Table

|Aspect|Star|Snowflake|
|---|---|---|
|**Normalization**|Denormalized|Normalized|
|**Query Speed**|Faster|Slower|
|**Storage**|Higher|Lower|
|**Complexity**|Simple|Complex|
|**Joins**|Few|Many|
|**Best For**|Simple analytics|Complex hierarchies|

---

## When to Use Each

**Use Star Schema when:**

- Query performance is critical
- Users are non-technical (need simplicity)
- Dimension hierarchies are shallow
- Storage isn't a constraint

**Use Snowflake Schema when:**

- Data integrity is paramount
- Complex hierarchical dimensions exist
- Storage efficiency matters
- Updates to dimension data are frequent
- You have skilled analysts who understand normalization

---

## Real-World Example

**Retail Sales Analysis:**

**Star:** Single Product Dimension (Product ID, Name, Category, Brand, Price, Supplier) + Fact Table  
→ _One simple join, fast queries, but Category/Brand/Supplier data repeats_

**Snowflake:** Product ID → Product Table → Category Table; Category Table → Supplier Table  
→ _More joins, less redundancy, more flexible if supplier changes_

---

This should give your readers a solid foundation for understanding when and why to choose each approach!