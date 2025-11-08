# Magitrickle + SingBox Руководство

[MagiTrickle](https://magitrickle.dev) | [SingBox](https://sing-box.sagernet.org)

[Видео гайд по прошивке роутера Netis N6](https://www.youtube.com/watch?v=AQLE734jDZ8s)
[Видео руководство по установке Entware, MagiTrickle, Sing-box](https://www.youtube.com/watch?v=NjPYz0YgSMA&)

## Что потребуется

- Роутер Keenetic с USB портом (или с прошивкой Keenetic), подключенный к интернету;
- USB накопитель (хватит 1 ГБ, USB Hub работает с USB модемами);
- Свободное время.

Протестировано на: Keenetic Giga KN‑1011/KN‑1012, Keenetic Viva KN‑1913, Netis N6, Xiaomi MiRouter 3Gv1 (прошивка Keenetic).

## Настройка DoT и DoH вместо провайдерского DNS

Если уже настроено, пропустите этот пункт.

### Вариант 1: Настройка через интерфейс роутера

Перейдите в: «Сетевые правила» > «Интернет фильтры» > «Настройка DNS».

Добавьте серверы DNS‑over‑TLS:

- 1.1.1.1 — `cloudflare-dns.com`
- 1.0.0.1 — `cloudflare-dns.com`
- 8.8.4.4 — `dns.google.com`
- 8.8.8.8 — `dns.google.com`

![Настройка DNS серверов](https://kiarant.github.io/media/image1.png)

### Вариант 2: Настройка через CLI

Перейдите по ссылке `http://192.168.1.1/a`, или через «Шестеренку» > «Командная строка».

Поочередно введите команды и нажмите «Отправить»:

```
dns-proxy tls upstream 1.1.1.1 sni cloudflare-dns.com
dns-proxy tls upstream 1.0.0.1 sni cloudflare-dns.com
dns-proxy tls upstream 8.8.4.4 sni dns.google
dns-proxy tls upstream 8.8.8.8 sni dns.google
system configuration save
```

![Настройка DNS через CLI](https://kiarant.github.io/media/image2.png)
![Дополнительная настройка CLI](https://kiarant.github.io/media/image3.png)

В настройках интернет‑соединения включите «Игнорировать DNSv4 интернет‑провайдера» (и DNSv6, если используется IPv6).

![Игнорирование DNS провайдера](https://kiarant.github.io/media/image4.png)

## Установка Entware

### Подготовка флешки

Отформатируйте USB накопитель в ext4. Используйте утилиты, такие как ~~MiniTool Partition Wizard~~ (не рекомендуется) Paragon Partition Manager, так как Windows не поддерживает ext4 стандартными средствами.

[Ссылка на Keenetic Help](https://help.keenetic.com)

![Форматирование флешки](https://kiarant.github.io/media/image5.png)

### Подготовка роутера

В общих настройках системы «Изменить набор компонентов», установите компонент «Поддержка открытых пакетов» (OPKG).

![Установка компонента OPKG](https://kiarant.github.io/media/image6.png)

### Вариант установки 1: Через CLI

1. Зайдите в командную строку роутера: `http://192.168.1.1/a` или через «Шестеренку» > «Командная строка».  
   ![Доступ к командной строке](https://kiarant.github.io/media/image8.png)
2. Введите: `opkg disk`, нажмите Tab, выберите имя флешки (как вы назвали ее при форматировании).  
   ![Выбор флешки в CLI](https://kiarant.github.io/media/image9.png)
3. Вставьте ссылку для вашей архитектуры:
   - mipsel: https://bin.entware.net/mipselsf-k3.4/installer/mipsel-installer.tar.gz
   - mips: https://bin.entware.net/mipssf-k3.4/installer/mips-installer.tar.gz
   - aarch: https://bin.entware.net/aarch64-k3.10/installer/aarch64-installer.tar.gz  
   ![Установка Entware через CLI](https://kiarant.github.io/media/image10.png)
4. Нажмите «Отправить запрос».
5. Перейдите во вкладку «Диагностика», откройте журнал, дождитесь: `Opkg::Manager: /opt/etc/init.d/doinstall: [5/5] "Entware" installed!`  
   ![Журнал установки](https://kiarant.github.io/media/image15.png)

### Вариант установки 2: Через интерфейс

1. Скачайте инсталлятор Entware для вашего роутера (ссылки в варианте 1), например: `mipsel-installer.tar.gz`.
2. Вставьте флешку в роутер, зайдите в «Приложения», выберите флешку.  
   ![Выбор флешки в приложениях](https://kiarant.github.io/media/image11.png)
3. Создайте папку `install` в корне флешки, загрузите туда инсталлятор.  
   ![Создание папки install](https://kiarant.github.io/media/image12.png)  
   ![Загрузка инсталлятора](https://kiarant.github.io/media/image13.png)
4. Перейдите во вкладку OPKG, выберите накопитель, нажмите «Сохранить».  
   ![Настройка OPKG](https://kiarant.github.io/media/image14.png)

## Установка Magitrickle

Используйте SSH‑клиент (PuTTY, Kitty, Termius). Подключитесь к роутеру:

- Адрес: IP роутера, порт: 222 (или 22, если «Сервер SSH» не установлен).  
- Логин: `root`, пароль: `keenetic` (смените пароль командой `passwd`).

![Подключение через SSH](https://kiarant.github.io/media/image16.png)
![Ввод логина и пароля / смена пароля](https://kiarant.github.io/media/image17.png)

Поочередно введите команды:

```
wget -qO- http://bin.magitrickle.dev/packages/add_repo.sh | sh
opkg update && opkg install magitrickle
/opt/etc/init.d/S99magitrickle start
```

Для обновления пакета в дальнейшем можно использовать команду:

```
opkg update && opkg install magitrickle
/opt/etc/init.d/S99magitrickle restart
```

Magitrickle запущен. Проверить можно через браузер: `адрес_роутера:8080`.

![Интерфейс Magitrickle](https://kiarant.github.io/media/image19.png)

## Установка Sing‑Box

Установите Sing‑Box:

```
opkg install sing-box-go
```

![Установка Sing‑Box](https://kiarant.github.io/media/image20.png)

### Создание конфига

Создать конфиг можно вручную, или использовать генераторы конфига:

- [Kiarant Converter](https://kiarant.github.io/converter)

Примечание: 100% работоспособность конфига не гарантируется ввиду отсутствия возможности проверить все конфигурации.

Загрузите конфиг в `/opt/etc/sing-box/` (замените стандартный). Я использовал WinSCP. Новое подключение: адрес роутера, порт 222, логин `root`, пароль от entware (если не изменили: `keenetic`).

![Загрузка конфига через WinSCP](https://kiarant.github.io/media/image21.png)
![Настройка WinSCP](https://kiarant.github.io/media/image22.png)

## Настройка маршрутизации

Перейдите в интерфейс Magitrickle: `192.168.1.1:8080`

1. Нажмите «+», создайте группу.
2. В выпадающем списке выбираем нужный нам интерфейс: `tun0` или `singtun0` (или как вы его обозвали).  
   ![Создание группы в Magitrickle](https://kiarant.github.io/media/image27.png)
3. Для проверки укажите в Pattern, например, `2ip.ru`, сохраните.  
   ![Настройка Pattern](https://kiarant.github.io/media/image28.png)
4. Откройте браузер в режиме инкогнито, зайдите на https://2ip.ru, проверьте.  
   ![Проверка на 2ip.ru](https://kiarant.github.io/media/image29.png)

Можете создавать разные группы с разными интерфейсами. Дальше ваша фантазия.

Список доменов для разных сервисов можете найти по ссылке https://iplist.opencck.org (формат «Text», тип «Домены», только wildcard).

![Список доменов](https://kiarant.github.io/media/image30.png)

Для импорта больших списков можете использовать кнопку «Импортировать список»

![Импорт списков](https://kiarant.github.io/media/image_mass_import.png)
![Вставка списков](https://kiarant.github.io/media/image_mass_import2.png)
![Результат импорта списков](https://kiarant.github.io/media/image_mass_import3.png)

Нажимаем «Сохранить изменения».

![Сохранение изменений](https://kiarant.github.io/media/image_MT_save.png)

## Типы правил маршрутизации

В интерфейсе Magitrickle доступны следующие типы правил:

- **Namespace (Именное пространство)**: Охватывает домен и поддомены. Например, `example.com`.  
  ![Namespace](https://kiarant.github.io/media/image31.png)
- **Wildcard (Подстановочный шаблон)**: Использует `*` и `?`. Например, `*example.com`.  
  ![Wildcard](https://kiarant.github.io/media/image32.png)
- **Domain (Точный домен)**: Только указанный домен, без поддоменов. Например, `sub.example.com`.  
  ![Domain](https://kiarant.github.io/media/image33.png)
- **RegExp (Регулярное выражение)**: Для опытных пользователей, использует парсер `dlclark/regexp2`. Например, `^[a-z]*example\.com$`.  
  ![RegExp](https://kiarant.github.io/media/image34.png)
