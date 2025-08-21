---

# 🚀 Ansible Nginx High Availability Load Balancer

![Ansible](https://img.shields.io/badge/Ansible-2.18-red) ![Nginx](https://img.shields.io/badge/Nginx-1.23-blue) ![HA](https://img.shields.io/badge/HA-LoadBalancer-green) ![Ubuntu](https://img.shields.io/badge/OS-Ubuntu-orange)

---

## 📌 Описание проекта

Проект создаёт **высокодоступный балансировщик нагрузки (HA Load Balancer) на Nginx** с динамическим пулом бэкендов через **Ansible**.

**Возможности:**

* Автоматическая установка и настройка Nginx на LB и backend.
* Динамическое формирование upstream пула из Ansible inventory.
* Безопасная проверка конфигурации Nginx (`nginx -t`) перед reload.
* Расширенные логи с указанием конкретного backend сервера.
* Легко масштабируемая структура с ролевыми playbook’ами.

---

## 🏗 Архитектура системы

```text
                           ┌─────────────────────┐
                           │     Clients         │
                           │  (Web Browsers /    │
                           │   curl / Postman)   │
                           └─────────┬──────────┘
                                     │ HTTP Requests
                                     ▼
                           ┌─────────────────────┐
                           │    LB Server        │
                           │   Nginx LoadBalancer│
                           │   - Listen: 80      │
                           │   - Upstream Pool   │
                           └─────────┬──────────┘
                                     │ Proxied Requests
              ┌──────────────────────┼───────────────────────┐
              ▼                      ▼                       ▼
     ┌───────────────┐       ┌───────────────┐       ┌───────────────┐
     │ Backend 1     │       │ Backend 2     │  ...  │ Backend N     │
     │ Nginx App     │       │ Nginx App     │       │ Nginx App     │
     └───────────────┘       └───────────────┘       └───────────────┘
```

---

## 📁 Файловая структура

```text
ansible-nginx-ha-lb/
├── inventory/
│   └── hosts.txt            # LB и backend серверы
├── playbooks/
│   └── site.yml             # Главный playbook
├── roles/
│   ├── lb/                  # Роль Load Balancer
│   │   ├── tasks/
│   │   │   └── configure.yml
│   │   └── templates/
│   │       ├── lb.conf.j2
│   │       └── upstream_backends.j2
│   └── backend/             # Роль backend серверов
│       └── tasks/main.yml
└── README.md
```

---

## ⚙ Роли Ansible

### `lb` — Load Balancer

* Устанавливает Nginx.
* Деплоит конфигурации:

  * `lb.conf.j2` — основной конфиг LB.
  * `upstream_backends.j2` — пул backend-серверов.
* Валидирует конфигурацию перед reload.
* Убирает дефолтный сайт Debian/Ubuntu.

### `backend` — Backend Servers

* Устанавливает Nginx.
* Настраивает базовую страницу для теста.
* Проверяет доступность порта 80.

---

---

## 🚀 Запуск проекта

```bash
git clone <repo-url>
cd ansible-nginx-ha-lb
ansible-playbook -i ./inventory/hosts.txt playbooks/site.yml
```

> Playbook установит Nginx на LB и backend, настроит конфиги и безопасно reload.

---

## 🔍 Проверка работы

1. Проверка страницы LB (должен проксировать на backend):

```bash
curl -sv http://<LB_IP>/
```

2. Проверка логов LB:

```bash
sudo tail -f /var/log/nginx/lb_access.log
```

* В логах увидите: `upstream: "10.0.2.15:80"` или `"10.0.2.16:80"` — значит, балансировка работает.

3. Проверка upstream Nginx:

```bash
sudo nginx -T | grep upstream -A 5
```

---

## 💡 Возможные улучшения

* Настроить **health checks** для backend.
* Добавить **SSL (HTTPS)** на LB.
* Включить **sticky sessions**.
* Поддержка **RedHat/CentOS**.

---

## 🔧 Полезные команды

```bash
# Проверка конфигурации Nginx
sudo nginx -t

# Перезагрузка Nginx
sudo systemctl reload nginx

# Просмотр логов LB
sudo tail -f /var/log/nginx/lb_access.log
```

---

