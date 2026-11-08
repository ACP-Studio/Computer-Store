# PC Parts Store

A command-line computer parts store written in object-oriented C++17. Browse real PC parts, add them to a basket, remove them again, check that your parts work together, and check out with a saved receipt.

It is a single file (`pcstore.cpp`) with no external libraries. It uses only `using namespace std;`, with no other `using` or `namespace` anywhere.

---

## Features

- **62 real products** in 10 categories: CPUs, graphics cards, motherboards, RAM, storage, power supplies, cases, coolers, monitors and peripherals
- **Browse** by category, **search** by any word, or list every product
- **Product pages** with full specs, price and stock
- **Basket:** add items, remove an item, change a quantity, or empty the basket
- **Stock control:** you can't add more than what is in stock
- **Compatibility check** that warns about parts that don't work together
- **Checkout** that prints a receipt and saves it to a text file
- **Input validation:** typing letters or an out-of-range number just asks again

---

## Build and run

```bash
g++ -std=c++17 -O2 -Wall -Wextra -o pcstore pcstore.cpp
./pcstore
```

On Windows (MinGW), build the same way and run `pcstore.exe`. The program works on Linux, macOS and Windows.

---

## How to use it

```
--- MAIN MENU ---   Basket: 0 item(s), $0.00
  1. Browse by category
  2. Search products
  3. Show every product
  4. View basket (remove items / change quantity)
  5. Checkout
  0. Exit
```

1. **Find a part:** use *Browse* or *Search*. Search matches the brand, name, category and specs, so `rtx 5070`, `ddr5`, `am5`, `oled` or `samsung` all work. Every word you type must match.
2. **Open it:** type a product ID from the list to see its specs.
3. **Add it:** choose *Add to basket* and enter a quantity.
4. **Manage the basket:** from the main menu choose *View basket*, then:
   - *Remove an item* by its number in the basket
   - *Change an item's quantity* (set it to 0 to remove it)
   - *Empty the basket*
5. **Check out:** confirm the order and enter your name. The stock is reduced, a receipt is shown, and it is saved as `receipt_<date>_<time>_<order>.txt` in the folder you ran the program from.

### Example basket

```
=== YOUR BASKET ===
#   PRODUCT                                       PRICE  QTY        TOTAL
-------------------------------------------------------------------------
1   AMD Ryzen 7 7800X3D                         $399.00    1      $399.00
2   ASUS TUF GAMING B650-PLUS WIFI              $199.00    1      $199.00
3   Kingston FURY Beast 16GB                     $39.00    2       $78.00
-------------------------------------------------------------------------
    TOTAL (4 items)                                               $676.00

Compatibility check:
  [OK] CPU and motherboard share the AM5 socket.
  [!] The memory is DDR4 but the motherboard needs DDR5.
```

---

## Compatibility check

The basket checks the parts you have chosen and shows `[OK]` or `[!]` lines:

| Check | Rule |
|-------|------|
| CPU ↔ motherboard | The CPU socket must match the motherboard socket (AM4, AM5, LGA1700, LGA1851) |
| RAM ↔ motherboard | The memory type must match (DDR4 or DDR5) |
| Motherboard ↔ case | The case must take that size: Mini-ITX, Micro-ATX or ATX |
| Power supply | Wattage must cover the CPU power + GPU power + 150 W for the rest of the PC |
| Duplicates | Warns if you have more than one CPU, motherboard, case or power supply |

These are warnings only. You can still check out, so you stay in control.

---

## Code structure (OOP)

```
Product (abstract base class)
 ├─ Cpu
 ├─ Gpu
 ├─ Motherboard
 ├─ Memory
 ├─ Storage
 ├─ PowerSupply
 ├─ PcCase
 ├─ Cooler
 ├─ Monitor
 └─ Peripheral
```

| Class | Job |
|-------|-----|
| `Product` | Base class with ID, brand, name, price and stock. Each child class overrides `specs()` to describe itself (polymorphism) |
| `Catalog` | Owns all products with `unique_ptr`. Finds by ID, filters by category, searches |
| `BasketItem` / `Basket` | Holds the chosen products and quantities. Add, remove, change quantity, totals, stock checks |
| `CompatibilityChecker` | Uses `dynamic_cast` to find the parts in the basket and compare them |
| `Console` | Safe keyboard input (numbers within a range, text lines) |
| `Store` | The menus and the whole customer flow, including checkout and receipts |

Prices are stored as whole **cents** (`long long`), so there are no floating-point rounding errors.

---

## Adding or changing products

All products are listed in `Catalog::populate()`. Each product is one line, and its ID is assigned automatically.

```cpp
//               brand,    name,               price,    stock, VRAM GB, power W
add<Gpu>("NVIDIA", "GeForce RTX 5070", usd(549), 10, 12, 250);
```

Each product type takes different details, and the comment above each group in the file shows the order. For example, a CPU line is `brand, name, price, stock, socket, cores, threads, base GHz, boost GHz, TDP`.

To add a **new kind** of product, create a class that inherits from `Product` and overrides `specs()`, add a value to the `Category` enum, then add its products in `populate()`.

> **Important:** The prices in the program are approximate US retail prices, **not live prices**. Update them to match your supplier before you use the store for real.

---

## Limitations

- Stock resets every time the program starts. Nothing is saved between runs except the receipt files.
- There is no payment, tax, shipping or customer account system.
- The compatibility check covers the main points (sockets, RAM type, case size, power). It does not check things like cooler socket support, GPU length or case clearance.
- Prices and availability are example data.
