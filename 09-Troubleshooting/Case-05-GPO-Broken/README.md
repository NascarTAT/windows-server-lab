# Case 05. Удаление привязки GPO для OU Sales



## Ситуация



Клиент **User4 (Илья Солдатов, отдел Sales)** сообщает: пропали корпоративные обои, панель управления снова открывается.



- **Тип проблемы:** Group Policy

- **Клиент:** User4 (Sales)

- **Затронутый объект:** привязка GPO `Sales_Desktop_Policy` к OU Sales



## Диагностика



### 1. Проверка на клиенте



```cmd

gpresult /r

```



**Обнаружено:**

- `Применённые объекты групповой политики: Н/Д`

- Пользователь в OU `Sales` (`OU=Sales,OU=lab,OU=LabUsers,DC=lab,DC=local`).

- `Sales_Desktop_Policy` **не применён**.



**Вывод:** GPO не доходит до пользователя.



### 2. Проверка привязок на сервере



```powershell

Get-GPInheritance -Target "OU=Sales,OU=lab,OU=LabUsers,DC=lab,DC=local" | Select -ExpandProperty GpoLinks

```



**Результат:** пустой вывод — **привязки нет**.



### 3. Визуальная проверка в GPMC



**Server Manager** → **Средства** → **Управление групповой политикой**.



В дереве: у OU `HR` привязка `HR-Drive-Mapping` **есть**, у OU `Sales` — **пусто**.



**Причина подтверждена:** GPO `Sales_Desktop_Policy` существует, но **не связан** с OU Sales.



## Решение



### 1. Восстановление привязки



В **GPMC**:

1. ПКМ по **OU Sales** → **«Связать существующий объект групповой политики…»**.

2. Выбрать `Sales_Desktop_Policy` → **ОК**.



**Альтернатива через PowerShell:**



```powershell

New-GPLink -Name "Sales_Desktop_Policy" -Target "OU=Sales,OU=lab,OU=LabUsers,DC=lab,DC=local"

```



### 2. Применение политики



На клиенте:



```cmd

gpupdate /force

```



Затем **logoff** → вход под User4.



### 3. Проверка



```cmd

gpresult /r

```



В разделе **«Применённые объекты групповой политики»** появится `Sales_Desktop_Policy`.



**Подтверждено:**

- Запрет панели управления снова активен — при попытке открыть «Параметры» появляется сообщение **«Операция отменена из-за ограничений»**.

- Обои применяются согласно GPO.



## Причина сбоя



**В рамках учебного кейса:** привязка GPO была удалена вручную:



```powershell

Remove-GPLink -Name "Sales_Desktop_Policy" -Target "OU=Sales,OU=lab,OU=LabUsers,DC=lab,DC=local"

```



Сам GPO при этом **не удалялся** — только связь с OU.



**Что бы я делал в реальной работе для поиска причины:**



1. Проверил бы **журнал Directory Service** на контроллере — аудит изменений AD-объектов.

2. Настроил бы **SACL** на объектах OU/домена для аудита изменений привязок GPO — без этого события удаления ссылок **не логируются**.

3. Проверил бы **историю изменений** в системе управления конфигурацией (если есть).

4. Опросил бы коллег — не менял ли кто-то политики.



**Важно:** для логирования удаления GPO-ссылок **недостаточно** стандартного аудита AD. Требуется настроить **Audit Directory Service Access** и **SACL** на контейнере — это отдельная задача администрирования.



## Ключевые выводы



- **GPO и права на папки — независимые вещи.** Если GPO не настраивает File System permissions — удаление привязки **не влияет** на доступ к сетевым папкам.

- **`Get-GPInheritance`** — быстрая проверка привязок GPO через PowerShell.

- **`gpresult /r`** — основной инструмент диагностики на клиенте. Показывает, какие GPO применены, а какие — нет.

- **Сам GPO ≠ привязка GPO.** GPO хранится в `Group Policy Objects`, привязка — на OU. Удаление привязки **не удаляет** GPO, его можно переиспользовать.

- **Аудит GPO-изменений** требует настройки SACL — стандартные логи AD **не показывают**, кто удалил привязку.



## Скриншоты



- [01-gpmc-no-link.png](screenshots/01-gpmc-no-link.png) — GPMC: у OU Sales нет привязки

- [02-gpresult-not-applied.png](screenshots/02-gpresult-not-applied.png) — gpresult: GPO не применён

- [03-symptom-wallpaper.png](screenshots/03-symptom-wallpaper.png) — стандартные обои (симптом)

- [04-symptom-panel.png](screenshots/04-symptom-panel.png) — панель управления открывается (симптом)

- [05-gpmc-restored.png](screenshots/05-gpmc-restored.png) — GPMC: привязка восстановлена

- [06-panel-blocked.png](screenshots/06-panel-blocked.png) — запрет панели управления снова работает



## Использованные технологии



- Group Policy Management (GPMC)

- PowerShell: `Get-GPInheritance`, `Remove-GPLink`, `New-GPLink`

- cmd: `gpupdate /force`, `gpresult /r`

- Active Directory — OU, привязки GPO

- Event Viewer — журнал Directory Service

