# Развертывание и запуск приложения

Приложение `Prometheus exporter for Nimble SRT` может быть развернуто разными способами:

1. Из исходного кода.
2. С помощью `Docker`.
3. С помощью `Kubernetes` (основной способ).

Т.к. `Kubernetes` — основной способ развертывания приложения, рассмотрим этот вариант.

## Развертывание с помощью Kubernetes

Чтобы развернуть приложение с помощью `Kubernetes`:

1. Создайте манифест для Kubernetes (например `deployment.yaml`).
2. В раздел `spec -> containers` добавьте следующую конфигурацию:

```yaml
      - name: srt-exporter
        image: {{ $.Values.werf.image.exporter }}
        command:
        - /nimble_exporter
        - -auth_salt=590
        - -auth_hash=xxxx
        ports:
        - containerPort: 9017
          name: exporter
          protocol: TCP
        resources:
          {{ toYaml $.Values.resources.exporter |  nindent 10 }}
```

Здесь:

- `name` — имя контейнера или пода в Kubernetes.
В данном случае — `srt-exporter`.
- `image` — Docker-образа, который будет использоваться для создания контейнера.
В данном случае — `{{ $.Values.werf.image.exporter }}`: значение подставляется из конфигурационного файла `Helm`.
- `command` — список команд и аргументов, которые будут выполнены и заданы при запуске контейнера.
- `ports` — раздел, определяющий порты, которые будут открыты для контейнера:
    - `containerPort` — указывает порт, на котором будет слушать контейнер (в данном случае —`9017`).
    - `name` — имя порта (для удобства). В данном случае — `exporter`.
    - `protocol` — протокол, который будет использоваться. В данном случае `TCP`.
- resources — раздел определяет ресурсы, которые будут выделены для контейнера.
    - `{{ toYaml $.Values.resources.exporter | nindent 10 }}` — указывает на то, что ресурсы описаны в шаблоне `Helm`. Здесь значения ресурсов из файла конфигурации преобразуются в формат YAML и к срокам добавляются отступы (`10` пробелов) для правильного форматирования манифеста.

3. Подготовьте шаблоны Helm с именем Docker-образа и описанием ресурсов.
Пример шаблона Helm (`values.yaml`):

```yaml
werf:
  image:
    exporter: nimble_exporter:latest
resources:
  exporter:
    requests:
      cpu: "100m"
      memory: "128Mi"
    limits:
      cpu: "200m"
      memory: "256Mi"
```

4. Примените манифест. Для этого используйте команду:

```yaml
kubectl apply -f deployment.yaml
```

После запуска приложение будет слушать порт `9017`.
Настройте `Prometheus` и другие компоненты для сборов метрик с этого порта по протоколу `TCP`.
