# Coding Test - Student Instructions

This test evaluates your skills in backend development, database queries, frontend React components, CI/CD pipelines, and AI prompt engineering.

---

## 📋 Test Overview

- **Total Questions**: 5 (Choose ANY 3 to complete)
- **Recommended Time**: 45 minutes
- **Submission**: Push your code to your GitHub repository before time ends
- **Scoring**: Your top 3 completed questions will be graded

### Question Breakdown

| # | Topic | Points |
|---|-------|--------|
| 1 | Backend API (Python Flask OR Java Spring Boot) | 25 |
| 2 | Database (SQL + MongoDB) | 20 |
| 3 | Frontend (React Component) | 20 |
| 4 | CI/CD & GitHub Actions | 15 |
| 5 | Prompt Engineering (AI) | 20 |

**Important**: You only need to complete **3 out of 5** questions. Choose the ones that match your strengths!

---

## 🚀 Getting Started with GitHub Codespaces

### Step 1: Fork This Repository
1. Click the **Fork** button at the top-right of this repository
2. This creates a copy in YOUR GitHub account
3. All your work will be saved there

### Step 2: Open in GitHub Codespaces (Recommended)
**This is the easiest way - no local setup needed!**

1. Go to your forked repository on GitHub
2. Click the green **Code** button
3. Select the **Codespaces** tab
4. Click **Create codespace on main**
5. Wait ~30 seconds for the environment to load with all tools pre-installed

✅ **You'll get:**
- VS Code running in your browser
- Python 3.11, Java 17, Node.js 18 pre-installed
- PostgreSQL and MongoDB clients ready
- All extensions configured
- Direct commit/push to your repository

### Alternative: Use VS Code Locally
If you prefer working locally:
```bash
# Replace YOUR_USERNAME with your GitHub username
git clone https://github.com/YOUR_USERNAME/erp-coding-test.git
cd erp-coding-test
# Install dependencies
pip install -r requirements.txt
```

---

## 📝 Questions

### QUESTION 1: Backend API – Choose Python or Java

**Context:** You are building an ERP Inventory Module with a PostgreSQL table `inventory` containing: `id (UUID)`, `product_name (VARCHAR)`, `quantity (INT)`, `reorder_level (INT)`, `last_updated (TIMESTAMP)`.

**Task:** Create a REST API endpoint `GET /api/inventory/alerts` that returns all products where `quantity <= reorder_level`.

**Expected JSON Format:**
```json
[{"id": "...", "product_name": "Widget", "quantity": 5, "reorder_level": 10}]
```

**Files to Complete:**
- **Python (Flask):** Edit `backend/python/app.py` - Complete the `get_alerts()` function
- **Java (Spring Boot):** Edit `backend/java/InventoryController.java` - Complete the `getAlerts()` method

*Choose ONE language only.*

---
1 Create Table--
CREATE TABLE inventory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_name VARCHAR(255) NOT NULL,
    quantity INT NOT NULL DEFAULT 0,
    reorder_level INT NOT NULL DEFAULT 0,
    last_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

2 Create API--
from flask import Flask, jsonify
@app.route("/api/inventory/alerts", methods=["GET"])
def get_alerts():
    conn = get_db_connection()
    cur = conn.cursor()

    cur.execute("""
        SELECT id, product_name, quantity, reorder_level
        FROM inventory
        WHERE quantity <= reorder_level
        ORDER BY quantity ASC
    """)

    rows = cur.fetchall()

    cur.close()
    conn.close()

    alerts = [
        {
            "id": str(row[0]),
            "product_name": row[1],
            "quantity": row[2],
            "reorder_level": row[3]
        }
        for row in rows
    ]

    return jsonify(alerts), 200

3. JSON File-- 
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "product_name": "Widget",
    "quantity": 5,
    "reorder_level": 10
  }
]


### QUESTION 2: Database – SQL + NoSQL

**Context:** Your ERP has:
- SQL `orders` table: `order_id`, `customer_id`, `total_amount`, `order_date`
- MongoDB `order_audit` collection: `{order_id, old_status, new_status, changed_by, timestamp}`

**Tasks:**

1. **SQL (10 pts):** Write a query to find the **top 5 customers** by total order value for year 2025.
   - File: `database/queries.sql`

2. **MongoDB (10 pts):** Find all audit entries for `order_id = "ORD-1001"` where status changed from `"PENDING"` to `"SHIPPED"`.
   - File: `database/queries.mongodb`

---
1 SQL--
SELECT
    customer_id,
    SUM(total_amount) AS total_order_value
FROM orders
WHERE order_date >= '2025-01-01'
  AND order_date < '2026-01-01'
GROUP BY customer_id
ORDER BY total_order_value DESC
LIMIT 5;

2 MongoDB--
db.order_audit.find({
    order_id: "ORD-1001",
    old_status: "PENDING",
    new_status: "SHIPPED"
})



### QUESTION 3: Frontend – React Dashboard Widget

**Context:** A React component shell exists. The backend API `/api/inventory/alerts` returns low-stock items.

**Task:**
- Fetch data from `/api/inventory/alerts` using `fetch()` inside `useEffect`
- Display results in a **table** with columns: Product Name, Quantity, Reorder Level
- If empty array, show: `<p>All inventory levels are healthy.</p>`

**File to Complete:** `frontend/Dashboard.jsx`

---
import React, { useEffect, useState } from "react";

function Dashboard() {
  const [alerts, setAlerts] = useState([]);

  useEffect(() => {
    fetch("/api/inventory/alerts")
      .then((response) => {
        if (!response.ok) {
          throw new Error("Failed to fetch inventory alerts");
        }
        return response.json();
      })
      .then((data) => {
        setAlerts(data);
      })
      .catch((error) => {
        console.error("Error fetching inventory alerts:", error);
      });
  }, []);

  return (
    <div>
      <h2>Inventory Alerts</h2>

      {alerts.length === 0 ? (
        <p>All inventory levels are healthy.</p>
      ) : (
        <table>
          <thead>
            <tr>
              <th>Product Name</th>
              <th>Quantity</th>
              <th>Reorder Level</th>
            </tr>
          </thead>

          <tbody>
            {alerts.map((item) => (
              <tr key={item.id}>
                <td>{item.product_name}</td>
                <td>{item.quantity}</td>
                <td>{item.reorder_level}</td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}
export default Dashboard;

### QUESTION 4: CI/CD & GitHub Actions

**Context:** Your repository needs automated testing on every push.

**Task:** Create a GitHub Actions workflow that:
- Triggers on `push` to `main` branch
- Runs `npm install` and `npm run build` for frontend
- Runs `python -m pytest` (Python) OR `mvn test` (Java) for backend

**File to Create/Edit:** `ci-cd/deploy.yml`

---
name: CI/CD

on:
  push:
    branches:
      - main

jobs:
  frontend:
    name: Frontend Build
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: frontend

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install

      - name: Build frontend
        run: npm run build

  backend:
    name: Backend Tests
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: backend/python

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
          pip install pytest

      - name: Run tests
        run: python -m pytest

### QUESTION 5: Prompt Engineering

**Context:** Your PM wants a **"Supplier Performance"** widget showing:
- On-time delivery percentage per supplier
- Average response time to purchase orders  
- Color-coded status (Green/Yellow/Red)

**Task:** Write a detailed AI prompt to generate this React component. Your prompt must include:
- Component name: `SupplierPerformance`
- Expected props/API data structure
- UI layout (cards or table)
- Color-coding logic

**File to Complete:** `prompt.txt`

---
Create a production-ready React component named `SupplierPerformance` for an ERP dashboard.

Component requirements:

1. Component name
- The component must be named `SupplierPerformance`.
- Use a functional React component.
- Use modern React with hooks where appropriate.
- Export the component as the default export.

2. Data source / API
The component should fetch supplier performance data from:

GET /api/suppliers/performance

Expected API response:

[
  {
    "supplier_id": "SUP-001",
    "supplier_name": "ABC Supplies",
    "on_time_delivery_percentage": 95.5,
    "average_response_time_hours": 4.2
  },
  {
    "supplier_id": "SUP-002",
    "supplier_name": "XYZ Traders",
    "on_time_delivery_percentage": 82.0,
    "average_response_time_hours": 12.5
  }
]

Also design the component so the data can optionally be supplied through a prop:

<SupplierPerformance suppliers={suppliers} />

Expected `suppliers` prop structure:

[
  {
    supplier_id: string,
    supplier_name: string,
    on_time_delivery_percentage: number,
    average_response_time_hours: number
  }
]

If the `suppliers` prop is provided, use it instead of fetching from the API.

3. UI layout
Create a clean and responsive dashboard widget.

Display suppliers in a table with these columns:

- Supplier
- On-Time Delivery
- Avg. Response Time
- Status

Each supplier should appear as one table row.

Format:
- On-time delivery as a percentage, for example `95.5%`.
- Response time in hours, for example `4.2 hrs`.

Add a widget heading:
"Supplier Performance"

The table should be responsive and readable on desktop and mobile screens.

4. Color-coded status logic

Calculate the supplier status using both performance metrics.

GREEN:
- On-time delivery >= 90%
- AND average response time <= 8 hours

YELLOW:
- On-time delivery >= 75%
- AND average response time <= 24 hours
- But the supplier does not meet the GREEN criteria.

RED:
- On-time delivery < 75%
- OR average response time > 24 hours.

Display the status using both text and color:
- Green: `Good`
- Yellow: `Needs Attention`
- Red: `Poor`

Use accessible colors and do not rely on color alone. Include the status text or an appropriate accessible label.

5. Loading and error handling
- Show a loading state while fetching data.
- Show a clear error message if the API request fails.
- Handle an empty supplier list gracefully with:
  "No supplier performance data available."

6. Technical requirements
- Use `fetch()` for the API request.
- Use `useEffect` and `useState` when fetching data.
- Do not use external UI libraries unless necessary.
- Keep the component self-contained and reusable.
- Use semantic HTML.
- Add reasonable CSS/classes for the dashboard card, table, status badges, loading state, and error state.
- Avoid hardcoded supplier records in the component.
- Handle invalid or missing numeric values gracefully.

Return the complete React component code and any required CSS. The implementation should be clean, maintainable, accessible, and ready to integrate into an existing ERP dashboard.

## ✅ Submission Checklist

Before time ends, ensure:
- [ ] You have forked the repository to YOUR GitHub account
- [ ] You have completed exactly 3 questions (or more if you want options)
- [ ] All your code is committed (`git commit -m "my solution"`)
- [ ] All commits are pushed to `main` branch (`git push origin main`)
- [ ] Your GitHub repository is public (so graders can access it)

---

## 🎯 Scoring Rules

- Each question is graded independently
- If you complete more than 3 questions, your **top 3 scores** count
- Final score = Sum of your best 3 question scores
