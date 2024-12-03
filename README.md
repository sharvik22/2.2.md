# Домашнее задание к занятию «Хранение в K8s. Часть 2» Шарапат Виктор

### Цель задания

В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке NFS в MicroK8S](https://microk8s.io/docs/nfs). 
2. [Описание Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). 
3. [Описание динамического провижининга](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/). 
4. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
6. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

* Создал манифест Deployment, состоящего из контейнеров busybox и multitool.
* Создал PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
* Проверил, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории.

![image](https://github.com/user-attachments/assets/0c770f5b-9632-42b3-a086-a4259955f9a4)


* удалил Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.

![image](https://github.com/user-attachments/assets/f4c43ea6-3ace-49f0-a4bc-ed6212645ab3)


![image](https://github.com/user-attachments/assets/e87607f1-0a95-4503-a516-5b1e4304873e)

после удаления PVC, PV останется в состоянии Released, так как мы использовали политику Retain. Это означает, что данные на диске сохранятся, но PV больше не будет доступен для использования.

* Продемонстрировать, что файл сохранился на локальном диске ноды.

![image](https://github.com/user-attachments/assets/ee32f44d-3f2c-437d-964d-588d9f33d180)


* Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
  
![image](https://github.com/user-attachments/assets/4add7948-5d74-4045-8e32-0c95a9d891d0)

Когда мы удаляем PV, Kubernetes не удаляет физические данные на узле. Это происходит потому, что PV — это просто абстракция над физическим хранилищем, и удаление PV не влияет на физические данные.


![image](https://github.com/user-attachments/assets/d9bf9dd1-ea93-47d1-b0af-0fd734b00d49)

---

### Решение 2

* Включил NFS-сервера в MicroK8S

### sudo apt update
### sudo apt install nfs-kernel-server

![image](https://github.com/user-attachments/assets/2b977774-a259-4e52-b754-59fbd8791e0a)

* Создал директорию, которая будет использоваться как NFS-шар

### sudo mkdir -p /mnt/nfs_share
### sudo chown nobody:nogroup /mnt/nfs_share
### sudo chmod 777 /mnt/nfs_share

![image](https://github.com/user-attachments/assets/3d10810e-af2a-47d4-a40f-38db6788394e)

* Настройка NFS-сервера

 Добавил запись в файл /etc/exports, чтобы настроить NFS-шар

### echo "/mnt/nfs_share *(rw,sync,no_subtree_check)" | sudo tee -a /etc/exports

![image](https://github.com/user-attachments/assets/791b62a5-c69b-4080-adb2-2d637cc19932)

* Применил изменения

### sudo exportfs -a
### sudo systemctl restart nfs-kernel-server

![image](https://github.com/user-attachments/assets/2fb26d66-7f17-4236-b9ac-2e37ae7c79e1)

* Создал и применил манифесты
  
* Проверка

![Screenshot_3](https://github.com/user-attachments/assets/edfc6637-ab0b-4cbf-9b34-6e884018451a)

* Подключился к поду создал файл, проверил его наличие
  
![Screenshot_2](https://github.com/user-attachments/assets/50c86885-49e4-45d1-ba02-8f3e9a383c62)


* Подключился к k8s проверил шару NFS

![image](https://github.com/user-attachments/assets/de1417f1-1313-4bdd-bf7e-b7052139d510)

* Обратная проверк NFS

![Screenshot_1](https://github.com/user-attachments/assets/90e55dd3-8825-4506-87e5-525d2bb4b836)

![Screenshot_4](https://github.com/user-attachments/assets/61d7feb7-2216-4374-85ed-c39d165fd493)
















---



