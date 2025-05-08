# Nutrient Calculation

## Overview

This project calculates the nutritional breakdown of a meal based on individual food items and their servings. It includes several steps, each performing a specific task to achieve the final result.

---

## Steps

### **Step 1: Nutrient Fetch Function**
```python
def get_nutrient_per_serving(food_code, df_nutrients):
    """Return the nutrient values for a given food_code (per serving)."""
    row = df_nutrients[df_nutrients['food_code'] == food_code]
    if row.empty:
        raise ValueError(f"Food code {food_code} not found in nutrient data.")
    
    return {
        'kcal': row['unit_serving_energy_kcal'].values[0],
        'carbs_g': row['unit_serving_carb_g'].values[0],
        'protein_g': row['unit_serving_protein_g'].values[0],
        'fat_g': row['unit_serving_fat_g'].values[0],
        'fiber_g': row['unit_serving_fibre_g'].values[0],
        'sugar_g': row['unit_serving_freesugar_g'].values[0],
    }
```

**Explanation**:  
This function retrieves the nutrient values for a specific food item based on its `food_code`. It looks up the `df_nutrients` DataFrame to find the row corresponding to the given `food_code`. If the `food_code` is not found, it raises an error. Otherwise, it extracts and returns the nutrient values (e.g., calories, carbs, protein, etc.) for one serving of the food.

---

### **Step 2: Scaling by User Portion**
```python
def scale_nutrients(nutrient_dict, servings):
    """Scale nutrient values based on number of servings."""
    return {k: v * servings for k, v in nutrient_dict.items()}
```

**Explanation**:  
This function scales the nutrient values based on the number of servings. It multiplies each nutrient value in the `nutrient_dict` by the number of servings provided.

---

### **Step 3: Total Nutrient Calculation Utility (with Names)**
```python
def calculate_meal_nutrients_with_names(components, df_nutrients):
    """
    Returns both the total nutrients and a breakdown by item with names.
    """
    total = {
        'kcal': 0, 'carbs_g': 0, 'protein_g': 0, 
        'fat_g': 0, 'fiber_g': 0, 'sugar_g': 0
    }

    breakdown = []  # To hold individual food item names and scaled nutrients

    for food_code, servings in components:
        row = df_nutrients[df_nutrients['food_code'] == food_code]
        if row.empty:
            raise ValueError(f"Food code {food_code} not found.")
        
        food_name = row['food_name'].values[0]
        nutrient_per_serving = get_nutrient_per_serving(food_code, df_nutrients)
        scaled = scale_nutrients(nutrient_per_serving, servings)

        # Add to breakdown
        breakdown.append({
            'food_name': food_name,
            'servings': servings,
            **scaled
        })

        # Add to total
        for k in total:
            total[k] += scaled.get(k, 0)
    
    return total, breakdown
```

**Explanation**:  
This function calculates the total nutrients for a meal and provides a breakdown of nutrients for each food item. It iterates through the `components` list, where each item is a tuple of `food_code` and `servings`. For each food item:
1. It fetches the nutrient values per serving using `get_nutrient_per_serving`.
2. It scales the nutrients based on the number of servings using `scale_nutrients`.
3. It adds the scaled nutrients to the `total` dictionary and appends the breakdown for the food item (including its name and scaled nutrients).

---

### **Step 4: Example Meal**
```python
meal = [("ASC144", 5), ("ASC167", 1)]
total, breakdown = calculate_meal_nutrients_with_names(meal, df_nutrients)
```

**Explanation**:  
This is an example meal consisting of 5 servings of `ASC144` (e.g., idli) and 1 serving of `ASC167` (e.g., sambar). The `calculate_meal_nutrients_with_names` function is used to compute the total nutrients and breakdown.

---

### **Step 5: Display Result Nicely**
```python
def pretty_print_with_breakdown(total_nutrients, breakdown):
    print("\n🍽️ Meal Components Breakdown:")
    for item in breakdown:
        print(f"\n🧾 {item['food_name']} (x {item['servings']}):")
        print(f"  Calories : {item['kcal']:.2f} kcal")
        print(f"  Carbs    : {item['carbs_g']:.2f} g")
        print(f"  Protein  : {item['protein_g']:.2f} g")
        print(f"  Fat      : {item['fat_g']:.2f} g")
        print(f"  Fiber    : {item['fiber_g']:.2f} g")
        print(f"  Sugar    : {item['sugar_g']:.2f} g")

    print("\n✅ Total Nutrient Summary:")
    for k, v in total_nutrients.items():
        print(f"{k:<12}: {v:.2f}")
```

**Explanation**:  
This function formats and prints the nutrient breakdown for each food item and the total nutrient summary in a user-friendly way.

---

## Example Output
```
🍽️ Meal Components Breakdown:

🧾 Idli (x 5):
  Calories : 250.00 kcal
  Carbs    : 50.00 g
  Protein  : 10.00 g
  Fat      : 2.50 g
  Fiber    : 5.00 g
  Sugar    : 1.00 g

🧾 Sambar (x 1):
  Calories : 150.00 kcal
  Carbs    : 30.00 g
  Protein  : 5.00 g
  Fat      : 2.00 g
  Fiber    : 3.00 g
  Sugar    : 0.50 g

✅ Total Nutrient Summary:
kcal        : 400.00
carbs_g     : 80.00
protein_g   : 15.00
fat_g       : 4.50
fiber_g     : 8.00
sugar_g     : 1.50
```
