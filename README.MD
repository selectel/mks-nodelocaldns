# Настройка NodeLocal DNS Cache в кластере

В случаях, когда на DNS в вашем кластере оказывается повышенная нагрузка, мы рекомендуем вам использовать NodeLocal DNS Cache. Это позволяет уменьшить нагрузку на DNS, а также уменьшить задержку резолвинга внутренних и внешних ресурсов.

Скачиваем себе [манифест](https://raw.githubusercontent.com/selectel/mks-nodelocaldns/master/nodelocaldns.yaml).

Далее, применяем этот манифест командой `kubectl create -f nodelocaldns.yaml`.

Проверить корректность работы DNS можно следующим образом.

Вам необходимо включить режим логирования для подов NodeLocal DNS. Отредактируйте манифест, добавив слово log в место, на которое указывает стрелка:

```conf
data:
  Corefile: |
    cluster.local:53 {
        log <-- место, в котором необходимо добавить запись
        errors
        cache {
                success 9984 30
                denial 9984 5
        }
```

После этого, сохраните манифест и примените его:
`kubectl apply -f nodelocaldns.yaml`

Затем, вам необходимо развернуть под с утилитами для проверки DNS и войти в tty данного пода. Сделать это можно помощью команд:

```shell
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
kubectl exec -i -t dnsutils -- bash
```

Подключившись к поду, выполните команду:

```shell
dig @169.254.20.25 selectel.ru
```

После этого, вы сможете увидеть попытку получить адрес в логах. Чтобы посмотреть весь лог, выполните команду:

```shell
kubectl logs --namespace=kube-system -l k8s-app=kube-dns -f
```
