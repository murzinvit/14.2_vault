## Домашнее задание к занятию "14.2 Синхронизация секретов с внешними сервисами. Vault" </br>

### Задача 1: Работа с модулем Vault </br>
Запустить модуль Vault конфигураций через утилиту kubectl в установленном minikube </br>
```kubectl apply -f 14.2/vault-pod.yml```
![vault_pod_run](https://github.com/murzinvit/screen_1/blob/40c163a3a2845515d19186777154b8bd5d01f682/Kuber_vault_pod_run.jpg) </br>

Получить значение внутреннего IP пода(Прим: jq - утилита для работы с JSON в командной строке) </br>
```kubectl get pod 14.2-netology-vault -o json | jq -c '.status.podIPs'```
![ip_vault_pod](https://github.com/murzinvit/screen_1/blob/b08900bd98e311e43847251d54aaa2be8f4a4361/Kuber_ip_vault_pod.jpg) </br>

Запустить второй модуль для использования в качестве клиента </br>
```kubectl run -i --tty fedora --image=fedora --restart=Never -- sh```</br>
![vault_run_client](https://github.com/murzinvit/screen_1/blob/016aed66152a595f95fc64441ba427555fc0fe67/Kuber_vault_run_client.jpg) </br>

Установить дополнительные пакеты </br>
```dnf -y install pip & pip install hvac```</br>
![grep_hvac](https://github.com/murzinvit/screen_1/blob/ddc8ff0084306e3b07fa4e2046293ab64dae46f4/Kuber_grep_hvac.jpg) </br>

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
![output_secret_python](https://github.com/murzinvit/screen_1/blob/129960dca030273dd12fcd3f4578bd8172067b29/Kuber_output_secret_python.jpg) </br>

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
Установка vault: </br>
https://www.dmosk.ru/instruktions.php?object=vault-hashicorp </br>
