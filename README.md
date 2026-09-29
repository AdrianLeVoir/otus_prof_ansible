# Домашнее задание: Первые шаги с Ansible

## 1. Стенд

| Параметр | Значение |
|---|---|
| Гипервизор | Proxmox VE |
| ВМ | Rocky Linux 10, SELinux Enforcing |
| Управление | ansible-playbook с ноутбука (ansible-core 2.21), inventory `inventory.ini` |

## 2. Выполнение условий задания

| Условие | Как выполнено |
|---|---|
| модуль yum/apt | `ansible.builtin.yum: name: nginx, state: present` (Rocky → yum) |
| конфиг из jinja2-шаблона с переменными | `templates/nginx.conf.j2` → `template` в `/etc/nginx/nginx.conf`; в шаблоне подставляются `{{ nginx_port }}`, `{{ worker_processes }}`, `{{ keepalive_timeout }}` |
| nginx в enabled | handler и verify-таск: `ansible.builtin.systemd: name: nginx, state: started, enabled: true` |
| notify для старта после установки | handler `start nginx`; на него подписаны задачи установки и шаблона |
| нестандартный порт 8080 через переменные | `vars: nginx_port: 8080`; шаблон слушает `listen {{ nginx_port }};`, проверка ходит на `http://127.0.0.1:{{ nginx_port }}/` |

## 3. Запуск

```bash
cd hw10
ansible-playbook playbook.yml        # inventory указан в ansible.cfg
```

Результат прогона:

```
TASK [Check port 8080 responds] ************ ok: [iris]
TASK [Show check result] *******************
ok: [iris] => {"msg": "nginx on port 8080: HTTP 200"}

PLAY RECAP **** iris: ok=7 changed=2 unreachable=0 failed=0
```

## 4. Проверка результата

На ВМ и снаружи (с хоста управления):

```
curl http://127.0.0.1:8080/      → 200
curl http://192.168.1.184:8080/  → 200
systemctl is-enabled nginx       → enabled
systemctl is-active nginx        → active
ss -tlnp | grep 8080             → LISTEN 0.0.0.0:8080 (nginx)
```

Идемпотентность: повторный `ansible-playbook playbook.yml` — `changed=0`, notify не срабатывает лишний раз.
