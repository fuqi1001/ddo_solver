# pgBouncer Configuration Formulas for Session Mode and Transaction Mode

When using **`session mode`** or **`transaction mode`** in pgBouncer, the following formulas help calculate `default_pool_size` and `max_client_conn`, considering **PostgreSQL `max_connections`**, **number of users**, and **number of databases**.

---

## 1. Session Mode

### Key Points:
- Each pgBouncer connection maps directly to a PostgreSQL connection.
- Connections are dedicated per **user/database pair**, so the count is proportional to the number of users and databases.

### Formulas:
1. **`default_pool_size = PostgreSQL max_connections ÷ (number_of_users × number_of_databases)`**  
   Ensure connections are divided proportionally among all user/database combinations.

2. **`max_client_conn = (default_pool_size × number_of_users × number_of_databases) + overhead`**  
   Overhead is typically **10%-20%** for admin or monitoring connections.

### Example:
- **PostgreSQL max_connections** = 500
- **Number of users** = 5
- **Number of databases** = 2

1. **`default_pool_size`**:

`default_pool_size = 500 ÷ (5 × 2) = 50`


2. **`max_client_conn`**:

`max_client_conn = (50 × 5 × 2) + (10% overhead) = 500 + 50 = 550`


---

## 2. Transaction Mode

### Key Points:
- Connections are reused between transactions, so `default_pool_size` can be smaller.
- Clients can far exceed physical connections, so `max_client_conn` can be much larger.

### Formulas:
1. **`default_pool_size = PostgreSQL max_connections ÷ 2 ÷ (number_of_users × number_of_databases)`**  
Reduce pool size by 50% due to transaction-level reuse.

2. **`max_client_conn = (default_pool_size × number_of_users × number_of_databases × multiplier)`**  
Multiplier is typically **5-10**, depending on concurrency and query patterns.

### Example:
- **PostgreSQL max_connections** = 500
- **Number of users** = 5
- **Number of databases** = 2
- **Multiplier** = 5

1. **`default_pool_size`**:

`default_pool_size = (500 ÷ 2) ÷ (5 × 2) = 25`


2. **`max_client_conn`**:

`max_client_conn = (25 × 5 × 2 × 5) = 1250`


---

## Comparison Table

| **Mode**           | **PostgreSQL max_connections** | **Number of Users** | **Number of Databases** | **`default_pool_size`** | **`max_client_conn`** |
|---------------------|-------------------------------|----------------------|--------------------------|-------------------------|-----------------------|
| **Session Mode**    | 500                           | 5                    | 2                        | 50                      | 550                   |
| **Transaction Mode**| 500                           | 5                    | 2                        | 25                      | 1250                 |

---

## Summary

### Session Mode Formulas:
- **`default_pool_size = PostgreSQL max_connections ÷ (number_of_users × number_of_databases)`**
- **`max_client_conn = (default_pool_size × number_of_users × number_of_databases) + overhead`**

### Transaction Mode Formulas:
- **`default_pool_size = PostgreSQL max_connections ÷ 2 ÷ (number_of_users × number_of_databases)`**
- **`max_client_conn = (default_pool_size × number_of_users × number_of_databases × multiplier)`**

These formulas dynamically adjust for varying numbers of users and databases. You can tweak the overhead or multiplier based on your workload characteristics.
