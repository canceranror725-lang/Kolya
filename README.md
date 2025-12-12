import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry
import json
import time

# --- Настройки ---
BASE_URL = "https://jsonplaceholder.typicode.com" 
MAX_RETRIES = 3  # Максимальное количество повторных попыток
RETRY_BACKOFF = 1.0 # Время ожидания между повторами (в секундах)

# --- 1. Создание отказоустойчивой сессии (Session) ---

def create_resilient_session():
    """
    Создает объект requests.Session с логикой повторных попыток (Retry Logic).
    Это обеспечивает, что при временных ошибках (500, 502 и т.д.) клиент
    автоматически попытается выполнить запрос снова.
    """
    # 
    
    # Определяем, какие статусы HTTP должны вызывать повторную попытку
    retry_statuses = [500, 502, 503, 504]
    
    retry_strategy = Retry(
        total=MAX_RETRIES,  # Общее количество попыток
        backoff_factor=RETRY_BACKOFF, # Фактор экспоненциальной задержки
        status_forcelist=retry_statuses, # Статусы, при которых нужно повторять
        method_whitelist=["HEAD", "GET", "PUT", "DELETE", "OPTIONS", "TRACE"] # Методы для повтора
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session = requests.Session()
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    
    return session

# --- 2. Универсальная функция запроса ---

def make_request(session, method, endpoint, headers=None, data=None):
    """
    Универсальная функция для выполнения HTTP-запросов (GET, POST, PUT, DELETE).
    """
    url = f"{BASE_URL}/{endpoint}"
    
    print(f"\n-> 🛠️ {method} Запрос к: {url}")
    
    try:
        # Выполнение запроса с использованием объекта Session
        response = session.request(
            method,
            url,
            json=data,          # Автоматическая сериализация для POST/PUT
            headers=headers,    # Пользовательские заголовки
            timeout=10          # Общий таймаут запроса
        )
        response.raise_for_status() # Вызывает исключение для ошибок 4xx/5xx
        
        # Обработка ответа
        if response.status_code == 204: # No Content (типично для DELETE)
            return {"status": "Success (No Content)"}
            
        return response.json()

    except requests.exceptions.HTTPError as e:
        print(f"❌ HTTP Ошибка (Код {e.response.status_code}): {e}")
    except requests.exceptions.RequestException as e:
        print(f"❌ Сетевая Ошибка/Таймаут: {e}")
        
    return None

# --- Главный запуск программы ---

# Создаем единую, отказоустойчивую сессию
resilient_session = create_resilient_session()

# --- 1. Демонстрация GET с повторными попытками (если бы сервер ломался) ---
print("--- 🚀 ОТКАЗОУСТОЙЧИВЫЙ HTTP-КЛИЕНТ ---")

posts_data = make_request(resilient_session, "GET", "posts/1")

if posts_data:
    print(f"✅ Успех: Получен пост ID: {posts_data.get('id')}, Заголовок: {posts_data.get('title')[:30]}...")

# --- 2. Демонстрация POST с кастомными заголовками (Авторизация) ---
print("\n--- Демонстрация Авторизации и POST ---")

# Имитация токена JWT
AUTH_TOKEN = "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eS5nLnBvdw"

custom_headers = {
    "Authorization": AUTH_TOKEN,
    "X-Client-ID": "MyApp-v2.0"
}

new_resource_payload = {
    "title": "Новая задача",
    "completed": False
}

created_resource = make_request(
    resilient_session, 
    "POST", 
    "todos", 
    headers=custom_headers, 
    data=new_resource_payload
)

if created_resource:
    print(f"✅ Успех: Создан новый TODO, ID: {created_resource.get('id')}")

# --- 3. Демонстрация PUT (Обновление) ---
print("\n--- Демонстрация PUT ---")

update_payload = {"title": "Обновленный заголовок", "completed": True}
updated_resource = make_request(resilient_session, "PUT", "posts/1", data=update_payload)

if updated_resource:
    print(f"✅ Успех: Пост ID: 1 обновлен. Заголовок: {updated_resource.get('title')}")

# Закрываем сессию
resilient_session.close()
