# 🛡️ Splunk SIEM Home Lab & Adversary Emulation

Репозиторій містить покроковий процес розгортання домашньої SIEM-лабораторії на базі Splunk Enterprise та тестування правил детекції за допомогою фреймворку Atomic Red Team.

## 🏢 Архітектура та компоненти
* **SIEM-Сервер:** Ubuntu Server (Splunk Enterprise)
* **Ендпоінт:** Windows 11 (Splunk Universal Forwarder)
* **Емуляція атак:** Atomic Red Team (MITRE ATT&CK)

---

## 📖 Зміст лабораторної роботи

Для зручності навігації весь процес розбито на окремі технічні модулі:

### 🛠️ [Модуль 1: Встановлення Splunk та налаштування мережі](docs/1-installation-and-network.md)
* Запуск Splunk Enterprise на Linux.
* Налаштування Port Forwarding у VirtualBox.
* Вирішення проблем із блокуванням порту `9997` фаєрволом (UFW Troubleshooting).

### 🔍 [Модуль 2: Емуляція атаки Brute Force (T1110)](docs/2-attack-emulation-t1110.md)
* Генерація невдалих спроб входу через Atomic Red Team.
* Розробка аналітичного SPL-запиту для виявлення аномалій.
* Аналіз маркерів компрометації (Event ID 4625, Sub Status `0xC000006A`).
* Стратегії протидії (Mitigation).

### 🔬 [Модуль 3: Просунутий моніторинг за допомогою Sysmon](docs/3-sysmon-and-advanced-detection.md)
* Встановлення Sysmon з конфігом SwiftOnSecurity.
* Troubleshooting помилки доступу до логів (errorCode=5: Access Denied).
* Перехоплення хакерських команд PowerShell (EventCode=1: Process Creation).
* Налаштування автозапуску Splunk сервісу через Linux systemd.