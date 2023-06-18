# ДЗ 1


## Cassandra

По умолчанию Cassnadra позиционируется как БД класса AP

Цитата из [официальной документации](https://cassandra.apache.org/doc/latest/cassandra/architecture/guarantees.html):

*High availability is a priority in web based applications and to this objective Cassandra chooses **Availability** and **Partition Tolerance** from the CAP guarantees, compromising on data Consistency to some extent.*

Однако, согласно [другой части официальноq документации](https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html) можно настроить уровень консистентности, среди которых в том числе есть ALL, тогда Cassandra может рабоать в режиме CP, потому что:

*Write operations are always sent to all replicas, regardless of consistency level. The consistency level simply controls how many responses the coordinator waits for before responding to the client.*

То есть пока не будет ответа ото всех узлов, координтор не ответит клиенту, что показывает, что в случае проблем с доступностью теряется Availability, зато появляется Consistency

## MongoDB

Согласно [официальной документации](https://www.mongodb.com/docs/manual/replication/#replication-in-mongodb):

*By default, clients read from the primary [1]; however, clients can specify a read preference to send read operations to secondaries.*

Это означает, что по умолчанию в MongoDB обеспечивается Consistency, так как операции чтения и записи будут всегда проводиться с одного и того же узла. Однако это говорит о том, что Availability не будет обеспечиваться, так как это один узел

Тем не менее репликация данных в MongoDB асинхронная путем чтения secondary узлами oplog (operation log) журнала primary узла. Таким образом у MongoDB имеет место Partition tolerance

Можно сделать вывод, что MongoDB по умолчанию относится к классу CP

Но, тем не менее есть возможность дать клиентам доступ на чтение seconday узлов, что повышает Availability, но репликация все равно остается асинхронной, из-за чего MongoDB теряет Consistency, что также подтверждает официальная документация:

*Asynchronous replication to secondaries means that reads from secondaries may return data that does not reflect the state of the data on the primary.*

Таким образом, если дать клиентам читать secondary узлы, то MongoDB становится AP класса


## MSSQL

Так как MSSQL является реляиционной БД, то Consistency и Availability стоят на первом месте и, конечно же, никакой устойчивости к разделению у MSSQL нет

Таким образом MSSQL относится к CA классу

Однако, если копнуть глубже, то по сути СА - это недостижимый класс, так как в CAP теореме все крутится на самом деле вокруг P. Основаня проблема - это нестабильность сети. Как только разные узлы кластера перестают "видеть" друг-друг друга теряется либо C из-за невозможности произвести репликацию, либо А из-за того, что невозможно сделать запись, если без подтверждения от другого узла запись данных невозможно. 

СА может быть достижим, в случае если под Availability мы понимаем **ТОЛЬКО** чтение, так как можно не записывать данные, но все равно иметь возможность с любого узла выдавать одну и ту же информацию, даже если они не "видят" друг-друга.