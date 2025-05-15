# shopping_tracker.py
پروژه برای مدیریت خرید محصولات با امکانات و قابلیت های زیاد#

import json
import os
from datetime import datetime

# نام فایل برای ذخیره داده‌ها
data_file = "shopping_data.json"

# تابع بارگذاری داده‌ها از فایل JSON (اگر وجود نداشته باشد، ساختار اولیه را ایجاد می‌کند)
def load_data():
    if os.path.exists(data_file):
        try:
            with open(data_file, "r") as file:
                data = json.load(file)
            if isinstance(data, dict) and "categories" in data:
                return data
            else:
                print("Error: Invalid data structure. Starting with empty data.")
                return {"categories": {}}
        except json.JSONDecodeError:
            print("Error: Invalid JSON data. Starting with empty data.")
            return {"categories": {}}
    return {"categories": {}}

# بارگذاری داده‌ها در زمان شروع برنامه
data = load_data()

# تابع ذخیره داده‌ها در فایل JSON
def save_data():
    # ذخیره داده‌ها در فایل
    with open(data_file, "w") as file:
        json.dump(data, file)

# گزینه 1: اضافه کردن ایتم با دسته‌بندی، نام، تعداد، قیمت و تاریخ/زمان اضافه شدن
def add_item():
    # نمایش دسته‌بندی‌های موجود
    if data["categories"]:
        print("Available categories:")
        categories = list(data["categories"].keys())
        for i, category in enumerate(categories, start=1):
            print(f"{i}. {category}")
    else:
        print("No categories found. You need to create a new one.")
    
    # دریافت نام یا شماره دسته‌بندی
    while True:
        category_input = input("Enter category name or number: ").strip()
        if category_input.isdigit():
            index = int(category_input) - 1
            if 0 <= index < len(categories):
                category = categories[index]
                break
            else:
                print("Invalid selection. Try again.")
        else:
            category = category_input
            if category not in data["categories"]:
                data["categories"][category] = []
                print(f"New category created ({category}).")
            break
    
    # نمایش آیتم‌های موجود در دسته‌بندی انتخاب شده (بدون تکرار)
    existing_items = {item["name"] for item in data["categories"].get(category, [])}
    existing_items_list = list(existing_items)
    
    if existing_items_list:
        print("Available items in this category:")
        for i, item in enumerate(existing_items_list, start=1):
            print(f"{i}. {item}")
    
    # دریافت نام یا شماره آیتم
    while True:
        item_input = input("Enter item name or number: ").strip()
        if item_input.isdigit():
            index = int(item_input) - 1
            if 0 <= index < len(existing_items_list):
                name = existing_items_list[index]
                break
            else:
                print("Invalid selection. Try again.")
                
        else:
            name = item_input.strip()
            if name in existing_items:
                print("Item already exists, selecting existing item.")
            else:
                existing_items.add(name)
                print(f"New item '{name}' added to selection.")
            break
    
    while not name:
        print("Item name cannot be empty.")
        name = input("Enter item name: ").strip()
    
    # انتخاب واحد (یکا)
    units = ["kg", "g", "number", "cm", "m", "mm", "m3", "m2", "liter", "pack", "box"]
    print("Available unit items:")
    for i, unit in enumerate(units, start=1):
        print(f"{i}. {unit}")
    
    while True:
        unit_choice = input("Select unit by number: ").strip()
        if unit_choice.isdigit():
            index = int(unit_choice) - 1
            if 0 <= index < len(units):
                unit = units[index]
                break
            else:
                print("Invalid selection. Please enter a valid number.")
        else:
            print("Invalid input. Please enter a number.")
    
    # دریافت تعداد با اعتبارسنجی
    while True:
        try:
            quantity = int(input(f"Enter quantity ({unit}): ").strip())
            if quantity <= 0:
                print("Quantity must be positive.")
                continue
            break
        except ValueError:
            print("Invalid quantity. Please enter an integer.")
    
    # دریافت قیمت با اعتبارسنجی
    while True:
        try:
            price = float(input("Enter price per unit: ").strip())
            if price <= 0:
                print("Price must be positive.")
                continue
            break
        except ValueError:
            print("Invalid price. Please enter a number.")
    
    # دریافت تاریخ و زمان فعلی به صورت ISO
    timestamp = datetime.now().isoformat()
    
    # ایجاد دیکشنری ایتم
    item = {
        "name": name,
        "unit": unit,
        "quantity": quantity,
        "price": price,
        "timestamp": timestamp
    }
    
    # اضافه کردن ایتم به دسته‌بندی مربوطه
    data["categories"][category].append(item)
    save_data()
    print(f"Added {quantity} {unit} of {name} in category '{category}' with price {price:,} per {unit} (added at {timestamp})")


# گزینه 2: نمایش تمام ایتم‌ها از تمامی دسته‌بندی‌ها
def display_all():
    print("\nAll Items:")
    print("-" * 50)
    total_price = 0
    if not data["categories"]:
        print("No categories found.")
    else:
        for category, items in data["categories"].items():
            print(f"Category: {category}")
            if not items:
                print("  No items in this category.")
            else:
                for i, item in enumerate(items, start=1):
                    total = item["quantity"] * item["price"]
                    total_price += total
                    # نمایش واحد (unit) در اینجا
                    print(f"  {i}. {item['name']} ({item['quantity']} {item['unit']} x {item['price']:,} = {total:,})")
            print("-" * 50)
    print(f"Total Price of All Items: {total_price:,}")

# گزینه 3: حذف کردن ایتم (امکان حذف جزئی به ازای تعداد مشخص)
def delete_item():
    if not data["categories"]:
        print("No categories found.")
        return
    # نمایش دسته‌بندی‌های موجود
    print("Available categories:")
    for i, category in enumerate(data["categories"].keys(), start=1):
        print(f"{i}. {category}")
    cat_choice = input("Select category by name or number: ").strip()
    if cat_choice.isdigit():
        index = int(cat_choice) - 1
        categories = list(data["categories"].keys())
        if index < 0 or index >= len(categories):
            print("Invalid category selection.")
            return
        category = categories[index]
    else:
        category = cat_choice
        if category not in data["categories"]:
            print("Category not found.")
            return
    # نمایش ایتم‌های موجود در دسته‌بندی انتخابی
    items = data["categories"][category]
    if not items:
        print("No items in this category.")
        return
    print(f"Items in category '{category}':")
    for i, item in enumerate(items, start=1):
        total = item["quantity"] * item["price"]
        print(f"{i}. {item['name']} ({item['quantity']} {item['unit']} x {item['price']} = {total}) added at {item['timestamp']}")
    try:
        item_choice = int(input("Select item number to delete: ").strip()) - 1
        if item_choice < 0 or item_choice >= len(items):
            print("Invalid item number.")
            return
        item = items[item_choice]
        max_qty = item["quantity"]
        # درخواست تعداد برای حذف (امکان حذف جزئی)
        while True:
            try:
                qty = int(input(f"Enter quantity to delete (1-{max_qty}): ").strip())
                if qty < 1 or qty > max_qty:
                    print("Invalid quantity.")
                    continue
                break
            except ValueError:
                print("Invalid input. Enter an integer.")
        if qty == max_qty:
            removed = items.pop(item_choice)
            action = "completely removed"
        else:
            item["quantity"] -= qty
            removed = item.copy()
            removed["quantity"] = qty
            action = "partially removed"
        save_data()
        print(f"{action} {qty} x {removed['name']} from category '{category}'.")
    except ValueError:
        print("Invalid selection.")

# گزینه 4: ویرایش ایتم (ویرایش نام، تعداد و قیمت)
def edit_item():
    if not data["categories"]:
        print("No categories found.")
        return
    
    print("Available categories:")
    categories = list(data["categories"].keys())
    for i, category in enumerate(categories, start=1):
        print(f"{i}. {category}")
    
    cat_choice = input("Select category by name or number: ").strip()
    if cat_choice.isdigit():
        index = int(cat_choice) - 1
        if 0 <= index < len(categories):
            category = categories[index]
        else:
            print("Invalid category selection.")
            return
    else:
        if cat_choice not in data["categories"]:
            print("Category not found.")
            return
        category = cat_choice
    
    new_category_name = input(f"Current category name is '{category}'. Enter new name (leave blank to keep): ").strip()
    if new_category_name and new_category_name != category:
        if new_category_name in data["categories"]:
            print("Category name already exists. Keeping the old name.")
        else:
            data["categories"][new_category_name] = data.get(category, [])
            del data["categories"][category]
            category = new_category_name
            print(f"Category renamed to '{new_category_name}'")
    
    items = data["categories"].get(category, [])
    if not items:
        print("No items in this category.")
        return
    
    print(f"Items in category '{category}':")
    for i, item in enumerate(items, start=1):
        total = item["quantity"] * item["price"]
        print(f"{i}. {item['name']} ({item['quantity']} {item['unit']} x {item['price']} = {total}) ")
    
    try:
        item_choice = int(input("Select item number to edit: ").strip()) - 1
        if item_choice < 0 or item_choice >= len(items):
            print("Invalid item number.")
            return
        
        item = items[item_choice]
        new_name = input(f"Current name is '{item['name']}'. Enter new name (leave blank to keep): ").strip()
        new_quantity = input(f"Current quantity is {item['quantity']}. Enter new quantity (leave blank to keep): ").strip()
        new_price = input(f"Current price is {item['price']}. Enter new price (leave blank to keep): ").strip()
        
        # بخش جدید: ویرایش واحد (unit)
        units = ["kg", "g", "number", "cm", "m", "mm", "m3", "m2", "liter", "pack", "box"]
        print(f"\nCurrent unit is '{item['unit']}'. Available units:")
        for i, unit in enumerate(units, start=1):
            print(f"{i}. {unit}")
        
        while True:
            unit_choice = input("Select new unit by number (leave blank to keep current): ").strip()
            if not unit_choice:
                break  # عدم تغییر اگر کاربر چیزی وارد نکرد
            if unit_choice.isdigit():
                index = int(unit_choice) - 1
                if 0 <= index < len(units):
                    item["unit"] = units[index]
                    break
                else:
                    print("Invalid selection. Please enter a valid number.")
            else:
                print("Invalid input. Please enter a number.")
        
        if new_name:
            item["name"] = new_name
        
        if new_quantity:
            try:
                qty = int(new_quantity)
                if qty > 0:
                    item["quantity"] = qty
                else:
                    print("Quantity must be positive. Keeping old value.")
            except ValueError:
                print("Invalid quantity. Keeping old value.")
        
        if new_price:
            try:
                price = float(new_price)
                if price > 0:
                    item["price"] = price
                else:
                    print("Price must be positive. Keeping old value.")
            except ValueError:
                print("Invalid price. Keeping old value.")
        
        save_data()
        print("Item and category (if changed) updated successfully!")
    except ValueError:
        print("Invalid selection.")

# گزینه 5: نمایش خلاصه‌ای از دسته‌بندی‌ها و ایتم‌ها
def display_summary():
    print("\nSummary of Categories and Items:")
    print("-" * 50)
    if not data["categories"]:
        print("No categories found.")
        return
    grand_total = 0.0
    for category, items in data["categories"].items():
        cat_total_quantity = 0
        cat_total_spent = 0.0
        print(f"Category: {category}")
        if not items:
            print("  No items in this category.")
        else:
            for item in items:
                cat_total_quantity += item["quantity"]
                cat_total_spent += item["quantity"] * item["price"]
            print(f"  Total Items: {len(items)}")
            print(f"  Total Quantity: {cat_total_quantity:,}")
            print(f"  Total Spent: {cat_total_spent:,}")
            grand_total += cat_total_spent
        print("-" * 50)
    print(f"GRAND TOTAL SPENT: {grand_total:,}")

# گزینه 6: گزارش گیری پیشرفته (زیرمنو برای سه گزارش)
def advanced_report():
    print("\nAdvanced Reporting:")
    print("1. Most expensive/cheapest purchase per category")
    print("2. Price trend for a product in a time interval")
    print("3. Monthly cost comparison")
    choice = input("Select report option: ").strip()
    if choice == "1":
        report_expensive_cheapest()
    elif choice == "2":
        report_price_trend()
    elif choice == "3":
        report_monthly_comparison()
    else:
        print("Invalid selection.")

# زیرتابع گزارش: نمایش گران‌ترین و ارزان‌ترین خرید در هر دسته
def report_expensive_cheapest():
    print("\nMost Expensive/Cheapest Purchase per Category:")
    print("-" * 50)
    if not data["categories"]:
        print("No categories found.")
        return
    for category, items in data["categories"].items():
        if not items:
            print(f"Category: {category} has no items.")
            continue
        most_expensive = max(items, key=lambda x: x["price"])
        cheapest = min(items, key=lambda x: x["price"])
        print(f"Category: {category}")
        print(f"  Most Expensive: {most_expensive['name']} at {most_expensive['price']:,}")
        print(f"  Cheapest: {cheapest['name']} at {cheapest['price']:,}")
        print("-" * 50)

# زیرتابع گزارش: نمایش سیر تغییرات قیمت برای یک محصول در بازه زمانی مشخص
def report_price_trend():
    print("\nPrice Trend Report for a Product:")
    if not data["categories"]:
        print("No categories found.")
        return
    
    print("Available categories:")
    categories = list(data["categories"].keys())
    for i, category in enumerate(categories, start=1):
        print(f"{i}. {category}")
    
    while True:
        cat_choice = input("Select category by name or number: ").strip()
        if cat_choice.isdigit():
            index = int(cat_choice) - 1
            if 0 <= index < len(categories):
                category = categories[index]
                break
            else:
                print("Invalid selection. Please enter a valid category number.")
        elif cat_choice in categories:
            category = cat_choice
            break
        else:
            print("Category not found. Please enter a valid category name or number.")
    
    items = data["categories"].get(category, [])
    if not items:
        print("No items found in this category.")
        return
    
    print(f"Available items in category '{category}':")
    unique_items = sorted(set(item["name"] for item in items))
    for i, item_name in enumerate(unique_items, start=1):
        print(f"{i}. {item_name}")
    
    while True:
        item_choice = input("Select product by name or number: ").strip()
        if item_choice.isdigit():
            index = int(item_choice) - 1
            if 0 <= index < len(unique_items):
                product = unique_items[index]
                break
            else:
                print("Invalid selection. Please enter a valid item number.")
        elif item_choice in unique_items:
            product = item_choice
            break
        else:
            print("Item not found. Please enter a valid item name or number.")
    
    default_start_date = "1999-01-01"
    default_end_date = datetime.now().strftime("%Y-%m-%d 23:59:59")
    
    start_date_str = input(f"Enter start date (YYYY-MM-DD) [default: {default_start_date}]: ").strip() or default_start_date
    end_date_str = input(f"Enter end date (YYYY-MM-DD) [default: {default_end_date}]: ").strip() or default_end_date
    
    try:
        start_date = datetime.fromisoformat(start_date_str)
        end_date = datetime.fromisoformat(end_date_str)
    except ValueError:
        print("Invalid date format.")
        return
    
    trend_items = []
    for item in items:
        if item["name"].lower() != product.lower():
            continue
        
        if "timestamp" not in item:
            print(f"Skipping item '{item['name']}' due to missing timestamp.")
            continue
        
        try:
            item_date = datetime.fromisoformat(item["timestamp"])
        except ValueError:
            print(f"Skipping item '{item['name']}' due to invalid timestamp format.")
            continue
        
        if start_date <= item_date <= end_date:
            trend_items.append((item_date, item["price"]))
    
    if not trend_items:
        print(f"No records found for '{product}' in the specified time interval.")
        return
    
    trend_items.sort(key=lambda x: x[0])
    print(f"\nPrice trend for '{product}' in category '{category}':")
    for date_val, price in trend_items:
        print(f"{date_val.strftime('%Y-%m-%d %H:%M:%S')} - Price: {price}")


# زیرتابع گزارش: مقایسه هزینه‌ها در سطح ماهیانه
def report_monthly_comparison():
    print("\nMonthly Cost Comparison:")
    monthly_totals = {}
    for category, items in data["categories"].items():
        for item in items:
            try:
                dt = datetime.fromisoformat(item["timestamp"])
            except ValueError:
                continue
            month_key = dt.strftime("%Y-%m")
            total = item["quantity"] * item["price"]
            monthly_totals[month_key] = monthly_totals.get(month_key, 0) + total
    if not monthly_totals:
        print("No records found.")
        return
    for month in sorted(monthly_totals.keys()):
        print(f"{month}: Total Spent = {monthly_totals[month]}")

# گزینه 7: گزارش ماهانه برای یک محصول در یک دسته‌بندی
def monthly_report():
    print("\nMonthly Report for a Product:")
    if not data["categories"]:
        print("No categories found.")
        return
    
    print("Available categories:")
    categories = list(data["categories"].keys())
    for i, cat in enumerate(categories, start=1):
        print(f"{i}. {cat}")
    
    while True:
        cat_choice = input("Select category by name or number: ").strip()
        if cat_choice.isdigit():
            index = int(cat_choice) - 1
            if 0 <= index < len(categories):
                category = categories[index]
                break
            else:
                print("Invalid selection. Please enter a valid category number.")
        elif cat_choice in categories:
            category = cat_choice
            break
        else:
            print("Category not found. Please enter a valid category name or number.")
    
    items = data["categories"].get(category, [])
    if not items:
        print("No items found in this category.")
        return
    
    print(f"Available items in category '{category}':")
    unique_items = sorted(set(item["name"] for item in items))
    for i, item_name in enumerate(unique_items, start=1):
        print(f"{i}. {item_name}")
    
    while True:
        item_choice = input("Select product by name or number: ").strip()
        if item_choice.isdigit():
            index = int(item_choice) - 1
            if 0 <= index < len(unique_items):
                product = unique_items[index]
                break
            else:
                print("Invalid selection. Please enter a valid item number.")
        elif item_choice in unique_items:
            product = item_choice
            break
        else:
            print("Item not found. Please enter a valid item name or number.")
    
    monthly_data = {}
    for item in items:
        if item["name"].lower() != product.lower():
            continue
        
        if "timestamp" not in item:
            print(f"Skipping item '{item['name']}' due to missing timestamp.")
            continue
        
        try:
            dt = datetime.fromisoformat(item["timestamp"])
        except ValueError:
            print(f"Skipping item '{item['name']}' due to invalid timestamp format.")
            continue
        
        month_key = dt.strftime("%Y-%m")
        if month_key not in monthly_data:
            monthly_data[month_key] = {"quantity": 0, "total": 0.0}
        
        monthly_data[month_key]["quantity"] += item["quantity"]
        monthly_data[month_key]["total"] += item["quantity"] * item["price"]
    
    if not monthly_data:
        print("No records found for the specified product.")
        return
    print('-'*50)
    print(f"\nMonthly report for '{product}' in category '{category}':")
    for month in sorted(monthly_data.keys()):
        print(f"{month}: Quantity = {monthly_data[month]['quantity']}, Total Spent = {monthly_data[month]['total']:,}\n")
    print('-'*50)

# گزینه 8: پیشنهاد خرید هوشمند بر اساس تاریخچه قیمت
def intelligent_suggestion():
    print("\nIntelligent Purchase Suggestion:")
    suggestions = []

    product_dict = {}
    for category, items in data["categories"].items():
        for item in items:
            key = (category, item["name"])
            if key not in product_dict:
                product_dict[key] = []

            if "timestamp" not in item:
                print(f"Skipping item '{item['name']}' due to missing timestamp.")
                continue

            try:
                dt = datetime.fromisoformat(item["timestamp"])
            except ValueError:
                print(f"Skipping item '{item['name']}' due to invalid timestamp format.")
                continue

            product_dict[key].append((dt, item["price"]))

    for key, records in product_dict.items():
        records.sort(key=lambda x: x[0])
        prices = [price for (_, price) in records]
        avg_price = sum(prices) / len(prices)
        last_price = prices[-1]

        if avg_price > 0 and last_price < avg_price:
            drop_percent = ((avg_price - last_price) / avg_price) * 100
            suggestions.append((key, last_price, avg_price, drop_percent))

    if not suggestions:
        print("No suggestions available at this time.")
        return

    suggestions.sort(key=lambda x: x[3], reverse=True)
    for (category, product), last_price, avg_price, drop_percent in suggestions:
        print(f"Category: {category}, Product: {product}, Last Price: {last_price:,}, Average Price: {avg_price:,.2f}, Drop: {drop_percent:,.2f}%")

# گزینه 9: ریست کردن و پاک کردن تمام داده‌ها
def reset_data():
    global data
    confirmation = input("Are you sure you want to reset all data? (yes/no): ").strip().lower()
    if confirmation == "yes":
        data = {"categories": {}}
        save_data()
        print("All data has been reset.")
    else:
        print("Reset canceled.")

def analyze_trends():
    print("\nInventory Analysis Report:")
    print("-" * 50)
    total_price_change = 0
    total_quantity_change = 0
    for category, items in data["categories"].items():
        category_price_change = 0
        category_quantity_change = 0
        print(f"Category: {category}")
        item_trends = {}
        
        for item in items:
            name = item["name"]
            if name not in item_trends:
                item_trends[name] = {"initial_qty": item["quantity"], "final_qty": item["quantity"],
                                     "initial_price": item["price"], "final_price": item["price"]}
            else:
                item_trends[name]["final_qty"] = item["quantity"]
                item_trends[name]["final_price"] = item["price"]
        
        for name, values in item_trends.items():
            qty_change = values["final_qty"] - values["initial_qty"]
            price_change = ((values["final_price"] - values["initial_price"]) / values["initial_price"]) * 100 if values["initial_price"] > 0 else 0
            print(f"  {name}: Quantity Change: {qty_change}, Price Change: {price_change:.2f}%")
            category_price_change += values["final_price"] - values["initial_price"]
            category_quantity_change += qty_change
        
        print(f"Total category quantity change: {category_quantity_change}")
        print(f"Total category price change: {category_price_change:.2f}")
        total_price_change += category_price_change
        total_quantity_change += category_quantity_change
        print("-" * 50)
    
    print(f"Total inventory quantity change: {total_quantity_change}")
    print(f"Total inventory price change: {total_price_change:.2f}")


# تابع اصلی: نمایش منو و دریافت دستورات از کاربر
def main():
    print("Welcome to the Shopping Tracker!")
    while True:
        print('''
1. Add Item (with Category, Name, Quantity, Price, Date/Time)
2. Display All Items
3. Delete Item
4. Edit Item
5. Summary of Categories and Items
6. Advanced Reporting (Expensive/Cheapest, Price Trend, Monthly Comparison)
7. Monthly Report for a Product
8. Intelligent Purchase Suggestion
9. Analyze Inventory Trends (Quantity & Price Changes)
10. Reset All Data
11. Exit
''')
        command = input("Enter a command (1-10): ").strip()
        if command == "1":
            add_item()
        elif command == "2":
            display_all()
        elif command == "3":
            delete_item()
        elif command == "4":
            edit_item()
        elif command == "5":
            display_summary()
        elif command == "6":
            advanced_report()
        elif command == "7":
            monthly_report()
        elif command == "8":
            intelligent_suggestion()
        elif command == "9":
            analyze_trends()
        elif command == "10" :
            reset_data()
        elif command == "11":
            print("Goodbye!")
            break
        else:
            print("Invalid command. Please try again.")

if __name__ == "__main__":
    main()
