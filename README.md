# Nourish — User Guide

**Your personal meal planning and grocery shopping system.**
Live at: `sunilahluwalia0777-ship-it.github.io/nourish`

> **2026-04-26 cutover:** the apps now read and write to a private NUC
> backend (`nutrition-webhook` Cloudflare Worker → SQLite on the NUC)
> instead of the GitHub Contents API. The HTML lives here on GitHub
> Pages; the data does not. See `data/README.md` for what happened to
> the old JSON files.

---

## The two apps

| App | URL | What it does |
|---|---|---|
| **Meal Planner** | `/nourish` | Plan your week, generate grocery lists |
| **Grocery** | `/nourish/grocery.html` | Shop in-store with your list |

---

## First-time setup

1. Open the Meal Planner — you'll be redirected to a sign-in page
2. Enter the password (from your password manager — entry: `nourish/login_password`)
3. You're back at the Meal Planner. Cookie set for 30 days on this browser.

The session cookie is `HttpOnly` (JavaScript can't see it) and `SameSite=None; Secure`. It carries automatically on every cross-origin fetch from the apps to the NUC backend.

> **On a new device or browser** — open the Meal Planner. You'll be redirected to sign in. Enter the same password. Done.

> **Inside the app:** click **⚙ Settings** (Meal Planner header) or the gear in Grocery to reach the in-app settings panel: an Instacart API key field (optional), a Sign-out button, and the "Saved weeks" history list.

> **What happened to the old GitHub PAT and Anthropic key?**
> The GitHub PAT is no longer used (NUC is the data store now).
> The Anthropic key moved server-side — the browser never holds it.
> The standalone `settings.html` page was deleted in the 2026-04-26 cleanup;
> remaining preferences (Instacart key, sign out) moved into the in-app panel.

---

## Meal Planner

### Planning your week

The calendar shows 7 days across the top with four meal slots per day — Breakfast, Lunch, Snack, and Dinner.

**Clicking a meal type label** (e.g. *Breakfast*) opens the meal carousel for that meal type, targeting the first unfilled day.

**Clicking an empty slot** opens the carousel for that specific day and meal.

**The carousel** slides up from the bottom showing all your meal options as cards. Use the tabs to filter by type (Chicken, Fish, Indian, Veggie etc.) or search by name. Tap **Select** to assign a meal.

**Special options** — every carousel has two special cards at the start:
- 🚫 **Skip / None** — mark a slot as intentionally empty (e.g. intermittent fasting)
- 🍽 **Eating out** — marks the slot without adding groceries

**Flipping a card** — click any filled meal card to see a large flip card with the full recipe on the back: ingredients, step-by-step method, and complete nutrition facts (calories, protein, carbs, fat, fiber, sodium, cholesterol). Flip back by clicking again or pressing Escape.

**The recipe source** shown on each card tells you where to find the full recipe:
- 📖 Chef D'ole Recipe Book (recipe number)
- ▶ YouTube (links directly to the video)
- ✓ Quick assemble (no recipe needed)

**Adding a custom meal** — scroll to the end of any carousel and tap **+ Add a new meal**. Fill in the name, calories, macros, ingredients and steps. It's saved permanently and appears in future carousels.

**Daily macro totals** run along the bottom of the calendar, updating live as you fill in meals — Calories, Protein, Carbs and Fat per day.

---

### Generating your grocery list

When your week is planned, click **Done — get grocery list** in the top right.

This does three things:
1. Asks the NUC's `/claude/grocery-list` endpoint to consolidate the ingredients across the week into a categorised grocery list (Anthropic key lives on the NUC, not in the browser)
2. Saves your weekly plan to the NUC (`PUT /api/plan/week`)
3. Saves the grocery list to the NUC (`PUT /api/grocery/current`) so the Grocery app can read it

The grocery list appears in a panel on the right, grouped into: Produce, Proteins, Dairy & Eggs, Grains & Pantry, Condiments & Spices, and Frozen & Other.

---

### Instacart integration

**Per meal** — when viewing a recipe on the flip card back, tap **🛒 Shop on Instacart** to send that meal's ingredients directly to an Instacart cart. A link is generated and opens in Instacart where you can review and check out.

**Full week** — in the grocery panel, tap **Send full week to Instacart** to send all week's ingredients as one cart. Requires an Instacart Developer API key in Settings.

---

### Saving and history

Every completed week is automatically saved to the NUC's SQLite. The
relevant tables on the server side:

```
config         — your settings (servings, stores, family members)
meals          — preloaded + custom meal library
week_plans     — every saved week's slot assignments
grocery_items  — current week's grocery list
pantry         — what you have on hand (drives the pantry-check)
```

To load a past week, click **⬡ NUC** in the header — saved weeks appear in the panel with their average daily calories (computed live from `week_plans` joined with `meals`). Click **Load** to restore any past week to the calendar.

---

### Exporting to Excel

Click **↓ Export** in the header to download a spreadsheet with three sheets:
- **Weekly Plan** — the 7-day grid with daily macro totals
- **Daily Nutrition** — one row per meal with full nutritional detail, status and recipe source
- **Summary** — weekly and daily averages, macro ratios, meal counts (home cooked / eating out / skipped)

Use these weekly exports for historical nutrition analysis.

---

## Grocery app

Open at `/nourish/grocery.html`. Designed for use on your phone while in-store.

### Importing your list

Tap **↓ Nourish** to import the grocery list generated in the Meal Planner. The list loads automatically from GitHub.

> You need to click "Done — get grocery list" in Nourish at least once before the import works.

### Store tabs

Items are automatically assigned to stores based on what they are:

| Store | Typical items |
|---|---|
| **Costco** | Bulk proteins, oils, eggs, nuts, yogurt, cheese |
| **Indian Shop** | Spices, dals, paneer, ghee, atta, specialty items |
| **Instacart** | Fresh produce, dairy, everyday grocery items |
| **Safeway** | Same-day fresh items, gap fills |

Tap any **store pill** on an item to cycle it to the next store. Store assignments are remembered — if you move chicken to Costco this week, it stays there on future imports.

### Shopping

Switch between store tabs to see only that store's items. Tap an item to check it off — it fades and moves to the bottom of its category with a strikethrough so you can still see what you've got.

Tap **Clear done** to permanently remove all checked items from the list.

### Adding items manually

Type in the bar at the top of the list and tap **Add**. Items are auto-assigned to a store based on their name.

### Ordering Instacart items

Tap **🛒 Order Instacart items** at the bottom to open Instacart with your Instacart-assigned items.

---

## Settings

| Setting | What it does |
|---|---|
| GitHub token | Allows all apps to read/write your data repo |
| Anthropic key | Powers the grocery list generation |
| Instacart key | Enables direct cart creation on Instacart |
| Servings per meal | Default used by the meal planner |
| Default cook time | Filters out meals that take too long |
| Allergies / avoid | Passed to Claude when generating lists |
| Stores | Which stores appear in the Grocery app |
| Family members | Names and dietary preferences |

Click **Test connection** after entering GitHub details to verify the token works before saving.

---

## Tips

- **Plan Sunday evening** — fill in next week's meals, click Done, and your grocery list is ready to import on Monday
- **Store assignments stick** — assign items to stores once and they're remembered every week
- **Flip any card** to read the recipe before cooking — especially useful for the Chef D'ole recipes
- **Skip slots** you don't need rather than leaving them empty — it keeps the macro totals accurate
- **Export weekly** for nutrition tracking — stack multiple week exports in Excel to chart trends over time

---

## Your meal library

The app comes pre-loaded with **57 meals** drawn from your Chef D'ole recipe book and meal planning spreadsheets, organised across four meal types.

**Breakfasts (10):** Soaked Oats, Banana Nut Protein Muffins, Savoury Veg Muffins, Wow Protein Pancakes, Chobani Bowl, Veggie Omelette, Blueberry Scones, Avocado & Egg Toast, Greek Yogurt Parfait, Cloud Bread + Eggs

**Lunches (12):** Broccoli Chicken Bowl, Healthy Salad, Turkey Sandwich, Café Yumm Bowl, Chicken Quesadilla, Chipotle Bowl, Turkey Burger, Rice Noodle Stir-fry, Potstickers + Edamame, Tuna Lettuce Wraps, Pesto Pasta, Apple + Almond Butter

**Snacks (10):** Apple + Almond Butter, Chobani + Blueberries, Protein Muffin, Walnuts + Fruit, Hummus + Veggie Sticks, Spinach Protein Shake, Rice Cakes + Almond Butter, Hard-boiled Eggs, Keto Granola Bar, Cottage Cheese + Pineapple

**Dinners (25):** Honey Garlic Chicken, Shawarma Plate, Indian Chicken Curry, Chicken Kaathi Roll, Thai Curry, Pecan-Crusted Chicken, Buffalo Chicken Bowl, Chipotle Chicken Bowl, Salmon + White Wine, Shrimp + Cilantro Rice, Fish Tacos, Dijon Salmon, Blackened Cod, Colombian Salmon, Turkey Chili, Turkey Balls, Minced Turkey Bowl, Seekh Kabab, Daal Makhni, Paneer Kathi Roll, Rajma Chawal, Aloo Gobi + Daal, Saag + Makki Roti, Bhindi + Daal + Roti, Egg Curry + Roti

You can add unlimited custom meals through the carousel's **+ Add a new meal** card.
