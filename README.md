# Alexei-Paradiev
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime

class WeatherDiaryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary")
        self.entries = []
        self.load_data()

        # Поля ввода
        ttk.Label(root, text="Дата (YYYY-MM-DD):").grid(row=0, column=0, padx=5, pady=5)
        self.date_entry = ttk.Entry(root)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(root, text="Температура (°C):").grid(row=1, column=0, padx=5, pady=5)
        self.temp_entry = ttk.Entry(root)
        self.temp_entry.grid(row=1, column=1, padx=5, pady=5)

        ttk.Label(root, text="Описание погоды:").grid(row=2, column=0, padx=5, pady=5)
        self.desc_entry = ttk.Entry(root)
        self.desc_entry.grid(row=2, column=1, padx=5, pady=5)

        ttk.Label(root, text="Осадки:").grid(row=3, column=0, padx=5, pady=5)
        self.precip_var = tk.BooleanVar()
        ttk.Checkbutton(root, variable=self.precip_var).grid(row=3, column=1, sticky='w', padx=5, pady=5)

        # Кнопка добавления
        ttk.Button(root, text="Добавить запись", command=self.add_entry).grid(row=4, column=0, columnspan=2, pady=10)

        # Таблица для отображения записей
        self.tree = ttk.Treeview(root, columns=('Date', 'Temp', 'Desc', 'Precip'), show='headings')
        self.tree.heading('Date', text='Дата')
        self.tree.heading('Temp', text='Температура')
        self.tree.heading('Desc', text='Описание')
        self.tree.heading('Precip', text='Осадки')
        self.tree.grid(row=5, column=0, columnspan=2, padx=5, pady=5, sticky='nsew')

        # Фильтры
        ttk.Label(root, text="Фильтр по дате:").grid(row=6, column=0, padx=5, pady=5)
        self.filter_date_entry = ttk.Entry(root)
        self.filter_date_entry.grid(row=6, column=1, padx=5, pady=5)

        ttk.Label(root, text="Фильтр по температуре (>):").grid(row=7, column=0, padx=5, pady=5)
        self.filter_temp_entry = ttk.Entry(root)
        self.filter_temp_entry.grid(row=7, column=1, padx=5, pady=5)

        ttk.Button(root, text="Применить фильтры", command=self.apply_filters).grid(row=8, column=0, columnspan=2, pady=10)
        ttk.Button(root, text="Сбросить фильтры", command=self.reset_filters).grid(row=9, column=0, columnspan=2, pady=5)

    def add_entry(self):
        date_str = self.date_entry.get()
        temp_str = self.temp_entry.get()
        desc = self.desc_entry.get()
        precip = self.precip_var.get()

        # Валидация
        if not self.validate_input(date_str, temp_str, desc):
            return

        try:
            date = datetime.strptime(date_str, '%Y-%m-%d').date()
            temp = float(temp_str)
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты или температуры")
            return

        entry = {
            'date': date_str,
            'temperature': temp,
            'description': desc,
            'precipitation': 'да' if precip else 'нет'
        }
        self.entries.append(entry)
        self.update_table()
        self.save_data()  # Автосохранение
        self.clear_inputs()

    def validate_input(self, date_str, temp_str, desc):
        if not date_str:
            messagebox.showerror("Ошибка", "Дата не может быть пустой")
            return False
        try:
            datetime.strptime(date_str, '%Y-%m-%d')
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты. Используйте YYYY-MM-DD")
            return False

        if not temp_str:
            messagebox.showerror("Ошибка", "Температура не может быть пустой")
            return False
        try:
            float(temp_str)
        except ValueError:
            messagebox.showerror("Ошибка", "Температура должна быть числом")
            return False

        if not desc:
            messagebox.showerror("Ошибка", "Описание не может быть пустым")
            return False
        return True

    def apply_filters(self):
        filtered = self.entries
        date_filter = self.filter_date_entry.get()
        temp_filter_str = self.filter_temp_entry.get()

        if date_filter:
            filtered = [e for e in filtered if e['date'] == date_filter]
        if temp_filter_str:
            try:
                temp_filter = float(temp_filter_str)
                filtered = [e for e in filtered if e['temperature'] > temp_filter]
            except ValueError:
                messagebox.showerror("Ошибка", "Температура фильтра должна быть числом")
                return
        self.update_table(filtered)

    def reset_filters(self):
        self.filter_date_entry.delete(0, tk.END)
        self.filter_temp_entry.delete(0, tk.END)
        self.update_table()

    def update_table(self, entries=None):
        for item in self.tree.get_children():
            self.tree.delete(item)
        entries = entries or self.entries
        for entry in entries:
            self.tree.insert('', 'end', values=(
                entry['date'],
                f"{entry['temperature']}°C",
                entry['description'],
                entry['precipitation']
            ))

    def save_data(self):
        with open('data.json', 'w', encoding='utf-8') as f:
            json.dump(self.entries, f, ensure_ascii=False, indent=4)

    def load_data(self):
        try:
            with open('data.json', 'r', encoding='utf-8') as f:
                self.entries = json.load(f)
            self.update_table()
        except FileNotFoundError:
            self.entries = []

    def clear_inputs(self):
        self.date_entry.delete(0, tk.END)
        self.temp_entry.delete(0, tk.END)
        self.desc_entry.delete(0, tk.END)
        self.precip_var.set(False)

# Запуск приложения
if __name__ == "__main__":
    root = tk.Tk()
    app = WeatherDiaryApp(root)
    root.mainloop()
