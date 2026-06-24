РАЗВЕРТЫВАНИЕ СТЕНДА
Создадим в PNetLab такую схему:
<img width="489" height="612" alt="image" src="https://github.com/user-attachments/assets/fbd194f9-9ff4-4e87-b35e-97cfc1ffebfb" />
1.	Net: cloud_nat
2.	ISP: 1 CPU, 1024 RAM, 3 Ethernet
3.	HQ-RTR: 1 CPU, 1024 RAM, 4 Ethernet
4.	BR-RTR: 1 CPU, 1024 RAM, 2 Ethernet
5.	HQ-SRV: 1 CPU, 1024 RAM, 1 Ethernet
6.	HQ-CLI: 1 CPU, 1024 RAM, 1 Ethernet
7.	BR-SRV: 1 CPU, 1024 RAM, 1 Ethernet

Все устройства на базе Linux Debian 13.
 
ВНИМАНИЕ!!! ТО ЧТО ОПИСАНО НИЖЕ ОЧЕНЬ ВАЖНО!!!
ПРОДЕЛАТЬ НА ВСЕХ УСТРОЙСТВАХ!!!
На экране входа в пользователя вводим стандартные данные учетной записи root. Пароль Test123.
<img width="801" height="601" alt="image" src="https://github.com/user-attachments/assets/9ba7795c-f74c-40dc-b4df-23857f5d5359" />
Первым делом открываем терминал
<img width="710" height="534" alt="image" src="https://github.com/user-attachments/assets/42a53b78-e566-43be-8a1f-f247f6b6f1d8" />
И редактируем репозиторий через редактор nano
<img width="769" height="67" alt="image" src="https://github.com/user-attachments/assets/2c6f7a07-f42b-4df3-8e2c-b75352d93d27" />
Для тех у кого не работает инет в пинете надо открыть конфиг nano
/etc/resolv.conf и nameserver поменять на 8.8.8.8 или если стоит search полностью стереть строку и вписать nameserver 8.8.8.8
НА 10 ПУНКТЕ МОЖЕТ ВЫЛЕЗТИ ОШИБКА ПРИ СКАЧИВАНИИ bind9, ЕСЛИ ЭТО ПРОИЗОЙДЕТ, ПИШЕМ sudo apt-get update
Ставим комментарий на первой строчке символом #
<img width="837" height="416" alt="image" src="https://github.com/user-attachments/assets/6e124d25-0b34-47d0-bb47-905e42e032cb" />
Выходим Ctrl+X, y, Enter.
! Почему это важно?
—	Это важно, потому что Debian по умолчанию использует репозиторий с локального .iso образа (CD-ROM). Так как Debian уже установлен и CD-ROM извлечен, установщик пакетов apt будет ждать образ Debian в виртуальном дисководе и брать оттуда пакеты. Но он его не дождется, и пакеты устанавливаться не начнут. Поэтому отключаем этот репозиторий на всех устройствах, чтобы установка производилась через Интернет.
 
МОДУЛЬ 1

1.	Произведите базовую настройку устройств:
•	Настройте имена устройств согласно топологии. Используйте полное доменное имя
•	На всех устройствах необходимо сконфигурировать IPv4:
o	IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918
o	Локальная сеть в сторону HQ-SRV(VLAN 100) должна вмещать не более 32 адресов
o	Локальная сеть в сторону HQ-CLI(VLAN 200) должна вмещать не менее 16 адресов
o	Локальная сеть для управления(VLAN 999) должна вмещать не более 8 адресов
o	Локальная сеть в сторону BR-SRV должна вмещать не более 16 адресов
Зададим имена устройствам полным доменным именем:
На ISP:
<img width="972" height="35" alt="image" src="https://github.com/user-attachments/assets/538702b1-2a7a-4063-a185-1e79f54f293b" />
На HQ-RTR:
<img width="979" height="32" alt="image" src="https://github.com/user-attachments/assets/ad4ab2b3-6425-41fe-a672-7fe4aab64972" />
На BR-RTR:
<img width="976" height="31" alt="image" src="https://github.com/user-attachments/assets/54fdb86e-e764-4335-8faa-d61759f2da2e" />
На HQ-SRV:
<img width="981" height="36" alt="image" src="https://github.com/user-attachments/assets/00c6814f-0ced-4242-9160-eb1c5a05a321" />
На HQ-CLI:
<img width="971" height="42" alt="image" src="https://github.com/user-attachments/assets/ab7fe3d4-053c-4a6a-8f07-1284bbd75cac" />
На BR-SRV:
<img width="951" height="32" alt="image" src="https://github.com/user-attachments/assets/dfd18abf-6d73-405a-925f-8e9cead7c70b" />

Сконфигурируем IPv4.
Руководствуясь требованиями задания составим таблицу IP-адресации.

Имя устройства	IP-адрес	Шлюз	по умолчанию	Сеть
ISP	DHCP		Интернет
	172.16.1.1/28	-	От ISP к HQ-RTR
	172.16.2.1/28	-	От ISP к BR-RTR
HQ-RTR	172.16.1.2/28	172.16.1.1	От ISP к HQ-RTR
	192.168.100.1/27	-	От HQ-RTR к
HQ-SRV (VLAN100)
	192.168.100.33/28	-	От HQ-RTR к HQ-CLI (VLAN200)
	192.168.100.49/29	-	VLAN999
HQ-SRV	192.168.100.2/27	192.168.100.1	От	HQ-RTR	к
HQ-SRV
HQ-CLI	DHCP	192.168.100.33 (DHCP)	От	HQ-RTR	к
HQ-CLI
BR-RTR	172.16.2.2/28	172.16.2.1	От ISP к BR-RTR
	192.168.200.1/28	-	От BR-RTR к BR- SRV
BR-SRV	192.168.200.2/28	192.168.200.1	От BR-RTR к BR- SRV

Как мы видим, ISP, HQ-RTR и BR-RTR используют сетевую часть 172.16.*.* с маской подсети 28.
Данная сетевая часть будет использоваться для соединения между ISP и HQ- RTR, а также между ISP и BR-RTR.
Все что далее, использует сетевую часть 192.168.*.*

2.	Настройте доступ к сети Интернет, на маршрутизаторе ISP:
Настройте адресацию на интерфейсах:
•	Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP
•	Настройте маршрут по умолчанию, если это необходимо
•	Настройте интерфейс, в сторону HQ-RTR, интерфейс подключен к сети
172.16.1.0/28
•	Настройте интерфейс, в сторону BR-RTR, интерфейс подключен к сети
172.16.2.0/28
•	На ISP настройте динамическую сетевую трансляцию портов для доступа к сети Интернет HQ-RTR и BR-RTR.
P.S. В данном задании мы сразу сделаем задание 8, чтобы не терять время.
8.	Настройка	динамической	трансляции	адресов	маршрутизаторах
HQ-RTR и BR-RTR:
•	Настройте динамическую трансляцию адресов для обоих офисов в сторону ISP, все устройства в офисах должны иметь доступ к сети Интернет
Проведем	базовую	настройку	сети	через	конфигурационный	файл
/etc/network/interfaces.
На ISP:
<img width="719" height="57" alt="image" src="https://github.com/user-attachments/assets/9c6f7f80-576e-4cf3-85ef-3b6ce2c4f23e" />
<img width="786" height="800" alt="image" src="https://github.com/user-attachments/assets/e4f9798f-2bd3-428d-b5c0-ed854d69c5e7" />
•	auto ens3 – автоматическое включение интерфейса
•	iface ens3 inet dhcp/static – тип интерфейса (динамический/статический)
•	address 172.16.1.1/28 – задание IP-адреса на интерфейс
•	post-up nft -f /etc/nftables.conf – исполнение команды после загрузки конфигурационного файла (в данном случае это применение настроек файла nftables)
Выходим с файла: Ctrl+X, y, Enter.
СДЕЛАТЬ НА ISP, HQ-RTR И BR-RTR СТРОГО, ИНАЧЕ ИНЕТ НЕ БУДЕТ РАБОТАТЬ
Редактируем sysctl.conf 
<img width="766" height="42" alt="image" src="https://github.com/user-attachments/assets/3a865d38-6933-41c0-9f68-f0ce13eddfb0" />
<img width="491" height="120" alt="image" src="https://github.com/user-attachments/assets/221034f2-1795-4cfe-b6f0-3a1730bdaffc" />
Выходим с файла: Ctrl+X, y, Enter.
Применяем правила sysctl
<img width="531" height="47" alt="image" src="https://github.com/user-attachments/assets/a18b19d5-4966-4681-8abe-1deee20698cd" />
Сделаем динамическую трансляцию адресов (проделать также на HQ- RTR, BR-RTR!)
<img width="635" height="47" alt="image" src="https://github.com/user-attachments/assets/84ad1c13-6845-4056-8352-d8b7989e600e" />
<img width="974" height="693" alt="image" src="https://github.com/user-attachments/assets/6caae1c6-1e6d-4e9d-b6af-6a9057bd6e62" />
<img width="592" height="323" alt="image" src="https://github.com/user-attachments/assets/e2e7b846-0319-4dcd-84b9-d60320dd2c70" />
Приводим файл к такому виду (делаем маскарадинг)
<img width="975" height="625" alt="image" src="https://github.com/user-attachments/assets/8c42c255-a9b9-4d9b-85aa-4a1d362e4cb7" />
ОБЯЗАТЕЛЬНО ПРОВЕРИТЬ ПРАВИЛЬНОСТЬ НАПИСАНИЯ!!! ОДНА ОШИБКА И ТЫ ОШИБСЯ
Перезагружаем	службу	сети	(делать	рекомендую	почаще	на	всех устройствах, если какие-то проблемы)
<img width="724" height="36" alt="image" src="https://github.com/user-attachments/assets/3539edff-37fa-4d7e-b4f2-c77f0dffbd8a" />
Проверяем IP-адреса
<img width="977" height="126" alt="image" src="https://github.com/user-attachments/assets/b202607c-6f59-4876-81ed-9e889f5c29a1" />
В случае, если какие-то IP-адреса пропадают — перезагружаем службы NetworkManager и networking по очереди:
<img width="814" height="42" alt="image" src="https://github.com/user-attachments/assets/9affc7f9-165c-491a-96bb-8b16657e1d46" />
<img width="695" height="34" alt="image" src="https://github.com/user-attachments/assets/c6db7d34-6e7d-454b-90f8-5a77133c0fd0" />
На HQ-RTR:
<img width="821" height="46" alt="image" src="https://github.com/user-attachments/assets/44365661-c453-4fc8-ab1e-efd0621c52ad" />
<img width="821" height="46" alt="image" src="https://github.com/user-attachments/assets/0301b322-f501-4471-82dc-89597b3d8206" />
•	gateway 172.16.1.1 – указание шлюза (IP-адрес предыдущего устройства, к которому подключено устройство по сети. В нашем случае это ISP.)
Выходим с файла: Ctrl+X, y, Enter.
Перезагружаем службу сети
<img width="783" height="62" alt="image" src="https://github.com/user-attachments/assets/b3bbac5d-fb0a-4d5d-a967-1407f6019f4b" />
Проверяем IP-адреса
<img width="976" height="204" alt="image" src="https://github.com/user-attachments/assets/7812b5c4-6a36-4c00-941a-d1c31a820c89" />
Попробуем пингануть с ISP HQ-RTR
<img width="974" height="259" alt="image" src="https://github.com/user-attachments/assets/f00d57c5-1798-49ee-9566-d21b5f20be64" />
А также в обратную сторону
<img width="977" height="298" alt="image" src="https://github.com/user-attachments/assets/26f03211-8509-4624-b41e-30b1bab14ba0" />
Проверим также и Интернет
<img width="954" height="278" alt="image" src="https://github.com/user-attachments/assets/3495b6b9-014f-4a22-a451-492f60341c0a" />
На BR-RTR:
<img width="783" height="39" alt="image" src="https://github.com/user-attachments/assets/c89cf2fb-c428-4403-9ded-78d4a7209385" />
<img width="748" height="805" alt="image" src="https://github.com/user-attachments/assets/82f965a7-6797-4b60-b965-148d0895481d" />
Перезагружаем службу сети
<img width="781" height="42" alt="image" src="https://github.com/user-attachments/assets/11913bf2-2713-441d-800c-769530325979" />
Проверяем IP-адреса
<img width="936" height="102" alt="image" src="https://github.com/user-attachments/assets/b667363f-733f-4947-95ff-4e262be55f83" />
На HQ-SRV:
<img width="803" height="50" alt="image" src="https://github.com/user-attachments/assets/8bad17ba-caa1-4206-84ee-c8a188ca8e08" />
<img width="691" height="810" alt="image" src="https://github.com/user-attachments/assets/994fde5d-f482-4973-a378-b2dea68cb8df" />
Перезагружаем службу сети
<img width="780" height="41" alt="image" src="https://github.com/user-attachments/assets/e5dbb523-0f41-40c8-87bd-d5893618bc6d" />
Проверяем IP-адреса
<img width="952" height="72" alt="image" src="https://github.com/user-attachments/assets/63b462da-0f61-4cb4-b127-57e813c6cbca" />
На HQ-CLI (временно, до 9 задания, позже он IP будет получать по DHCP):
<img width="805" height="46" alt="image" src="https://github.com/user-attachments/assets/8c8927a8-1b59-4861-a2b3-bdd43f7d8279" />
<img width="803" height="44" alt="image" src="https://github.com/user-attachments/assets/cff04ba5-b1b3-4d9b-ad54-32c1722b931b" />
Перезагружаем службу сети
<img width="700" height="814" alt="image" src="https://github.com/user-attachments/assets/fae20a9c-8bed-4e90-8ba6-4506b53dbb1f" />
Проверяем IP-адреса
<img width="954" height="74" alt="image" src="https://github.com/user-attachments/assets/24d24a81-fe42-4188-9087-38900c4f35c5" />
На BR-SRV:
<img width="810" height="45" alt="image" src="https://github.com/user-attachments/assets/98a8222e-2cd9-4c68-b388-8d792d5446ab" />
<img width="713" height="804" alt="image" src="https://github.com/user-attachments/assets/51dbf307-670b-45d9-8daf-57fed28123d3" />
Перезагружаем службу сети
<img width="803" height="50" alt="image" src="https://github.com/user-attachments/assets/cb48ccaa-7fd4-4bae-9b31-ee4b05670017" />
Проверяем IP-адреса
<img width="947" height="121" alt="image" src="https://github.com/user-attachments/assets/bad6b389-b7fa-4ea2-b101-deb401832ed2" />
Рекомендую повторно проверить везде IP-адреса. Если что-то отвалилось:
<img width="538" height="49" alt="image" src="https://github.com/user-attachments/assets/cd383ccd-fee7-4149-ae88-c13862f9bfb9" />

3.	Создайте локальные учетные записи на серверах HQ-SRV и BR-SRV:
•	Создайте пользователя sshuser
•	Пароль пользователя sshuser с паролем P@ssw0rd
•	Идентификатор пользователя 2026
•	Пользователь sshuser должен иметь возможность запускать sudo
без ввода пароля
•	Создайте пользователя net_admin на маршрутизаторах HQ-RTR и
BR-RTR
•	Пароль пользователя net_admin с паролем P@ssw0rd
•	При настройке ОС на базе Linux, запускать sudo без ввода пароля
•	При настройке ОС отличных от Linux пользователь должен обладать максимальными привилегиями.

ЕСЛИ  СЛУЧАЙНО ДОПУСТИЛИ ОШИБКУ В СОЗДАНИИ ПОЛЬЗОВАТЕЛЯ, МОЖНО ЕГО УДАЛИТЬ С ПОМОЩЬЮ ЭТИХ КОМАНД:
rm –r /home/имя пользователя
userdel имя пользователя
После этого уже внимательно можно создать снова

Создаем пользователей sshuser на HQ-SRV и BR-SRV
<img width="967" height="37" alt="image" src="https://github.com/user-attachments/assets/2261fe09-413b-4df3-960a-297d93600c73" />
<img width="977" height="39" alt="image" src="https://github.com/user-attachments/assets/1b72d76e-ba06-41b1-adf2-4929d97b07bb" />
Даем привилегии sudo
<img width="736" height="42" alt="image" src="https://github.com/user-attachments/assets/243dab70-5b03-40ee-950c-4df208d3d2e8" />
<img width="706" height="41" alt="image" src="https://github.com/user-attachments/assets/c7d0a936-4b2f-48be-83ef-eb7f0003fc3b" />
Ставим пароль P@$$word на пользователя sshuser
<img width="569" height="133" alt="image" src="https://github.com/user-attachments/assets/1fd21122-ae63-4ad9-ae6a-30df786e7b2b" />
Заходим в visudo
<img width="403" height="45" alt="image" src="https://github.com/user-attachments/assets/6153eaf0-5831-4a75-8bc2-9ac06fc470ff" />
<img width="372" height="42" alt="image" src="https://github.com/user-attachments/assets/0ebea14a-2ff1-4a09-bba7-9adc6c91d198" />
И на HQ-SRV и на BR-SRV ОБЯЗАТЕЛЬНО в конце файла visudo
добавляем строчку
<img width="617" height="41" alt="image" src="https://github.com/user-attachments/assets/97ea55eb-6982-4266-b5d1-b6f62bd48466" />
После слова sshuser нажимаем TAB, пишем ALL=(ALL:ALL), затем жмем пробел и пишем NOPASSWD:ALL
<img width="483" height="40" alt="image" src="https://github.com/user-attachments/assets/b79b29bd-6b18-4b1b-88be-a55c68ecdb01" />
Проверяем наших пользователей sshuser
<img width="903" height="820" alt="image" src="https://github.com/user-attachments/assets/aaad3d1e-a98a-49dd-9a0c-1fc456c0c054" />
Обратим внимание, что при логине пароль не требуется
<img width="408" height="42" alt="image" src="https://github.com/user-attachments/assets/376242dd-8d40-4df2-a123-90a941b724ee" />
Повышаем права через sudo –i, видим что также все работает без пароля
<img width="314" height="89" alt="image" src="https://github.com/user-attachments/assets/d65d7e72-fe14-47e4-ab00-6298d238d5c0" />
Создаем пользователей net_admin на HQ-RTR и BR-RTR
<img width="919" height="42" alt="image" src="https://github.com/user-attachments/assets/b8c3edc6-26d2-4126-a913-52c7c768789f" />
<img width="917" height="42" alt="image" src="https://github.com/user-attachments/assets/4854aa43-d9de-4a11-8405-4c6c60ec86cf" />
Даем привилегии sudo
<img width="764" height="45" alt="image" src="https://github.com/user-attachments/assets/dde6dbe4-1a25-4e01-8a01-74ffa6f1de29" />
<img width="761" height="44" alt="image" src="https://github.com/user-attachments/assets/eca3d014-7fdc-4302-a22d-c6c6870d7bb6" />
Ставим пароль P@ssw0rd на пользователя net_admin
<img width="792" height="173" alt="image" src="https://github.com/user-attachments/assets/bb4dcb86-87cb-412d-8efd-8715ae059757" />
Заходим в visudo
<img width="416" height="46" alt="image" src="https://github.com/user-attachments/assets/d95cbd04-9ff6-44eb-ad0b-ba2b95332387" />
<img width="397" height="47" alt="image" src="https://github.com/user-attachments/assets/ca1bfd79-629b-47a8-be11-8dee4d3500d2" />
И на HQ-RTR и на BR-RTR ОБЯЗАТЕЛЬНО в конце файла visudo
добавляем строчку
<img width="731" height="43" alt="image" src="https://github.com/user-attachments/assets/7ad69d0d-91ba-49db-8877-4091f1ab21cd" />
<img width="786" height="811" alt="image" src="https://github.com/user-attachments/assets/5ca8a12d-4f21-4b04-8ded-2215f11f89b2" />
Также все работает без пароля
<img width="560" height="159" alt="image" src="https://github.com/user-attachments/assets/38f9183c-0990-48d0-93e1-0df15cce26fe" />

4.	Настройте коммутацию в сегменте HQ следующим образом:
•	Трафик HQ-SRV должен принадлежать VLAN 100
•	Трафик HQ-CLI должен принадлежать VLAN 200
•	Предусмотреть возможность передачи трафика управления в VLAN 999
•	Реализовать на HQ-RTR маршрутизацию трафика всех указанных VLAN с использованием одного сетевого адаптера ВМ/физического порта
•	Сведения о настройке коммутации внесите в отчёт На HQ-RTR установим OpenvSwitch:
•	Если не скачивается, выдает Игн:... Проверяем интернет.
<img width="975" height="632" alt="image" src="https://github.com/user-attachments/assets/08aeb68c-ce23-41e2-958f-cc1c85bcde98" />
Создаем мост на виртуальном коммутаторе
<img width="683" height="45" alt="image" src="https://github.com/user-attachments/assets/0a18af18-5055-4bde-9e6c-194b943cfc7f" />
Добавляем физические интерфейсы на виртуальный коммутатор с тегами
<img width="888" height="101" alt="image" src="https://github.com/user-attachments/assets/437fdd82-5b42-4c22-8253-f6f0a60d431a" />
Добавляем VLAN-интерфейсы
<img width="975" height="142" alt="image" src="https://github.com/user-attachments/assets/f2f4c0f6-c4b9-4be0-bd96-70a4fd17626f" />
Отредактируем файл /etc/network/interfaces. В нем нужно будет поднять
VLAN-интерфейсы, назначить им IP-адреса, а также поднять мост
<img width="792" height="41" alt="image" src="https://github.com/user-attachments/assets/cf3a44fd-3437-476b-bdf2-972fe7da2e4a" />
Модифицируем файл таким образом:
<img width="794" height="46" alt="image" src="https://github.com/user-attachments/assets/39209748-a35e-4ea1-98b8-c0f64e4976fe" />
Перезагружаем сеть
<img width="805" height="984" alt="image" src="https://github.com/user-attachments/assets/ac6656b3-1865-4c2c-94cc-a7acc1126b3c" />
Проверяем адресацию
<img width="974" height="261" alt="image" src="https://github.com/user-attachments/assets/fa4919e7-8936-459c-bc15-f22ab8ffe35b" />
Теперь попробуем пингануть HQ-SRV и HQ-CLI с HQ-RTR
<img width="975" height="444" alt="image" src="https://github.com/user-attachments/assets/f1d72d24-b97c-40e8-b5a3-d06df8d261a7" />
Также на HQ-SRV и на HQ-CLI теперь появился интернет
<img width="937" height="267" alt="image" src="https://github.com/user-attachments/assets/001f6b43-8b77-455c-be6d-77242dbcfdc0" />

5.	Настройте безопасный удаленный доступ на серверах HQ-SRV и
BR-SRV:
•	Для подключения используйте порт 2026
•	Разрешите подключения исключительно пользователю sshuser
•	Ограничьте количество попыток входа до двух
•	Настройте баннер «Authorized access only».
Устанавливаем SSH-сервер на HQ-SRV и BR-SRV
<img width="778" height="43" alt="image" src="https://github.com/user-attachments/assets/31854ce4-1185-406b-b13c-0f50d0204565" />
<img width="772" height="41" alt="image" src="https://github.com/user-attachments/assets/affc023d-9c1a-4f26-bfae-ddab9ce1b620" />
Редактируем конфигурацию
<img width="749" height="36" alt="image" src="https://github.com/user-attachments/assets/f5f15f54-0914-439e-aad8-92606409cb83" />
<img width="955" height="716" alt="image" src="https://github.com/user-attachments/assets/9313bbe0-44ea-418e-8698-63136a977722" />
Раскомментируем строчку Port 22, и изменим порт на 2026
<img width="391" height="143" alt="image" src="https://github.com/user-attachments/assets/7bbc8edb-f8df-41ad-b7f6-2f340eccfee2" />
Разрешим вход только пользователем sshuser
<img width="908" height="804" alt="image" src="https://github.com/user-attachments/assets/02598e58-3327-4e6c-a98e-11216ba09494" />
Ограничиваем на две попытки входа
<img width="403" height="131" alt="image" src="https://github.com/user-attachments/assets/cbb0403b-e030-4207-af91-2e236c563e88" />
Зададим также баннер
<img width="446" height="156" alt="image" src="https://github.com/user-attachments/assets/d6d01cbb-a877-40d2-8275-f2841a27d84b" />
Выходим с файла
Создадим баннер по пути /etc/ssh_banner
<img width="653" height="47" alt="image" src="https://github.com/user-attachments/assets/d26ca0b0-cd6f-4c6f-b54c-77daf5b4213b" />
В баннере звездочками делаем рамку, и вписываем туда «Authorized access
only»
<img width="814" height="811" alt="image" src="https://github.com/user-attachments/assets/576f2a33-9321-48c2-814c-a2aa7dd67703" />
Тоже самое делаем и на BR-SRV.
Перезагружаем SSH
<img width="702" height="39" alt="image" src="https://github.com/user-attachments/assets/84c1c70b-2a3d-4102-9eb5-daed22edbd8e" />
<img width="695" height="39" alt="image" src="https://github.com/user-attachments/assets/11f8302a-1c45-4136-8465-ee984678b79a" />
Попробуем подключиться по SSH с HQ-CLI на HQ-SRV
<img width="902" height="44" alt="image" src="https://github.com/user-attachments/assets/9acc2c66-4e2c-4bed-8a70-be8a38d29ae4" />
Пишем «yes»
<img width="979" height="80" alt="image" src="https://github.com/user-attachments/assets/9301818a-c4fe-4d8d-a794-a34f5d572245" />
Баннер отображается корректно. Вводим пароль
<img width="728" height="255" alt="image" src="https://github.com/user-attachments/assets/e9c8f1c2-f52e-4ec5-a65d-f79f74e9731f" />
Как видим, все работает
<img width="954" height="259" alt="image" src="https://github.com/user-attachments/assets/f25b3851-489a-4055-ab62-e826b9c51bad" />

6.	Между офисами HQ и BR, на маршрутизаторах HQ-RTR и BR-
RTR необходимо сконфигурировать ip туннель:
•	На выбор технологии GRE или IP in IP
•	Сведения о туннеле занесите в отчёт.
Производим настройку на HQ-RTR.
•	Выбираем «Редактировать подключения»
•	Выбираем «Добавить»
•	Выбираем «Туннель IP»
•	Задаём понятные имена «Имя профиля» и «Устройство»
•	«Режим» выбираем «GRE»
•	«Родительский» указываем интерфейс в сторону ISP (ens3)
•	Задаём «Локальный IP» (IP на интерфейсе HQ-RTR в сторону ISP 172.16.1.2)
•	Задаём «Удаленный IP» (IP на интерфейсе BR-RTR в сторону ISP 172.16.2.2)
•	Переходим к «КОНФИГУРАЦИЯ IPv4», переключаем на «Вручную»
•	Задаём адрес IPv4 для туннеля (10.10.0.1/30)
<img width="806" height="42" alt="image" src="https://github.com/user-attachments/assets/47cad913-2157-4a68-9a46-f27f369c9038" />
<img width="780" height="611" alt="image" src="https://github.com/user-attachments/assets/b25274c0-3c1d-4e73-97ca-0b15db912e96" />
Выходим
Заходим в файл /etc/network/interfaces
<img width="531" height="83" alt="image" src="https://github.com/user-attachments/assets/2e6eb6e5-49e7-4955-90f1-0c7c76b1ec95" />
В самом конце файла добавляем строчку для поднятия интерфейса GRE
<img width="975" height="753" alt="image" src="https://github.com/user-attachments/assets/89baf958-1c03-4cfa-ae5a-ca4c27b50848" />
Перезапускаем службу сети
<img width="775" height="37" alt="image" src="https://github.com/user-attachments/assets/9c2dbc98-a632-4ecc-a315-f90c144d6f12" />
Проверяем, что gre0 перешел в статус UNKNOWN и tun1 приобрел IP-адрес
<img width="973" height="114" alt="image" src="https://github.com/user-attachments/assets/70c77b89-8748-4700-ab31-dd4ceeddee79" />
Изменяем TTL интерфейса GRE на 64
<img width="975" height="35" alt="image" src="https://github.com/user-attachments/assets/f67367f4-e296-4d70-9930-7c44e2bc9b3e" />
Производим настройку на BR-RTR.
•	Выбираем «Редактировать подключения»
•	Выбираем «Добавить»
•	Выбираем «Туннель IP»
•	Задаём понятные имена «Имя профиля» и «Устройство»
•	«Режим» выбираем «GRE»
•	«Родительский» указываем интерфейс в сторону ISP (ens3)
•	Задаём «Локальный IP» (IP на интерфейсе BR-RTR в сторону ISP 172.16.2.2)
•	Задаём «Удаленный IP» (IP на интерфейсе HQ-RTR в сторону ISP 172.16.1.2)
•	Переходим к «КОНФИГУРАЦИЯ IPv4», переключаем на «Вручную»
•	Задаём адрес IPv4 для туннеля (10.10.0.2/30)
<img width="792" height="47" alt="image" src="https://github.com/user-attachments/assets/27ca002d-40a1-4e97-94cd-9e84c3f09ff4" />
Выходим
Заходим в файл /etc/network/interfaces
<img width="606" height="65" alt="image" src="https://github.com/user-attachments/assets/aaed39a8-cf13-4aea-9ce9-c469f0c87d57" />
В самом конце файла добавляем строчку для поднятия интерфейса GRE
<img width="799" height="52" alt="image" src="https://github.com/user-attachments/assets/6fe30b3b-9d09-4bf9-862c-a72c1b475f82" />
Перезапускаем службу сети
<img width="955" height="536" alt="image" src="https://github.com/user-attachments/assets/c74f2ed8-3e42-4a28-85d1-d08765de5368" />
Проверяем, что gre0 перешел в статус UNKNOWN и tun1 приобрел IP-адрес
<img width="926" height="114" alt="image" src="https://github.com/user-attachments/assets/401cd773-e1a6-4441-8d76-a3aecda20096" />
Изменяем TTL интерфейса GRE на 64
<img width="982" height="40" alt="image" src="https://github.com/user-attachments/assets/f3e93f9f-63bf-45c7-b92b-011bdfd56851" />
Если тут вылезает ошибка, проверьте внимательно nmtui, не должно стоять поcле gre пробела 
Попробуем пингануть с BR-RTR HQ-RTR по IP-адресу 10.10.0.1
<img width="800" height="268" alt="image" src="https://github.com/user-attachments/assets/fb8a3e03-13b6-4dc1-b124-9925183d8fce" />
Попробуем пингануть с HQ-RTR BR-RTR по IP-адресу 10.10.0.2
<img width="877" height="275" alt="image" src="https://github.com/user-attachments/assets/58156b0d-53ab-4955-936f-556b37783eef" />

7.	Обеспечьте динамическую маршрутизацию на маршрутизаторах HQ-RTR и BR-RTR: сети одного офиса должны быть доступны из другого офиса и наоборот. Для обеспечения динамической маршрутизации используйте link state протокол на усмотрение участника:
•	Разрешите выбранный протокол только на интерфейсах ip туннеля
•	Маршрутизаторы должны делиться маршрутами только друг с другом
•	Обеспечьте защиту выбранного протокола посредством парольной защиты
•	Сведения о настройке и защите протокола занесите в отчёт.
Устанавливаем FRR на HQ-RTR и BR-RTR
<img width="555" height="35" alt="image" src="https://github.com/user-attachments/assets/1b7a59c8-02dd-4235-83d4-72cde14bb1b4" />
<img width="555" height="39" alt="image" src="https://github.com/user-attachments/assets/502e1f9e-8ead-4eca-b6a0-385d9dcefb76" />
Сначала настроим HQ-RTR
Зайдем в конфигурационный файл модулей FRR
<img width="709" height="47" alt="image" src="https://github.com/user-attachments/assets/e8e334dc-2807-4f7a-91d4-31cd573ecd87" />
Ищем строчку ospfd=no и меняем на ospfd=yes
<img width="460" height="591" alt="image" src="https://github.com/user-attachments/assets/a44567d0-ed10-432a-a5b8-06de0df223f7" />
Перезапускаем FRR
<img width="664" height="37" alt="image" src="https://github.com/user-attachments/assets/dbe63eaf-c987-4ca8-b526-bc7ba1461ebe" />
Входим в эмуляцию интерфейса FRR (аналог Cisco)
<img width="424" height="41" alt="image" src="https://github.com/user-attachments/assets/487b7267-12fe-436e-b26c-4dc09a171fa6" />
Входим в режим конфигурации терминала
<img width="519" height="47" alt="image" src="https://github.com/user-attachments/assets/e14c3a79-1cee-4892-8587-82ef95273046" />
Входим в режим конфигурации OSPF
<img width="736" height="44" alt="image" src="https://github.com/user-attachments/assets/27969d45-7bdd-415c-b125-7ced80f72a31" />
Задаем ID для маршрутизатора (1.1.1.1)
<img width="968" height="44" alt="image" src="https://github.com/user-attachments/assets/8959eb93-28e4-46a6-9357-4d7de3d50cf1" />
<img width="977" height="38" alt="image" src="https://github.com/user-attachments/assets/c35c7b18-3835-4bd5-8968-d9cfa10377b1" />
Объявляем локальные сети офиса HQ (сеть VLAN100 и VLAN200) и сеть
GRE-туннеля
<img width="972" height="83" alt="image" src="https://github.com/user-attachments/assets/ecadc619-b32c-46ea-9f27-6e4cbe143987" />
Настройка аутентификации для области
<img width="965" height="43" alt="image" src="https://github.com/user-attachments/assets/02ab31cf-a63f-44a5-9d16-ee8577c33898" />
Переходим к конфигурированию интерфейса tun1
<img width="795" height="39" alt="image" src="https://github.com/user-attachments/assets/994e7a0f-5826-450b-ac9a-144763e68d94" />
Выводим интерфейс из пассивного режима
<img width="905" height="34" alt="image" src="https://github.com/user-attachments/assets/522d1736-9ffe-449d-ba6d-e10513376278" />
Туннельный интерфейс tun1 делаем активным, для установления соседства с
BR-RTR и обмена внутренними маршрутами
<img width="975" height="40" alt="image" src="https://github.com/user-attachments/assets/506d7a61-b5e7-4315-a855-067001965020" />
Настройка аутентификации с открытым паролем password
<img width="967" height="57" alt="image" src="https://github.com/user-attachments/assets/490eb586-b4ca-4566-ba2c-2fcebfe38739" />
Выходим и записываем изменения
<img width="944" height="236" alt="image" src="https://github.com/user-attachments/assets/297b6b77-8c59-4464-ae2b-bb38b69039aa" />
Настроим BR-RTR
Зайдем в конфигурационный файл модулей FRR
<img width="649" height="47" alt="image" src="https://github.com/user-attachments/assets/601e735f-1a6a-4b53-b7e7-65912f740347" />
Ищем строчку ospfd=no и меняем на ospfd=yes
<img width="680" height="44" alt="image" src="https://github.com/user-attachments/assets/0525faa6-d111-44a1-82b0-47f599278da4" />
Перезапускаем FRR
<img width="806" height="771" alt="image" src="https://github.com/user-attachments/assets/722ceaa5-b8be-4ce8-bfd5-c58242c128f1" />
Входим в эмуляцию интерфейса FRR (аналог Cisco)
<img width="385" height="36" alt="image" src="https://github.com/user-attachments/assets/f2cf0116-d50f-48fb-b7a3-df0ae7afcaf3" />
Входим в режим конфигурации терминала
<img width="516" height="55" alt="image" src="https://github.com/user-attachments/assets/980882e9-c231-4b5c-90f0-cdc30ef0d2c5" />
Входим в режим конфигурации OSPF
<img width="744" height="41" alt="image" src="https://github.com/user-attachments/assets/9e003cb5-1944-4652-8a35-b2f1b7fd36e7" />
Задаем ID для маршрутизатора (2.2.2.2)
<img width="975" height="39" alt="image" src="https://github.com/user-attachments/assets/7cfbbbb8-607e-4259-9b32-dd35126b3ca5" />
<img width="973" height="45" alt="image" src="https://github.com/user-attachments/assets/d15216de-87ea-4060-a863-19c2b6a4250d" />
Объявляем локальную сеть офиса BR и сеть GRE-туннеля
<img width="970" height="56" alt="image" src="https://github.com/user-attachments/assets/ead7a690-7dc6-4d4b-b55f-2e66ae7b30a4" />
Настройка аутентификации для области
<img width="966" height="43" alt="image" src="https://github.com/user-attachments/assets/77840ce2-c694-4ca8-9a5f-41f16336fb9f" />
Переходим к конфигурированию интерфейса tun1
<img width="805" height="36" alt="image" src="https://github.com/user-attachments/assets/ef8c5695-11eb-44c0-92ae-c88d1f6933cd" />
Выводим интерфейс из пассивного режима
<img width="911" height="38" alt="image" src="https://github.com/user-attachments/assets/14df734f-ed84-4a65-b861-d8858341c9cd" />
Туннельный интерфейс tun1 делаем активным, для установления соседства с
HQ-RTR и обмена внутренними маршрутами
<img width="968" height="40" alt="image" src="https://github.com/user-attachments/assets/34f9935e-5813-4014-a0c9-45f1e3b62466" />
Настройка аутентификации с открытым паролем password
<img width="953" height="62" alt="image" src="https://github.com/user-attachments/assets/f3edccda-7a2f-4b6f-8a6e-8fa7b410e1b2" />
Выходим и записываем изменения
<img width="975" height="238" alt="image" src="https://github.com/user-attachments/assets/04622e72-8e3f-4216-80e0-8cce27a6afbb" />
После этого перезагружаем HQ-RTR и BR-RTR
<img width="422" height="51" alt="image" src="https://github.com/user-attachments/assets/b0122746-d8df-48dd-9b11-6d86cdd2233b" />
<img width="406" height="53" alt="image" src="https://github.com/user-attachments/assets/714a08dd-ea3e-4436-bd22-82d4fba1f3af" />
Входим в эмуляцию интерфейса FRR
<img width="392" height="47" alt="image" src="https://github.com/user-attachments/assets/d5ddd5d8-543e-4761-a662-8a50293d443d" />
<img width="392" height="63" alt="image" src="https://github.com/user-attachments/assets/b02953d6-4b86-4e77-ab0b-69810d66b773" />
Проверяем соседство маршрутизаторов
<img width="978" height="159" alt="image" src="https://github.com/user-attachments/assets/5b054ada-b3df-49e5-8215-0c6bc0eeff34" />
<img width="977" height="171" alt="image" src="https://github.com/user-attachments/assets/b253557c-6ba6-4382-bbbc-1478f355cabc" />
Все работает, можно с HQ-SRV пингануть BR-SRV
ЕСЛИ СЕРВЕРА НЕ ВИДЯТ ДРУГ ДРУГА, ТО ПРОПИСЫВАЕМ НА HQ-RTR И BR-RTR systemctl restart frr
<img width="763" height="250" alt="image" src="https://github.com/user-attachments/assets/efffa0b4-6f6b-456a-a486-8aacc8360236" />

9.	Настройте протокол динамической конфигурации хостов для сети в сторону HQ-CLI:
•	Настройте нужную подсеть
•	В качестве сервера DHCP выступает маршрутизатор HQ-RTR
•	Клиентом является машина HQ-CLI
•	Исключите из выдачи адрес маршрутизатора
•	Адрес шлюза по умолчанию – адрес маршрутизатора HQ-RTR
•	Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV
•	DNS-суффикс – au-team.irpo
•	Сведения о настройке протокола занесите в отчёт.
Установим DHCP-сервер на HQ-RTR
<img width="778" height="36" alt="image" src="https://github.com/user-attachments/assets/dbdd97e8-eaef-4eff-8797-eb0072ae0d68" />
Зайдем в настройки интерфейсов DHCP
<img width="863" height="45" alt="image" src="https://github.com/user-attachments/assets/6022b345-73d1-4865-9b9b-a5daa18c8cec" />
Вписываем в INTERFACESv4 значение vlan200 (это HQ-CLI)
<img width="494" height="232" alt="image" src="https://github.com/user-attachments/assets/c50a0784-ab99-4090-996e-d7e0344c209a" />
Настроим сам DHCP
<img width="681" height="38" alt="image" src="https://github.com/user-attachments/assets/14ec6d88-1ff7-44e4-bba9-db913ce37f7d" />
<img width="975" height="647" alt="image" src="https://github.com/user-attachments/assets/e8bd8e39-4da1-4ce8-9cd8-0f8ac16644de" />
Нужно изменить эти две строчки
<img width="970" height="96" alt="image" src="https://github.com/user-attachments/assets/b4b8c521-861a-4723-92c8-9225780fa346" />
На
<img width="917" height="128" alt="image" src="https://github.com/user-attachments/assets/790cdbe4-f80b-4950-a978-4ecfe9942e04" />
Перемещаемся к концу файла
<img width="911" height="342" alt="image" src="https://github.com/user-attachments/assets/31e2ea3d-8e00-4230-9ed2-67f0b415fab4" />
Добавляем в конец настройку DHCP
<img width="975" height="703" alt="image" src="https://github.com/user-attachments/assets/504a3499-fb6e-4f79-93e2-f81b3b913240" />
Тут 2 пробела перед range и option
•	subnet - обозначает сеть, в области которой будет работать данная группа настроек;
•	range — диапазон, из которого будут браться IP-адреса;
•	option domain-name-servers — через запятую перечисляем DNS-сервера.
•	option domain-name — суффикс доменного имени
•	option routers — шлюз по умолчанию;
•	default-lease-time, max-lease-time — время и максимальное время в секундах, на которое клиент получит адрес, по его истечению будет выполнено продление срока
То есть, по сути наш DHCP будет работать в области 192.168.100.32 с маской 255.255.255.240 (/28), выдавать диапазоны адресов от 192.168.100.34 до 192.168.100.47 от нашего шлюза VLAN200 (192.168.100.33).
Перезагружаем службу DHCP
<img width="875" height="41" alt="image" src="https://github.com/user-attachments/assets/4c694f67-3157-4b0d-bddf-b42b78835e72" />
Теперь на HQ-CLI изменим файл /etc/network/interfaces
<img width="797" height="36" alt="image" src="https://github.com/user-attachments/assets/e237b685-e8fa-4cfe-9f7c-2e9f9e05fe95" />
Меняем настройку со статики
<img width="438" height="423" alt="image" src="https://github.com/user-attachments/assets/c306ba56-ef89-49ef-ab14-8261a085756f" />
На DHCP
<img width="487" height="365" alt="image" src="https://github.com/user-attachments/assets/be64385e-767c-4283-bb02-a750da42ca8a" />
P.S. В nmtui рекомендую удалить Wired connection 1, иначе интерфейс получит два IP-адреса
<img width="974" height="559" alt="image" src="https://github.com/user-attachments/assets/297e3a90-f32b-4373-b663-b2b3b35a87df" />
<img width="975" height="96" alt="image" src="https://github.com/user-attachments/assets/350760e8-bd29-4e90-a6e2-b73be3365dd0" />
Перезапускаем службу сети
<img width="814" height="44" alt="image" src="https://github.com/user-attachments/assets/24d2d012-a880-4dab-818d-d8556f32fce4" />
Проверяем выдачу IP-адреса
<img width="881" height="105" alt="image" src="https://github.com/user-attachments/assets/1847608d-83f0-441d-9e5e-3f2bb93a635f" />
Может	выдаться	и	192.168.100.35	(это	если	забыть	удалить	из	nmtui
подключение), но ничего страшного
<img width="866" height="102" alt="image" src="https://github.com/user-attachments/assets/7963c386-b4a6-450a-9f93-5a8f979037ff" />

10.	Настройте инфраструктуру разрешения доменных имён для офисов HQ и BR:
•	Основной DNS-сервер реализован на HQ-SRV
•	Сервер	должен	обеспечивать	разрешение	имён	в	сетевые	адреса устройств и обратно в соответствии с таблицей 3
•	В качестве DNS сервера пересылки используйте любой общедоступный DNS сервер(77.88.8.7, 77.88.8.3 или другие)
МОЖЕТ ВЫЛЕЗТИ ОШИБКА ПРИ СКАЧИВАНИИ dnsmasq, ЕСЛИ ЭТО ПРОИЗОЙДЕТ, ПИШЕМ sudo apt-get update
Заходим на HQ-SRV, скачиваем DNS-сервер
1. apt install dnsmasq -y
2. nano /etc/dnsmasq.conf
3. Конфигурируем как на скриншоте, в начале или конце файла
<img width="1006" height="757" alt="image" src="https://github.com/user-attachments/assets/009df4dc-d6db-4e40-8101-92fa4e5abbe0" />
4. nano /etc/resolv.conf
5. В resolv записываем nameserver 127.0.0.1
6. Проверяем интернет с разрешением DNS ping ya.ru
7. Проверяем ping br-rtr.au-team.irpo и прочие хосты
Если пингуется, значит DNS преобразуется в IP и все работает

11.	Настройте часовой пояс на всех устройствах (за исключением виртуального коммутатора, в случае его использования) согласно месту проведения экзамена
Проверяем часовой пояс.
<img width="975" height="271" alt="image" src="https://github.com/user-attachments/assets/7f023772-bfc3-4080-90d5-9eb8232498cd" />
Список доступных часовых поясов можно посмотреть командой.
<img width="977" height="153" alt="image" src="https://github.com/user-attachments/assets/1a06d966-7860-46b0-b891-2b1ae86eb8ac" />
Посмотреть список регионов и городов.
<img width="975" height="394" alt="image" src="https://github.com/user-attachments/assets/f777b200-daa3-4f63-ad2d-c02e5ef9f0a7" />
Выберем Красноярск.
<img width="955" height="53" alt="image" src="https://github.com/user-attachments/assets/202d03e9-351e-401b-8e84-7aa06256c934" />
Изменение даты и времени при необходимости.
<img width="937" height="42" alt="image" src="https://github.com/user-attachments/assets/3c7cb7e1-aabb-4571-9878-6d4d22cc50be" />
Проверяем изменения.
<img width="944" height="240" alt="image" src="https://github.com/user-attachments/assets/fde8ecfd-828b-46c3-9e7b-a66cf5e2bdb9" />
Повторяем задание 11 на всех устройствах – ISP, HQ-RTR, BR-RTR, HQ-SRV, HQ-CLI, BR-SRV.
