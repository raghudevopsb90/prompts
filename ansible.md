Create an Ansible project named `roboshop-ansible` that deploys the RoboShop e-commerce application, with each component on its own server. Write working, beginner-readable code on the first attempt. Do not ask questions, and do not run anything: no servers, no DNS records, no testing.

## Source of truth: read these first

My step-by-step setup docs have the raw commands, packages, config files and systemd units for every component. Read all 14 before writing any code, then convert those manual steps into Ansible:

- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/00-overview.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/01-frontend.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/02-mysql.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/03-catalogue.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/04-mongodb.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/05-user.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/06-valkey.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/07-cart.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/08-shipping.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/09-rabbitmq.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/10-payment.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/11-notification.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/12-orders.md
- https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/13-ratings.md

If you cannot open these links, stop immediately and say so. Do not guess their content or write any code.

Use the docs for every command, package name, port, environment variable and config value. Take systemd units and `nginx.conf` from the docs as they are, changing only what "Corrections to the docs" below says. Where the docs and this prompt differ, this prompt wins.

## Environment

- Targets: RHEL 10 x86_64, SELinux and firewalld off. Login is `ec2-user` / `DevOps321` with passwordless sudo.
- Each server is reachable, and known to the other servers, as `<component>.rdevopsb90.online`.

## Project design

- `roboshop.yml`: one play on `hosts: all` with `become: yes`, using `roles:` to run the single role named by the extra variable `component_name`.
- `run.sh`: takes the component as `$1` and runs the playbook against `$1.rdevopsb90.online,` as an inline inventory, passing `component_name`, `ansible_user` and `ansible_password` with `-e`. Credentials go nowhere else.
- `ansible.cfg`: `[defaults]` has no host key checking and silent Python interpreter discovery; `[ssh_connection]` has 3 retries.
- `README.md`: prerequisites, and the run order from `00-overview.md`.
- One role per component (13 roles, named as in the docs). Unit files, `nginx.conf` and repo files go in that role's `templates/` folder and are installed with the `template` module, never `copy`, because `copy` does not look in `templates/`.

## How roles reuse code

Shared logic lives only in a `common` role, and other roles reach it with `import_role`:

```yaml
- name: Deploy as a NodeJS application
  ansible.builtin.import_role:
    name: common
    tasks_from: nodejs
```

- Always use `ansible.builtin.import_role` with `name` and `tasks_from` to reuse from another role. Never use a file path to another role's tasks, and never use `include_role`.
- Inside the `common` role, one task file loads another with `ansible.builtin.import_tasks` and a relative name, such as `pre-reqs.yml`.
- `common/tasks/` has one file per concern: `pre-reqs`, `nodejs-install`, `nodejs`, `golang`, `java`, `python`, `mysql-schema`, `systemd`.
- `common` has no `main.yml`, `vars/` or `handlers/`.
- `nodejs`, `golang`, `java` and `python` are complete entry points. Each one runs the whole "Deployment order" below by itself.
- Each entry point has this shape, and its language install and build tasks are written directly in that file: first `import_tasks: pre-reqs.yml`, then the install and build tasks, last `import_tasks: systemd.yml`. `nodejs` also imports `nodejs-install.yml` in place of writing the install tasks.
- A task file must never import itself, and task files must never import each other in a loop. `golang.yml`, for example, must not import `golang.yml`. A loop stops Ansible with "A recursion loop was detected".
- Imports go one way only: entry points import `pre-reqs`, `nodejs-install` and `systemd`; `systemd` imports `mysql-schema`; those four import nothing else.
- An app role (catalogue, user, cart, shipping, payment, notification, orders, ratings) has exactly one task in `tasks/main.yml`, the `import_role` above, and nothing else. It has a `vars/main.yml` only if it needs one of the two variables below.
- The only variables an app role may set are `schema_files` (a list) and `extra_pip_packages`. The shared tasks read those variables, so no component name or schema file is hard-coded in `common`. `component_name` comes only from `-e`.
- `frontend`, `mysql`, `mongodb`, `valkey` and `rabbitmq` have their own tasks. `frontend` reuses `nodejs-install` through `import_role`, because it builds in `/tmp/frontend` and has no systemd unit of its own.
- `frontend` must not use `appuser`: do not create it, and do not import `pre-reqs` or `systemd` for frontend. Its build and the files in `/usr/share/nginx/html` run and stay as root, the default under `become`, so set no `owner` or `group` on them. Keep the docs' `nginx.conf` and the packaged nginx service unchanged.

## Corrections to the docs

1. **Artifact URLs:** the docs download from `raghudevopsb89`. Use `https://raw.githubusercontent.com/raghudevopsb90/roboshop-microservices/main/artifacts/<component>.zip`.
2. **Hostnames:** the docs use `localhost` or `<MYSQL-SERVER-IP>` placeholders and tell you to substitute private IPs. Use `<component>.rdevopsb90.online` everywhere instead, including `nginx.conf`, all systemd units, and the `mysql -h` commands. Example: `Environment=MYSQL_HOST=mysql.rdevopsb90.online`. Keep `localhost` only in the MySQL `root@'localhost'` statement.
3. **App user:** create `appuser` with home `/app` for all 8 app services (not frontend), not only notification, or gunicorn logs a permission error.
4. **Notification:** its `requirements.txt` does not include gunicorn, so also install the pip package `gunicorn`.
5. **Ratings:** in the unit file, use `ExecStart=/usr/local/bin/gunicorn -b 0.0.0.0:8006 app:app`.
6. **Valkey:** replace the whole `bind 127.0.0.1...` line with `bind 0.0.0.0`.
7. **MongoDB:** replace the whole `bindIp 127.0.0.1...` line with `bind 0.0.0.0`.

## Deployment order inside the common role

Every run, in this order:

1. Create `appuser`.
2. Delete and recreate `/app` with `owner: appuser` and `group: appuser`, then unpack the app zip into it with `owner: appuser` and `group: appuser` on the `unarchive` task too.
3. Install the language and build.
4. If `schema_files` is defined: install the MySQL client, then load each listed file from `/app/db/`, in the docs' order. The schema must load BEFORE the service starts.
5. Give `appuser` ownership of `/app` again, recursively, because the build runs as root and creates root-owned files, and remove all access for others (`mode: o-rwx`, as the docs' `chmod o-rwx` does).
6. Install the unit file with `template`, reload systemd, then enable and restart the service.

## Rules that decide whether this works

- **Code is always downloaded:** every component, including frontend, must download its zip before using it. Application code and SQL come only from the zip, so never copy them into the project.
- **No `creates` or `removes`:** do not use them on any task. `/app` is rebuilt every run, and a file from the zip already exists, so `creates` would silently skip the schema load.
- **Safe to re-run:** make the MySQL root password step and the RabbitMQ `add_user` step safe by checking first (for example, try the new password, or list users), then acting only if needed. A real failure must still fail.
- **Do not hide errors:** no `ignore_errors`, `failed_when: false`, `|| true` or `2>/dev/null`.
- **Config changes must apply:** services whose config you changed use `state: restarted`, not `started`, including nginx, mongod and valkey. Run `nginx -t` before restarting nginx.
- **RabbitMQ order:** start the service, enable the plugin, restart, then create the user.
- **One action per task:** a task has one module. Never repeat a module key inside a task.
- **Modules:** use real modules (`dnf`, `template`, `file`, `unarchive`, `user`, `replace`, `systemd_service`) wherever they fit, and `shell` only for builds, the NodeSource setup script, the MySQL client, `rabbitmqctl` and `rabbitmq-plugins`. Repo files already set `gpgcheck=0`, so do not add `disable_gpg_check`.
- **Nothing extra:** no vault, tags, handlers, `group_vars`, extra roles, extra packages, empty files, helper scripts or absolute paths from any machine. Do not install a package a component does not need.
- **Style:** give every task a clear name, and quote any YAML value that starts with `{{`.

## Review your own code before finishing

Mentally run `cart`, `catalogue`, `notification`, `frontend` and `mysql` on a clean server, then run them a second time, and fix whatever fails. Then confirm:

- Each task's input already exists when that task runs: the user before ownership changes, the zip before the build.
- Every file in the project is used, and every file referenced exists.
- `localhost` appears nowhere in `nginx.conf` or any unit file.
- The `raghudevopsb89` org appears nowhere.
- `notification` gets gunicorn and `ratings` gets cryptography, and no other component gets either.
- The task creating `/app` and the task unpacking into it both set `owner` and `group` to `appuser`.
- `appuser` appears nowhere in the `frontend` role.
- For every `import_tasks` and `import_role` line, the file it names is a different file from the one it is written in, and following the imports never returns to a file already visited.

## Output

If you have file-writing tools, create the files in a folder named `roboshop-ansible` and reply only with the file list. Otherwise, for each file print `### <path>` followed by one fenced code block with the full content. No placeholders and no explanations.
