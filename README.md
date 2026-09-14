# Домашняя лаборатория: Windows Server + Active Directory

## Цель
Получить практические навыки администрирования корпоративной инфраструктуры: развернуть контроллер домена, настроить DHCP, DNS, создать пользователей и подключить клиентскую машину к домену.

## Стек
- VirtualBox
- Windows Server 2019 (AD DS, DNS, DHCP)
- Windows 10 (клиент)

## Что сделано
- Установлен Windows Server 2019 в VirtualBox
- Настроен статический IP: 192.168.1.10
- Развёрнуты роли: AD DS, DNS, DHCP
- Создан домен lab.local
- Созданы пользователи: User1, User2, User3
- Настроена область DHCP (192.168.1.100–200)
- Клиент Windows 10 подключён к домену
- Проверена авторизация доменного пользователя (whoami → lab\user1)

## Проблемы и решения
- **APIPA у клиента (169.254.x.x)** — исправлено сменой типа сети в VirtualBox на «Внутренняя сеть» с именем `labnet`.
- **Ошибка «Install-ADDSForest не распознано»** — исправлено установкой роли через `Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools` и перезапуском PowerShell.
- **Ошибка пароля при вводе в домен** — использован формат `LAB\Администратор`.

## Скриншоты
- `screenshots/01-server-manager.png`
- `screenshots/02-ad-users.png`
- `screenshots/03-dhcp-scope.png`
- `screenshots/04-dns-zone.png`
- `screenshots/05-client-whoami-ipconfig.png`

