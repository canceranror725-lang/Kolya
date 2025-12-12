import requests
import json
import time

# --- Настройки ---
# Публичный API для тестирования HTTP-запросов
BASE_URL = "https://jsonplaceholder.typicode.com" 

def fetch_data(endpoint):
    """
    Выполняет HTTP GET-запрос для получения данных.
    """
    url = f"{BASE_URL}/{endpoint}"
    
    print(f"-> 🌐 Выполнение GET-запроса к: {url}")
    
    try:
        # 
        response = requests.get(url, timeout=5) # Таймаут 5 секунд
        response.raise_for_status() # Вызывает исключение для HTTP ошибок 4xx/5xx
        
        # Декодируем JSON-ответ в словарь Python
        data = response.json()
        
        print(f"✅ Статус: {response.status_code}")
        print(f"   Получено {len(data)} элементов.")
        
        return data

    except requests.exceptions.RequestException as e:
        print(f"❌ ОШИБКА GET-запроса: {e}")
        return None

def create_resource(endpoint, payload):
    """
    Выполняет HTTP POST-запрос для создания нового ресурса.
    """
    url = f"{BASE_URL}/{endpoint}"
    
    print(f"-> 📦 Выполнение POST-запроса к: {url}")
    
    try:
        # 
        # Используем json=payload, чтобы автоматически сериализовать данные и установить Content-Type: application/json
        response = requests.post(url, json=payload, timeout=5) 
        response.raise_for_status() # Вызывает исключение для HTTP ошибок 4xx/5xx
        
        # Декодируем JSON-ответ
        data = response.json()
        
        print(f"✅ Статус: {response.status_code}")
        print(f"   Новый ID ресурса: {data.get('id')}")
        
        return data

    except requests.exceptions.RequestException as e:
        print(f"❌ ОШИБКА POST-запроса: {e}")
        return None

# --- Главный запуск программы ---

print("--- 🌐 ИНСТРУМЕНТ ДЛЯ HTTP-ЗАПРОСОВ (Requests) ---")

# --- 1. Демонстрация GET-запроса (Получение списка постов) ---
print("\n[1. ПОЛУЧЕНИЕ ДАННЫХ (GET)]")

posts = fetch_data("posts")

if posts:
    # Выводим информацию о первом посте
    first_post_title = posts[0].get('title', 'N/A')
    print(f"   Первый пост: '{first_post_title[:40]}...'")


# --- 2. Демонстрация POST-запроса (Создание нового поста) ---
print("\n[2. ОТПРАВКА ДАННЫХ (POST)]")

new_post_payload = {
    "title": "Новый пост от API клиента",
    "body": "Это тело поста, созданного в {time.ctime()}",
    "userId": 99
}

created_post = create_resource("posts", new_post_payload)

if created_post:
    print(f"   Подтвержденный заголовок: '{created_post.get('title')}'")
    # Обратите внимание, что fake API вернет ID=101, имитируя создание ресурса


# --- 3. Демонстрация обработки ошибок (GET-запрос к несуществующему ресурсу) ---
print("\n[3. ОБРАБОТКА ОШИБОК]")

error_data = fetch_data("non_existent_endpoint") # Ожидаем 404 Not Found

if error_data is None:
    print("   Как ожидалось, запрос завершился с ошибкой (например, 404).")
