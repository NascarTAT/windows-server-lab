# Case 02. Остановка службы DHCP-сервера



## Ситуация



Клиент сообщает: **не работает интернет**, не открывается ни один сайт.



- **Тип проблемы:** Network / DHCP

- **Клиент:** User3 (HR)

- **Компьютер клиента:** DESKTOP-UJAHLGC



## Диагностика



### 1. Проверка сетевых параметров на клиенте



```cmd

ipconfig /all

```



**Обнаружено:**

- IP-адрес: `169.254.X.X` — это **APIPA** (Automatic Private IP Addressing).

- Маска: `255.255.0.0`

- Шлюз: **отсутствует**



**APIPA** — это сигнал, что клиент **не смог получить IP от DHCP-сервера** и выдал себе «аварийный» адрес. С таким адресом **интернета нет**.



### 2. Попытка получить IP заново



```cmd

ipconfig /release

ipconfig /renew

```



**Результат:** таймаут, ошибка «**Произошла ошибка при обновлении интерфейса Ethernet: не удаётся связаться с DHCP-сервером**». После — **снова APIPA**.



**Вывод:** DHCP-сервер **не отвечает**.



### 3. Проверка связи с сервером



```cmd

ping 192.168.1.1

ping 192.168.1.10

```



**Результат:** 100% потерь — сервер недоступен. Что логично: без корректного IP клиент не может достучаться.



### 4. Поиск причины на сервере



Проверил статус службы DHCP на сервере:



```powershell

Get-Service -Name DHCPServer | Select Name, Status, StartType

```



**Результат:**

```

Name        Status   StartType

----        ------   ---------

DHCPServer  Stopped  Disabled

```



**Причина найдена:** служба **остановлена** и **отключена** от автозапуска.



### 5. Подтверждение в Event Viewer



Через **Event Viewer** → **Журналы Windows** → **Система** → фильтр по **Event ID 7040**.



Найдено событие:



> **Тип запуска службы "DHCP-сервер" был изменён с "Автоматически" на "отключена".**



Это **прямое доказательство**, что службу остановили вручную.



## Решение



### 1. Включить службу через services.msc



1. **Win + R** → `services.msc` → **Enter**.

2. Найти **«DHCP-сервер»**.

3. Двойной клик → **Тип запуска: Автоматически**.

4. **Применить** → **Запустить** → **ОК**.



### 2. Проверка



```powershell

Get-Service -Name DHCPServer | Select Name, Status, StartType

```



Ожидаемо:

```

Name        Status   StartType

----        ------   ---------

DHCPServer  Running  Automatic

```



### 3. Проверка с клиента



На клиенте:



```cmd

ipconfig /release

ipconfig /renew

ipconfig

```



**Результат:** клиент получил корректный IP `192.168.1.100` от DHCP-сервера. Интернет работает.



## Ключевые выводы



### Для специалиста L1



- **APIPA (169.254.X.X)** — первый сигнал, что клиент не получил IP от DHCP.

- **Event ID 7040** в System log — показывает **изменения типа запуска служб**. Отличный инструмент для диагностики «кто и когда отключил службу».

- **`Get-Service DHCPServer`** — быстрая проверка состояния службы.

- **DHCP Client (на клиенте) ≠ DHCP Server (на сервере)** — это **разные службы**. DHCP Client **запрашивает**, DHCP Server **выдаёт**. Не путать.

- **Правильный порядок диагностики:** `ipconfig /all` → `ping` шлюза → `ping` сервера → `ipconfig /renew` → проверка DHCP на сервере → логи.



### Для пользователя



- Проблема была **на стороне сервера**, а не его компьютера.

- Сообщение об ошибке (`не удаётся связаться с DHCP-сервером`) **точно указывало** направление поиска.



## Скриншоты



- [01-symptom-apipa.png](screenshots/01-symptom-apipa.png) — APIPA на клиенте

- [02-event-7040.png](screenshots/02-event-7040.png) — событие 7040 в Event Viewer

- [03-service-disabled.png](screenshots/03-service-disabled.png) — отключённая служба

- [04-after-fix.png](screenshots/04-after-fix.png) — корректный IP после решения



## Использованные технологии



- DHCP (Dynamic Host Configuration Protocol)

- APIPA (Automatic Private IP Addressing)

- PowerShell: `Get-Service`, `Set-Service`, `Stop-Service`

- Event Viewer — System log

- Event ID 7040 — Service Control Manager

- cmd: `ipconfig /release`, `ipconfig /renew`, `ipconfig /all`, `ping`

- services.msc

