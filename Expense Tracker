import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime
import json
import os

DATA_FILE = "expenses.json"

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker - Трекер расходов")
        self.root.geometry("800x500")

        # Данные
        self.expenses = []
        self.load_data()

        # Виджеты ввода
        input_frame = tk.LabelFrame(root, text="Добавить расход", padx=10, pady=10)
        input_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(input_frame, text="Сумма:").grid(row=0, column=0, padx=5, pady=5)
        self.amount_entry = tk.Entry(input_frame)
        self.amount_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(input_frame, text="Категория:").grid(row=0, column=2, padx=5, pady=5)
        self.category_var = tk.StringVar()
        self.category_combo = ttk.Combobox(input_frame, textvariable=self.category_var,
                                           values=["Еда", "Транспорт", "Развлечения", "Здоровье", "Другое"])
        self.category_combo.grid(row=0, column=3, padx=5, pady=5)
        self.category_combo.set("Еда")

        tk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=4, padx=5, pady=5)
        self.date_entry = tk.Entry(input_frame)
        self.date_entry.grid(row=0, column=5, padx=5, pady=5)
        self.date_entry.insert(0, datetime.today().strftime("%Y-%m-%d"))

        add_btn = tk.Button(input_frame, text="Добавить расход", command=self.add_expense, bg="lightgreen")
        add_btn.grid(row=0, column=6, padx=10, pady=5)

        # Таблица расходов
        columns = ("ID", "Сумма", "Категория", "Дата")
        self.tree = ttk.Treeview(root, columns=columns, show="headings")
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=150)
        self.tree.pack(pady=10, padx=10, fill="both", expand=True)

        # Фильтры
        filter_frame = tk.LabelFrame(root, text="Фильтры", padx=10, pady=10)
        filter_frame.pack(pady=5, padx=10, fill="x")

        tk.Label(filter_frame, text="Категория:").grid(row=0, column=0, padx=5)
        self.filter_category = ttk.Combobox(filter_frame, values=["Все"] + list(set([e["category"] for e in self.expenses])))
        self.filter_category.set("Все")
        self.filter_category.grid(row=0, column=1, padx=5)
        self.filter_category.bind("<<ComboboxSelected>>", lambda e: self.apply_filters())

        tk.Label(filter_frame, text="Дата от (ГГГГ-ММ-ДД):").grid(row=0, column=2, padx=5)
        self.date_from = tk.Entry(filter_frame, width=12)
        self.date_from.grid(row=0, column=3, padx=5)

        tk.Label(filter_frame, text="Дата до (ГГГГ-ММ-ДД):").grid(row=0, column=4, padx=5)
        self.date_to = tk.Entry(filter_frame, width=12)
        self.date_to.grid(row=0, column=5, padx=5)

        filter_btn = tk.Button(filter_frame, text="Применить фильтр", command=self.apply_filters)
        filter_btn.grid(row=0, column=6, padx=10)

        reset_btn = tk.Button(filter_frame, text="Сбросить фильтры", command=self.reset_filters)
        reset_btn.grid(row=0, column=7, padx=5)

        # Подсчёт суммы
        self.total_label = tk.Label(root, text="Сумма за период: 0.00", font=("Arial", 12, "bold"), fg="blue")
        self.total_label.pack(pady=5)

        # Кнопки действий
        action_frame = tk.Frame(root)
        action_frame.pack(pady=10)

        delete_btn = tk.Button(action_frame, text="Удалить выбранный расход", command=self.delete_expense, bg="salmon")
        delete_btn.pack(side="left", padx=5)

        save_btn = tk.Button(action_frame, text="Сохранить в JSON", command=self.save_data, bg="lightblue")
        save_btn.pack(side="left", padx=5)

        load_btn = tk.Button(action_frame, text="Загрузить из JSON", command=self.load_data, bg="lightyellow")
        load_btn.pack(side="left", padx=5)

        # Обновить таблицу
        self.refresh_table()

    def add_expense(self):
        """Добавление расхода с проверкой"""
        try:
            amount = float(self.amount_entry.get())
            if amount <= 0:
                messagebox.showerror("Ошибка", "Сумма должна быть положительным числом")
                return
        except ValueError:
            messagebox.showerror("Ошибка", "Сумма должна быть числом")
            return

        category = self.category_var.get().strip()
        if not category:
            messagebox.showerror("Ошибка", "Введите категорию")
            return

        date_str = self.date_entry.get().strip()
        try:
            datetime.strptime(date_str, "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты. Используйте ГГГГ-ММ-ДД")
            return

        new_id = max([e["id"] for e in self.expenses], default=0) + 1
        self.expenses.append({
            "id": new_id,
            "amount": amount,
            "category": category,
            "date": date_str
        })
        self.save_data()
        self.refresh_table()
        self.amount_entry.delete(0, tk.END)
        messagebox.showinfo("Успех", "Расход добавлен")

    def delete_expense(self):
        selected = self.tree.selection()
        if not selected:
            messagebox.showwarning("Предупреждение", "Выберите расход для удаления")
            return
        for item in selected:
            exp_id = int(self.tree.item(item)["values"][0])
            self.expenses = [e for e in self.expenses if e["id"] != exp_id]
        self.save_data()
        self.refresh_table()
        messagebox.showinfo("Успех", "Расход(ы) удалён")

    def apply_filters(self):
        self.refresh_table()

    def reset_filters(self):
        self.filter_category.set("Все")
        self.date_from.delete(0, tk.END)
        self.date_to.delete(0, tk.END)
        self.refresh_table()

    def get_filtered_expenses(self):
        filtered = self.expenses[:]

        # Фильтр по категории
        cat = self.filter_category.get()
        if cat != "Все":
            filtered = [e for e in filtered if e["category"] == cat]

        # Фильтр по дате
        date_from_str = self.date_from.get().strip()
        date_to_str = self.date_to.get().strip()

        if date_from_str:
            try:
                date_from = datetime.strptime(date_from_str, "%Y-%m-%d")
                filtered = [e for e in filtered if datetime.strptime(e["date"], "%Y-%m-%d") >= date_from]
            except:
                pass

        if date_to_str:
            try:
                date_to = datetime.strptime(date_to_str, "%Y-%m-%d")
                filtered = [e for e in filtered if datetime.strptime(e["date"], "%Y-%m-%d") <= date_to]
            except:
                pass

        return filtered

    def refresh_table(self):
        for row in self.tree.get_children():
            self.tree.delete(row)

        filtered = self.get_filtered_expenses()
        total = 0
        for exp in filtered:
            self.tree.insert("", tk.END, values=(exp["id"], f"{exp['amount']:.2f}", exp["category"], exp["date"]))
            total += exp["amount"]

        self.total_label.config(text=f"Сумма за период: {total:.2f}")

        # Обновить список категорий в фильтре
        cats = sorted(set([e["category"] for e in self.expenses]))
        self.filter_category["values"] = ["Все"] + cats
        if self.filter_category.get() not in ["Все"] + cats:
            self.filter_category.set("Все")

    def save_data(self):
        try:
            with open(DATA_FILE, "w", encoding="utf-8") as f:
                json.dump(self.expenses, f, ensure_ascii=False, indent=4)
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить данные: {e}")

    def load_data(self):
        if os.path.exists(DATA_FILE):
            try:
                with open(DATA_FILE, "r", encoding="utf-8") as f:
                    self.expenses = json.load(f)
                self.refresh_table()
                messagebox.showinfo("Загрузка", "Данные загружены из JSON")
            except Exception as e:
                messagebox.showerror("Ошибка", f"Не удалось загрузить данные: {e}")
        else:
            self.expenses = []


if __name__ == "__main__":
    root = tk.Tk()
    app = ExpenseTracker(root)
    root.mainloop()
