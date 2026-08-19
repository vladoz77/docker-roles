# Ansible role: Docker

Ansible-роль для установки и базовой настройки Docker CE на системах семейства Debian и Red Hat.

Роль:

- добавляет официальный репозиторий Docker;
- устанавливает Docker CE, Docker CLI, containerd, Buildx и Compose Plugin;
- запускает и включает сервис Docker;
- создает группу `docker` и добавляет в нее указанных пользователей;
- записывает параметры Docker daemon в `/etc/docker/daemon.json`.

## Поддерживаемые системы

Роль выбирает набор задач по `ansible_facts.os_family`:

- `Debian` — используется APT-репозиторий Docker;
- `RedHat` — используется репозиторий Docker для CentOS/RHEL.

Для Debian-систем учитываются архитектуры `x86_64`, `aarch64` и `armv7l`.

## Требования

- Ansible `2.14` или новее;
- целевая система семейства Debian или Red Hat;
- доступ к официальным репозиториям Docker;
- запуск роли с повышенными привилегиями (`become: true`).

Роль использует только встроенные модули Ansible и не требует установки дополнительных коллекций.

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

Пример с настройкой пользователей и Docker daemon:

```yaml
---
- name: Configure Docker hosts
  hosts: docker_hosts
  become: true
  vars:
    docker_users:
      - deploy
      - molecule
    docker_daemon_options:
      log-opts:
        max-size: "50m"
  roles:
    - role: vlad.docker
```

После добавления пользователя в группу `docker` ему потребуется заново войти в систему.

## Переменные

| Переменная | Значение по умолчанию | Описание |
| --- | --- | --- |
| `docker_service_name` | `docker.service` | Имя systemd-сервиса Docker. |
| `docker_users` | `["{{ ansible_user_id }}"]` | Пользователи, добавляемые в группу `docker`. |
| `docker_daemon_options` | `{ log-opts: { max-size: "100m" } }` | Содержимое `/etc/docker/daemon.json`. При пустом словаре файл не создается и не изменяется. |

Параметры `docker_daemon_options` проверяются перед записью командой `python3 -m json.tool`.
При изменении конфигурации сервис Docker перезапускается.

## Тестирование

Для Molecule предусмотрен сценарий с Ubuntu Jammy и Rocky Linux 9:

```bash
molecule test
```

Отдельные этапы сценария:

```bash
molecule converge
molecule verify
```

Проверка подтверждает, что сервис Docker активен, команда `docker version` выполняется, а файл daemon-конфигурации существует.

## Лицензия

MIT