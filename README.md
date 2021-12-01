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
* Запустить модуль и проверить доступность сервиса Vault. <br>
Т.к столкнулся с проблемой, что не собираются контейнеры т.к не скачивается ничего при сборке и yum update и т.п не работает.Установка vault через helm имеет много нюансов </br>
Для примера испытал динамическое создание логина и пароля для MySQL в k8s, через vault на сторенней машине вне k8s </br>
Создание пода в k8s[mysql-pod.yaml](https://github.com/murzinvit/14.2_vault/blob/af59c55e2d702f1cd5f72a8ab2ca1df8ea60ab79/mysql-pod.yaml) </br>
Опубликовать под `kubectl expose pod k8s-mysql --type=NodePort --name=mysql-service` </br>
После всего имеем root пароль, логин, ip ноды, и динамический порт 30k+ для соединения с БД из внешнего мира </br>
На стронней машине устанавливаем vault: </br>
`wget https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo -O /etc/yum.repos.d/hashicorp.repo` </br>
`yum install vault && systemctl enable vault --now` </br>
vault запечатан: `vault status`</br>
![saeled_vault](https://github.com/murzinvit/screen_1/blob/42cbe17a2fbe4259c2d71ebe4fda4ebd755eda82/kuber_saeled_vault.jpg) </br>
Распечатываем командой `vault operator init`, и трёкратным введением команды `vault operator unseal` и полученных Unseal Key </br>
![vault_sealed_fauls](https://github.com/murzinvit/screen_1/blob/403e277a6ba507f8c06f410e0ba3c81ede49ae48/Kuber_vault_sealed_fauls.jpg) </br>
Залогиниться на сервер: `vault login` </br>
![vault_login](https://github.com/murzinvit/screen_1/blob/53677c88a2de116c999dd1dbbc0c5ea10748fb9d/Kuber_vault_login.jpg) </br>
Далее разрешаем database secrets engine: `vault secrets enable database` </br>
Настройка для подключения vault к базе данных: </br>
```
vault write database/config/news \
 plugin_name=mysql-database-plugin \
 connection_url="{{username}}:{{password}}@tcp(10.10.1.111:31812)/" \
 allowed_roles="test-role" \
 username="root" \
 password="admin"
 ```
 Создать role в vault: </br>
 ```
 vault write database/roles/test-role \
 db_name=news \
 creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';" \
 default_ttl="1h" \
 max_ttl="24h"
 ```
 Создать пользователя и пароль `vault read database/creds/test-role` </br>
 ![login_pass_in_vault](https://github.com/murzinvit/screen_1/blob/ae977917fb5af02760fb0b99fd7487c2050c6ce0/Kuber_vault_create_login_pass_in_vault.jpg) <br>
 Теперь можно логиниться в MySQL с помощью полученных логина и пароля: </br>
 ![login_in_heidi](https://github.com/murzinvit/screen_1/blob/c478a0d1437dce6ceb50b237acecc72c7f00c0b7/Kuber_login_in_heidi.jpg) </br>
 Логин прошёл успешно: </br>
 ![login_token_db](https://github.com/murzinvit/screen_1/blob/e8101ad364d1e419da9154d2709719328f376016/Kuber_vault_login_token_db.jpg) </br>
 
 В качестве мануала использовал статью: https://www.dmosk.ru/instruktions.php?object=vault-hashicorp </br>
 
### Рабочие заметки: </br>
https://gitlab.com/k11s-os/k8s-lessons/-/tree/main/Vault <br>
https://pythonru.com/biblioteki/ustanovka-i-podklyuchenie-sqlalchemy-k-baze-dannyh <br>
https://khashtamov.com/ru/postgresql-python-psycopg2/ <br>
https://khashtamov.com/ru/pyenv-python/ <br>
Установка vault: </br>
https://www.dmosk.ru/instruktions.php?object=vault-hashicorp </br>
https://linux-notes.org/ustanovka-vault-v-unix-linux/ </br>
https://learn.hashicorp.com/tutorials/vault/kubernetes-raft-deployment-guide?in=vault/kubernetes </br>
https://github.com/hashicorp/vault-helm/issues/139 </br>
https://adminunix.ru/podnimaem-nfs-na-debian/ </br>
https://marcofranssen.nl/install-hashicorp-vault-on-kubernetes-using-helm-part-1 </br>
