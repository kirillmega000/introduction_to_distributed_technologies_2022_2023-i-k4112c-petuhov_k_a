1. Что такое ConfigMap и Secrets? - основные понятия, виды ресурсов + манифесты для каждого типа ресурсов

ConfigMap - объект Kubernetes, используемый для хранения неконфединциальных данных в формате ключ-значение. Может быть 
использована для хранения значений переменных, аргументов командной строки. ConfigMap может быть использован 
как Volume для контейнеров Pod и постоянного хранения информации.

Пример ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
data:
  REACT_APP_USERNAME: Kirill
  REACT_APP_COMPANY_NAME: ITMO
```

Secret - объект Kubernetes для хранения защищенных данных, таких как данные учетной записи, токены доступа ...
Данные будут хранится в Kubernetes с повышенной безопасностью. Для них можно настроить политики доступа, шифрование,
ограничить доступ для определенных Pod. 

Пример Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-example
type: Opaque
data:
  USER_NAME: Kirill
  PASSWORD: ITMO
```

Secret может быть разных типов и предназначен для хранения различных видов данных (например, tls сертификат).
Стандартным значение является Opaque (хранение произвольных данных).

