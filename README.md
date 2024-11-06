# Проектная работа 3 спринта

## Анализ и проектирование

### Анализ и планирование

#### Функциональность системы

Управление отоплением: Пользователи могут удалённо включать/выключать отопление в своих домах. Пользователи могут устанавливать желаемую температуру. Система автоматически поддерживает заданную температуру, регулируя подачу тепла.

Мониторинг температуры: Система получает данные о температуре с датчиков, установленных в домах. Пользователи могут просматривать текущую температуру в своих домах через веб-интерфейс.

#### Архитектура монолитного приложения

Язык программирования: Java.

База данных: PostgreSQL.

Архитектура: Монолитная, все компоненты системы (обработка запросов, бизнес-логика, работа с данными) находятся в рамках одного приложения.

Взаимодействие: Синхронное, запросы обрабатываются последовательно.

Масштабируемость: Ограничена, так как монолит сложно масштабировать по частям.

Развертывание: Требует остановки всего приложения.

#### Домены и границы контекстов

- Домен: Управление пользователями
    - Поддомен: Аутентификация и авторизация пользователей
    - Поддомен: Управление профилем пользователя
- Домен: Управление устройствами
    - Поддомен: Регистрация новых устройств
    - Поддомен: Управление состоянием устройств
      - Контекст: Включение/выключение отопления
    - Поддомен: Отправка команд устройствам
      - Контекст: Регулирование температуры в отопительной системе
- Домен: Управление телеметрией
    - Поддомен: Получение данных с датчиков
      - Контекст: Сбор данных о температуре в доме     
    - Поддомен: Отображение данных
      - Контекст: Отображение текущей температуры дом

#### Контекст системы

![system context](./diagrams/1.1-Smart_Home__Context_Diagram_.png)

### Архитектура микросервисов

#### Декомпозиция на микросервисы

Произведено разбитие целевого монолита на микросервисы по выделеным доменам и границам контекстов:
- API Gateway:
  - Единая точка входа в систему для клиентов
  - Маршрутизация и балансировка запросов к соответствующим микросервисам
  - Обеспечение безопасности и контроля доступа
- Шина данных (Kafka):
  - Асинхронное взаимодействие между микросервисами
  - Обеспечение масштабируемости и отказоустойчивости микросервисов
- MQTT брокер:
  - Асинхронное взаимодействие с внешними устройствами
  - Обеспечение двусторонней связи для передачи команд и получения телеметрии и результатов команд
- Сервис управления пользователями (UserManagementService):
  - Аутентификация и авторизация
  - Управление профилями пользователей
  - Контроль доступа к ресурсам
- Сервис управления домами и помещениями (HouseManagementService):
  - Управление информацией о домах и помещениях
  - Привязка устройств к помещениям
  - Обработка сценариев автоматизации
- Сервис управления устройствами (DeviceManagementService):
  - Управление состоянием устройств (включение/выключение)
  - Отправка команд на устройства
  - Обработка ответов от устройств
- Сервис телеметрии (TelemetryService):
  - Сбор и хранение данных телеметрии
  - Обработка потоков данных
  - Передача данных другим сервисам
- Сервис уведомлений (Notification Service):
  - Уведомления пользователей о настроенных событиях
- Сервис аналитики (AnalyticsService):
  - Генерация отчётов о потреблении
  - Шаблонизация отчётов

#### Определение взаимодействия

- API Gateway:
  - Синхронное взаимодействие с микросервисами
  - Синхронная авторизация через UserManagementService
  - Ассинхронное прослушивание событий через WebSocket
- Шина данных (Kafka):
  - Асинхронное взаимодействие между микросервисами
- MQTT брокер:
  - Асинхронное взаимодействие с внешними устройствами
- Сервис управления пользователями (UserManagementService):
  - Синхнронное взаимодействие с API Gateway и БД
- Сервис управления домами и помещениями (HouseManagementService):
  - Синхнронное взаимодействие с API Gateway и БД
- Сервис управления устройствами (DeviceManagementService):
  - Синхнронное взаимодействие с API Gateway и БД
- Сервис телеметрии (TelemetryService):
  - Ассинхронный опрос телеметрии и ассинхронное уведомление через событийную модель
  - Разделены операции чтения и записи в БД
- Сервис уведомлений (Notification Service):
  - Ассинхронная отправка уведомлений через Kafka с использованием очереди
- Сервис аналитики (AnalyticsService):
  - Ассинхронная генерация отчетов и выгрузка отчетов на S3

#### Визуализация архитектуры

##### Уровень контейнеров (Containers)

![system containers](./diagrams/1.2.containers-Smart_Home__Containers_Diagram_.png)

##### Уровень компонентов (Components)

![house component](./diagrams/1.2.component-house_system-House_System__Component_Diagram_.png)
![device component](./diagrams/1.2.component-device_system-Device_System__Component_Diagram_.png)
![telemetry component](./diagrams/1.2.component-telemetry_system-Telemetry_System__Component_Diagram_.png)
![analytic component](./diagrams/1.2.component-analytic_system-Analytic_System__Component_Diagram_.png)
![user component](./diagrams/1.2.component-user_system-User_System__Component_Diagram_.png)
![notification component](./diagrams/1.2.component-notification_system-Notification_System__Component_Diagram_.png)

##### Уровень кода (Code)

![house code](./diagrams/1.2.code-house_system-House_System__Code_Diagram_.png)
![device code](./diagrams/1.2.code-device_system-Device_System__Code_Diagram_.png)
![telemetry code](./diagrams/1.2.code-telemetry_system-Telemetry_System__Code_Diagram_.png)
![analytic code](./diagrams/1.2.code-analytic_system-Analytic_System__Code_Diagram_.png)
![user code](./diagrams/1.2.code-user_system-User_System__Code_Diagram_.png)
![notification code](./diagrams/1.2.code-notification_system-Notification_System__Code_Diagram_.png)

### ER-диаграмма

![house ed](./diagrams/1.3.erd-house_system-House_System__ED_Diagram_.png)
![device ed](./diagrams/1.3.erd-device_system-Device_System__ED_Diagram_.png)
![telemetry ed](./diagrams/1.3.erd-telemetry_system-Telemetry_System__ED_Diagram_.png)
![analytic ed](./diagrams/1.3.erd-analytic_system-Analytic_System__ED_Diagram_.png)
![user ed](./diagrams/1.3.erd-user_system-User_System__ED_Diagram_.png)
![notification ed](./diagrams/1.3.erd-notification_system-Notification_System__ED_Diagram_.png)

### Создание и документирование API

[Спецификация OpenAPI](./api-docs/openapi.yml)

[Спецификация AsyncAPI](./api-docs/asyncapi.yml)

# Базовая настройка

## Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```


## Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```


## Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```


## Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)


Создайте файл ~/.terraformrc

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

## Применяем terraform конфигурацию 

```bash
cd terraform
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```


## Delete minikube

```bash
minikube delete
```
