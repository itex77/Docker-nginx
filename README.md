# Docker-nginx
Deploy nginx for monitoring in zabbix

Установка Docker
Рассмотрим примеры установки на базе операционных систем Red Hat/CentOS и Debian/Ubuntu.

Red Hat/CentOS
Устанавливаем репозиторий — для этого загружаем файл с настройками репозитория:

wget https://download.docker.com/linux/centos/docker-ce.repo

* если система вернет ошибку, устанавливаем wget командой yum install wget.

... и переносим его в каталог yum.repos.d:

mv docker-ce.repo /etc/yum.repos.d/

Устанавливаем docker:

yum install docker-ce docker-ce-cli containerd.io

Если система вернет ошибку Необходимо: container-selinux >= ..., переходим на страницу пакетов CentOS, находим нужную версию container-selinux и копируем на него ссылку:

Копирование ссылки на нужную версию container-selinux

... с помощью данной ссылки выполняем установку:

yum install http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.99-1.el7_6.noarch.rpm

После повторяем команду на установку докера:

yum install docker-ce docker-ce-cli containerd.io

Debian/Ubuntu
В deb-системе ставится командой:

apt-get install docker docker.io

После установки
Разрешаем запуск сервиса docker:

systemctl enable docker

... и запускаем его:

systemctl start docker

После проверяем:

docker run hello-world

... мы должны увидеть:

...
Hello from Docker!
This message shows that your installation appears to be working correctly.
...

Сборка нового образа
Сборка начинается с создания файла Dockerfile — он содержит инструкции того, что должно быть в контейнере. В качестве примера, соберем свой веб-сервер nginx.

И так, чтобы создать свой образ с нуля, создаем каталог для размещения Dockerfile:

mkdir -p /opt/docker/mynginx

* где /opt/docker/mynginx — полный путь до каталога, где будем создавать образ.

... переходим в данный каталог:

cd /opt/docker/mynginx

... и создаем Dockerfile:

vim Dockerfile

FROM centos:7

MAINTAINER Akhunzyanov Ilfat <itex77@yandex.ru>

ENV TZ=Europe/Moscow

RUN yum install -y epel-release && yum install -y nginx
RUN yum clean all
RUN echo "daemon off;" >> /etc/nginx/nginx.conf
RUN sed -i "0,/nginx/s/nginx/docker-nginx/i" /usr/share/nginx/html/index.html

CMD [ "nginx" ]

* в данном файле мы:

используем базовый образ centos 7;
в качестве автора образа указываем свое фио;
задаем временную зону внутри контейнера Europe/Moscow.
устанавливаем epel-release и nginx;
чистим систему от метаданных и кэша пакетов после установки;
указываем nginx запускаться на переднем плане (daemon off); 
в индексном файле меняем первое вхождение nginx на docker-nginx;
запускаем nginx.
* подробное описание инструкций Dockerfile смотрите ниже.

Запускаем сборку:

docker build -t aim/nginx:v1 .

* где aim — имя автора; nginx — название для сборки; v1 — тег с указанием версии. Точка на конце указывает, что поиск Dockerfile выполняем в текущей директории.

... начнется процесс сборки образа — после его завершения мы должны увидеть что-то на подобие:

Successfully built eae801eaeff2
Successfully tagged aim/nginx:v1

Посмотреть список образов можно командой:

docker images

Создаем и запускаем контейнер из образа:

docker run -d -p 8080:80 aim/nginx:v1

* в данном примере мы запустим контейнер из образа dmosk/nginx:v1 и укажем, что необходимо опубликовать внешний порт 8080, который будет перенаправлять трафик на порт 80 внутри контейнера.

Открываем браузер и переходим по адресу http://<IP-адрес нашего докера>:8080 — мы должны увидеть страницу приветствия с нашим docker-nginx:

Страница приветствия с docker-nginx

Посмотреть созданные контейнеры можно командой:

docker ps -a

Запустить или остановить контейнеры можно командами:

docker stop 5fe78aca2e1d

docker start 5fe78aca2e1d

* где 5fe78aca2e1d — идентификатор контейнера.

Редактирование образа
В примере выше мы рассмотрели создание нового образа с нуля. Также, мы можем взять любой другой образ, отредактировать его и сохранить под своим названием.

Скачаем образ операционной системы CentOS:

docker pull centos:latest

Войдем в скачанный образ для его изменения:

docker run -t -i centos:latest /bin/bash

Внесем небольшие изменения, например, создадим учетную запись:

[root@8f07ef93918f /]# useradd dmosk -G wheel -m

[root@8f07ef93918f /]# passwd aim

[root@8f07ef93918f /]# exit

* в данном примере мы создали пользователя aim и задали ему пароль.

Коммитим образ:

docker commit -m "Add user aim" -a "Akhunzyanov Ilfat" 8f07ef93918f centos:my

* где -m — параметр для указания комментария; -a — указывает автора; 8f07ef93918f — идентификатор контейнера, который был нами изменен (его можно было увидеть в приглашении командной строки); centos:my — название нашего нового образа.

Новый образ создан.

Загрузка образа на Docker Hub
Заходим на Docker Hub страницу регистрации. Создаем пользователя:

Регистрация на Docker Hub

На следующей странице также заполняем данные профиля. После переходим в почтовый ящик, который был указан при регистрации и переходим по ссылке Confirm Your Email With Docker для подтверждения регистрации. Регистрация закончена.

Переходим на страницу Repositories и создаем свой репозиторий, например, dmosk. Теперь можно загрузить наши образы в репозиторий.

Сначала авторизуемся в Linux под нашим зарегистрированным пользователем:

docker login --username aim

Задаем тег для одного из образов и загружаем его в репозиторий:

docker tag centos:my aim/aim:centos

docker push aim/aim:centos

Загрузим второй образ:

docker tag aim/nginx:v1 aim/aim:nginx

docker push aim/aim:nginx

В Docker Hub должны появиться наш образы:



Чтобы воспользоваться образом на другом компьютере, также авторизуемся под зарегистрированным пользователем docker:

docker login --username aim

Загружаем образ:

docker pull d--useaim:nginx

Запускаем его:

docker run -d -p 8080:80 aim/aim:nginx

Описание инструкций Dockerfile
Инструкция	Описание	Пример
FROM	Указывает, какой базовый образ нужно использовать. Обязательная инструкция для Dockerfile	FROM ubuntu:16.04
MAINTAINER	Автор образа.	MAINTAINER aim <mail>
RUN	Выполняет команду в новом слое при построении образа.	RUN apt-get install python
CMD	Запускает команду каждый раз при запуске контейнера. Может быть вызвана только один раз. Если в Dockerfile указать несколько таких инструкций, то выполнена будет последняя.	CMD ["openvpn"]
LABEL	Добавляет метаданные.	LABEL version="2"
EXPOSE	Указывает, какой порт должно использовать приложение внутри контейнера.	EXPOSE 8080
ENV	Задает переменные окружения в образе.	ENV PGPASSWORD pass
ADD	Добавляет файлы/папки из текущего окружения в образ. Если в качестве копируемого файла указать архив, то он будет добавлен в образ в распакованном виде. Также в качестве источника принимает URL.	ADD /root/.ssh/{id_rsa,id_rsa.pub} /root/.ssh/
COPY	Также как и ADD добавляет файлы в образ, но обладает меньшими функциями — не принимает URL и не распаковывает архивы. Рекомендован для использования в случаях, где не требуются возможности ADD или когда нужно перенести архив, как архив.	COPY ./mypasswd /root/
ENTRYPOINT	Указывает команду, которой будет передаваться параметр при запуске контейнера.	ENTRYPOINT ["/sbin/apache2"]
VOLUME	Добавляет том в контейнер.	VOLUME ["/opt/myapp"]
USER	Задает пользователя, от которого будет запущен образ.	USER user:group
WORKDIR	Можно задать каталог, откуда будут запускаться команды ENTRYPOINT и CMD.	WORKDIR /opt/apps
ARG	Создает переменную, которую может использовать сборщик.	ARG folder=/opt/apps
WORKDIR $folder
ONBUILD	Действия, которые выполняются, если наш образ используется как базовый для другой сборки.	ONBUILD ADD . /app/src
STOPSIGNAL	Переопределяет сигнал SIGTERM для завершения контейнера.	STOPSIGNAL SIGINT
HEALTHCHECK	Команда, которая будет проверять работоспособность контейнера.	HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1
SHELL	Позволяет заменить стандартную оболочку для выполнения команд на пользовательскую.	SHELL ["/bin/sh", "-c"]
Импорт и экспорт образа
Наши образы docker мы можем экспортировать для переноса на другой сервер. Рассмотрим процесс подробнее.

Создание резерва (экспорт)
Созданный нами образ можно сохранить в виде архива и, при необходимости, перенести на другой сервер или оставить как бэкап.

И так, для создания резервной копии образа, смотрим их список:

docker images

... и для нужного выполняем команду:

docker save -o /backup/docker/image_name.tar <image_name>

* в данном примере мы создаем архив контейнера <container image> в файл /backup/docker/container.tar.

Чтобы уменьшить размер, занимаемый созданным файлом, заархивируем его командой:

gzip /backup/docker/container.tar

* в итоге, мы получим файл container.tar.gz. 

Или, чтобы сразу создать заархивированную копию:

docker save <image_name> | gzip > /backup/docker/image_name.tar.gz

Для создания архива всех образов в нашей системе выполняем:

docker save $(docker images -q) | gzip > /backup/docker/all_image.tar.gz

* в файле /backup/docker/all_image.tar.gz мы получим архив со всем образами, которые есть в нашей системе.

Восстановление
Сначала распаковываем архив:

gunzip image_name.tar.gz

После восстанавливаем образ:

docker load -i image_name.tar

Смотрим, что нужный нам образ появился:

docker images

Volume
1. Для экспорта вольюма вводим:

docker run --rm --volumes-from container-name -v $(pwd):/backup ubuntu bash -c "cd /volume-path && tar zcf /backup/backup.tar.gz ."

* где container-name — имя контейнера, к которому привязан нужный нам вольюм; volume-path — путь монтирования вольюма внутри контейнера.

После ввода команды docker запустит контейнер с Ubuntu и создаст архив backup.tar.gz. Мы его должны увидеть в текущем каталоге:

ls -l

Данный файл можно переносить на другой сервер.

2. Для восстановления на целевом сервере переходим в каталог, где лежит наш файл backup.tar.gz и вводим:

docker run --rm --volumes-from container-name -v $(pwd):/backup ubuntu bash -c "cd /volume-path && tar zxf /backup/backup.tar.gz"

* где container-name — имя контейнера, к которому будет привязан вольюм; volume-path — путь монтирования вольюма внутри контейнера.

Дополнительные команды
В данном подразделе приведем примеры команд, которые могут оказаться полезными при работе с образами.

1. Удалить один образ:

docker rmi <название образа или его ID>

Например:

docker rmi dmosk/nginx:v1

2. Удалить все образы:

docker rmi $(docker images -q)

* опция -q возвращает только идентификаторы запрошенных объектов.

Мы можем получить ошибки на подобие:

Error response from daemon: conflict: unable to delete 857594f280c1 (must be forced) - image is being used by stopped container ...

Это значит, что для удаляемого образа есть действующие контейнеры — они могут быть как включены, так и находится в отключенном состоянии. Удалить все нерабочие контейнеры можно командой:

docker rm $(docker ps --filter status=exited -q)

Если нужно, можно остановить все действующие контейнеры командой:

docker stop $(docker ps -a -q)

Или мы можем остановить только те контейнеры, имя которых начинается на web:

docker stop $(docker ps -q --filter name=^web)

* в качестве фильтра указываются паттерны. Можно использовать ^ для определения начала строки и $ — конца.

Также мы можем принудительно удалить все образы, даже если они используются для контейнеров в данный момент:

docker rmi -f <название образа или его ID>

* добавлена опция -f. Обратите внимание, что даная опция является коротким вариантом опции --filter, однако, короткая форма не работает в конструкции с $(docker ...).

3. Для выявления проблем при запуске или в работе контейнера очень полезна опция для просмотра логов:

docker logs <имя или идентификатор>
