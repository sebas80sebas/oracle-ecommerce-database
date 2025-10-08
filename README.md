# CaffeineManiacs™ Database System

A comprehensive database management system for an e-commerce coffee company, developed as part of the "Files and Databases" course at Universidad Carlos III de Madrid.

---

## 📋 Project Overview

This project implements a complete relational database system for **CaffeineManiacs™ Inc**, an e-commerce company specializing in coffee products. The system manages customers, products, orders, suppliers, and user reviews through a well-structured Oracle database.

The project was developed in three progressive phases:

1. **Relational Design & Data Loading**
2. **Advanced Queries, Procedures & Triggers**
3. **Physical Design & Performance Optimization**

---

## 👥 Team Members

- **Iván Sebastián Loor Weir** (NIA: 100448737)
- **Álvaro González Fúnez** (NIA: 100451281)
- **Arturo Jiménez Tajuelo Pérez** (NIA: 100451161)

**Academic Year:** 2023/24 - 2nd Semester  
**Course:** Files and Databases (2nd Year)  
**Degree:** Computer Science Engineering

---

## 🗂️ Project Structure

```
caffeinemaniacs-database/
├── Practica1.pdf    # Practice 1: Relational Design & Data Loading
├── Practica2.pdf    # Practice 2: Queries, Procedures & Triggers
├── Practica3.pdf    # Practice 3: Physical Design & Optimization
└── README.md        # This file
```

---

## 📊 Practice 1: Relational Design & Data Loading

### Key Components

#### Database Schema Design
- Complete relational schema with implicit and explicit semantics
- Normalized tables for customers (registered/unregistered), products, suppliers, orders, reviews
- Proper constraint implementation (PKs, FKs, CHECKs)

#### Main Tables Created

| Table | Description |
|-------|-------------|
| `Cliente_Reg` / `Cliente_NoReg` | Registered and unregistered customers |
| `Producto` | Coffee products with origin and roast information |
| `Articulo` | Product references with stock management |
| `Proveedor` | Supplier information |
| `Compra` | Purchase transactions |
| `Pedido_Reposicion` | Supplier replenishment orders |
| `Publicacion` | Customer reviews and ratings |
| `Tarjeta` | Customer payment cards |
| `Direccion` | Customer addresses |
| `Entrega` | Delivery information |

### Data Validation

- **Email format validation** using `REGEXP_LIKE`
- **Phone number** length constraints (9 digits)
- **CIF validation** for suppliers (10 characters)
- **Barcode validation** (15 characters)
- **Card number validation** (20 characters)
- **Check constraints** for product attributes (decaf, roast type)
- **Score validation** (1-5 range)

### Semantic Constraints

#### ✅ Implemented (53 total)
- Cascade deletion and update rules
- Composite primary keys for addresses
- Auto-generated IDs for orders, deliveries, and publications
- Stock range validation (`min_stock ≤ stock ≤ max_stock`)
- Order status constraints (`'draft'`, `'placed'`, `'fulfilled'`)

#### ❌ Not Implemented (14 excluded)
- Automatic stock replenishment triggers
- Dynamic discount calculations
- Validated endorsement system for reviews

---

## 🔍 Practice 2: Advanced Queries, Procedures & Triggers

### Complex Queries

#### Query 1: Variety Analysis by Country
- Identifies potential consumer countries for each coffee variety
- Calculates total buyers, units sold, revenue, and average units per reference
- Filters countries representing >1% of total sales for each variety
- Uses year-over-year comparison

#### Query 2: Monthly Best-Selling References
- Identifies top-selling product reference each month
- Calculates order count, units sold, total revenue, and total profit
- Uses `ROW_NUMBER()` window function for ranking

### PL/SQL Package: `caffeine`

#### Procedure 1: `Informe_Proveedor`
Generates comprehensive supplier reports:
- **Statistics:** confirmed orders, completed orders, average delivery time
- **Offer analysis:** current cost, min/max costs, cost differentials
- **Comparison** against average and best offers

#### Procedure 2: `Set_Replacement_Orders`
- Converts draft orders to confirmed status
- Updates order states in bulk
- Error handling with custom messages

### Database Views

#### View 1: `Mis_compras`
- Personal purchase history for logged-in user
- Shows order details, products, prices, quantities
- Read-only access

#### View 2: `Mi_perfil`
- Complete user profile information
- Includes addresses and payment cards
- Uses `LEFT JOINs` for optional data
- Read-only access

#### View 3: `mis_comentarios`
- User's product reviews
- **Updatable view** with `INSTEAD OF` triggers
- **Business rules:**
  - Can only delete comments with 0 likes
  - Can only update text if likes = 0
  - Automatic insertion handling

### Database Triggers

#### Trigger 1: `trg_update_endorsed`
- Automatically sets endorsement flag on reviews
- Validates if user previously purchased the product
- Executes before `INSERT` or `UPDATE` on Posts

#### Trigger 2: `Move_To_Anonymous` ⚠️ (Implementation incomplete)
- Converts registered user data to anonymous on deletion
- Moves orders and reviews to anonymous tables
- **Issue:** Encountered mutating table error

#### Trigger 3: `Prevent_Anonymous_Purchase`
- Prevents anonymous purchases with registered user cards
- Validates card ownership before insertion
- Raises custom error on violation

#### Trigger 4: `Update_Stocks`
- Automatic stock management after purchases
- Generates replenishment orders when `stock < min_stock`
- Selects cheapest supplier automatically
- Handles cases: no supplier, one supplier, multiple suppliers
- Updates stock levels in real-time

---

## ⚡ Practice 3: Physical Design & Performance Optimization

### Performance Analysis

#### Initial Configuration
- **Organization:** Serial non-consecutive (Oracle default)
- **Block size:** 8KB
- **PCTFREE:** 10% (default)
- **PCTUSED:** 60% (default)

### Workload Queries

| Query | Description | Initial Cost | Frequency |
|-------|-------------|--------------|-----------|
| Q1 | Posts by barcode | High (full scan) | 0.1 |
| Q2 | Posts by product | High (full scan) | 0.1 |
| Q3 | Posts with score ≥ 4 | High (full scan) | 0.1 |
| Q4 | All posts | Very High | 0.2 |
| Q5 | User purchases (JOIN) | Medium | 0.5 |

### Optimization Strategies Tested

#### 1. Index Optimization ✅ **Best Result**

**Index 1** (Critical optimization)
```sql
CREATE INDEX ind_compuesto_usuario 
ON orders_clients(username, town, country);
```

**Impact on Q5:**
- Time: 0.07s → 0.01s (85% improvement)
- Cost: Significant reduction
- Consistent gets: Halved

**Index 2** (Complementary optimization)
```sql
CREATE INDEX ind_post 
ON posts(barcode, product, username);
```

**Impact on Q1-Q4:**
- Marginal cost reduction
- Time unchanged but stabilized
- Better query planning

#### 2. Block Size Adjustment ❌ **Rejected**

**Testing:** 2KB, 8KB, 16KB blocks

**Results (16KB):**
- Workload tests: Improved (29ms vs 32ms)
- Individual queries: Degraded significantly
  - Q3: 0.05s → 0.66s (1220% worse!)
  - Q4: 0.13s → 0.50s (285% worse!)

**Conclusion:** Optimization contradicts actual query performance

#### 3. Cluster Implementation ❌ **Rejected**

```sql
CREATE CLUSTER usuario (username VARCHAR2(30));
CREATE INDEX ind_usuario ON CLUSTER usuario;
```

**Results:**
- Q1: Excellent improvement (0.04s → 0.01s)
- Q2-Q4: Significant degradation
- Workload: 35.4ms vs 32.1ms initial (worse overall)

**Conclusion:** Optimizes one query at expense of others

#### 4. PCTUSED/PCTFREE Tuning ⚠️ **Minimal Impact**

- **Tested:** PCTUSED=99% (read-only workload)
- **Result:** Negligible performance difference
- **Reason:** Small dataset size

### Final Design Performance

| Configuration | Time (ms) | Consistent Gets | Improvement |
|---------------|-----------|-----------------|-------------|
| Initial | 33.5 | 7,063 | baseline |
| Index 1 only | 20.0 | 3,840 | 40% faster |
| Index 1 + 2 | 20.1 | 2,906 | 40% faster |
| Cluster | 38.0 | 7,714 | 13% slower ❌ |
| 16KB blocks | 28.0 | 3,965 | Query degradation ❌ |

---

## 🎯 Optimal Configuration

### 🏆 Winner: Dual Index Strategy

- **Block size:** 8KB (default)
- **Index 1:** `(username, town, country)` on `orders_clients`
- **Index 2:** `(barcode, product, username)` on `posts`
- **PCTUSED/PCTFREE:** Default values

### Performance Gains:
- ✅ 40% faster execution time
- ✅ 59% reduction in consistent gets
- ✅ All queries improved or maintained
- ✅ No degradation in any operation

---

## 🛠️ Technologies Used

- **DBMS:** Oracle Database
- **Language:** SQL, PL/SQL
- **Tools:** SQL*Plus
- **Design:** Relational algebra, ER diagrams

---

## 📈 Key Learning Outcomes

- **Relational Design:** Normalization, constraint modeling, semantic analysis
- **SQL Proficiency:** Complex queries, window functions, subqueries
- **PL/SQL Programming:** Procedures, triggers, packages, exception handling
- **Performance Tuning:** Index design, query optimization, execution plan analysis
- **Database Administration:** Physical design decisions, storage optimization

---

## 🎓 Academic Context

This project demonstrates comprehensive database management skills covering:

- Conceptual and logical database design
- Physical implementation in Oracle
- Query optimization and performance analysis
- Business logic implementation through triggers and procedures
- Real-world e-commerce data modeling

---

## 📝 Documentation

For comprehensive technical details, implementation specifications, and in-depth analysis, please refer to the complete project documentation:

- **Practice 1:** `Practica1.pdf` - Relational Design & Data Loading
- **Practice 2:** `Practica2.pdf` - Advanced Queries, Procedures & Triggers
- **Practice 3:** `Practica3.pdf` - Physical Design & Performance Optimization

All documents are available in Spanish and include detailed explanations, code examples, and performance metrics.
Each practice includes detailed documentation:

- Design decisions with relational algebra foundations
- Implementation code with extensive comments
- Test cases demonstrating functionality
- Performance metrics with before/after comparisons
- Lessons learned and improvement suggestions

---

## ⚠️ Known Limitations

- Mutating table error in `Move_To_Anonymous` trigger
- Some semantic constraints require application-level enforcement
- Performance improvements limited by small dataset size
- Email validation regex could be more restrictive
- No supplier handling in stock trigger could be improved

---

## 🏆 Achievements

- ✅ Successfully migrated legacy denormalized data to normalized schema
- ✅ Implemented 4 complex triggers with business logic
- ✅ Created 3 user views with appropriate access controls
- ✅ Achieved 40% performance improvement through indexing
- ✅ Demonstrated practical understanding of database optimization trade-offs

---

## 📧 Contact

- **Iván Sebastián Loor Weir:** 100448737@alumnos.uc3m.es
- **Álvaro González Fúnez:** 100451281@alumnos.uc3m.es
- **Arturo Jiménez Tajuelo Pérez:** 100451161@alumnos.uc3m.es

---

## 📝 License

Academic Project - Universidad Carlos III de Madrid © 2023-2024

---

**Universidad Carlos III de Madrid**  
Degree in Computer Science Engineering  
Course: Ficheros y Bases de Datos (Files and Databases)  
Academic Year: 2023/24
