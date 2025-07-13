# 💳 Non-Zero Balance Card Identification

## 🔍 Objective

Identify all customer cards that have a **non-zero balance** as of June 30, 2025, and belong to specific issuer programs (by BIN). This can support business actions such as refunds, customer outreach, or balance utilization campaigns.

---

## 🗃️ Data Tables Used

- **card_apps_balance_entries**: Stores balance entries per card and timestamp  
- **card**: Card master metadata  
- **card_program**: Maps each card to its program  
- **issuer_program**: Contains issuer details, including BINs  

---

## 🧠 Logic & Approach

1. Aggregate balance amounts from `card_apps_balance_entries` for each `card_id` up to **June 30, 2025**
2. Filter out cards where the **total balance is 0 or negative**
3. Join with the `card`, `card_program`, and `issuer_program` tables
4. Filter for specific **BINs** (`'981254'`, `'896734'`) to narrow down to relevant issuer programs

---

## 🛠️ SQL Techniques Used

- Subquery with `SUM()` aggregation
- `HAVING` clause for filtering grouped data
- Multi-level `INNER JOIN`s for data enrichment
- Date-based filtering
- Conditional filtering using `IN`

---

## 💡 Business Use Cases

- Identify cards with unutilized funds for **refund or closure**
- Drive **customer re-engagement** campaigns based on unused balances
- Monitor cards for **compliance** and **inactivity audits**

---

## 🧾 SQL Query

```sql
SELECT 
    filtered.card_id AS card_id,
    filtered.amount
FROM 
    (
        SELECT 
            card_id, 
            SUM(amount) AS amount
        FROM 
            card_apps_balance_entries
        WHERE 
            created_time <= '2025-06-30 18:30:00'
        GROUP BY 
            card_id
        HAVING 
            SUM(amount) > 0
    ) AS filtered
INNER JOIN 
    card AS c ON filtered.card_id = c.card_id
INNER JOIN 
    card_program AS cp ON c.card_program_id = cp.card_program_id
INNER JOIN 
    issuer_program AS ip ON cp.issuer_program_id = ip.id
WHERE 
    ip.bin IN ('981254', '896734');

