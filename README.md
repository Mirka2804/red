import tkinter as tk
from tkinter import ttk, messagebox
import json
import os

class BookTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Book Tracker")
        self.books = []
        self.load_data()
        self.create_widgets()

    def create_widgets(self):
        # Поля ввода
        tk.Label(self.root, text="Название книги:").grid(row=0, column=0, sticky="w", padx=5, pady=5)
        self.title_entry = tk.Entry(self.root, width=30)
        self.title_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Автор:").grid(row=1, column=0, sticky="w", padx=5, pady=5)
        self.author_entry = tk.Entry(self.root, width=30)
        self.author_entry.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Жанр:").grid(row=2, column=0, sticky="w", padx=5, pady=5)
        self.genre_entry = tk.Entry(self.root, width=30)
        self.genre_entry.grid(row=2, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Количество страниц:").grid(row=3, column=0, sticky="w", padx=5, pady=5)
        self.pages_entry = tk.Entry(self.root, width=30)
        self.pages_entry.grid(row=3, column=1, padx=5, pady=5)

        # Кнопка добавления
        self.add_button = tk.Button(self.root, text="Добавить книгу", command=self.add_book)
        self.add_button.grid(row=4, column=0, columnspan=2, pady=10)

        # Таблица для отображения книг
        self.tree = ttk.Treeview(self.root, columns=("Title", "Author", "Genre", "Pages"), show="headings")
        self.tree.heading("Title", text="Название")
        self.tree.heading("Author", text="Автор")
        self.tree.heading("Genre", text="Жанр")
        self.tree.heading("Pages", text="Страниц")
        self.tree.grid(row=5, column=0, columnspan=2, padx=5, pady=5, sticky="nsew")

        # Фильтры
        tk.Label(self.root, text="Фильтр по жанру:").grid(row=6, column=0, sticky="w", padx=5, pady=5)
        self.filter_genre = tk.Entry(self.root, width=30)
        self.filter_genre.grid(row=6, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Фильтр страниц (>):").grid(row=7, column=0, sticky="w", padx=5, pady=5)
        self.filter_pages = tk.Entry(self.root, width=30)
        self.filter_pages.grid(row=7, column=1, padx=5, pady=5)

        self.filter_button = tk.Button(self.root, text="Применить фильтры", command=self.apply_filters)
        self.filter_button.grid(row=8, column=0, columnspan=2, pady=10)

        # Кнопки сохранения/загрузки
        self.save_button = tk.Button(self.root, text="Сохранить в JSON", command=self.save_data)
        self.save_button.grid(row=9, column=0, pady=5)

        self.load_button = tk.Button(self.root, text="Загрузить из JSON", command=self.load_data)
        self.load_button.grid(row=9, column=1, pady=5)

        self.update_table()

    def add_book(self):
        title = self.title_entry.get().strip()
        author = self.author_entry.get().strip()
        genre = self.genre_entry.get().strip()
        pages_text = self.pages_entry.get().strip()

        # Проверка корректности ввода
        if not title or not author or not genre:
            messagebox.showerror("Ошибка", "Все поля, кроме фильтра, должны быть заполнены!")
            return

        try:
            pages = int(pages_text)
            if pages <= 0:
                raise ValueError
        except ValueError:
            messagebox.showerror("Ошибка", "Количество страниц должно быть положительным числом!")
            return

        # Добавление книги
        book = {
            "title": title,
            "author": author,
            "genre": genre,
            "pages": pages
        }
        self.books.append(book)

        # Очистка полей ввода
        self.title_entry.delete(0, tk.END)
        self.author_entry.delete(0, tk.END)
        self.genre_entry.delete(0, tk.END)
        self.pages_entry.delete(0, tk.END)

        self.update_table()

    def save_data(self):
        with open("books.json", "w", encoding="utf-8") as f:
            json.dump(self.books, f, ensure_ascii=False, indent=4)
        messagebox.showinfo("Успех", "Данные сохранены в books.json")

    def load_data(self):
        if os.path.exists("books.json"):
            with open("books.json", "r", encoding="utf-8") as f:
                self.books = json.load(f)
        else:
            self.books = []
        self.update_table()

    def apply_filters(self):
        genre_filter = self.filter_genre.get().strip().lower()
        pages_filter_text = self.filter_pages.get().strip()

        filtered_books = self.books

        if genre_filter:
            filtered_books = [book for book in filtered_books if genre_filter in book["genre"].lower()]

        if pages_filter_text:
            try:
                pages_filter = int(pages_filter_text)
                filtered_books = [book for book in filtered_books if book["pages"] > pages_filter]
            except ValueError:
                messagebox.showwarning("Предупреждение", "Некорректное значение для фильтра страниц")
                return

        self.update_table(filtered_books)

    def update_table(self, books=None):
        for item in self.tree.get_children():
            self.tree.delete(item)

        target_books = books if books is not None else self.books
        for book in target_books:
            self.tree.insert("", "end", values=(book["title"], book["author"], book["genre"], book["pages"]))

if __name__ == "__main__":
    root = tk.Tk()
    app = BookTracker(root)
    root.mainloop()
