# Alexei-Paradiev
import tkinter as tk
from tkinter import ttk, messagebox
import json
import random
from datetime import datetime, timedelta

# Вспомогательные функции
def validate_date(date_string):
    try:
        datetime.strptime(date_string, "%Y-%m-%d")
        return True
    except ValueError:
        return False

def load_tasks():
    try:
        with open("tasks.json", "r", encoding="utf-8") as f:
            return json.load(f)
    except (FileNotFoundError, json.JSONDecodeError):
        return []

def save_tasks(tasks):
    with open("tasks.json", "w", encoding="utf-8") as f:
        json.dump(tasks, f, ensure_ascii=False, indent=4)

def generate_random_task():
    titles = ["Подготовить отчёт", "Провести встречу", "Проверить документы", "Ответить на письма", "Обновить базу данных"]
    descriptions = ["Выполнить анализ данных за последний квартал", "Обсудить план на следующий месяц", "Убедиться в корректности информации", "Ответить всем отправителям", "Добавить новые записи"]
    statuses = ["В работе", "Завершена", "Отложена"]
    
    random_date = datetime.now() + timedelta(days=random.randint(1, 30))
    
    return {
        "id": random.randint(1000, 9999),
        "title": random.choice(titles),
        "description": random.choice(descriptions),
        "due_date": random_date.strftime("%Y-%m-%d"),
        "status": random.choice(statuses)
    }

# Класс приложения
class TaskManagerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Менеджер задач")
        self.tasks = load_tasks()
        self.setup_ui()

    def setup_ui(self):
        # Поля ввода
        tk.Label(self.root, text="Название:").grid(row=0, column=0, sticky="w")
        self.title_entry = tk.Entry(self.root, width=30)
        self.title_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Описание:").grid(row=1, column=0, sticky="w")
        self.desc_entry = tk.Text(self.root, width=30, height=3)
        self.desc_entry.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Срок выполнения (YYYY-MM-DD):").grid(row=2, column=0, sticky="w")
        self.date_entry = tk.Entry(self.root, width=30)
        self.date_entry.grid(row=2, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Статус:").grid(row=3, column=0, sticky="w")
        self.status_var = tk.StringVar(value="В работе")
        status_combo = ttk.Combobox(self.root, textvariable=self.status_var,
                                   values=["В работе", "Завершена", "Отложена"])
        status_combo.grid(row=3, column=1, padx=5, pady=5)

        # Кнопки
        tk.Button(self.root, text="Добавить задачу", command=self.add_task).grid(row=4, column=0, pady=10)
        tk.Button(self.root, text="Сгенерировать случайную задачу",
                 command=self.generate_random).grid(row=4, column=1, pady=10)
        tk.Button(self.root, text="Сохранить задачи", command=lambda: save_tasks(self.tasks)).grid(row=5, column=0, pady=5)
        tk.Button(self.root, text="Загрузить задачи", command=self.load_and_refresh).grid(row=5, column=1, pady=5)

        # Фильтрация
        tk.Label(self.root, text="Фильтр по статусу:").grid(row=6, column=0, sticky="w", pady=(10, 0))
        self.filter_var = tk.StringVar(value="Все")
        filter_combo = ttk.Combobox(self.root, textvariable=self.filter_var,
                               values=["Все", "В работе", "Завершена", "Отложена"])
        filter_combo.grid(row=6, column=1, pady=(10, 0))

        tk.Button(self.root, text="Применить фильтр", command=self.apply_filter).grid(row=7, column=0, columnspan=2, pady=5)

        # Таблица задач
        columns = ("ID", "Название", "Срок", "Статус")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings", height=8)
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=120)
        self.tree.grid(row=8, column=0, columnspan=2, padx=10, pady=10, sticky="nsew")

        self.refresh_task_list()

    def add_task(self):
        title = self.title_entry.get().strip()
        desc = self.desc_entry.get("1.0", "end-1c").strip()
        date_str = self.date_entry.get().strip()
        status = self.status_var.get()

        if not title:
            messagebox.showerror("Ошибка", "Название задачи обязательно!")
            return
        if date_str and not validate_date(date_str):
            messagebox.showerror("Ошибка", "Некорректный формат даты! Используйте YYYY-MM-DD.")
            return

        task = {
            "id": len(self.tasks) + 1,
            "title": title,
            "description": desc,
            "due_date": date_str if date_str else "Не указана",
            "status": status
        }
        self.tasks.append(task)
        save_tasks(self.tasks)
        self.refresh_task_list()
        self.clear_inputs()

    def clear_inputs(self):
        self.title_entry.delete(0, tk.END)
        self.desc_entry.delete("1.0", tk.END)
        self.date_entry.delete(0, tk.END)
        self.status_var.set("В работе")

    def refresh_task_list(self, filtered_tasks=None):
        for item in self.tree.get_children():
            self.tree.delete(item)
        tasks_to_show = filtered_tasks if filtered_tasks is not None else self.tasks
        for task in tasks_to_show:
            self.tree.insert("", "end", values=(
                task["id"], task["title"], task["due_date"], task["status"]
            ))

    def apply_filter(self):
        filter_status = self.filter_var.get()
        if filter_status == "Все":
            self.refresh_task_list()
        else:
            filtered = [t for t in self.tasks if t["status"] == filter_status]
            self.refresh_task_list(filtered)

    def load_and_refresh(self):
        self.tasks = load_tasks()
        self.refresh_task_list()

    def generate_random(self):
        random_task = generate_random_task()
        self.tasks.append(random_task)
        save_tasks(self.tasks)
        self.refresh_task_list()

# Запуск приложения
if __name__ == "__main__":
    root = tk.Tk()
    app = TaskManagerApp(root)
    root.mainloop()
