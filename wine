import streamlit as st
import sqlite3
import datetime

# Connect to SQLite DB
conn = sqlite3.connect('wine_mart.db', check_same_thread=False)
cursor = conn.cursor()

# Create tables
cursor.execute('''
CREATE TABLE IF NOT EXISTS wines (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    type TEXT,
    price REAL,
    stock INTEGER
)
''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS sales (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    wine_id INTEGER,
    quantity INTEGER,
    total_price REAL,
    date TEXT,
    FOREIGN KEY (wine_id) REFERENCES wines(id)
)
''')
conn.commit()

# Streamlit UI
st.title("🍷 Wine Mart App")

menu = ["Add Wine", "View Inventory", "Purchase Wine", "Sales Report"]
choice = st.sidebar.selectbox("Menu", menu)

# Add Wine
if choice == "Add Wine":
    st.header("➕ Add New Wine")
    name = st.text_input("Wine Name")
    wine_type = st.selectbox("Wine Type", ["Red", "White", "Rose"])
    price = st.number_input("Price", min_value=0.0)
    stock = st.number_input("Stock Quantity", min_value=0)

    if st.button("Add Wine"):
        cursor.execute("INSERT INTO wines (name, type, price, stock) VALUES (?, ?, ?, ?)",
                       (name, wine_type, price, stock))
        conn.commit()
        st.success(f"{name} added successfully!")

# View Inventory
elif choice == "View Inventory":
    st.header("📦 Inventory")
    cursor.execute("SELECT * FROM wines")
    data = cursor.fetchall()
    if data:
        st.table(data)
    else:
        st.info("No wines available.")

# Purchase Wine
elif choice == "Purchase Wine":
    st.header("🛒 Purchase Wine")
    cursor.execute("SELECT * FROM wines")
    wines = cursor.fetchall()

    if wines:
        wine_names = [f"{w[0]} - {w[1]} (₹{w[3]}) | Stock: {w[4]}" for w in wines]
        selected = st.selectbox("Select Wine", wine_names)
        selected_id = int(selected.split(' - ')[0])
        quantity = st.number_input("Quantity", min_value=1)

        if st.button("Purchase"):
            selected_wine = [w for w in wines if w[0] == selected_id][0]
            if selected_wine[4] >= quantity:
                total_price = selected_wine[3] * quantity
                cursor.execute("UPDATE wines SET stock = stock - ? WHERE id = ?", (quantity, selected_id))
                cursor.execute("INSERT INTO sales (wine_id, quantity, total_price, date) VALUES (?, ?, ?, ?)",
                               (selected_id, quantity, total_price, datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
                conn.commit()
                st.success(f"Purchased {quantity} x {selected_wine[1]} for ₹{total_price}")
            else:
                st.error("Not enough stock.")
    else:
        st.warning("No wines available.")

# Sales Report
elif choice == "Sales Report":
    st.header("📈 Sales Report")
    cursor.execute('''
    SELECT wines.name, sales.quantity, sales.total_price, sales.date
    FROM sales
    JOIN wines ON wines.id = sales.wine_id
    ORDER BY sales.date DESC
    ''')
    report = cursor.fetchall()

    if report:
        st.table(report)
    else:
        st.info("No sales yet.")


streamlit run wine_mart_app.py
