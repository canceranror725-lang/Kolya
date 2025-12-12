import sqlite3
import os
from pathlib import Path
import time

# --- Настройки ---
DB_FILE = Path("app_data.db")
TABLE_NAME = "tasks"

def create_connection(db_file):
    """Создает соединение с базой данных SQLite."""
    conn = None
    try:
        # 
        conn = sqlite3.connect(db_file)
        print(f"✅ Успешное подключение к базе данных: {db_file}")
        return conn
    except sqlite3.Error as e:
        print(f"❌ Ошибка подключения к SQLite: {e}")
        return None

def create_table(conn):
    """Создает таблицу задач, если она не существует."""
    
    # SQL-запрос для создания таблицы
    sql_create_tasks_table = f""" CREATE TABLE IF NOT EXISTS {TABLE_NAME} (
                                    id INTEGER PRIMARY KEY,
                                    name TEXT NOT NULL,
                                    priority INTEGER,
                                    status TEXT,
                                    start_date TEXT
                                ); """
    try:
        cursor = conn.cursor()
        cursor.execute(sql_create_tasks_table)
        conn.commit()
        print(f"✅ Таблица '{TABLE_NAME}' готова.")
    except sqlite3.Error as e:
        print(f"❌ Ошибка при создании таблицы: {e}")

def insert_task(conn, task):
    """Вставляет новую задачу в таблицу."""
    sql = f''' INSERT INTO {TABLE_NAME}(name, priority, status, start_date)
              VALUES(?, ?, ?, ?) '''
    try:
        cursor = conn.cursor()
        cursor.execute(sql, task)
        conn.commit()
        return cursor.lastrowid # Возвращает ID вставленной строки
    except sqlite3.Error as e:
        print(f"❌ Ошибка при вставке данных: {e}")
        return None

def select_all_tasks(conn):
    """Выбирает все строки из таблицы задач."""
    sql = f'SELECT * FROM {TABLE_NAME}'
    cursor = conn.cursor()
    cursor.execute(sql)
    # fetchall() возвращает список кортежей
    rows = cursor.fetchall()
    return rows

def update_task_status(conn, task_id, new_status):
    """Обновляет статус задачи по ID."""
    sql = f''' UPDATE {TABLE_NAME}
              SET status = ?
              WHERE id = ?'''
    try:
        cursor = conn.cursor()
        cursor.execute(sql, (new_status, task_id))
        conn.commit()
        print(f"⚙️ Обновлена задача ID {task_id}: новый статус '{new_status}'")
    except sqlite3.Error as e:
        print(f"❌ Ошибка при обновлении: {e}")

def delete_task(conn, task_id):
    """Удаляет задачу по ID."""
    sql = f'DELETE FROM {TABLE_NAME} WHERE id=?'
    try:
        cursor = conn.cursor()
        cursor.execute(sql, (task_id,))
        conn.commit()
        print(f"➖ Удалена задача ID {task_id}.")
    except sqlite3.Error as e:
        print(f"❌ Ошибка при удалении: {e}")

# --- Главный запуск программы ---

print("--- 🗄️ ДЕМОНСТРАЦИЯ РАБОТЫ С SQLite ---")

# 1. Подключение к базе данных (создаст файл, если не существует)
conn = create_connection(DB_FILE)

if conn:
    # 2. Создание таблицы
    create_table(conn)

    # 3. Вставка данных
    task_list = [
        ('Спроектировать API', 1, 'In Progress', str(time.time())),
        ('Написать Unit-тесты', 2, 'To Do
