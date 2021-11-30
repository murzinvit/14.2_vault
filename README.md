## Домашнее задание к занятию "14.2 Синхронизация секретов с внешними сервисами. Vault" </br>

### Задача 1: Работа с модулем Vault </br>
Запустить модуль Vault конфигураций через утилиту kubectl в установленном minikube </br>
```
kubectl apply -f 14.2/vault-pod.yml
```

Получить значение внутреннего IP пода </br>
```
kubectl get pod 14.2-netology-vault -o json | jq -c '.status.podIPs'
```

Примечание: jq - утилита для работы с JSON в командной строке </br>
Запустить второй модуль для использования в качестве клиента </br>
```
kubectl run -i --tty fedora --image=fedora --restart=Never -- sh
```

Установить дополнительные пакеты </br>
```
dnf -y install pip
pip install hvac
```

Запустить интепретатор Python и выполнить следующий код, предварительно поменяв IP и токен </br>
```
import hvac
client = hvac.Client(
    url='http://10.10.133.71:8200',
    token='aiphohTaa0eeHei'
)
client.is_authenticated()

# Пишем секрет
client.secrets.kv.v2.create_or_update_secret(
    path='hvac',
    secret=dict(netology='Big secret!!!'),
)

# Читаем секрет
client.secrets.kv.v2.read_secret_version(
    path='hvac',
)
```

### Задача 2 (*): Работа с секретами внутри модуля

* На основе образа fedora создать модуль;
* Создать секрет, в котором будет указан токен;
* Подключить секрет к модулю;
* Запустить модуль и проверить доступность сервиса Vault.


### Рабочие заметки: </br>
https://gitlab.com/k11s-os/k8s-lessons/-/tree/main/Vault <br>
https://pythonru.com/biblioteki/ustanovka-i-podklyuchenie-sqlalchemy-k-baze-dannyh <br>
https://khashtamov.com/ru/postgresql-python-psycopg2/ <br>
https://khashtamov.com/ru/pyenv-python/ <br>

