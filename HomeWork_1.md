1. **Создание проекта**

	- Создайте виртуальную машину в Яндекс.Облаке // любых других облаках // у себя на компьютере с параметрами: 1-2 ядра, 2-4 ГБ памяти, операционная система Ubuntu.. Назовите проект в формате bananaflow-<yyyymmdd>, где yyyymmdd — год, месяц и день вашего рождения (например, bananaflow-19900101).
		Создал-"bananaflow-19940315":
			ОС -> Ubuntu 24.04
   			SSD: 10 gb
   			RAM: 2gb
   			CPU: 2

2. **Настройка SSH-доступа**

	- Добавьте свой SSH-ключ к виртуальной машине.
		Добавил свой SSH-ключ во время создания машины

	- Подключитесь к машине через SSH (первая сессия). Не забудьте использовать ssh-add  для добавления ключа в агент.
		Так как добавил свой SSH-ключ без passphrase, использовать ssh-add не пришлось

3. **Установка PostgreSQL**

	- Установите PostgreSQL из пакетов с помощью команды apt install.
		Устнановил postgres 
			```console
			sudo apt install postgresql
			```
4. **Подключение к PostgreSQL**

	- Откройте вторую SSH-сессию и подключитесь к PostgreSQL через psql от имени пользователя postgres.
		Открыл новую сессию в новом терминале 
		```console
		ssh farrukh@153.51.170.11
		```
		и с помощью команды 
		```console
		sudo -u postgres psql
		```
		подключился к БД postgres

5. **Работа с транзакциями**

	- Выключите автоматический коммит (auto commit) в обеих сессиях.
		```sql
		\set AUTOCOMMIT off
		```
	- В первой сессии создайте таблицу shipments (перевозки) и добавьте в неё данные:
		```sql
		create table shipments(id serial, product_name text, quantity int, destination text);
		insert into shipments(product_name, quantity, destination) values('bananas', 1000, 'Europe');
		insert into shipments(product_name, quantity, destination) values('coffee', 500, 'USA');
		commit;
		```

6. **Изучение уровней изоляции**
	
	- Проверьте текущий уровень изоляции с помощью команды:
		```sql
		show transaction isolation level;
		```
		Ответ: read committed
	
	- Начните новую транзакцию в обеих сессиях с уровнем изоляции по умолчанию.
		**Ввел:**
		```sql	
		BEGIN;
		```
	- В первой сессии добавьте новую запись:
		```sql
		insert into shipments(product_name, quantity, destination) values('sugar', 300, 'Asia');
		```
		**Выполнил**
	
	- Во второй сессии выполните:
		```sql
		select * from shipments;
		```
	
	- Видите ли вы новую запись? 
		**Нет не вижу, потому что при уровне изоляции read committed, другие сессии будут видеть только то что commited.**

	- Завершите первую транзакцию с помощью commit; и снова выполните select \* from shipments во второй сессии. Видите ли вы новую запись теперь?
		**Вижу, потому что данные закоммитились и они будут отображаться у всех у кого уровень транзакции read committed.**

7. **Эксперименты с уровнем изоляции Repeatable Read**

	- Начните новые транзакции в обеих сессиях с уровнем изоляции repeatable read:
		```sql
		set transaction isolation level repeatable read;
		```
		**Начал**

	- В первой сессии добавьте новую запись:
		```sql
		insert into shipments(product_name, quantity, destination) values('bananas', 2000, 'Africa');
		```
		**Выполнил**

	- Во второй сессии выполните:
		```sql
		select * from shipments;
		```
		**Выполнил**

	- Видите ли вы новую запись?
		**Нет. Не в какой другой сесси данные не будут видны потому что не было комита**

	- Завершите первую транзакцию с помощью commit; и снова выполните select \* from shipments во второй сессии. Видите ли вы новую запись теперь? Объясните.
		**Нет. Потому что выставил уровень изоляции repeatable read и буду видеть только те данные которые были актуальны до выставления уровня изоляции.**

	- Завершите вторую транзакцию и снова выполните select \* from shipments. Видите ли вы новую запись? Объясните.
		**Вижу, потому что уровень изоляции теперь read committed**

