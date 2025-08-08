<img width="3188" height="1202" alt="frame (3)" src="https://github.com/user-attachments/assets/517ad8e9-ad22-457d-9538-a9e62d137cd7" />


SASSY CALCULATOR


### Team Name: Fames


### Team Members
- Team Lead: Dawn Baby- Viswajyothi College of Engineering and Technology 
- Member 2: Neithan Benoy John - Viswajyothi College of Engineering and Technology

### Project Description
The Sassy Calculator is a playful twist on a normal calculator built using Python’s Tkinter GUI library.
Instead of just silently showing results, it sometimes gives funny or sarcastic remarks when you enter overly simple math (like 2+2) or certain patterns.

### The Problem (that doesn't exist)
The main problem of the Sassy Calculator is that it’s not purely a serious tool — its sass can sometimes get in the way of actual calculation.
For certain inputs like 2+2 or 3-3, instead of giving the numerical answer, it gives a sarcastic remark.
It supports only basic arithmetic; advanced operations like square roots, percentages, or trigonometry aren’t built in.

### The Solution (that nobody asked for)
We are creating a sassy calculator that displays taunts in Malayalam when simple arithmetic operations are given ( for example 2+2 ) 

## Technical Details
### Technologies/Components Used
For Software:
- The main language used to implement the sassy calculator is Pythong
- Tkinter and pyttsx3 are some frameworks used to create the GUI library for creating buttons etc and pyttsx3 is used to make the calculator deliver the sassy comments.
- VS code is used 

# Installation

#!/usr/bin/env python3
"""
Sassy Calculator — Full Tkinter Keypad Edition
Safe arithmetic evaluator with sarcastic responses for trivial problems.
"""

import ast
import operator
import random
import tkinter as tk

# --- Optional speech ---
try:
    import pyttsx3
    SPEECH = True
    engine = pyttsx3.init()
except Exception:
    SPEECH = False

# --- Sass ---
TAUNTS = [
    "നിനക്കിനിയും മതിയായില്ലേ",
    "തരത്തില്ല മാമനോട് ഒന്നും തോന്നല്ലേ",
    "കഴിവില്ലെടാ നിനക്ക്",
    "പറ്റണ പണി വല്ലോ ചെയ്യടെ"
]

def speak(text: str):
    display_var.set(text)
    if SPEECH:
        try:
            engine.say(text)
            engine.runAndWait()
        except Exception:
            display_var.set("(speech failed) " + text)

# --- Safe evaluation setup ---
ALLOWED_BINOPS = {
    ast.Add: operator.add,
    ast.Sub: operator.sub,
    ast.Mult: operator.mul,
    ast.Div: operator.truediv,
    ast.FloorDiv: operator.floordiv,
    ast.Mod: operator.mod,
    ast.Pow: operator.pow
}
ALLOWED_UNARYOPS = {
    ast.UAdd: operator.pos,
    ast.USub: operator.neg
}

def eval_node(node):
    if isinstance(node, ast.Expression):
        return eval_node(node.body)
    if isinstance(node, ast.Constant):
        if isinstance(node.value, (int, float)):
            return node.value
        raise ValueError("Only numerical constants allowed")
    if isinstance(node, ast.Num):
        return node.n
    if isinstance(node, ast.BinOp):
        op_type = type(node.op)
        if op_type not in ALLOWED_BINOPS:
            raise ValueError(f"Operator {op_type} not allowed")
        left = eval_node(node.left)
        right = eval_node(node.right)
        return ALLOWED_BINOPS[op_type](left, right)
    if isinstance(node, ast.UnaryOp):
        op_type = type(node.op)
        if op_type not in ALLOWED_UNARYOPS:
            raise ValueError(f"Unary operator {op_type} not allowed")
        operand = eval_node(node.operand)
        return ALLOWED_UNARYOPS[op_type](operand)
    raise ValueError(f"Expression element {type(node)} not allowed")

def is_trivial_expression(expr_ast):
    node = expr_ast
    if isinstance(node, ast.Expression):
        node = node.body
    if isinstance(node, ast.BinOp):
        left = node.left
        right = node.right
        if isinstance(left, (ast.Constant, ast.Num)) and isinstance(right, (ast.Constant, ast.Num)):
            lval = left.n if isinstance(left, ast.Num) else left.value
            rval = right.n if isinstance(right, ast.Num) else right.value
            if isinstance(lval, int) and isinstance(rval, int):
                if abs(lval) <= 12 and abs(rval) <= 12:
                    if type(node.op) in (ast.Add, ast.Sub, ast.Mult, ast.Div):
                        return True
    return False

def safe_eval(expr: str):
    parsed = ast.parse(expr, mode='eval')
    return eval_node(parsed)

# --- Calculator logic ---
def button_click(value):
    current = display_var.get()
    if "Sassy Calculator" in current or current.startswith("=") or "Nope" in current:
        display_var.set("")
        current = ""
    display_var.set(current + str(value))

def clear_display():
    display_var.set("")

def calculate():
    expr = display_var.get().strip()
    if not expr:
        return

    force_answer = False
    if expr.endswith("!!"):
        force_answer = True
        expr = expr[:-2].strip()

    try:
        parsed = ast.parse(expr, mode='eval')
    except Exception:
        speak("I can't even parse that. Try a real expression.")
        return

    try:
        if not force_answer and is_trivial_expression(parsed):
            speak(random.choice(TAUNTS))
            return
        result = safe_eval(expr)
        speak(f"= {result}")
    except Exception:
        speak("Nope. That expression is not allowed or is invalid.")

# --- GUI Setup ---
root = tk.Tk()
root.title("Sassy Calculator")

display_var = tk.StringVar()
display_var.set("Sassy Calculator")

# Longer display box
display = tk.Entry(
    root,
    textvariable=display_var,
    font=("Arial", 18),
    justify='right',
    bd=10,
    relief="sunken",
    width=35  # Increased width so longer outputs fit
)
display.grid(row=0, column=0, columnspan=4, pady=5)

buttons = [
    ('7', 1, 0), ('8', 1, 1), ('9', 1, 2), ('/', 1, 3),
    ('4', 2, 0), ('5', 2, 1), ('6', 2, 2), ('*', 2, 3),
    ('1', 3, 0), ('2', 3, 1), ('3', 3, 2), ('-', 3, 3),
    ('0', 4, 0), ('.', 4, 1), ('**', 4, 2), ('+', 4, 3),
    ('(', 5, 0), (')', 5, 1), ('//', 5, 2), ('%', 5, 3),
    ('!!', 6, 0), ('C', 6, 1), ('=', 6, 2),
]

for (text, row, col) in buttons:
    if text == '=':
        btn = tk.Button(root, text=text, width=5, height=2, font=("Arial", 14), command=calculate)
    elif text == 'C':
        btn = tk.Button(root, text=text, width=5, height=2, font=("Arial", 14), command=clear_display)
    else:
        btn = tk.Button(root, text=text, width=5, height=2, font=("Arial", 14), command=lambda val=text: button_click(val))
    btn.grid(row=row, column=col, padx=3, pady=3)

root.mainloop()


# Run
[commands]

### Project Documentation
For Software:

# Screenshots (Add at least 3)
<img width="591" height="605" alt="image" src="https://github.com/user-attachments/assets/682e0149-fa3e-44ac-a1dd-450abf9c6b86" />
This shows the arithmetic operation of adding two numbers (2+2)

![Screenshot1]
<img width="592" height="608" alt="image" src="https://github.com/user-attachments/assets/39205854-b747-4e7f-b042-3f6cf83cdee1" />
This show the result showing that taunts after giving a simple arithmetic operation.


![Screenshot2]
<img width="595" height="607" alt="image" src="https://github.com/user-attachments/assets/9863971c-2fe7-4e38-ae1c-db97815d418b" />
This shows the arithmetic operation of adding two numbers (10+9)

![Screenshot3]
<img width="585" height="605" alt="image" src="https://github.com/user-attachments/assets/7026dee4-deaf-408a-affd-97f5e78e3479" />
This show the result showing that taunts after giving a simple arithmetic operation.

## Team Contributions
- [Name 1]: Dawn Baby
- [Name 2]: Neithan Benoy

---
Made with ❤️ at TinkerHub Useless Projects 

![Static Badge](https://img.shields.io/badge/TinkerHub-24?color=%23000000&link=https%3A%2F%2Fwww.tinkerhub.org%2F)
![Static Badge](https://img.shields.io/badge/UselessProjects--25-25?link=https%3A%2F%2Fwww.tinkerhub.org%2Fevents%2FQ2Q1TQKX6Q%2FUseless%2520Projects)



