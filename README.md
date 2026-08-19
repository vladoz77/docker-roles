# Ansible role: Docker

Роль устанавливает и настраивает Docker CE на системах Debian/Ubuntu и Red Hat/Rocky Linux.

Роль выполняет следующие действия:

- добавляет официальный репозиторий Docker;
- устанавливает Docker и Python SDK для Ansible-модулей Docker;
- запускает и включает сервис Docker;
- добавляет указанных пользователей в группу `docker`;
- записывает настройки демона в `/etc/docker/daemon.json`;
- создает Docker-сеть.

## Требования

- Ansible `2.14` или новее;
- целевая система семейства Debian или Red Hat;
- коллекция `community.docker` версии `3.4.0` или новее;
- подключение к официальным репозиториям Docker;
- выполнение роли с `become: true`.

Установить зависимость можно командой:

```bash
ansible-galaxy collection install -r requirements.yaml
```

## Использование

```yaml
---
- name: Install Docker
  hosts: docker_hosts
  become: true
  roles:
    - role: vlad.docker
```

Для локального checkout роль можно подключить по пути:

```yaml
roles:
  - role: /path/to/docker-roles
```

После добавления пользователя в группу `docker` ему потребуется заново войти в систему.

## Переменные

| Переменная | Значение по умолчанию | Описание |
| --- | --- | --- |
| `docker_network_name` | `docker_default` | Имя создаваемой Docker-сети. |
| `docker_network_driver` | `bridge` | Драйвер Docker-сети. |
| `docker_network_subnet` | `""` | Подсеть сети. Пустое значение отключает явную настройку IPAM. |
| `docker_service_name` | `docker.service` | Имя systemd-сервиса Docker. |
| `docker_users` | `["{{ ansible_user_id }}"]` | Пользователи, добавляемые в группу `docker`. |
| `docker_daemon_options` | `{ log-opts: { max-size: "100m" } }` | Содержимое `/etc/docker/daemon.json`. Пустой словарь отключает создание файла. |

Пример настройки:

```yaml
docker_users:
  - deploy
  - molecule

docker_network_name: app_network
docker_network_subnet: 172.28.0.0/16
docker_daemon_options:
  log-driver: json-file
  log-opts:
    max-size: 50m
```

## Тестирование

Тесты Molecule запускаются на Ubuntu Jammy и Rocky Linux 9:

```bash
molecule test
```

Для запуска только converge или verify:

```bash
molecule converge
molecule verify
```

## Лицензия

MIT