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

Редактируем конфигурацию





