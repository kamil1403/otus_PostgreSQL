<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/2/29/Postgresql_elephant.svg" alt="PostgreSQL Logo" width="150">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-postgres__replication-00758F?style=for-the-badge&logo=postgresql&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-17.12.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание
1. Поднять 3 сервера (Master, Slave, Backup) через Vagrant.
2. Настроить потоковую репликацию (Streaming Replication).
3. Настроить сервер резервного копирования (Barman).
4. Реализовать сценарий восстановления (WAL streaming).

### ✅ Результат
- [x] Стенд развернут (Vagrant + Ansible).
- [x] Репликация Master-Slave работает (Postgres 14).
- [x] Бэкапы снимаются через Barman без ошибок.

**1. Статус репликации (Master):**
Результат см. на скриншоте 🖼️ ["1.png"](ссылка_на_скриншот)

**2. Статус Barman и успешный бэкап:**
Результат см. на скриншоте 🖼️ ["2.png"](ссылка_на_скриншот)

### 🧭 Оглавление
- [🧰 Шаг 1 - Инфраструктура](#one)
- [🛠️ Шаг 2 - Настройка Ansible](#two)
- [🔍 Шаг 3 - Проверка](#three)

---

<a id="one"></a>
## 🧰 Шаг 1 - Инфраструктура
**Master:** 192.168.57.11 (`node1`)   
**Slave:** 192.168.57.12 (`node2`)   
**Barman:** 192.168.57.13 (`barman`)   
Развертывание ВМ описано в `Vagrantfile`.

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

MACHINES = {
  :node1 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "node1",
        :cpus => 2,
        :memory => 1024,
        :ip => "192.168.57.11",
  },
  :node2 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "node2",
        :cpus => 2,
        :memory => 1024,
        :ip => "192.168.57.12",
  },
  :barman => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "barman",
        :cpus => 1,
        :memory => 1024,
        :ip => "192.168.57.13",
  },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.hostname = boxconfig[:vm_name]
      box.vm.network "private_network", ip: boxconfig[:ip]
      
      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
        v.name = "pg_#{boxconfig[:vm_name]}" 
      end
      
    end
  end
end
```

<a id="two"></a>
## 🧰 Шаг 2 - Настройка Ansible
Конфигурация выполняется через Ansible.
Запуск:
```bash
cd ansible
ansible-playbook -i hosts provision.yml
```

<a id="three"></a>
## 🧰 Шаг 3 - Проверка
1. Проверка репликации (на Master):
```bash
vagrant ssh node1
sudo -u postgres psql -c "select usename, application_name, client_addr, state, sync_state from pg_stat_replication;"
Ожидаем: state = streaming
```
2. Проверка Barman (на сервере Backup):
```bash
vagrant ssh barman
sudo su - barman
barman check node1 && barman list-backup node1
Ожидаем: OK по всем пунктам и наличие бэкапа в списке.
```
