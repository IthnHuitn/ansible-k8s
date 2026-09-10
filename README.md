# Ansible Playbooks — K8s Infrastructure

Ansible-плейбук для развёртывания Kubernetes-кластера в Yandex Cloud: 
настройка нод, инициализация control plane, подключение worker-нод, 
установка ingress-nginx и kube-prometheus-stack (Prometheus, Grafana, Alertmanager).
---
## Структура

```text
.
├── ansible.cfg
├── inventory/
│   ├── group_vars/
│   │   └── all.yml             # Общие переменные (подсети, CIDR, версии и т. д.)
│   └── hosts.yml.example       # Пример инвентаря (заполняется Terraform)
├── playbooks/
│   └── site.yml                # Главный плейбук — запускает все роли
├── roles/
│   ├── common/                 # Базовая настройка всех нод
│   ├── kubernetes/             # Установка container runtime и k8s-компонентов
│   ├── master/                 # Инициализация control plane, ingress, мониторинг
│   └── worker/                 # Подключение worker-нод к кластеру
└── README.md

```

#### Роли
|Роль |	Назначение |
|-----|------------|
|common | Обновление пакетов, установка утилит, настройка sysctl, отключение swap |
|kubernetes | Установка containerd, kubeadm, kubelet, kubectl |
|master | kubeadm init, установка Flannel, ingress-nginx, kube-prometheus-stack, копирование kubeconfig |
|worker | kubeadm join — подключение worker-нод через bastion (ProxyJump) |
---

#### Запуск

```bash
ansible-playbook playbooks/site.yml -i inventory/hosts.yml
```

#### Доступ к Grafana

После выполнения плейбука Grafana доступна по адресу:
```bash
http://<LB_PUBLIC_IP>/
```

Первичный вход:

    login: admin
    password: admin

#### Проверка
```bash
# Все поды
kubectl get pods --all-namespaces

# Ingress-nginx
kubectl get pods -n ingress-nginx

# Ingress-маршруты
kubectl get ingress -n monitoring

# Ноды кластера
kubectl get nodes
```
