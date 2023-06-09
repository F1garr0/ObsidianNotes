---
share: true  
---
**Внедрение зависимостей (*DI*)** - частный случай подхода инверсии зависимостей (*DIP*) при котором в конструктор класса передаются все объекты от которых данный класс зависит. 

**Что это значит:** Внутри класса не должны создаваться объекты через new, они все должны быть переданы внутри конструктора и присвоены переменным класса.

**Из этого можно сделать вывод:** зависимости класса это объекты, которые необходимы для работы этого класса. При том речь идёт не про внешние библиотеки и не про систему сборки, а именно про создание экземпляров класса.

```java
Class MyClass
	dependency1
	dependency2

	MyClass(dep1, dep2)
		this.dependency1 = dep1
		this.dependency2 = dep2
		
	func doSomething()
		dep1.doSomething()
		dep2.doSomethingElse()
```

**Какие плюсы это несет:** Мы выносим логику по созданию всех зависимых объектов из класса и оставляем там только необходимую логику работы с этими объектами и всё что за этим следует.

**Какие минусы это несет:** Мы выносим логику по созданию всех зависимых объъектов из класса куда-то, где-то их придется создавать и передавать по цепочке из одного класса в другой и чем больше эта цепочка, тем больше нам объектов надо одновременно хранить и передавать глубже по цепочке даже учитывая то, что промежуточные классы могут их не использовать. А еще все эти объекты всегда должны храниться в памяти всегда.

Частично эту проблемы, которые мы породили решают **конетейеры зависомостей.**

**Что делают контейнеры зависимостей:** Они берут на себя ответственность за создание объектов, которые мы помечаем определенным образом, соответственно берут на себя управление зависимостями и их жизненным циклом - передачу их в конструкторы. По сути мы просто добавляем все зависимости в некий контейнер, откуда они потом будут доставаться чтобы быть передаными внутрь конструкторов. Чаще всего (в зависимости от используемых библиотек) большинство этих дейвствий выполняется автоматически.
При этом все вышеописанное в основном применимо к объектам, которые не зависят от пользовательского ввода, т.е. предопределены заранее на момент запуска программы, с теми же данными, которые должны попадать в контейнер в момент получения программой данных извне всё немного сложнее.

Прежде чем перейти к контейнерам, надо понять несколько основных понятий:
1. **Точка входа** - место где мы *объявляем контейнер* (*который чаще всего называется как приложение*) добавляем туда зависимости тем или иным способом. Согласно подходу: только в точке входа мы имеет доступ к контейнеру чтобы достать из него экзмепляр какого-либо класса содержащегося там (При этом все его зависимости будут созданы и переданы по цепочке автоматически если они есть в контейнере)
2. **Сервис** - класс, который мы помечаем как сервис, может быть автоматически помещен в контейнер и для него могут работать все правила по его созданию и передачу в него его зависимостей.
3. **Конфигурация** - класс, который является обёрткой для классов, которые мы не можем изменять чтобы пометить их как сервисы т.к. это встроенны классы языка программирования или внешние библиотеки. 

**_Пример использования_**

```java
// Класс EntryPoint является точкой входа в котором объявляется контейнер
@Application
class EntryPoint
	Application container
	
	EntryPoiny()
		container = new Application()
		// Чаще всего сервисы могут добавляться автоматически и нет 
		// необходимости добавлять их вручную
		container.addDependency(A.class)
		container.addDependency(B.class)
		// Также может поддерживаться добавление не только 
		// по имени класса
		// Но и инстанса уже инициализированного объекта, например
		// container.addInstanse(new B())
	
	func doSomething()
		// Т.к. это точка входа мы можем здесь достать из контейнера
		// объект вручную и вызвать его метод 
		container.getDependency(A.class).doSomething()

```
---
```java 
// Класс A зависит от B
@Service
class A
	B bDependency
	
	// Мы помечаем конструктор который будет использовать контейнер 
	// для использования внедрения зависимостей из него
	@Inject
	A(B bDep)
		this.bDependency = bDep
		
	func doSomething()
		bDependency.doSomething()
```
---
```java
// Класс B ни от чего не зависит, но его можно пометить как сервис, 
// потому что он используется для инъекции зависимости в класс A, так 
// что может автоматом попасть в контейнер явно его туда не передавая 
// в точке входа
@Service
class B
		
	func doSomething()
		...
	
```
---

Важным моментом при попытках использовать контейнеры зависимостей это не начать использовать *Service Locator*, который считается антипаттерном, по крайней мере с точки зрения нашего подхода

**В чем заключается суть Service Locator:** Примером для представления может служить статический класс, который хранит в себе различные сервисы и может вернуть их по запросу.
При этом необходимо зарегестрировать в локаторе эти сервисы. Так например мы можем реализовать два класса выполняющих отправку почты, но через разные сервисы, унаследовав их от одного базового класса и регистрируя только один из них получать из локатора текущую реализацию по имени базового класса или же зарегестрировав оба использовать *Named Bindings* (Именованые связки) чтобы передать например строку в локатор, которая будет идентифицировать нужную реализацию и переключаться между ними динамически. Чем-то это напоминает фабрику и не зря, по сути это она и есть, но с некоторыми доработками.

> Контейнеры зависимостей позволяют вам делать всё то же самое

**Чем же плох Service Locator:** При использовании данного подхода в случае если какой-либо сервис не будет зарегестрирован и он потребуется для одного из вызванных сервисов, то мы не сможем отследить это на этапе написания кода и ошибка возникнет только в runtime.
(К слову говоря некоторые библиотеки зависимостей тоже не всегда имеют нормальной валидации ошибок на этапе написания кода, например в Intelij Idea нет валидации для guice, а проверка Sprig DI-containers есть только в PRO версии Idea). Такой код может быть сложнее сопровождать и мб-что то еще.


Полезные ссылки:
1. [# Service Locator — антипаттерн](https://habr.com/ru/companies/otus/articles/694458/)
2. Тык
3. Тык
4. А нечего тыкать больше, можешь тыкнуть себе в глаз если очень хочется
