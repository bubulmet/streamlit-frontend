# Docker-образ для запуска Streamlit-скриптов  

[DockerHub](https://hub.docker.com/r/bubulmet/streamlit-frontend)

## Зачем  

Вот собрали мы какое-то API, скажем. Завернули в docker. И нужно к нему сделать простенький визуальный клиент. И всё это в один *docker-compose* собрать. Для таких случаев и подходит этот образ.  

Здесь взят [python](https://hub.docker.com/_/python) (`3.7-slim`), установлен [Streamlit](https://streamlit.io/) и еще пара пакетов. В т.ч. [requests](https://requests.readthedocs.io/en/master/), что делает возможной удобную работу с API. Что еще - см. в [requirements.txt](./requirements.txt).

## Использование  

Предположим, имеется такая структура директорий:  
```
    my_project
    ├── streamlit_app
    │   ├── main.py
    │   └── ...
    └── ...
```

Тогда нужно просто сделать так:  
```
    docker run -p 8501:8501 -v ./streamlit_app:/app bubulmet/streamlit-frontend
```

Или в `docker-compose.yml`:  
```
    services:
      streamlit_app:
        image: bubulmet/streamlit-frontend
        volumes:
          - ./streamlit_app:/app
        ports:
          - 8501:8501
```

**Принципиальные моменты:**  
- ожидает, что основной Streamlit-скрипт будет называться `main.py`. Именно его запускает.  
- ищет `main.py` в директории `/app`, поэтому именно к ней  mount'им локальную директорию с проектом.  

## Полезные советы  

Если Streamlit-скрипту нужно передать какие-то параметры, можно это делать через переменные среды. Например, при локальной отладке (без докера) я хочу, чтобы скрипт обращался к API по адресу `localhost`; а когда будет собран *docker-compose*, нужно будет обращаться к сервису `my_api`. Тогда можно поступить так:  

1. в Streamlit-скрипте:  
```
    import os  
    API_HOST = os.getenv('STREAMLIT_API_HOST')  
```

2. при разработке (без docker) (в bash):  
```
    export STREAMLIT_API_HOST=localhost  
```

3. в `docker-compose.yml`:  
```
    services:  
      my_api:  
        ...  

      streamlit_app:  
        image: bubulmet/streamlit-frontend  
        volumes:  
          - ./streamlit_app:/app  
        ports:  
          - 8501:8501  
        environment:  
          - STREAMLIT_API_HOST=my_api   
```  

## На правах рекламы  

API здорово писать на [FastAPI](https://fastapi.tiangolo.com/), и потом полное приложение завернуть в [Traefik](https://traefik.io/traefik/). И в итоге у нас есть полноценное веб-приложение.

## push'им на DockerHub

1. `docker build --tag streamlit-frontend .`  
2. `docker tag streamlit-frontend bubulmet/streamlit-frontend`  
3. `docker logout`  
3. `docker login`  
4. `docker push bubulmet/streamlit-frontend`  

А там можно подключить автобилд! Так что более вручную это делать не требуется: после коммита в ветку main на GitHub, на DockerHub автоматом новая версия сбилдится.
