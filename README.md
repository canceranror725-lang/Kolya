import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry
import json
import time

# --- Настройки API ---
# В реальной жизни здесь был бы URL Authena API
AUTHENA_REGISTRATION_URL = "https://mock-authena-api.com/v1/auth/register" 
MAX_RETRIES = 3 
RETRY_BACKOFF = 1.0 

# --- 1. Отказоустойчивая HTTP-Сессия ---

def create_resilient_session():
    """Создает requests.Session с логикой повторных попыток для надежности."""
    retry_strategy = Retry(
        total=MAX_RETRIES,
        backoff_factor=RETRY_BACKOFF,
        status_forcelist=[500, 502, 503, 504], # Повторять при ошибках сервера
    )
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session = requests.Session()
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    return session

# --- 2. Функция регистрации ---

def register_user(session, user_data, api_key):
    """
    Имитирует отправку данных регистрации пользователя на API.
    """
    
    # Данные, которые мы отправляем (Payload)
    registration_payload = {
        "email": user_data["email"],
        "password": user_data["password"],
        "firstName": user_data["first_name"],
        "lastName": user_data["last_name"],
        "companyName": user_data["company_name"],
        "countryCode": user_data["country_code"]
        # Возможно, здесь нужно добавить согласие с TOS и Captcha-токен
    }
    
    # Заголовки:
    # 1. Content-Type: Указывает, что тело запроса - JSON.
    # 2. X-API-Key: Часто используется для идентификации клиента.
    headers = {
        "Content-Type": "application/json",
        "X-API-Key": api_key, 
        "Accept": "application/json"
    }

    print(f"\n-> 🔐 Отправка запроса на регистрацию для {user_data['email']}...")
    
    try:
        # Выполняем POST-запрос
        response = session.post(
            AUTHENA_REGISTRATION_URL,
            json=registration_payload,
            headers=headers,
            timeout=15 # Установка таймаута
        )
        
        # Вызов исключения для кодов ошибок 4xx/5xx
        response.raise_for_status() 
        
        # Проверка кода ответа
        if response.status_code == 201:
            print("✅ УСПЕХ: Пользователь успешно зарегистрирован (HTTP 201 Created).")
            # 
            return response.json()
        
        # Если статус 200, но не 201 (тоже может быть успехом, зависит от API)
        elif response.status_code == 200:
             print("✅ УСПЕХ: Регистрация принята (HTTP 200 OK).")
             return response.json()
        
        # Непредвиденный успешный ответ
        return {"status": "success", "response_code": response.status_code}

    except requests.exceptions.HTTPError as e:
        # Обработка ошибок, специфичных для регистрации (400 Bad Request, 409 Conflict)
        error_code = e.response.status_code
        
        if error_code == 400:
            print(f"❌ ОШИБКА 400: Неверные данные. Проверьте форматы полей (например, email).")
        elif error_code == 409:
            print(f"❌ ОШИБКА 409: Пользователь с этим email уже существует (Conflict).")
        else:
            print(f"❌ HTTP Ошибка (Код {error_code}): {e.response.text}")
        
    except requests.exceptions.RequestException as e:
        print(f"❌ Сетевая ошибка или таймаут: Не удалось связаться с сервером. {e}")
        
    return None

# --- Главный запуск программы ---

if __name__ == "__main__":
    
    # --- 1. Подготовка данных ---
    
    # ВАЖНО: Никогда не используйте реальный пароль в коде!
    TEST_USER_DATA = {
        "email": f"test.user_{int(time.time())}@example.com", # Уникальный email для теста
        "password": "SecurePassword123!", 
        "first_name": "Тест",
        "last_name": "Пользователь",
        "company_name": "Demo Corp",
        "country_code": "US" 
    }
    
    MOCK_API_KEY = "Your-Authena-Public-Key-12345" # Имитация вашего публичного ключа

    print("--- 📋 ИМИТАЦИЯ РЕГИСТРАЦИИ ПОЛЬЗОВАТЕЛЯ ---")
    
    # 2. Создание отказоустойчивой сессии
    session = create_resilient_session()

    # 3. Выполнение регистрации
    registration_result = register_user(session, TEST_USER_DATA, MOCK_API_KEY)
    
    print("-" * 60)
    if registration_result:
        print("Подробности ответа сервера (имитация):")
        # Имитация успешного ответа, который может включать токен доступа
        print(json.dumps(registration_result, indent=4))
    else:
        print("Регистрация не удалась. Смотрите сообщения об ошибках выше.")

    session.close()
