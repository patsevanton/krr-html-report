# Kubernetes Resource Recommender (KRR)

## Введение

KRR (Kubernetes Resource Recommender) — это CLI-инструмент для оптимизации использования ресурсов в Kubernetes. Он анализирует метрики Pod'ов, собираемые в Prometheus, и предлагает оптимальные настройки `requests` и `limits` для CPU и памяти. Это помогает снизить расходы на облачные ресурсы и повысить производительность приложений.

## Как работает KRR

KRR получает данные из Prometheus и рассчитывает оптимальные лимиты на основе истории использования ресурсов. Основные алгоритмы:
- **CPU request** устанавливается на 95-й перцентиль.
- **CPU limit** может быть отключен или равен request.
- **Память** рассчитывается по максимальному использованию + буфер.

## Развертывание KRR в Kubernetes

Для работы KRR в кластере необходимо создать:
- **Namespace** (`ns.yaml`)
- **ServiceAccount** (`sa.yaml`)
- **ClusterRole** и **ClusterRoleBinding** (`ClusterRole.yaml`, `ClusterRoleBinding.yaml`)
- **Deployment** с контейнером KRR (`deploy.yaml`)
- **Service** (`svc.yaml`)
- **Ingress** для доступа к результатам (`ingress.yaml`)

Пример `deploy.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: krr
  namespace: krr
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: krr
          image: robustadev/krr:v1.22.0
          command:
            - /bin/sh
            - -c
            - |
              while true; do
                python krr.py simple-limit --formatter html --fileoutput /output/index.html;
                sleep 1d;
              done
          resources:
            requests:
              memory: 1Gi
            limits:
              memory: 2Gi
```

## Использование KRR

Примеры запуска KRR:

- Получение рекомендаций в терминале:
  ```sh
  krr simple-limit -p <Prometheus_URL> --formatter yaml
  ```
- Запуск с фильтрацией по namespace:
  ```sh
  krr simple-limit -p <Prometheus_URL> -n my-namespace
  ```

## Интеграции и возможности

- Веб-интерфейс KRR для просмотра рекомендаций.
- Отправка отчетов в Slack.
- Возможность подключения к `k9s` как плагин.

## Заключение

KRR — мощный инструмент для автоматической оптимизации ресурсов в Kubernetes, который помогает экономить до **69% CPU** и **18% памяти**. Простота настройки и гибкость интеграции делают его отличным выбором для DevOps-инженеров.



# Особенности KRR

Ширина html страницы регулируется с помощью переменной `COLUMNS`.

Приложение krr ходит в текущий k8s и смотрит там текущие pod, поэтому ему нужны `ClusterRole`, `ClusterRoleBinding`, `ServiceAccount`.

Для параметра `--prometheus-label cluster` нужно указать контекст для текущего кластера в VictoriaMetrics.

Используется стратегия `simple-limit`, у которой `cpu limit` = `cpu request`, так как в обычной стратегии `simple` cpu limit отключен.

Время после команды sleep указывает сколько ждать перед тем как запустить следующий раз.

Параметр `--cpu-min` по умолчанию равен 10. Но мы выставляем минимум 100.

Параметр `--cpu_request` равен 100 персентилю использования cpu

Параметр `--cpu_limit` равен 100 персентилю использования cpu. Максимальное значение лоя cpu_limit=100.

Параметр `--memory_buffer_percentage` по умолчанию 15 - Процентное соотношение добавленного буфера к пиковому объему используемой памяти для рекомендации по использованию памяти

Параметр `--oom_memory_buffer_percentage` по умолчанию 25 - На какой процент увеличить объем памяти при возникновении событий OOM Killed
