import tkinter as tk
from tkinter import ttk, messagebox
import sympy as sp
import re

def validate_expression(expr_str):
    """
    Checks if the input is a valid mathematical expression 
    and blocks random text string inputs.
    """
    clean_expr = expr_str.replace(" ", "")
    
    valid_keywords = ['pi', 'E', 'oo', 'sin', 'cos', 'tan', 'exp', 'log', 'sqrt']
    
    temp_expr = clean_expr
    for kw in valid_keywords:
        temp_expr = temp_expr.replace(kw, "")
        
    words = re.findall(r'[a-zA-Z]+', temp_expr)
    
    for word in words:
        if len(word) > 2:
            return False, f"Invalid variable or function: '{word}'. Please use standard math symbols (e.g., x, y, t)."
            
    return True, ""


def calculate_results():
    """
    Core Evaluation Function
    Automatically detects variables and evaluates calculus operations.
    """
    root.focus()
    
    expr_str = expr_entry.get().strip()
    op_type = op_combobox.get()
    
    if not expr_str:
        result_label_output.config(text="INVALID EXPRESSION! Please enter a math expression.", fg="#D32F2F")
        messagebox.showerror("Input Error", "Please provide a mathematical expression.")
        status_label.config(text="Status: Missing expression", fg="#FFCDD2")
        return

    is_valid, err_msg = validate_expression(expr_str)
    if not is_valid:
        result_label_output.config(text="INVALID EXPRESSION! Please type a valid math expression.", fg="#D32F2F")
        messagebox.showerror("Input Error", err_msg)
        status_label.config(text="Status: Invalid input format", fg="#FFCDD2")
        return

    try:
        local_dict = {
            'pi': sp.pi,
            'E': sp.E,
            'oo': sp.oo,
            'sin': sp.sin,
            'cos': sp.cos,
            'tan': sp.tan,
            'exp': sp.exp,
            'log': sp.log,
            'sqrt': sp.sqrt
        }
        
        parsed_expr = sp.sympify(expr_str, locals=local_dict)
        
        free_syms = sorted(parsed_expr.free_symbols, key=lambda s: s.name)
        
        if not free_syms:
            var_symbol = sp.Symbol('x')
        else:
            var_symbol = free_syms[0]
            
        var_str = str(var_symbol)

        if op_type == "Derivative (1st Order)":
            result = sp.diff(parsed_expr, var_symbol)
            result_str = f"d/d{var_str} [{parsed_expr}] = {result}"
            
        elif op_type == "Derivative (2nd Order)":
            result = sp.diff(parsed_expr, var_symbol, 2)
            result_str = f"d²/d{var_str}² [{parsed_expr}] = {result}"

        elif op_type == "Derivative (3rd Order)":
            result = sp.diff(parsed_expr, var_symbol, 3)
            result_str = f"d³/d{var_str}³ [{parsed_expr}] = {result}"

        elif op_type == "Indefinite Integral":
            result = sp.integrate(parsed_expr, var_symbol)
            result_str = f"∫ ({parsed_expr}) d{var_str} = {result} + C"
            
        elif op_type == "Definite Integral":
            lower_str = lower_limit_entry.get().strip()
            upper_str = upper_limit_entry.get().strip()
            
            if not lower_str or not upper_str:
                result_label_output.config(text="INVALID INPUT! Please specify lower and upper limits.", fg="#D32F2F")
                messagebox.showerror("Input Error", "Please specify upper and lower limits.")
                status_label.config(text="Status: Missing limits", fg="#FFCDD2")
                return
                
            lower_lim = sp.sympify(lower_str, locals=local_dict)
            upper_lim = sp.sympify(upper_str, locals=local_dict)
            
            result = sp.integrate(parsed_expr, (var_symbol, lower_lim, upper_lim))
            result_str = f"∫[{lower_lim} to {upper_lim}] ({parsed_expr}) d{var_str} = {result}"

        elif op_type == "Limit":
            target_str = limit_point_entry.get().strip()
            if not target_str:
                result_label_output.config(text="INVALID INPUT! Please specify limit point (e.g., 0, 1, oo).", fg="#D32F2F")
                messagebox.showerror("Input Error", "Please specify the limit point (e.g., 0, 1, oo).")
                status_label.config(text="Status: Missing limit point", fg="#FFCDD2")
                return
            target_point = sp.sympify(target_str, locals=local_dict)
            result = sp.limit(parsed_expr, var_symbol, target_point)
            result_str = f"lim ({var_str} → {target_point}) [{parsed_expr}] = {result}"
            
        else:
            raise ValueError("Unknown operation selected.")

        result_label_output.config(text=result_str, fg="#300000")
        status_label.config(text=f"Status: Success (Detected variable: '{var_str}')", fg="#A5D6A7")

    except Exception as e:
        result_label_output.config(text="INVALID SYNTAX! Please re-type a valid expression.", fg="#D32F2F")
        status_label.config(text="Status: Invalid mathematical syntax", fg="#FFCDD2")


def update_ui_fields(event):
    """Dynamically enables/disables input fields based on selected operation."""
    op_combobox.selection_clear()
    root.focus()

    selected_op = op_combobox.get()
    
    lower_limit_entry.config(state="disabled", bg="#E0E0E0")
    upper_limit_entry.config(state="disabled", bg="#E0E0E0")
    limit_point_entry.config(state="disabled", bg="#E0E0E0")

    lower_limit_entry.delete(0, tk.END)
    upper_limit_entry.delete(0, tk.END)
    limit_point_entry.delete(0, tk.END)

    if selected_op == "Definite Integral":
        lower_limit_entry.config(state="normal", bg="#FFFBEB")
        upper_limit_entry.config(state="normal", bg="#FFFBEB")
    elif selected_op == "Limit":
        limit_point_entry.config(state="normal", bg="#FFFBEB")
        limit_point_entry.insert(0, "0")

root = tk.Tk()
root.title("Advanced CE Calculus & Analysis Toolkit")
root.geometry("600x560")
root.configure(bg="#F3F4F6")
root.resizable(False, False)

style = ttk.Style()
style.theme_use('clam')

style.configure(
    "TCombobox", 
    fieldbackground="#FFFBEB", 
    background="#790000", 
    selectbackground="#FFFBEB", 
    selectforeground="#300000",
    borderwidth=1
)
style.map(
    "TCombobox", 
    fieldbackground=[("readonly", "#FFFBEB")],
    selectbackground=[("readonly", "#FFFBEB")],
    selectforeground=[("readonly", "#300000")]
)

main_card = tk.Frame(root, bg="#790000", highlightbackground="#F3E19C", highlightthickness=2)
main_card.pack(fill="both", expand=True, padx=25, pady=20)

header_frame = tk.Frame(main_card, bg="#790000")
header_frame.pack(fill="x", pady=(15, 10))

header = tk.Label(header_frame, text="CE Structural Calculus & Analysis", font=("Segoe UI", 15, "bold"), fg="#FFFBEB", bg="#790000")
header.pack()

sub_header = tk.Label(header_frame, text="Automated Evaluation & Symbolic Math Engine", font=("Segoe UI", 9), fg="#F3E19C", bg="#790000")
sub_header.pack(pady=(2, 0))

ttk.Separator(main_card, orient='horizontal').pack(fill='x', padx=20, pady=10)

input_frame = tk.Frame(main_card, bg="#790000")
input_frame.pack(pady=5, anchor="center")

def create_entry(parent, width=30, state="normal"):
    bg_color = "#FFFBEB" if state == "normal" else "#E0E0E0"
    return tk.Entry(
        parent,
        width=width,
        font=("Segoe UI", 10),
        bd=1,
        relief="solid",
        highlightthickness=1,
        highlightcolor="#F3E19C",
        highlightbackground="#A04040",
        bg=bg_color,
        state=state
    )

tk.Label(input_frame, text="Expression:", font=("Segoe UI", 9, "bold"), fg="#FFFBEB", bg="#790000").grid(row=0, column=0, sticky="e", pady=6, padx=8)
expr_entry = create_entry(input_frame, width=30)
expr_entry.grid(row=0, column=1, sticky="ew", pady=6, padx=8, ipady=3)
expr_entry.insert

tk.Label(input_frame, text="Operation:", font=("Segoe UI", 9, "bold"), fg="#FFFBEB", bg="#790000").grid(row=1, column=0, sticky="e", pady=6, padx=8)
op_combobox = ttk.Combobox(input_frame, values=[
    "Derivative (1st Order)",
    "Derivative (2nd Order)",
    "Derivative (3rd Order)",
    "Indefinite Integral",
    "Definite Integral",
    "Limit"
], state="readonly", width=30, font=("Segoe UI", 9))
op_combobox.current(0)
op_combobox.grid(row=1, column=1, sticky="ew", pady=6, padx=8, ipady=3)
op_combobox.bind("<<ComboboxSelected>>", update_ui_fields)

tk.Label(input_frame, text="Limit Point (x → a):", font=("Segoe UI", 9), fg="#F3E19C", bg="#790000").grid(row=2, column=0, sticky="e", pady=6, padx=8)
limit_point_entry = create_entry(input_frame, width=30, state="disabled")
limit_point_entry.grid(row=2, column=1, sticky="ew", pady=6, padx=8, ipady=3)

tk.Label(input_frame, text="Lower Limit:", font=("Segoe UI", 9), fg="#F3E19C", bg="#790000").grid(row=3, column=0, sticky="e", pady=6, padx=8)
lower_limit_entry = create_entry(input_frame, width=30, state="disabled")
lower_limit_entry.grid(row=3, column=1, sticky="ew", pady=6, padx=8, ipady=3)

tk.Label(input_frame, text="Upper Limit:", font=("Segoe UI", 9), fg="#F3E19C", bg="#790000").grid(row=4, column=0, sticky="e", pady=6, padx=8)
upper_limit_entry = create_entry(input_frame, width=30, state="disabled")
upper_limit_entry.grid(row=4, column=1, sticky="ew", pady=6, padx=8, ipady=3)

def on_btn_enter(e):
    calc_button.config(bg="#D6AF6A", fg="#300000")

def on_btn_leave(e):
    calc_button.config(bg="#F3E19C", fg="#300000")

calc_button = tk.Button(
    main_card,
    text="Calculate / Evaluate",
    command=calculate_results,
    bg="#F3E19C",
    fg="#300000",
    activebackground="#D6AF6A",
    activeforeground="#300000",
    font=("Segoe UI", 10, "bold"),
    bd=0,
    cursor="hand2",
    padx=15,
    pady=6
)
calc_button.pack(pady=(15, 5))
calc_button.bind("<Enter>", on_btn_enter)
calc_button.bind("<Leave>", on_btn_leave)

status_label = tk.Label(main_card, text="Status: Ready", font=("Segoe UI", 9, "italic"), fg="#F3E19C", bg="#790000")
status_label.pack()

result_frame = tk.Frame(main_card, bg="#F3E19C", bd=1, relief="solid")
result_frame.pack(fill="x", padx=25, pady=(15, 15), ipady=8)

tk.Label(result_frame, text="OUTPUT RESULT", font=("Segoe UI", 8, "bold"), fg="#790000", bg="#F3E19C").pack(pady=(4, 2))

result_label_output = tk.Label(
    result_frame,
    text="",
    font=("Consolas", 10, "bold"),
    fg="#300000",
    bg="#F3E19C",
    justify="center",
    wraplength=500
)
result_label_output.pack(fill="x", padx=15, pady=3)

root.mainloop()