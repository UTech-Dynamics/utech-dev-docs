# Thinking vs Coding Checklist

## 1️⃣ Before Writing Code (Thinking / Design Stage)

***Goal: Avoid mistakes that are expensive later.***

Example Tasks:
```
Focus   	                | Questions to Ask Yourself	                   |  Examples
Domain Modeling	            What are the real entities?                     How do they relate?	Product, Brand, Category
Cardinality	                One-to-many, many-to-many?	                    Product → Brand (1:N), Product → Categories (M:N)
Constraints	                What must never happen?	                        base_price ≥ 0, NOT NULL, brand must exist
Lifecycle / Status	        How does the entity evolve over time?	          active, discontinued, archived
Business Rules	            What rules must be preserved in DB or app?	    Products always have a primary category
Indexing / Performance	    Which columns will be filtered/queried often?	  brand_id, primary_category_id
Future-proofing	            Could entities get metadata or hierarchy later?	Category hierarchy, brand active/inactive flag
Edge Cases	                What happens if rules are violated?	            Trying to delete a brand in use
```
Outcome: You should be able to sketch the table schema, relationships, and constraints on paper or mentally.

## 2️⃣ While Writing Code (Implementation Stage)

Goal: Translate your design into working DB & code safely.

- Create migrations according to your schema design.

- Add FKs, CHECK constraints, NOT NULL, UNIQUE indexes.

- Implement enums or status columns for lifecycle.

- Apply soft deletion / lifecycle logic at DB + app level if needed.

- Write basic seed data to test constraints.

Tip: If you already thought about edge cases, coding becomes mechanical.

##  3️⃣ After Writing Code (Validation / Testing Stage)

- Ensure constraints work (try inserting invalid data).

- Check queries for active vs admin-only visibility.

- Ensure FKs prevent invalid references.

- Test lifecycle transitions (active → discontinued → archived).

##  4️⃣ How to Measure Productivity

- Time spent thinking/design is not wasted.

- If you can write code faster and with fewer bugs, that thinking time is paid off.

- A good ratio in real projects: 30–40% thinking/design, 50–60% coding, 10% testing/review

## Why This Is Valuable

Many juniors skip this step and jump straight into code. Consequences:

- Buggy or inconsistent database

- Hard-to-change design later

- Data integrity issues

- Refactors that take much longer

By doing this upfront:

- You reduce future rework

- You write higher-quality migrations and code

- You learn how real systems are designed

- You build intuition for business rules + technical constraints

***Spending 1–2 hours thinking deeply at this stage is 100× more valuable than writing code blindly.***


**✅ Key Insight:**

“Writing code fast doesn’t mean writing good code. Spending time thinking about domain, relationships, and constraints makes you a software engineer, not just a coder.”