## Дипломная работа по профессии «Системный администратор» - Андрей Обухов
- Содержание
- Задача
- Инфраструктура
- Сайт
- Мониторинг
- Логи
- Сеть
- Резервное копирование
- Дополнительно
- Выполнение работы
- Критерии сдачи
- Как правильно задавать вопросы дипломному руководителю

#### Задача
________________________________

Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в Yandex Cloud и отвечать минимальным стандартам безопасности: запрещается выкладывать токен от облака в git. Используйте инструкцию.

###### Перед началом работы над дипломным заданием изучите Инструкция по экономии облачных ресурсов.

#### Инфраструктура
Для развёртки инфраструктуры используйте Terraform и Ansible.

Не используйте для ansible inventory ip-адреса! Вместо этого используйте fqdn имена виртуальных машин в зоне ".ru-central1.internal". Пример: example.ru-central1.internal

Важно: используйте по-возможности минимальные конфигурации ВМ:2 ядра 20% Intel ice lake, 2-4Гб памяти, 10hdd, прерываемая.

Так как прерываемая ВМ проработает не больше 24ч, перед сдачей работы на проверку дипломному руководителю сделайте ваши ВМ постоянно работающими.

Ознакомьтесь со всеми пунктами из этой секции, не беритесь сразу выполнять задание, не дочитав до конца. Пункты взаимосвязаны и могут влиять друг на друга.

#### Сайт
Создайте две ВМ в разных зонах, установите на них сервер nginx, если его там нет. ОС и содержимое ВМ должно быть идентичным, это будут наши веб-сервера.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/97b75965-8c8d-42c0-8c5e-31ec0271b5c5)


Используйте набор статичных файлов для сайта. Можно переиспользовать сайт из домашнего задания.

Создайте Target Group, включите в неё две созданных ВМ.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/4b82d3ea-22fc-4369-99db-7967354887a7)


Создайте Backend Group, настройте backends на target group, ранее созданную. Настройте healthcheck на корень (/) и порт 80, протокол HTTP.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/a7ea2327-cf50-420b-bd4a-7d7d18999861)


Создайте HTTP router. Путь укажите — /, backend group — созданную ранее.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/fe63ecdc-f3ca-4db4-855f-b1d18557f66c)


Создайте Application load balancer для распределения трафика на веб-сервера, созданные ранее. Укажите HTTP router, созданный ранее, задайте listener тип auto, порт 80.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/a3d9021f-fd73-450e-bb3d-e5af239b64a6)


Протестируйте сайт curl -v <публичный IP балансера>:80

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/035a7ecc-2673-4faa-89b4-8fed3a54d9c9)
   
#### Мониторинг
Создайте ВМ, разверните на ней Zabbix. На каждую ВМ установите Zabbix Agent, настройте агенты на отправление метрик в Zabbix.

Zabbix сервер на виртуальной машине развернут с использованием ansibe [ansible/zabbix_server.yaml](https://github.com/vioas/DIPLOM_OBUKHOV.A/blob/main/Ansible/zabbix_server.yaml) 

Веб-консоль zabbix доступна из сети интернет по адресу (http://10.8.0.22)

Данные для авторизации:

Логин: Admin

Пароль: zabbix

На вирутальные машины с вебсайтом с использованием ansibe [ansible/zabbix_agent.ayml](https://github.com/vioas/DIPLOM_OBUKHOV.A/blob/main/Ansible/zabbix_agent.yaml) установлены zabbix агенты.

Настроены дашборды.


![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/6d680307-270b-40a1-b65f-0883b92bfa63)


Настройте дешборды с отображением метрик, минимальный набор — по принципу USE (Utilization, Saturation, Errors) для CPU, RAM, диски, сеть, http запросов к веб-серверам. Добавьте необходимые tresholds на соответствующие графики.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/3dc2dce6-7ef4-49a1-9a0e-e82238efc860)


#### Логи
Cоздайте ВМ, разверните на ней Elasticsearch. Установите filebeat в ВМ к веб-серверам, настройте на отправку access.log, error.log nginx в Elasticsearch.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/e4b1fa23-1ac5-4557-a77e-04260a21df25)


Создайте ВМ, разверните на ней Kibana, сконфигурируйте соединение с Elasticsearch.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/d77233ce-62b0-4252-8e09-accd10a5bbec)


#### Сеть
Разверните один VPC. Сервера web, Elasticsearch поместите в приватные подсети. Сервера Zabbix, Kibana, application load balancer определите в публичную подсеть.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/7c5024e3-8ec8-40b3-a358-045c23570a67)


Настройте Security Groups соответствующих сервисов на входящий трафик только к нужным портам.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/c78352df-6d8a-4208-a9a3-fabbd709ee7a)

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/2119e89d-6ce2-4389-af95-4d81545e0bc5)


Настройте ВМ с публичным адресом, в которой будет открыт только один порт — ssh. Эта вм будет реализовывать концепцию bastion host . Синоним "bastion host" - "Jump host". Подключение ansible к серверам web и Elasticsearch через данный bastion host можно сделать с помощью ProxyCommand . Допускается установка и запуск ansible непосредственно на bastion host.(Этот вариант легче в настройке)

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/1f384834-d45f-4b2c-9f7b-5888fd359ab6)


#### Резервное копирование
Создайте snapshot дисков всех ВМ. Ограничьте время жизни snaphot в неделю. Сами snaphot настройте на ежедневное копирование.

![image](https://github.com/vioas/DIPLOM_OBUKHOV.A/assets/142601752/068de8cc-7db7-469a-984c-1f3bd3a4f18e)


#### Дополнительно
Не входит в минимальные требования.

1. Для Zabbix можно реализовать разделение компонент - frontend, server, database. Frontend отдельной ВМ поместите в публичную подсеть, назначте публичный IP. Server поместите в приватную подсеть, настройте security group на разрешение трафика между frontend и server. Для Database используйте Yandex Managed Service for PostgreSQL. Разверните кластер из двух нод с автоматическим failover.
2. Вместо конкретных ВМ, которые входят в target group, можно создать Instance Group, для которой настройте следующие правила автоматического горизонтального масштабирования: минимальное количество ВМ на зону — 1, максимальный размер группы — 3.
3. В Elasticsearch добавьте мониторинг логов самого себя, Kibana, Zabbix, через filebeat. Можно использовать logstash тоже.
4. Воспользуйтесь Yandex Certificate Manager, выпустите сертификат для сайта, если есть доменное имя. Перенастройте работу балансера на HTTPS, при этом нацелен он будет на HTTP веб-серверов.

#### Выполнение работы
На этом этапе вы непосредственно выполняете работу. При этом вы можете консультироваться с руководителем по поводу вопросов, требующих уточнения.

⚠️ В случае недоступности ресурсов Elastic для скачивания рекомендуется разворачивать сервисы с помощью docker контейнеров, основанных на официальных образах.

Важно: Ещё можно задавать вопросы по поводу того, как реализовать ту или иную функциональность. И руководитель определяет, правильно вы её реализовали или нет. Любые вопросы, которые не освещены в этом документе, стоит уточнять у руководителя. Если его требования и указания расходятся с указанными в этом документе, то приоритетны требования и указания руководителя.

#### Критерии сдачи
1. Инфраструктура отвечает минимальным требованиям, описанным в Задаче.
2. Предоставлен доступ ко всем ресурсам, у которых предполагается веб-страница (сайт, Kibana, Zabbix).
3. Для ресурсов, к которым предоставить доступ проблематично, предоставлены скриншоты, команды, stdout, stderr, подтверждающие работу ресурса.
4. Работа оформлена в отдельном репозитории в GitHub или в Google Docs, разрешён доступ по ссылке.
5. Код размещён в репозитории в GitHub.
6. Работа оформлена так, чтобы были понятны ваши решения и компромиссы.
7. Если использованы дополнительные репозитории, доступ к ним открыт.

#### Как правильно задавать вопросы дипломному руководителю
_________________________________________________

Что поможет решить большинство частых проблем:

1. Попробовать найти ответ сначала самостоятельно в интернете или в материалах курса и только после этого спрашивать у дипломного руководителя. Навык поиска ответов пригодится вам в профессиональной деятельности.
2. Если вопросов больше одного, присылайте их в виде нумерованного списка. Так дипломному руководителю будет проще отвечать на каждый из них.
3. При необходимости прикрепите к вопросу скриншоты и стрелочкой покажите, где не получается. Программу для этого можно скачать здесь.

Что может стать источником проблем:

1. Вопросы вида «Ничего не работает. Не запускается. Всё сломалось». Дипломный руководитель не сможет ответить на такой вопрос без дополнительных уточнений. Цените своё время и время других.
2. Откладывание выполнения дипломной работы на последний момент.
3. Ожидание моментального ответа на свой вопрос. Дипломные руководители — работающие инженеры, которые занимаются, кроме преподавания, своими проектами. Их время ограничено, поэтому постарайтесь задавать правильные вопросы, чтобы получать быстрые ответы :)
