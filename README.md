# Дипломный практикум в Cloud: Amazon Web Services"

## Цели:

1. Подготовить облачную инфраструктуру на базе облачного провайдера AWS.
2. Запустить и сконфигурировать Kubernetes кластер.
3. Установить и настроить систему мониторинга.
4. Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. Настроить CI для автоматической сборки и тестирования.
6. Настроить CD для автоматического развёртывания приложения.

## Этапы выполнения:

### Создание облачной инфраструктуры

Для начала необходимо подготовить облачную инфраструктуру в облаке AWS при помощи [Terraform](https://www.terraform.io/).

Особенности выполнения:
- Для выполнения задания следует активировать купон AWS, полученный от координатора курса;
- Бюджет купона ограничен, что следует иметь в виду при проектировании инфраструктуры и использовании ресурсов;
- Следует использовать последнюю стабильную версию [Terraform](https://www.terraform.io/).

Предварительная подготовка к установке и запуску Kubernetes кластера.
1. При помощи IAM создайте service account, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой.
1. Подготовьте [backend](https://www.terraform.io/docs/language/settings/backends/index.html) для Terraform:
   1. Рекомендуемый вариант: [Terraform Cloud](https://app.terraform.io/)
   1. Альтернативный вариант: S3 bucket в созданном AWS аккаунте
1. Настройте [workspaces](https://www.terraform.io/docs/language/state/workspaces.html)
   1. Рекомендуемый вариант: создайте два workspace: *stage* и *prod*. В случае выбора этого варианта все последующие шаги должны учитывать факт существования нескольких workspace.
   1. Альтернативный вариант: используйте один workspace, назвав его *stage*. Пожалуйста, не используйте workspace, создавайемый Terraform-ом по-умолчанию (*default*).
1. Создайте VPC при помощи готового [модуля](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest) от AWS.
1. При помощи Terraform подготовьте как минимум 3 виртуальных машины EC2 для создания Kubernetes-кластера. Выберите тип виртуальной машины самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете, что необходимо сменить тип инстанса, используйте Terraform для внесения изменений.
1. Во время выполнения также понадобится создать [security groups](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) и некоторые другие ресурсы.
1. Следует учитывать, что доступ к EC2 должен быть возможен через Интернет, а не только по локальной сети.
1. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий.
1. В случае использования [Terraform Cloud](https://app.terraform.io/) в качестве [backend](https://www.terraform.io/docs/language/settings/backends/index.html) убедитесь, что применение изменений успешно проходит, используя web-интерфейс Terraform cloud.


Ожидаемые результаты:
1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий.
1. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

### Создание Kubernetes кластера

На этом этапе необходимо создать [Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфрастуктуры.

Это можно сделать двумя способами:
1. Рекомендуемый вариант: самостоятельная установка Kubernetes кластера.
   1. подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)
   1. задеплоить Kubernetes на подготовленные ранее ec2 инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
1. Альтернативый вариант: воспользуйтесь сервисом [EKS](https://aws.amazon.com/eks/) (Amazon Elastic Kubernetes Service)
   1. воспользуйте модулем [terraform-aws-eks](https://github.com/terraform-aws-modules/terraform-aws-eks)
   1. дополните уже существующую Terraform конфигурацию, новый проект создавать не нужно.

Ожидаемый результат:
1. Работоспособный Kubernetes кластер.
2. В файле `~/.kube/config` находятся данные для доступа к кластеру.
3. Команда `kubectl get pods --all-namespaces` отрабатывает без ошибок.

### Создание тестового приложения

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:
1. Рекомендуемый вариант:
   1. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.
   2. Подготовьте Dockerfile для создания образа приложения.
2. Альтернативный вариант:
   1. Используйте любой другой код, главное, чтобы был самостоятельно создан Dockerfile.

Ожидаемый результат:
1. Git репозиторий с тестовым приложением и Dockerfile.
2. Dockerhub регистр с собранным docker image.

### Подготовка Kubernetes конфигурации

Уже должны быть готовы конфигурации для автоматического создания облачной инфраструктуры и поднятия Kubernetes кластера.  
Теперь необходимо подготовить конфигурационные файлы для настройки нашего Kubernetes кластера.

Цель:
1. Задеплоить в кластер [prometheus](https://prometheus.io/), [grafana](https://grafana.com/), [alertmanager](https://github.com/prometheus/alertmanager), [экспортет](https://github.com/prometheus/node_exporter) основных метрик Kubernetes.
1. Задеплоить тестовое приложение, например [nginx](https://www.nginx.com/) сервер отдающий статическую страницу.

Рекомендуемые способ выполнения:
1. Воспользовать пакетом [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus), который уже включает в себя [Kubernetes оператор](https://operatorhub.io/) для [grafana](https://grafana.com/), [prometheus](https://prometheus.io/), [alertmanager](https://github.com/prometheus/alertmanager) и [node_exporter](https://github.com/prometheus/node_exporter). При желании можете собрать все эти приложения отдельно.
1. Для организации конфигурации использовать [qbec](https://qbec.io/), основанный на [jsonnet](https://jsonnet.org/). Обратите внимание на имеющиеся функции для интеграции helm конфигов и [helm charts](https://helm.sh/)
1. Если на первом этапе вы не воспользовались [Terraform Cloud](https://app.terraform.io/), то задеплойте в кластер [atlantis](https://www.runatlantis.io/) для отслеживания изменений инфраструктуры.

Альтернативный вариант:
1. Для организации конфигурации можно использовать [helm charts](https://helm.sh/)

Ожидаемый результат:
1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.
2. Http доступ к web интерфейсу grafana.
3. Дашборды в grafana отображающие состояние Kubernetes кластера.
4. Http доступ к тестовому приложению.

###  Установка и настройка CI/CD

Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:
1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
2. Автоматический деплой нового docker образа.

Можно использовать [teamcity](https://www.jetbrains.com/ru-ru/teamcity/), [jenkins](https://www.jenkins.io/) либо [gitlab ci](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/)

Ожидаемый результат:
1. Интерфейс ci/cd сервиса доступен по http.
2. При любом коммите в репозиторий с тестовым приложением происходит сборка и отправка в регистр Docker образа.
3. При создании тега в репозитории происходит деплой соответсвующего Docker образа.


###  Что необходимо для сдачи задания?

1. Репозиторий с конфигурационными файлами Terraform и готовность продемонстировать создание всех ресурсов с нуля.
```
https://github.com/OlegAnanyev/terraform_io
```
2. Пример pull request с комментариями созданными atlantis'ом или снимки экрана из Terraform Cloud.
![image](https://user-images.githubusercontent.com/32748936/139711300-39b5eda9-eedd-46b5-9b55-69173cf416ba.png)
3. Репозиторий с конфигурацией ansible, если был выбран способ создания Kubernetes кластера при помощи ansible.
```
https://github.com/OlegAnanyev/devops-kubespray/blob/master/inventory/diplom-cluster/inventory.ini
```
![image](https://user-images.githubusercontent.com/32748936/139713545-cb50c86c-adb6-41c1-a693-15c4b8fbaf44.png)

4. Репозиторий с Dockerfile тестового приложения и ссылка на собранный docker image.
```
https://gitlab.com/OlegAnanyev/devops-nginx-app/
https://hub.docker.com/repository/docker/olegananyev/diplom-nginx-app
```
![image](https://user-images.githubusercontent.com/32748936/139713689-f10149a6-7903-4df9-a508-bfc28dae6275.png)

5. Репозиторий с конфигурацией Kubernetes кластера.
```
https://github.com/OlegAnanyev/devops-kubernetes-config
```
![image](https://user-images.githubusercontent.com/32748936/139713263-5b59948a-f5a9-4df1-b4ea-a9074bb65869.png)

6. Ссылка на тестовое приложение и веб интерфейс Grafana с данными доступа.
```
http://3.137.163.70:30001 -- приложение
http://3.137.163.70:30000/d/85a562078cdf77779eaa1add43ccec1e/kubernetes-compute-resources-namespace-pods?orgId=1&refresh=10s&var-datasource=prometheus&var-cluster=&var-namespace=monitoring
```
![image](https://user-images.githubusercontent.com/32748936/139713398-fb2e63a4-fafd-41cb-a4b5-c7703f88d00a.png)

![image](https://user-images.githubusercontent.com/32748936/139713365-898d9533-d584-434e-b718-eb814c51b3ab.png)
