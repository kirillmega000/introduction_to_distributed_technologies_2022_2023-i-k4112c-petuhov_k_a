## 1. Что такое ConfigMap и Secrets? - основные понятия, виды ресурсов + манифесты для каждого типа ресурсов

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

## 2. Механизм стейкинга токенов в различных сетях.

Proof of stake - алгоритм консенсуса, который использует активы пользователя в качестве стейка (ценность, которой
рискует валидатор при проверке для гарантии честной работы). Он является более эффективной и менее ресурсоемкой
альтернативой алгоритму Proof of Work. 

При майнинге стейком стоимость оьорудования и затраченные ресурсы майнера. При этом майнеры гонятся за решением сложной 
криптографической головоломки. Головоломка, которую пытаются решить майнеры, не служит никакой другой цели, 
кроме обеспечения безопасности сети. Таким образом получается избыточная и излишняя трата электроэнергии.

Альтернатива - стекинг. Его основная идея в том, что участники могут блокировать свою долю монет (в стейкинге), 
и через определенные промежутки времени протокол случайным образом предоставляет одному из них право на валидацию
следующего блока. При этом вероятность выбора валидатора пропорциональна количеству валюты: чем больше манет участник предоставляет на блокировку, тем выше шанс, 
что он будет выбран. Таким образом упраздняется гонка между участниками валидации.

При стекинге отпадает необходимость в покупке дорогостоющего оборудования для участия в валидации, уменьшается порог входа.
Таким образом увеличивается количество нод в сети и ее эффективность.