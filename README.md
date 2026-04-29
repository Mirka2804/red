# Итоговая аттестация 
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.expenses = []
        self.load_data()
        self.create_widgets()

    def create_widgets(self):
        # Поля ввода
        tk.Label(self.root, text="Сумма:").grid(row=0, column=0, padx=5, pady=5)
        self.amount_entry = tk.Entry(self.root)
        self.amount_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Категория:").grid(row=1, column=0, padx=5, pady=5)
        self.category_var = tk.StringVar()
        categories = ["Еда", "Транспорт", "Развлечения", "Жильё", "Прочее"]
        self.category_combo = ttk.Combobox(self.root, textvariable=self.category_var, values=categories)
        self.category_combo.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Дата (ГГГГ-ММ-ДД):").grid(row=2, column=0, padx=5, pady=5)
        self.date_entry = tk.Entry(self.root)
        self.date_entry.grid(row=2, column=1, padx=5, pady=5)

        # Кнопка добавления
        tk.Button(self.root, text="Добавить расход", command=self.add_expense).grid(row=3, column=0, columnspan=2, pady=10)

        # Таблица
        self.tree = ttk.Treeview(self.root, columns=("Сумма", "Категория", "Дата"), show="headings")
        self.tree.heading("Сумма", text="Сумма")
        self.tree.heading("Категория", text="Категория")
        self.tree.heading("Дата", text="Дата")
        self.tree.grid(row=4, column=0, columnspan=2, padx=5, pady=5)

        # Фильтры
        tk.Label(self.root, text="Фильтр по категории:").grid(row=5, column=0, padx=5, pady=5)
        self.filter_category_var = tk.StringVar()
        filter_categories = ["Все"] + categories
        self.filter_combo = ttk.Combobox(self.root, textvariable=self.filter_category_var, values=filter_categories)
        self.filter_combo.set("Все")
        self.filter_combo.grid(row=5, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Период с (ГГГГ-ММ-ДД):").grid(row=6, column=0, padx=5, pady=5)
        self.start_date_entry = tk.Entry(self.root)
        self.start_date_entry.grid(row=6, column=1, padx=5, pady=5)

        tk.Label(self.root, text="по (ГГГГ-ММ-ДД):").grid(row=7, column=0, padx=5, pady=5)
        self.end_date_entry = tk.Entry(self.root)
        self.end_date_entry.grid(row=7, column=1, padx=5, pady=5)

        # Кнопки фильтрации и подсчёта
        tk.Button(self.root, text="Применить фильтр", command=self.apply_filter).grid(row=8, column=0, pady=10)
        tk.Button(self.root, text="Подсчитать сумму за период", command=self.calculate_sum).grid(row=8, column=1, pady=10)

        # Метка для суммы
        self.sum_label = tk.Label(self.root, text="Общая сумма: 0")
        self.sum_label.grid(row=9, column=0, columnspan=2, pady=5)

        self.refresh_table()

    def validate_input(self):
        try:
            amount = float(self.amount_entry.get())
            if amount <= 0:
                raise ValueError("Сумма должна быть положительным числом")
        except ValueError:
            messagebox.showerror("Ошибка", "Некорректная сумма")
            return False

        try:
            date_str = self.date_entry.get()
            datetime.strptime(date_str, "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты. Используйте ГГГГ-ММ-ДД")
            return False
        return True

    def add_expense(self):
        if not self.validate_input():
            return

        expense = {
            "amount": float(self.amount_entry.get()),
            "category": self.category_var.get(),
            "date": self.date_entry.get()
        }
        self.expenses.append(expense)
        self.save_data()
        self.refresh_table()
        self.clear_inputs()

    def clear_inputs(self):
        self.amount_entry.delete(0, tk.END)
        self.category_var.set("")
        self.date_entry.delete(0, tk.END)

    def refresh_table(self, filtered_expenses=None):
        for item in self.tree.get_children():
            self.tree.delete(item)

        expenses_to_show = filtered_expenses if filtered_expenses is not None else self.expenses
        for expense in expenses_to_show:
            self.tree.insert("", "end", values=(expense["amount"], expense["category"], expense["date"]))

    def apply_filter(self):
        category = self.filter_category_var.get()
        start_date_str = self.start_date_entry.get()
        end_date_str = self.end_date_entry.get()

        filtered_expenses = self.expenses

        if category != "Все":
            filtered_expenses = [e for e in filtered_expenses if e["category"] == category]

        if start_date_str:
            start_date = datetime.strptime(start_date_str, "%Y-%m-%d")
            filtered_expenses = [e for e in filtered_expenses if datetime.strptime(e["date"], "%Y-%m-%d") >= start_date]

        if end_date_str:
            end_date = datetime.strptime(end_date_str, "%Y-%m-%d")
            filtered_expenses = [e for e in filtered_expenses if datetime.strptime(e["date"], "%Y-%m-%d") <= end_date]

        self.refresh_table(filtered_expenses)

    def calculate_sum(self):
        start_date_str = self.start_date_entry.get()
        end_date_str = self.end_date_entry.get()

        total = 0
        for expense in self.expenses:
            expense_date = datetime.strptime(expense["date"], "%Y-%m-%d")

            if start_date_str:
                start_date = datetime.strptime(start_date_str, "%Y-%m-%d")
                if expense_date < start_date:
                    continue

            if end_date_str:
                end_date = datetime.strptime(end_date
