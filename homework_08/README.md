### Запуск тестового python (flask) приложения, используя docker+nginx+uWSGI (как на production)

В помощь:
Справочник на русском https://dker.ru/docs/docker-engine/engine-reference/dockerfile-reference/
uWSGI Quick Start https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html
https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04-ru

Подготовка файлов Docker-образа приложения:
Создадим точку входа нашего приложения wsgi.py
```
from flask-app import app

if __name__ == "__main__":
    app.run()
```
В файл requirements.txt добавим зависимость uWSGI=2.0.18
Dockerfile:
```
FROM python:alpine

COPY flask-app.py  /opt/flask-app.py
COPY wsgi.py /opt/wsgi.py
WORKDIR /opt/
COPY requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt
RUN chmod +x /opt/flask-app.py
RUN chmod +x /opt/wsgi.py
RUN rm -rf /tmp/requirements.txt

CMD ["/opt/flask-app.py"]
CMD uWSGI --workers 2 --bind unix:socket/flaskapp.sock wsgi:app
```
Собираем Docker image приложения
```
snuffy@linux-learn:/opt/flask_test_app$ docker build . -t flask_app
Sending build context to Docker daemon  6.144kB
Step 1/8 : FROM python:alpine
 ---> d5e5ad4a4fc0
Step 2/8 : COPY flask-app.py  /opt/flask-app.py
 ---> a3767e9f2219
Step 3/8 : WORKDIR /opt/
 ---> Running in 8b5cf0b40634
Removing intermediate container 8b5cf0b40634
 ---> 0d2e8c552994
Step 4/8 : COPY requirements.txt /tmp/requirements.txt
 ---> 33bd7b4c5d69
Step 5/8 : RUN pip install -r /tmp/requirements.txt
 ---> Running in 21953bcc0427
Collecting click==7.1.1
  Downloading click-7.1.1-py2.py3-none-any.whl (82 kB)
Collecting Flask==1.1.1
  Downloading Flask-1.1.1-py2.py3-none-any.whl (94 kB)
Collecting itsdangerous==1.1.0
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting Jinja2==2.11.1
  Downloading Jinja2-2.11.1-py2.py3-none-any.whl (126 kB)
Collecting MarkupSafe==1.1.1
  Downloading MarkupSafe-1.1.1.tar.gz (19 kB)
Collecting Werkzeug==1.0.0
  Downloading Werkzeug-1.0.0-py2.py3-none-any.whl (298 kB)
Building wheels for collected packages: MarkupSafe
  Building wheel for MarkupSafe (setup.py): started
  Building wheel for MarkupSafe (setup.py): finished with status 'done'
  Created wheel for MarkupSafe: filename=MarkupSafe-1.1.1-py3-none-any.whl size=12629 sha256=6c0ee58512fba87a10faeadc7dece71c97b62dc0f058aa59a0c04f35fe184903
  Stored in directory: /root/.cache/pip/wheels/0c/61/d6/4db4f4c28254856e82305fdb1f752ed7f8482e54c384d8cb0e
Successfully built MarkupSafe
Installing collected packages: click, MarkupSafe, Jinja2, itsdangerous, Werkzeug, Flask
Successfully installed Flask-1.1.1 Jinja2-2.11.1 MarkupSafe-1.1.1 Werkzeug-1.0.0 click-7.1.1 itsdangerous-1.1.0
Removing intermediate container 21953bcc0427
 ---> d2287a2f843a
Step 6/8 : RUN chmod +x /opt/flask-app.py
 ---> Running in 8ead868659f0
Removing intermediate container 8ead868659f0
 ---> 37587671b4a9
Step 7/8 : RUN rm -rf /tmp/requirements.txt
 ---> Running in 4f3c6b552ea1
Removing intermediate container 4f3c6b552ea1
 ---> 8e7c4ee0c639
Step 8/8 : CMD ["/opt/flask-app.py"]
 ---> Running in a767fe13c09e
Removing intermediate container a767fe13c09e
 ---> 8ce8115b79ad
Successfully built 8ce8115b79ad
Successfully tagged flask_app:latest
```
Создаем  создаем bridge сетевой интерфейс docker
```
snuffy@linux-learn:/opt/flask_test_app$ docker network create -d bridge flas_bridge
d527a19338b1a8f3cc828da7e15b93e605aab1ce92e2dd7991873566afb51956

snuffy@linux-learn:/opt/flask_test_app$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 08:00:27:b1:90:a7 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:9e:96:fd brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.34/24 brd 192.168.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 58038sec preferred_lft 58038sec
    inet6 fe80::76e8:6b55:1d65:8132/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:50:c9:be:d0 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:50ff:fec9:bed0/64 scope link
       valid_lft forever preferred_lft forever
26: vethc1f42fa@if25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether d2:83:a8:69:85:2b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::d083:a8ff:fe69:852b/64 scope link
       valid_lft forever preferred_lft forever
33: br-d527a19338b1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:5e:10:5d:e7 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-d527a19338b1
       valid_lft forever preferred_lft forever
```
Делаем конфиг nginx 
```
server {
    listen       80;
    listen       [::]:80;
    server_name ubuntu arkada.jumpingcrab.com;
	
    location / {
        include uwsgi_params;
        uwsgi_pass unix:/opt/flask_test_app/flaskapp.sock;
    }
}
```
и запускаем его в контейнере
```
docker run --network=flas_bridge --hostname nginx --name nginx_fa -ti -p 80:80 -v $(pwd)/nginx_fa.conf:/et/nginx/conf.d/nginx_fa.conf nginx-alpine:latest
```
flaskapp запускаем в контейнере
```
docker run -e FLASK_HOST=0.0.0.0 --network=flas_bridge --name flask_app --rm -td flask_app:latest
```

