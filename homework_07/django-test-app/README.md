## Установка Docker и запуск тестового django-приложения 

Туториал поустановке из официального репозитория https://www.digitalocean.com/community/tutorials/docker-ubuntu-18-04-1-ru

Сборка
```
snuffy@linux-learn:~/linux-learn/homework_07/django-test-app$ docker build . -t arkada/django_test_app
Sending build context to Docker daemon  183.8kB
Step 1/5 : FROM python:alpine
 ---> d5e5ad4a4fc0
Step 2/5 : COPY . /home/snuffy/linux-learn/homework_07/django-test-app
 ---> 3cbafc9b5dd5
Step 3/5 : WORKDIR /home/snuffy/linux-learn/homework_07/django-test-app
 ---> Running in 64d307933bc6
Removing intermediate container 64d307933bc6
 ---> 24f0f66cc8c0
Step 4/5 : RUN pip install -r /home/snuffy/linux-learn/homework_07/django-test-app/requirements.txt
 ---> Running in bc88ea11445e
Collecting Django==2.0.8
  Downloading Django-2.0.8-py3-none-any.whl (7.1 MB)
Collecting pytz==2018.5
  Downloading pytz-2018.5-py2.py3-none-any.whl (510 kB)
Installing collected packages: pytz, Django
Successfully installed Django-2.0.8 pytz-2018.5
Removing intermediate container bc88ea11445e
 ---> c10df69c38f1
Step 5/5 : CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
 ---> Running in d7dfa40ab982
Removing intermediate container d7dfa40ab982
 ---> 66eff20c5065
Successfully built 66eff20c5065
Successfully tagged arkada/django_test_app:latest
```
Запуск
```
snuffy@linux-learn:~/linux-learn/homework_07/django-test-app$ docker run -itp 8000:8000 arkada/django_test_app                                               Performing system checks...
System check identified no issues (0 silenced).
March 25, 2020 - 10:33:39
Django version 2.0.8, using settings 'demo.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```
Логинимся на https://hub.docker.com
```
snuffy@linux-learn:~/linux-learn/homework_07/django-test-app$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: arkada
Password:
WARNING! Your password will be stored unencrypted in /home/snuffy/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
```
Публикация
```
snuffy@linux-learn:~/linux-learn/homework_07/django-test-app$ docker push arkada/django_test_app
The push refers to repository [docker.io/arkada/django_test_app]
ca194c44d6fb: Pushed
88c2510c5d88: Pushed
b93bfa4af0a6: Pushed
63d7c2284384: Pushed
89785e89a1f2: Pushed
d87eb7d6daff: Pushed
beee9f30bc1f: Pushed
latest: digest: sha256:2d03b6a3048d826178c329c64da747d71ddf160d069eed96c957af72927b8005 size: 1789
```
