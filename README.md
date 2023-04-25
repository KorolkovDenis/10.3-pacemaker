# Домашнее задание к занятию 10.3 «Pacemaker»


### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c Шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и вашу фамилию и имя
   - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
   - для корректного добавления скриншотов воспользуйтесь инструкцией ["Как вставить скриншот в шаблон с решением"](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
   - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.

Желаем успехов в выполнении домашнего задания!

---

### Задание 1

Опишите основные функции и назначение Pacemaker.

*Приведите ответ в свободной форме.*

Ответ:

Pacemaker — менеджер ресурсов кластера. Он позволяет использовать службы и объекты в рамках одного кластера из двух или более нод.
Основные функции:
- позволяет находить и устранять сбои на уровне нод и служб;
- не зависит от подсистемы хранения: можем забыть общий накопитель, как страшный сон;
- не зависит от типов ресурсов: все, что можно прописать в скрипты, можно кластеризовать;
- поддерживает STONITH (Shoot-The-Other-Node-In-The-Head), то есть умершая нода изолируется и запросы к ней не поступают, пока нода не отправит сообщение о том, что она снова в рабочем состоянии;
- поддерживает кворумные и ресурсозависимые кластеры любого размера;
- поддерживает практически любую избыточную конфигурацию;
- может автоматически реплицировать конфиг на все узлы кластера — не придется править все вручную;
-  можно задать порядок запуска ресурсов, а также их совместимость на одном узле;
-  поддерживает расширенные типы ресурсов: клоны (когда ресурс запущен на множестве узлов) и дополнительные состояния (master/slave и подобное) — актуально для СУБД (MySQL, MariaDB, PostgreSQL, Oracle);
-  имеет единую кластерную оболочку CRM с поддержкой скриптов.


---

### Задание 2

Опишите основные функции и назначение Corosync.

*Приведите ответ в свободной форме.*

Ответ:

Corosync — программный продукт, который позволяет создавать единый кластер из нескольких аппаратных или виртуальных серверов. Corosync отслеживает и передает состояние всех участников (нод) в кластере.
В основе работы заложены следующие функции:
- мониторить статус приложений;
- оповещать приложения о смене активной ноды в кластере;
- отправлять идентичные сообщения процессам на всех нодах;
- предоставлять доступ к общей базе данных с конфигурацией и статистикой;
- отправлять уведомления об изменениях, произведенных в базе.
Идея заключается в том, что с помощью Corosync мы построим кластер, а следить за его состоянием будем с помощью Pacemaker.


---

### Задание 3

Соберите модель, состоящую из двух виртуальных машин. Установите Pacemaker, Corosync, Pcs. Настройте HA кластер.

*Пришлите скриншот рабочей конфигурации и состояния сервиса для каждого нода.*

Ответ:

Последовательность выполнения:

Создаю две ноды: node-1 и node-2

Добавляю в /etc/hosts/ запись:

192.168.43.181  node-1.dv.local node-1
192.168.43.199  node-2.dv.local node-2

Далее задам нужные hostname: node-1, node-2.

systemctl restart systemd-logind.service
exec bash
 
Далее установлю на двух нодах: nginx, в качестве веб-сервера, Pacemaker и Corosync.

apt install nginx
apt install corosync pacemaker pcs

Активирую и запускаю nginx:

systemctl enable nginx && systemctl start nginx

![screen1](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen1.jpg)

Теперь на каждой ноде отдельно, делаем страницу nginx по умолчанию:

Node-1:

echo "This is the default page for node-1.dv.local" | tee /var/www/html/index.nginx-debian.html

![screen2](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen2.jpg)

Node-2:

echo "This is the default page for node-2.dv.local" | tee /var/www/html/index.nginx-debian.html

![screen3](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen3.jpg)

Активирую и запускаю Pacemaker:

systemctl enable pcsd
systemctl start pcsd

![screen4](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen4.jpg)

При установке pacemaker у нас создался пользователь hacluster, зададим ему пароль:

cat /etc/passwd | grep hacluster

![screen5](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen5.jpg)

passwd hacluster

![screen6](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen6.jpg)

Развертка кластера

Перехожу к node-1 и работаю с ней:

pcs host auth node-1.dv.local node-2.dv.local

![screen7](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen7.jpg)

Следующим шагом запускаем ноды:

Но предварительно проверьте включена ли служба pcsd на
обоих кластерах:

service --status-all

![screen8](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen8.jpg)

pcs cluster setup mycluster1 node-1.dv.local node-2.dv.local --force
pcs cluster enable –all

![screen9](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen9.jpg)

pcs cluster start –all

![screen10](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen10.jpg)

pcs cluster status

![screen11](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen11.jpg)

pcs status

![screen12](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen12.jpg)

Отключаем вторичные свойства/параметры:

pcs property set stonith-enabled=false	
pcs property set no-quorum-policy=ignore

Добавлению запуск служб при старте системы на главном сервере (кластере), если вдруг не включены:

systemctl enable pcsd
systemctl enable corosync
systemctl enable pacemaker

Создадим виртуальный ip для heartbeat:
pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=192.168.43.100 cidr_netmask=24 op monitor interval=60s

pcs status

![screen13](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen13.jpg)

Добавлю ресурс веб-сервера nginx:

pcs resource create nginx ocf:heartbeat:nginx configfile="/etc/nginx/nginx.conf" op monitor timeout="20s" interval="60s"

Вот только он у меня выдает ошибку:

root@node-1:~# pcs resource create nginx ocf:heartbeat:nginx configfile="/etc/nginx/nginx.conf" op monitor timeout="20s" interval="60s"
Error: Agent 'ocf:heartbeat:nginx' is not installed or does not provide valid metadata: crm_resource: Metadata query for ocf:heartbeat:nginx failed: No such device or address, Error performing operation: No such object, use --force to override

И если сделать через --force, то он добавляется как ресурс, но в статусе стоп:

![screen14](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen14.jpg)

Проверим, что при отключении node-1 ресурсы запускаются на node-2.

Выключаем первую ноду. Запрашиваем статус второй ноды, чтобы убедиться, что первая недоступна.

pcs status

![screen15](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen15.jpg)
![screen16](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen16.jpg)

Включил первую ноду:

![screen17](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen17.jpg)

Осталось запросить статус кластера и убедиться, что адрес присвоен первой ноде.

pcs resource show

![screen18](https://github.com/KorolkovDenis/10.3-pacemaker/blob/main/screenshots/screen18.jpg)


---

### Задания со звёздочкой*
Эти задания дополнительные. Выполнять их не обязательно. Это не повлияет на зачёт. Вы можете их выполнить, если хотите глубже разобраться в материале.
 
---

### Задание 4

Установите и настройте DRBD-сервис для настроенного кластера.

*Пришлите скриншот рабочей конфигурации и состояние сервиса для каждого нода.*
