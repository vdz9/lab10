[![CI](https://github.com/vdz9/lab06/actions/workflows/ci.yml/badge.svg)](https://github.com/vdz9/lab06/actions/workflows/ci.yml)
### Tasks: Изучение систем автоматизации развёртывания и управления приложениями на примере Docker

1. Создать публичный репозиторий с названием **lab10** на сервисе **GitHub**
2. Ознакомиться со ссылками учебного материала
3. Выполнить инструкцию учебного материала

### 1. Настройка переменных окружения

```bash
export GITHUB_USERNAME=vdz9
export GITHUB_TOKEN=<сохраненный_токен>
source scripts/activate

cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate

git clone https://github.com/${GITHUB_USERNAME}/lab09.git projects/lab10
cd projects/lab10
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab10.git
```

Результат: Копирование репозитория из предыдущей лабораторной работы в текущую и последующая его привязка к новому репозиторию
### 2. Установка vagrant

```bash
$ vagrant version
$ vagrant init bento/ubuntu-22.04
$ less Vagrantfile
$ vagrant init -f -m bento/ubuntu-22.04
```

Результат: Установка vagrant

### 3. Cоздание директории для общих файлов

```bash
mkdir -p shared
```
Результат: Создание директория shared для синхронизации файлов между хостовой и виртуальной системами

### 4. Загрузка, добавление образа Ubuntu 22.04 и инициализая vagrant

```bash
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64-vagrant.box -O /tmp/ubuntu-22.04.box
vagrant box add ubuntu-22.04 /tmp/ubuntu-22.04.box
vagrant init ubuntu-22.04
```
Результат: Образ загружен напрямую из официального репозитория Ubuntu Cloud Images и добавлен локально, создание файла Vagrantfile с указанием образа ubuntu-22.04

### 5. Создание Vagrantfile

```bash
cat > Vagrantfile <<'EOF'
$script = <<-SCRIPT
sudo apt update
sudo apt install -y docker.io
sudo docker pull ubuntu:22.04
sudo docker create -ti --name fastide ubuntu:22.04 bash
sudo docker cp fastide:/home/ubuntu /home/
sudo useradd developer
sudo usermod -aG sudo developer
echo "developer:developer" | sudo chpasswd
sudo chown -R developer /home/developer
SCRIPT
EOF
```
```bash
cat >> Vagrantfile <<'EOF'
Vagrant.configure("2") do |config|
EOF
```
```bash
cat >> Vagrantfile <<'EOF'
  config.vm.box = "ubuntu-22.04"
  config.vm.network "public_network"
  config.vm.synced_folder('shared', '/vagrant', type: 'rsync')
  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
  end
  config.vm.provision "shell", inline: $script, privileged: true
  config.ssh.extra_args = "-tt"
end
EOF
```

Результат: Создание Vagrantfilе с настройками: ос - ubuntu-22.04, публичная сеть, синхронизация папки shared, 2 ГБ ОЗУ, скрипт для установки Docker и создания пользователя developer
### 6. Проверка статуса и запуск виртуальной машины

```bash
vagrant status
vagrant up --provider=virtualbox
```

Результат: Запуск виртуальной машины
### 7. Проверка портов и статуса

```bash
vagrant port
```
```bash
vagrant status
```

Результат: 

```bash
The forwarded ports for the machine are listed below. Please note that
these values may differ from values configured in the Vagrantfile if the
provider supports automatic port collision detection and resolution.

    22 (guest) => 2222 (host)
```
```bash
Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run vagrant halt to
shut it down forcefully, or you can run vagrant suspend to simply
suspend the virtual machine. In either case, to restart it again,
simply run vagrant up.
```
### 8. Подключение к виртуальной машине по SSH

```bash
vagrant ssh
```

Результат: 

```bash
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-179-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri May 22 20:04:00 UTC 2026

  System load:  0.1               Processes:               106
  Usage of /:   4.1% of 38.70GB   Users logged in:         0
  Memory usage: 21%               IPv4 address for enp0s3: 10.0.2.15
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '24.04.4 LTS' available.
Run 'do-release-upgrade' to upgrade to it.
```

### 9. Работа со снапшотами

```bash
vagrant snapshot list
vagrant snapshot push
vagrant snapshot list
vagrant halt
vagrant snapshot pop
```
Результат:
```bash
==> default: No snapshots have been taken yet!
    default: You can take a snapshot using vagrant snapshot save. Note that
    default: not all providers support this yet. Once a snapshot is taken, you
    default: can list them using this command, and use commands such as
    default: vagrant snapshot restore to go back to a certain snapshot.
```
```bash
==> default: Snapshotting the machine as 'push_1779480943_677'...
==> default: Snapshot saved! You can restore the snapshot at any time by
==> default: using vagrant snapshot restore. You can delete it using
==> default: vagrant snapshot delete.
```
```bash
==> default: 
push_1779480943_677
```
```bash
==> default: Attempting graceful shutdown of VM...
```
```bash
==> default: Restoring the snapshot 'push_1779480943_677'...
==> default: Deleting the snapshot 'push_1779480943_677'...
==> default: Snapshot deleted!
==> default: Checking if box 'ubuntu/jammy64' version '20241002.0.0' is up to date...
==> default: Resuming suspended VM...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Machine already provisioned. Run vagrant provision or use the --provision
==> default: flag to force provisioning. Provisioners marked to run always will still run.
```
