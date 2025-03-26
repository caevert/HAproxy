# HAproxy

[Файлы конфигурации](files/)
Дашборд Grafana
![Dashbord-1](./files/Graf1.png)

![Dashbord-2](./files/Graf2.png)
Метрики Grafana
![Dashbord-3](./files/Graf_metrics.png)

![Dashbord-4](./files/haproxy.cfg)
```
[Конфигурация terraform](I.Terraform/)

2. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

---
## Этап второй - Создание Kubernetes кластера

На данном этапе необходимо развернуть `Kubernetes` кластер, для данной задачи будем использовать набор конфигураций _Ansible_ [`Kubespray`](https://github.com/kubernetes-sigs/kubespray)

1. Клонирем `kubespray` командой `git clone https://github.com/kubernetes-sigs/kubespray`
2. Создаем конфигурацию своего кластера:

```shell
cd kubespray
cp inventory/sample inventory/netology
```

3. Выясняем айпи машин кластера на которые будет производится установка:

![Kube](./assets/K-1.png)
