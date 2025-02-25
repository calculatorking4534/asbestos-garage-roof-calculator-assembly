# 🏠 Asbestos Garage Roof Cost Calculator (x86-64 Assembly)

A lightweight **CLI-based cost estimator** for replacing an asbestos garage roof, written in **x86-64 Assembly (NASM)** for **Linux**. This program calculates the **total estimated cost** based on the **garage size** and **roofing material**, using real-world data sourced from **Excel Roofing & Asbestos Ltd (2025)**.

## 🚀 Features

- **Three garage size options** (Single, Double, Three-Car)
- **Five roofing material choices** (Steel, EPDM, GRP Fibreglass, Onduline, Concrete Fibre)
- **Accurate cost breakdown** including:
  - **Asbestos removal cost**
  - **Material cost per m²**
  - **Additional structural expenses**
- **Efficient integer math operations** using Assembly
- **Lightweight** - no dependencies, runs directly in Linux

## 📜 How It Works

The total estimated cost is calculated as:

Total Cost = Asbestos Removal Cost + (Garage Area × Material Cost per m²) + Additional Costs

### 🔢 Example Calculation

For a **Single Garage (18m²) with Steel Roofing**:

Removal Cost: £800 Material Cost: £60 per m² Additional Cost: £750
Total Cost = 800 + (18 × 60) + 750 = £2,630


## 🛠 Installation & Compilation

### Prerequisites

- **NASM** (Netwide Assembler)
- **Linux (x86-64)**
- **GNU LD (linker)**

### Compilation

Clone this repository and assemble the program:

git clone https://github.com/yourusername/asbestos-roof-cost-calculator.git
cd asbestos-roof-cost-calculator
nasm -f elf64 calculator.asm -o calculator.o
ld -o calculator calculator.o

🖥️ Usage

The program prompts for two user inputs:

Garage Type:
Select Garage Type:
(1) Single Garage (18 m²)
(2) Double Garage (33 m²)
(3) Three-Car Garage (54 m²)
Enter choice (1-3): _

Roofing Material:

Select Roofing Material:
(1) Steel
(2) EPDM Rubber
(3) GRP Fibreglass
(4) Onduline Bitumen
(5) Concrete Fibre
Enter choice (1-5): _

🔧 Code Structure
Prompt user input for garage type and material
Validate selection (default to first option if invalid)
Perform integer calculations (imul, add)
Convert result to string for display (itoa)
Print total cost to terminal

📝 Future Enhancements
Add floating-point support for more precise calculations
Implement a Windows-compatible version (using MASM)
Support dynamic pricing input instead of fixed values

📜 License
This project is licensed under the MIT License.

👨‍💻 References
Inspired by Excel Roofing & Asbestos' handy guide to garage roof replacement costs https://asbestos-roofing.co.uk/asbestos-roof-replacements/



