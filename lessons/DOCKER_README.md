# RabbitMQ запуск сервиса в Docker

Быстрый запуск RabbitMQ для разработки и тестирования с компонентой 1CRabbitMQ.

> 📄 **Файл конфигурации:** [`docker-compose.yml`](./docker-compose.yml)

## 🚀 Быстрый старт

### Шаг 1: Установка Docker

#### Windows

1. Скачайте и установите **Docker Desktop for Windows**:
   - 📥 [Скачать Docker Desktop](https://www.docker.com/products/docker-desktop/)
   - Минимальные требования: Windows 10/11 64-bit, WSL 2
   
2. Запустите Docker Desktop
   - После установки Docker Desktop запустится автоматически
   - Убедитесь, что Docker работает (иконка в трее должна быть зелёной)

3. Проверьте установку:
   ```powershell
   docker --version
   docker-compose --version
   ```

> **💡 Примечание:** Docker Desktop для Windows включает Docker Compose автоматически.

#### Linux (Ubuntu/Debian)

1. Обновите список пакетов:
   ```bash
   sudo apt update
   ```

2. Установите необходимые пакеты:
   ```bash
   sudo apt install -y ca-certificates curl gnupg lsb-release
   ```

3. Добавьте официальный GPG ключ Docker:
   ```bash
   sudo mkdir -p /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   ```

4. Добавьте репозиторий Docker:
   ```bash
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. Установите Docker Engine и Docker Compose:
   ```bash
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```

6. Добавьте текущего пользователя в группу docker (чтобы не использовать sudo):
   ```bash
   sudo usermod -aG docker $USER
   newgrp docker
   ```

7. Проверьте установку:
   ```bash
   docker --version
   docker compose version
   ```

> **💡 Примечание:** В новых версиях Docker команда `docker-compose` заменена на `docker compose` (без дефиса).

#### Linux (CentOS/RHEL/Fedora)

1. Установите необходимые пакеты:
   ```bash
   sudo dnf install -y dnf-plugins-core
   ```

2. Добавьте репозиторий Docker:
   ```bash
   sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
   ```

3. Установите Docker:
   ```bash
   sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```

4. Запустите Docker:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

5. Добавьте пользователя в группу docker:
   ```bash
   sudo usermod -aG docker $USER
   newgrp docker
   ```

### Шаг 2: Запуск RabbitMQ

1. Скопируйте [`docker-compose.yml`](./docker-compose.yml) в отдельную папку
2. Перейдите в эту папку в терминале
3. Выполните команды:

```bash
# Запустить RabbitMQ
docker-compose up -d

# Проверить статус
docker-compose ps

# Просмотреть логи
docker-compose logs -f rabbitmq

# Остановить
docker-compose down

# Остановить и удалить данные
docker-compose down -v
```

## 📋 Конфигурация

### Порты
- **5672** — AMQP порт (для подключения из 1С)
- **15672** — Management UI (веб-интерфейс)

### Учетные данные
- **Пользователь:** `rmuser`
- **Пароль:** `rmpassword`

> **⚠️ Важно:** Для production измените учетные данные в файле `docker-compose.yml`!

## 🌐 Management UI

После запуска откройте в браузере: **http://localhost:15672**

- Логин: `rmuser`
- Пароль: `rmpassword`

В веб-интерфейсе вы можете:
- Просматривать очереди и сообщения
- Создавать обменники (exchanges)
- Мониторить подключения и каналы
- Управлять пользователями и правами доступа

## 💾 Персистентность данных

Все данные RabbitMQ сохраняются в папке `./rabbitmq_data` на хосте. Это означает, что:
- Очереди, обменники и сообщения сохраняются при перезапуске контейнера
- При выполнении `docker-compose down` данные НЕ удаляются
- Для полной очистки используйте `docker-compose down -v`

## 🔧 Настройка под production

Для production окружения рекомендуется:

1. **Изменить учетные данные:**
   ```yaml
   environment:
     RABBITMQ_DEFAULT_USER: your_secure_user
     RABBITMQ_DEFAULT_PASS: your_secure_password_here
   ```

2. **Ограничить доступ к портам:**
   ```yaml
   ports:
     - "127.0.0.1:5672:5672"   # Только localhost
     - "127.0.0.1:15672:15672" # Только localhost
   ```

3. **Добавить кастомную конфигурацию:**
   ```yaml
   volumes:
     - ./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf
   ```

4. **Настроить ресурсы:**
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '2'
         memory: 2G
       reservations:
         cpus: '1'
         memory: 1G
   ```

## 🐛 Troubleshooting

### Контейнер не запускается

```bash
# Проверить логи
docker-compose logs rabbitmq

# Проверить статус
docker-compose ps
```

### Порт уже занят

Если порт 5672 или 15672 уже используется другим приложением, измените в `docker-compose.yml`:

```yaml
ports:
  - "5673:5672"    # Используем 5673 вместо 5672
  - "15673:15672"  # Используем 15673 вместо 15672
```

### Нет доступа к Management UI

Убедитесь, что:
1. Контейнер запущен: `docker-compose ps`
2. Порт 15672 не заблокирован файрволом
3. Используете правильные учетные данные

### Недостаточно места на диске

RabbitMQ требует минимум 2GB свободного места (настроено в `disk_free_limit`). Проверьте:

```bash
df -h
```

## 📚 Дополнительные ресурсы

- [Официальная документация RabbitMQ](https://www.rabbitmq.com/documentation.html)

