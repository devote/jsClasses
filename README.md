﻿Эмуляция классов на JavaScript
==============================

Одно важное отличие от других реализаций описанных во многих статьях, это возможность создавать аксессоры (setter's/getter's). Которые будут работать не только в современных браузерах, но и в долгоживущем IE ниже 9-ой версии. Об этом читайте ниже.

Для начала я опишу как создавать классы нужных нам типов, классы могут иметь обычные публичные свойства, приватные свойства и статические свойства.

Создание классов
----------------

Для создание класса достаточно объявить имя класса и присвоить ему объект.

Пример создания пустого класса:

```javascript
jClass("Empty", {}); // создали пустой класс Empty

alert(Empty); // увидим [class Empty]
```

Как вы уже поняли создание класса не требует огромных затрат на написание кода.

Для создания класса с приватными свойствами достаточно объявить вторым параметром не объект а функцию, в которой будет возвращен объект класса

Пример класса с приватными свойствами:

```javascript
jClass("PrivateProperty", function() {
	// наши приватные переменные/свойства
	var privateProp = "tratata",
		twoPrivateProp = "lalala";

	// возвращаем объект самого класса
	return {
	}
});

// Создадим экземпляр класса
var privateTest = new PrivateProperty();

// пробуем получить приватные свойства
alert(privateTest.privateProp); // увидим undefined
```

Создавать классы можно не только в глобальном контексте но и в любом другом.

Для примера я покажу несколько способов как это делается, вы можете выбрать любой приемлемый для вас способ, не ограничивая себя в чем либо.

Вот способы создания класса в любом удобном контексте:

```javascript
// создание класса например в контексте window
jClass(window, "Global", {});
// создание класса в текущем контексте
var CurrentContext = jClass({});
// создать класс в текущем контексте но при этом он будет
// доступен и в глобальном контексте c именем ClassesContext
var CurrentContext = jClass("ClassesContext", {});
```

На этом с созданием классов собственно и закончим, других способов думаю и не надо.

Работа с классами
-----------------

Теперь я покажу как работать с классами, принцип их работы ничем не отличается например от классов существующих в PHP. "Не может такого быть!" спросите вы, да, конечно не может. Есть тут свои тонкости, конечно же нет возможности создания интерфейсов, абстракции и прочих полноценных прелестей ООП. Но используя существующие возможности, программист смело может использовать знания классового программирования, поведение классов предсказуемо, контекст не бегает туда/сюда, а имеет тот самый экземпляр порожденного класса.

Для начала давайте мы создадим простой класс, который будет выводит информацию в окно браузера

```javascript
jClass("Debug", function() {

	// приватные переменные
	var
		// здесь будет хранится ссылка на тег BODY нашего документа
		body = null,
		// здесь будем складывать элементы с текстом до тех пор пока body не определен
		cache = [];

	return {

		// конструктор класса, будет вызван во время создания экземпляра класса
		// параметр callback нам понадобится позже, об этом читайте далее
		constructor: function(callback) {

			// определим какой метод нам использовать что бы повесить событие
			var listener = window.addEventListener ? ["addEventListener", ""] : 
									["attachEvent", "on"];

			// перед тем как вешать событие мы проверим,
			// возможно наш документ давно загружен
			if (document.readyState === "complete") {

				// если документ и правда был загружен, в этом случаем назначим
				// нашей приватной переменной ссылку на объект BODY
				body = document.body;

				// выполним функцию переданную первым параметром в конструкторе
				// если она была передана
				if (callback && typeof callback === "function") {
					callback.call(this);
				}

				// затем просто выйдем из конструктора
				return;
			}

			// сохраним текущий контекст что бы передать его callback'у
			var self = this;

			// при создании класса, повесим обработчик на событие загрузки документа
			window[listener[0]](listener[1] + "load", function() {

				// после того как документ загрузился, можно смело назначить нашей
				// приватной переменной ссылку на объект BODY
				body = document.body;

				// отобразим все что накопилось у нас в кеше, и сбросим его.
				for(var i = 0; i < cache.length; i++) {
					body.appendChild(cache[i]);
					cache[i] = null;
				}

				// очистим кеш
				cache.length = 0;

				// выполним функцию переданную первым параметром в конструкторе
				// если она была передана
				if (callback && typeof callback === "function") {
					callback.call(self);
				}

			}, false);
		},

		// наш метод с помощью которого мы будем выводить сообщения на нашу страницу
		write: function() {

				// создадим DIV в который положим наш текст
			var div = document.createElement("DIV"),

				// проверим что хотят вставить в окно вывода, если последний
				// параметр нашей функции имеет болевое значение TRUE значит
				// мы хотим просто распечатать текст не конвертируя теги в DOM
				// элементы.
				isPlainText = arguments.length ? 
						arguments[arguments.length - 1] === true : false,

				// переведем наши аргументы в массив
				dataArray = Array.prototype.slice.call(arguments);

			// если хотим распечатать текст не переводя HTML в структуру DOM объектов
			if (isPlainText && dataArray.pop()) {
				// последний аргумент как вы видите мы удалили, который информирует 
				// нас о том что мы не желаем переводить текст в структуру DOM
				div.appendChild(
					document.createTextNode(dataArray.join(", "))
				);
			} else {
				// здесь теги в тексте будут обработаны в DOM элементы.
				div.innerHTML = dataArray.join(", ");
			}

			// здесь мы выводим или отложим данные до возможности их вывести
			if (body) {
				// выводим в браузер сразу так как элемент BODY определен
				body.appendChild(div);
			} else {
				// положим пока что в наш кеш до определения элемента BODY
				cache[cache.length] = div;
			}
		}
	}
});
```

Вот мы с вами создали наш полноценный класс, в нем мы применили подход с приватными свойствами, этот класс особо хитрого ничего не делает, а просто выводит текст в окно браузера, при этом дожидается полной загрузки документа что бы не произошла ошибка.

Например мы можем теперь создать экземляр этого класса и распечатать наше первое сообщение.

```javascript
var debug = new Debug();

debug.write("Наш класс <var>Debug</var> отлично работает!");
```

"Ничего особенного!" Скажете вы, обычное ненужное создание классов иным способом. Да, отвечу я вам, особо ничего заумного тут нет, но самые вкусности еще не были рассказаны.

Наследование
------------

Давайте теперь создадим наш второй класс, который будет наследовать свойства нашего класса Debug. Наш новый класс будет обычной кнопкой, которая будет менять цвет при клике на нее.

```javascript
// Создадим класс Button и расширим его от класса Debug
jClass("Button extends Debug", function() {

		// статус мыши
	var mouseState = 0,
		// наша будущая кнопка, обычный DOM элемент
		button = null;

	// приватная функция
	function switchState(type) {

		// тип изменения статуса мыши
		if (type === 1) {

			mouseState++;

			// здесь мы меняем стиль кнопки в случае если мышь зажата на кнопке
			button.style.backgroundColor = "green";

			return;

		} else if (type === 2) {

			mouseState--;

		} else {

			mouseState = 0;
		}

		// стиль кнопки по умолчанию
		button.style.backgroundColor = "red";
	}

	return {

		// наш конструктор для кнопки
		Button: function() {

			// создадим элемент для кнопки
			button = document.createElement("SPAN");

			// зададим свойства кнопки по умолчанию
			button.style.border = "1px solid blue";
			button.style.color = "white";
			button.style.textAlign = "center";
			button.style.backgroundColor = "red";
			button.style.borderRadius = "5px";
			button.style.padding = "4px";
			button.style.cursor = "default";

			// начальный текст для нашей кнопки
			button.innerHTML = "Наша первая кнопка";

			// вызываем родительский конструктор то-есть конструктор класса Debug
			// обратите внимание на то что здесь я передаю первым параметром родителю
			// нашу функцию, которую класс Debug вызовет когда документ будет загружен
			this.parent.constructor(function() {

				// сохраним ссылку на текущий контекст
				var self = this;

				// добавим нашу кнопку в структуру DOM
				document.body.appendChild(button);

				// запретим выделение текста в IE при двойном клике на кнопку
				button.onselectstart = function() {
					return false;
				}

				// обработаем событие нажатия мыши
				button.onmousedown = function(e) {

					// получаем объект события мыши
					var e = e || window.event;

					// меняем статус кнопки, тоесть ее стиль
					switchState(1);

					// отменяем действие по умолчанию что бы текст
					// не выделялся в других браузерах.
					if (e.preventDefault) {
						e.preventDefault();
					} else {
						e.returnValue = false;
					}
				}

				// обработаем событие отпуска клавиши мыши
				button.onmouseup = function() {

					// меняем статус кнопки, то-есть стиль
					switchState(2);

					// если мышь нажали и отпустили на нашей кнопке
					if (mouseState === 0) {

						// запускаем обработчик действия после успешного
						// нажатия на нашу кнопку
						self.click();
					}
				}

				// обработаем уход мыши с нашей кнопки
				button.onmouseout = function() {

					// если статус мыши не нулевой, то прибавим статус
					if (mouseState && mouseState++) {

						// и восстановим стиль кнопки по умолчанию
						switchState(2);

					}
				}

				// обработаем событие прихода мыши на нашу кнопку
				button.onmouseover = function() {

					// если статус мыши не нулевой, убавляем его
					if (mouseState && mouseState--) {

						// и ставим стиль нажатой кнопки
						switchState(1);

					}
				}

				// перегрузим событие документа на поднятие клавиши мыши вне кнопки
				var handler = window.document.onmouseup;
				window.document.onmouseup = function(e) {

					// сбрасываем статус и ставим стиль по умолчанию
					switchState();

					// запустим старый обработчик если таков был
					if (handler) {
						handler.call(window, e);
					}
				}
			});
		},

		// глобальная функция которая возвращает DOM элемент нашей кнопки
		node: function() {
			return button;
		},

		// по сути абстрактная функция, которая вызывается при клике на кнопку
		// в нашем случае объявлять ее в дочернем классе не обязательно.
		click: function() { }

	}
});
```

И так мы с вами создали новый класс Button который наследует свойства класса Debug как вы уже заметили наследование делается методом добавления слова extends за которым идет имя класса с которого хотим унаследовать свойства.

Это не единственный способ наследования, это можно делать и другим способом, например:

```javascript
var Child = jClass(Debug, {});
```

Как мы видем класс Child стал наследником класса Debug


А теперь давайте опробуем нашу написанную кнопку

```javascript
	// Создадим экземпляр кнопки
	var button = new Button();

	// повесим событие на успешное нажатие по кнопке
	button.click = function() {
		// метод write мы унаследовали от класса Debug
		this.write("Вы нажали и отпустили кнопку мыши на нашей первой кнопке");
	}
```

Как вы видите у нас получилась полноценно работающая кнопка, может она и не красива, но это уже мелочи. Всегда можно изменить стиль, имя кнопки. Это лишь небольшой пример того как можно реализовывать проекты на классах.

Setter'ы/Getter'ы
-----------------

А теперь давайте перейдем на самые вкусности, которых так не хватает из-за ограничений, как вам известно Internet Explorer ниже 9-ой версии не позволяет нормально работать с геттерами/сеттерами, это огромный минус в разработке проектов. Да конечно же возможности языка от этого не уменьшаются, да и возможность написания программ тоже. Но я все же постарался реализовать их в текущих классах, можно скорее назвать это некими "magic getter/setter", тут не требуется вешать для каждого свойства всякие defineProperty а достаточно просто указать какие свойства должны иметь возможность перехвата.

Давайте мы с вами расширим наш класс кнопки и создадим некий супер класс который даст возможность менять текст кнопки посредством геттеров/сеттеров. В этом классе мы не будем использовать ни конструкторы ни приватных методов, а лишь создадим свойство которое будет перехватываться магическим геттером/сеттером

```javascript
jClass("SuperButton extends Button", {

	text: {
		set: function(value) {

			// пишем сообщение в браузер о том что был вызван сеттер для свойства
			this.write("Вызван SETTER для свойства <var>text</var> со значением <var>" + value + "</var>");

			// меняем текст кнопки на новое значение
			this.node().innerHTML = value;
		},

		get: function() {

			// пишем сообщение в браузер о том что был вызван геттер для свойства
			this.write("Вызван GETTER для свойства <var>text</var>");

			// возвращаем текущее значение нашего свойства
			return this.node().innerHTML;
		}
	},

	// другой способ работать с сеттерами/геттерами

	// сеттер для свойства name
	"set name": function(value) {
	},

	// геттер для свойства name
	"get name": function() {
	}
});
```

Вот мы с вами создали супер класс для кнопки, который просто дает возможность менять текст кнопки обычным назначением свойству text, нужного нам значения, это конечно не все возможности геттеров/сеттеров вы можете использовать их в любых условиях, с любым типом данных и т.д.

А теперь давайте посмотрим на то что у нас получилось:

```javascript
// создадим экземпляр нашей супер кнопки
var superButton = new SuperButton();

// испробуем геттер, просто получим текущее значение имени кнопки
// обратите внимание на сообщение в окне браузера
superButton.write("Текущее имя нашей супер кнопки: <var>" + superButton.text + "</var>");

// а теперь заменим текст кнопки и мы снова увидим сообщение в окне браузера
// информирующее нас о том что был вызван сеттер
superButton.text = "Наша вторая супер кнопка";
```

Все описанные примеры вы можете увидеть в действии вот <a href="http://spb-piksel.ru/classes/">по этой ссылке</a>.

Статические свойства
--------------------

Статические свойства добавлять очень просто, покажу на примерах создания классов:

```javascript
// создание класса например в контексте window
jClass(window, "Global", {
	staticProperty: 1
}, {
	dinamicProperty: function() {
		alert(Global.staticProperty);
	}
});
// создание класса в текущем контексте
var CurrentContext = jClass({
	staticProperty: 1
}, {
	dinamicProperty: function() {
		alert(CurrentContext.staticProperty);
	}
});
// создать класс в текущем контексте но при этом он будет
// доступен и в глобальном контексте c именем ClassesContext
var CurrentContext = jClass("ClassesContext", {
	staticProperty: 1
}, {
	dinamicProperty: function() {
		alert(ClassesContext.staticProperty);
	}
});
```

Напоследок хочу обратить внимание на то, что при обращении к родительским методам вам не нужно указывать явно контекст. Я думаю вы заметили что я вызываю конструктор класса Debug из нашего класса кнопки, обычным вызовом this.parent.constructor() при этом класс debug будет уже иметь контекст последнего потомка, то-есть инициатора классов. Вам не нужно вызывать родительские методы через всем известные call, apply и т.д. Достаточно просто вызвать this.parent.parentMethod( args ); и родственник будет работать с контекстом потомка.

Так же добавлю что создание дополнительных геттеров/сеттеров в уже существующий экземпляр класса добавить конечно же не получиться в таком браузере как ИЕ ниже 9-ой версии. Поэтому есть небольшие ограничения по динамике, так же при использовании геттеров/сеттеров в классах потомка и/или его наследников нельзя будет добавить динамически каких либо свойств. Но это ограничение распространяется лишь на ИЕ ниже 9-ой версии и в случае если присутствует хоть один геттер/сеттер.

Допустим мы хотим создать дополнительное свойство у экземпляра класса SuperButton или его потомков, которых пока у нас нет. Но в будущем они в любом случае у вас будут. То попытка создания приведет к ошибке в ИЕ ниже 9-ой версии, потому как объект с сеттерами/геттерами порожден через VBScript а там как вам известно есть ограничение которое не позволяет объявить дополнительное свойство если оно явно не указано, но конструктор классов создает в каждом экземпляре пустой объект shared, в котором вы можете создавать любые угодные свойства динамически.

Но у экземпляра класса Button мы спокойно можем создать дополнительные свойства, так как у нас не используются сеттеры/геттеры у этого класса и его потомков.


Так же хочу добавить что нативный instanceof не будет реагировать корректно на эти классы поэтому для этих случаев я добавил метод jClass.instanceOf для проверки принадлежности экземпляра к нужному нам классу в нашем случаем вызов:

```javascript
alert(jClass.instanceOf(superButton, Debug)); // отобразит TRUE
```
