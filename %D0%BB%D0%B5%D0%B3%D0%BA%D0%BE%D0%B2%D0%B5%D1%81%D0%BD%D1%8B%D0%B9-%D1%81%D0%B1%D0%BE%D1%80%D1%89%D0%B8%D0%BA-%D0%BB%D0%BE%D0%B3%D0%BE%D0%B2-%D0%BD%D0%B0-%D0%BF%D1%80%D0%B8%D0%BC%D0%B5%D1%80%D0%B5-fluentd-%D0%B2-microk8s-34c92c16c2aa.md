
# Легковесный сборщик логов на примере FluentD в MicroK8s



Занимаясь проектом в домашнем кластере построенным на MicroK8s я столкнулся с проблемой падения приложения, если сборщик логов не доступен в данный момент.

Используя классическую связку [Elasticsearch](https://www.elastic.co/), [Logstash](https://www.elastic.co/logstash), [Kibana](https://www.elastic.co/kibana), приложение поставляло логи по TCP на порт 12201 в Logstash, который был развернут в другом пространстве имен (*infrastructure*) и был урезан по ресурсами.

Каждый раз когда Logstash умирал, приложение было какое то время недоступно и переход на UDP не дал бы существенных улучшений. Логи ценны и выбрасывать их было нельзя.

Поэтому мой выбор пал на [FluentD](https://www.fluentd.org/). Это легковесный лог-брокер написанный на [Ruby](https://www.ruby-lang.org/), который может работать на 32 MB RAM и 0.1 CPU. Сталкиваться с теми же проблемы с TCP + GLEF не хотелось, поэтому принял решение читать логи из файлов.

*Если POD запущенный в kubernetes пишет свои логи в stdout, то их можно найти в **/var/logs/containers**. Осталось дело за малым, просто считать файлы логов и отправить их в elasticsearch.*

Я буду использовать [WERF](https://werf.io/) для сборки и доставки образа и [Docker Hub ](https://hub.docker.com/)для хранения образов и GitHub Actions как runner для запуска всего pipeline.

В корне проекта создам werf.yml. Собирать буду на основе образа linux alpine. Чистый fluentD:v1.9.1–1.0 занимает в сжатом состоянии всего 15 MB. Я буду использовать elasticsearch + kibana от [logz.io](https://logz.io/).

*Плагин fluent-plugin-logzio нужен будет для отправки данных в logz.io.*

    project: fluentd
    configVersion: 1
    ---
    image: ~
    from: fluent/fluentd:v1.9.1-1.0
    docker:
      USER: root
    ansible:
      install:
        - name: Install plugins
          shell: gem install fluent-plugin-logzio
          become_user: root

В дальнейшем pod с FluentD будет читать логи с файловой системы **node** где он запущен для этого пользователь FluentD установлен **root**.

Теперь нужно создать простой chart для доставки кода

    # werf helm create NAME [flags] [options]

И в итоге получаем простую структуру helm chart.

    .helm/templates/_helpers.tpl
    .helm/templates/config.yaml
    .helm/templates/deployment.yaml
    .helm/values.yaml
    werf.yml

## Deployment

Выглядит совершенно обычно и я не буду добавлять возможность автоматического масштабирования PODов в этой статье. Тут нужно обратить внимание на секции *volumes*. Смонтирую системные каталоги, где *containers* это том, который содержит файлы логов контейнеров, а в */var/log* буду хранить файл с данным о текущей позиции курсора чтения файлов:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ include "fluentd.fullname" . }}
      labels:
        {{- include "fluentd.labels" . | nindent 4 }}
        kubernetes.io/cluster-service: "true"
    spec:
      selector:
        matchLabels:
          {{- include "fluentd.selectorLabels" . | nindent 6 }}
      template:
        metadata:
          labels:
            {{- include "fluentd.selectorLabels" . | nindent 8 }}
            kubernetes.io/cluster-service: "true"
        spec:
          volumes:
            - name: {{ include "fluentd.fullname" . }}-logs
              hostPath:
                path: /var/log
    
            - name: {{ include "fluentd.fullname" . }}-container-logs
              hostPath:
                path: /var/lib/docker/containers
    
            - name: {{ include "fluentd.fullname" . }}-config-volume
              configMap:
                name: {{ include "fluentd.fullname" . }}-config
                items:
                  - key: fluent.conf
                    path: fluent.conf
    
          tolerations:
            - key: node-role.kubernetes.io/master
              effect: NoSchedule
          containers:
            - name: {{ .Chart.Name }}
              {{ werf_container_image . | indent 0 }}
              imagePullPolicy: {{ .Values.image.pullPolicy }}
              volumeMounts:
                - name: {{ include "fluentd.fullname" . }}-logs
                  mountPath: /var/log
    
                - name: {{ include "fluentd.fullname" . }}-container-logs
                  mountPath: /var/lib/docker/containers
                  readOnly: true
    
                - mountPath: /fluentd/etc/fluent.conf
                  subPath: fluent.conf
                  readOnly: true
                  name: {{ include "fluentd.fullname" . }}-config-volume
              resources:
                {{- toYaml .Values.resources | nindent 12 }}

## Config map (.helm/templates/config.yaml)

На предыдущем шаге я указал, что хочу дать PODу возможность читать файлы логов, которые находятся на HOSTе где развернут MikroK8s.

Все мои проекты, которые пишут логи в stdout находятся в пространстве имен *apps *и искать файлы логов FluentD будет именно по этому паттерну.

*Важно, дать контейнеру возможность создавать файлы хранения позиции курсора, если POD умрет и файла курсора не будет, то лог будет перечитан с самого начала и будут созданы дубли.*

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: {{ include "fluentd.fullname" . }}-config
      labels:
        {{- include "fluentd.labels" . | nindent 4 }}
    data:
      fluent.conf: |
        <source>
          @type tail
          path /var/log/containers/*apps*.log
          pos_file /var/log/fluentd-containers.log.pos
          tag apps.*
          <parse>
            @type json
          </parse>
        </source>
    
        <match apps.**>
          @type logzio_buffered
          endpoint_url https://listener.logz.io:8071?token=&type=
          output_include_time true
          output_include_tags true
          http_idle_timeout 10
          <buffer>
              @type memory
              flush_thread_count 4
              flush_interval 3s
              chunk_limit_size 16m
              queue_limit_length 4096
          </buffer>
        </match>

Кратко про секции:
> Source — источники данных
> Match — output в хранилище

В дальнейшем придется немного корректировать данные, которые попадают в коллекторы к FluentD.

## GitHub Action

Кластер доступен из мира, резервировать отдельный *self-hosted runner* дорого и не хотелось бы.

Для начала заведем репозиторий на [github.com](https://github.com/). В проекте создадим файл **.github/workflows/main.yml**. В нем и опишем flow работы runner.

*Будет удобно если при каждом merge в master, FluentD автоматически бы доставлялся с обновленной конфигурацией в кластер. Сразу же нужно завести в настройках репозитория секреты, в которых будем хранить чувствительную информацию.*

### Secrets:

![](https://cdn-images-1.medium.com/max/3552/0*Ly7COHB_gInmUEeA.png)

![](https://cdn-images-1.medium.com/max/3616/1*a5fn1YG6jEPji-rQNKxc6g.png)
> KUBE_CONFIG_BASE64_DATE _STAGE— ~/.kube/config | base64
> REGISTRY_PASSWORD — пароль к нашему docker registry
> REGISTRY_USER — пользователь docker registry
> REGISTRY_REPOSITORY — имя репозитория

    name: Stage
    
    on:
      push:
        branches:
          - master
    
    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout code
            uses: actions/checkout@v2
    
          - name: Set tag
            id: vars
            run: |
              echo ::set-output name=tag::$(echo $GITHUB_REF | cut -d'/' -f 3)-$(echo $GITHUB_SHA | cut -c1-8)
    
          - name: Registry login
            run: echo $PASSWORD | docker login -u $USER --password-stdin
            env:
              USER: ${{ secrets.REGISTRY_USER }}
              PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
    
          - name: Сonverge
            uses: werf/actions/converge@master
            env:
              WERF_STAGES_STORAGE: ":local"
              WERF_TAG_BY_STAGES_SIGNATURE: false
              WERF_IMAGES_REPO: ${{ secrets.REGISTRY_REPOSITORY }}
              WERF_TAG_CUSTOM1: ${{ steps.vars.outputs.tag }}
              WERF_NAMESPACE: infrastructure
            with:
              kube-config-base64-data: ${{ secrets.KUBE_CONFIG_BASE64_DATA_STAGE }}
              env: stage

### Шаги workflow:

1. Runner клонирует код;

2. Назначаем тег будущего артефакта;

3. Осуществляем вход в наш docker registry;

4. Converge (build/push/publish);

Шаг “Converge” вызывает уже заранее подготовленный метод WERF, который делает сборку артефакта, push в docker registry, deploy helm chart в kubernetes cluster.
> Теперь при следующем merge в master актуальный FluentD будет развернут в кластере автоматически.
> Про переменные окружения WERF можно прочитать в официальной документации

## **Первые проблемы с разбором текста: JSON не совсем JSON.**

Когда Kubernetes захватывает stdout PODа, он добавляет к нему свою metadata, а FluentD ожидает чистый json в файлах:

    2020–10–27 16:48:32 +0000 [warn]: #0 pattern not matched: "2020–10–27T19:48:31.859783645+03:00 stdout F {\"trace_id\": \"3a86554a4d41197e829adef23d431301\"}

Менять log_driver в K8S не подходящее решение, но всегда можно подправить фильтрацию данных в настройках FluentD.

Во время чтения строки разобьем ее на компоненты: *timestamp*, *system*, *log*. Stdout контейнера находится в ключе **log **и json лог обернут и представляет собой строку, а не объект.

Применю фильтр, что бы преобразовать json-string в объект и поднять на верхний уровень все его значения.

    <source>
        @type tail
        path /var/log/containers/*.log
        pos_file /var/log/fluentd-containers.log.pos
        time_format %Y-%m-%dT%H:%M:%S.%NZ
        tag apps.*
        read_from_head true
        <parse>
            @type regexp
            expression /^(?<timestamp>.*?)\s(?<system>.*?)\s(?<log>\{.*\})$/
            time_key timestamp
        </parse>
    </source>

    <filter apps.**>
        @type parser
        key_name log
        reserve_data true
        remove_key_name_field true
        replace_invalid_sequence true
        reserve_time true
        <parse>
            @type json
            json_parser json
        </parse>
    </filter>

Результат:

    {
       "timestamp":"2020–10–28T12:25:43.787975275+03:00",
       "system":"stdout F",
       "trace_id":"bd4ba83bd42e9b4b036544db42fb33fc",
       "remote_addr":"194.61.2.200",
       "remote_user":"",
       "time_local":"28/Oct/2020:09:25:43 +0000",
       "request":"GET /health HTTP/1.1",
       "status":"200",
       "body_bytes_sent":"93",
       "http_referer":"",
       "http_user_agent":"kube-probe/1.19+"
    }

Итоговый файл конфигурации

    <match fluent.**>
        @type null
    </match>
    
    <match **fluentd**.log>
        @type null
    </match>
    
    <match **kube-system**.log>
        @type null
    </match>
    
    <source>
        @type tail
        path /var/log/containers/*.log
        pos_file /var/log/fluentd-containers.log.pos
        time_format %Y-%m-%dT%H:%M:%S.%NZ
        tag apps.*
        read_from_head true
        <parse>
            @type regexp
            expression /^(?<timestamp>.*?)\s(?<system>.*?)\s(?<log>\{.*\})$/
            time_key timestamp
        </parse>
    </source>
    
    <filter apps.**>
        @type parser
        key_name log
        reserve_data true
        remove_key_name_field true
        replace_invalid_sequence true
        reserve_time true
        <parse>
            @type json
            json_parser json
        </parse>
    </filter>
    
    <match apps.**>
      @type logzio_buffered
      endpoint_url https://listener.logz.io:8071?token=&type=
      output_include_time true
      output_include_tags true
      http_idle_timeout 10
      <buffer>
          @type memory
          flush_thread_count 4
          flush_interval 3s
          chunk_limit_size 16m
          queue_limit_length 4096
      </buffer>
    </match>

В итоге получаем корректно разобранные логи.

![](https://cdn-images-1.medium.com/max/2628/1*-JG8Nv0fUI926n4-_NQI8g.png)

### Итог:

1. Ускорилась работа приложения. Больше не тратится время на установку. соединения с коллектором.

1. Если коллектор перезагружается, то это никак не влияет на работу приложения.

1. Потребление ресурсов сократилось.

1. Если logz.io будет не доступен, то FluentD будет складывать логи в RAM, а если памяти выделенной на POD не останется, то FluentD просто перестанет читать логи до высвобождения оперативной памяти, после чего просто продолжит читать с места где остановился.
