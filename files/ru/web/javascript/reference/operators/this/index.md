---
title: this
slug: Web/JavaScript/Reference/Operators/this
---

{{jsSidebar("Operators")}}

Поведение ключевого слова `this` в JavaScript несколько отличается по сравнению с остальными языками. Имеются также различия при использовании `this` в [строгом](/ru/docs/Web/JavaScript/Reference/Strict_mode) и нестрогом режиме.

В большинстве случаев значение `this` определяется тем, каким образом вызвана функция. Значение `this` не может быть установлено путём присваивания во время исполнения кода и может иметь разное значение при каждом вызове функции. В ES5 представлен метод {{jsxref('Function.prototype.bind()', 'bind()')}}, который используется для {{jsxref('Operators/this','привязки значения ключевого слова this независимо от того, как вызвана функция','Метод_bind')}}. Также в ES2015 представлены {{jsxref('Functions/Arrow_functions', 'стрелочные функции')}}, которые не создают собственные привязки к `this` (они сохраняют значение `this` лексического окружения, в котором были созданы).

{{EmbedInteractiveExample("pages/js/expressions-this.html")}}

## Синтаксис

```
this
```

### Значение

Свойство контекста выполнения кода (global, function или eval), которое в нестрогом режиме всегда является ссылкой на объект, а в строгом режиме может иметь любое значение.

## Global контекст

В глобальном контексте выполнения (за пределами каких-либо функций) `this` ссылается на глобальный объект вне зависимости от режима (строгий или нестрогий).

```js
// В браузерах, объект window также является объектом global:
console.log(this === window); // true

a = 37;
console.log(window.a); // 37

this.b = "MDN";
console.log(window.b); // "MDN"
console.log(b); // "MDN"
```

> **Примечание:** вы всегда можете легко получить объект global, используя глобальное свойство {{jsxref ("globalThis")}}, независимо от текущего контекста, в котором выполняется ваш код.

## Function контекст

В пределах функции значение `this` зависит от того, каким образом вызвана функция.

### Простой вызов

Поскольку следующий код не в {{jsxref('Strict_mode', 'строгом режиме')}}, и значение `this` не устанавливается вызовом, по умолчанию будет использоваться объект global, которым в браузере является `{{domxref('window')}}`.

```js
function f1() {
  return this;
}

// В браузере:
f1() === window; // window - глобальный объект в браузере

// В Node:
f1() === global; // global - глобальный объект в Node
```

В строгом режиме, если значение `this` не установлено в контексте выполнения, оно остаётся `undefined`, как показано в следующем примере:

```js
function f2() {
  "use strict"; // см. strict mode
  return this;
}

f2() === undefined; // true
```

> **Примечание:** Во втором примере `this` должно иметь значение `{{jsxref("undefined")}}`, потому что функция `f2` была вызвана напрямую, а не как метод или свойство объекта (например, `window.f2()`). Реализация этой особенности не поддерживалась в некоторых браузерах, когда они впервые начали поддерживать {{jsxref('Strict_mode', 'строгий режим')}}. В результате они некорректно возвращали объект `window`.

Для того, чтобы при вызове функции установить `this` в определённое значение, используйте {{jsxref('Function.prototype.call()', 'call()')}} или {{jsxref('Function.prototype.apply()', 'apply()')}}, как в следующих примерах.

**Пример 1**

```js
// В качестве первого аргумента методов call или apply может быть передан объект,
// на который будет указывать this.
var obj = { a: "Custom" };

// Это свойство принадлежит глобальному объекту
var a = "Global";

function whatsThis() {
  return this.a; //значение this зависит от контекста вызова функции
}

whatsThis(); // 'Global'
whatsThis.call(obj); // 'Custom'
whatsThis.apply(obj); // 'Custom'
```

**Пример 2**

```js
function add(c, d) {
  return this.a + this.b + c + d;
}

var o = { a: 1, b: 3 };

// Первый параметр - это объект для использования в качестве
// 'this', последующие параметры передаются как
// аргументы функции call
add.call(o, 5, 7); // 16

// Первый параметр - это объект для использования в качестве
// 'this', второй - массив, чьи члены используются
// в качестве аргументов функции call
add.apply(o, [10, 20]); // 34
```

Обратите внимание, что в нестрогом режиме, если значение, переданное в `call` или `apply как` `this`, не является объектом, будет сделана попытка преобразовать его в объект с помощью внутренней операции `ToObject`. Таким образом, если переданное значение является примитивом, таким как `7` или `'foo'`, оно будет преобразовано в `Object` с использованием связанного конструктора, так что примитивное число `7` будет преобразовано в объект так, как будто с помощью `new Number(7)`, а строка `'foo'` - как будто с помощью `new String('foo')`, например

```js
function bar() {
  console.log(Object.prototype.toString.call(this));
}

bar.call(7); // [object Number]
bar.call("foo"); // [object String]
```

### Метод `bind`

ECMAScript 5 представил {{jsxref("Function.prototype.bind()")}}. Вызов `f.bind(someObject)` создаёт новую функцию с тем же телом и областью действия, что и `f`, но там, где в исходной функции используется `this`, в новой функции оно постоянно будет связано с первым аргументом `bind`, независимо от того, как функция используется.

```js
function f() {
  return this.a;
}

var g = f.bind({ a: "azerty" });
console.log(g()); // azerty

var h = g.bind({ a: "yoo" }); // bind only works once!
console.log(h()); // azerty

var o = { a: 37, f: f, g: g, h: h };
console.log(o.a, o.f(), o.g(), o.h()); // 37,37, azerty, azerty
```

### Стрелочные функции

В {{jsxref('Functions/Arrow_functions', 'стрелочных функциях')}}, `this` привязан к окружению, в котором была создана функция. В глобальной области видимости `this` будет указывать на глобальный объект.

```js
var globalObject = this;
var foo = () => this;
console.log(foo() === globalObject); // true
```

> **Примечание:** Note: если аргумент this передаётся в call, bind или apply при вызове стрелочной функции, он будет проигнорирован. Вы всё ещё можете добавить аргументы к вызову, но первый аргумент (thisArg) должен быть установлен в null.

Неважно, как стрелочная функция `foo()` будет вызвана, её значение this будет указывать на глобальный объект. `this` будет сохранять своё значение, даже если функция `foo()` будет вызвана как метод объекта (что в обычных функциях связывает `this` с объектом вызова) или с использованием методов `call`, `apply` или `bind`:

```js
// Вызов функции как метода объекта
var obj = { foo: foo };
console.log(obj.foo() === globalObject); // true

// Попытка установить this с помощью call
console.log(foo.call(obj) === globalObject); // true

// Попытка установить this с помощью bind
foo = foo.bind(obj);
console.log(foo() === globalObject); // true
```

Несмотря ни на что, `this` стрелочной функции `foo()` имеет то же значение, что и при создании функции (глобальный объект в примере выше). То же самое касается стрелочных функций, созданных внутри других функций: их `this` будет привязан к окружению.

```js
// Создаём объект obj, содержащий метод bar, который возвращает функцию,
// которая возвращает свой this. Возвращаемая функция создана
// как стрелочная функция, таким образом её this постоянно замкнут
// на this функции, в которой она создана. Значение bar может быть установлено
// в вызове, который, в свою очередь, устанавливает значение возвращаемой функции.
var obj = {
  bar: function () {
    var x = () => this;
    return x;
  },
};

// Вызываем bar как метод объекта obj, устанавливая его this на obj
// Присваиваем ссылку возвращаемой функции переменной fn
var fn = obj.bar();

// Вызываем fn без установки this, что в обычных функциях указывало бы
// на глобальный объект или undefined в строгом режиме.
console.log(fn() === obj); // true

// Но будьте осторожны, если вы ссылаетесь на метод obj, не вызывая его
var fn2 = obj.bar;
// Вызов this стрелочной функции изнутри метода bar вернёт теперь window,
// потому что он следует за this из fn2.
console.log(fn2()() == window); // true
```

В примере выше, функция (назовём её анонимной функцией A), присвоенная методу `obj.bar`, возвращает другую функцию (назовём её анонимной функцией B) которая создана как стрелочная функция. В результате, `this функции B при вызове замкнут на` `this,` принадлежащий `obj.bar` (функции A). `this` функции B всегда будет иметь то значение, которое он получил при создании. В примере выше, `this функции B` указывает на `this функции A,которым является` obj, таким образом this будет равен `obj` даже тогда, когда будет вызван методом, который в нормальных условиях устанавливал бы значение this равным `undefined` или глобальному объекту (или любым другим методом, как в предыдущем примере в глобальном контексте выполнения).

### В методе объекта

Когда функция вызывается как метод объекта, используемое в этой функции ключевое слово `this` принимает значение объекта, по отношению к которому вызван метод.

В следующем примере, когда вызвано свойство `o.f()` , внутри функции `this` привязано к объекту `o.`

```js
var o = {
  prop: 37,
  f: function () {
    return this.prop;
  },
};

console.log(o.f()); // logs 37
```

Необходимо отметить, что на поведение `this` совсем не влияет то, как или где была определена функция. В предыдущем примере мы определили функцию внутри свойства `f` во время определения объекта `o`. Однако, мы могли бы также просто определить сначала функцию, а затем закрепить её за свойством `o.f`. В этом случае поведение `this` не изменится:

```js
var o = { prop: 37 };

function independent() {
  return this.prop;
}

o.f = independent;

console.log(o.f()); // logs 37
```

Эти примеры показывают, что имеет значение только то, что функция была вызвана из свойства `f` объекта `o`.

Аналогично, привязывание `this` обуславливается наличием ближайшей ссылки на объект или свойство. В следующем примере, когда мы вызываем функцию, мы обращаемся к ней как к методу `g` объекта `o.b`. На этот раз во время выполнения, `this`, что находится внутри функции, будет ссылаться на `o.b`. Тот факт, что объект является членом объекта `o`, не имеет значения; важна только ближайшая ссылка.

```js
o.b = { g: independent, prop: 42 };
console.log(o.b.g()); // logs 42
```

#### `this` в цепочке object's prototype

Это же представление справедливо и для методов, определённых где-либо в цепочке object's prototype. Если метод находится в цепочке прототипов, то `this` ссылается на объект, на котором был вызван метод, т.е. так, словно метод является методом самого объекта, а не прототипа.

```js
var o = {
  f: function () {
    return this.a + this.b;
  },
};
var p = Object.create(o);
p.a = 1;
p.b = 4;

console.log(p.f()); // 5
```

В этом примере объект, который присвоен переменной `p`, не имеет собственного свойства `f`, а наследует это свойство от своего прототипа. Однако, совершенно неважно, что поиск свойства f в конце концов обнаружит его на объекте `o`. Поскольку поиск начался с `p.f`, то и свойство `this` внутри функции `f` будет ссылаться на объект `p`. Таким образом, если `f` вызывается как метод `p`, то и `this` относится к `p`. Это полезная особенность прототипного наследования JS.

#### `this` с геттерами/сеттерами

Все те же утверждения справедливы, если функция вызывается из геттера или сеттера. Для функции, которая используется как геттер или сеттер `this` привязан к объекту, свойство которого необходимо извлечь через геттер/сеттер.

```js
function modulus() {
  return Math.sqrt(this.re * this.re + this.im * this.im);
}

var o = {
  re: 1,
  im: -1,
  get phase() {
    return Math.atan2(this.im, this.re);
  },
};

Object.defineProperty(o, "modulus", {
  get: modulus,
  enumerable: true,
  configurable: true,
});

console.log(o.phase, o.modulus); // logs -0.78 1.4142
```

### В конструкторе

Когда функция используется как конструктор (с ключевым словом [`new`](/ru/docs/Web/JavaScript/Reference/Operators/new) ), `this` связано с создаваемым новым объектом.

Примечание: по умолчанию конструктор возвращает объект, на который ссылается `this`, но он может вернуть и другой объект (если возвращаемое значение не является объектом, тогда будет возвращён объект с `this`).

```js
/*
 * Конструктор работает таким образом:
 *
 * function MyConstructor(){
 *   // фактический код, составляющий тело функции.
 *   // создание свойств с |this| по
 *   // желанию, определяя их значения; например,
 *   this.fum = "nom";
 *   // и т.д.
 *
 *   // Если функция возвращает выражение,
 *   // возвращающее объект, этот объект будет
 *   // результатом выражения |new|. В обратном случае,
 *   // результат выражения - объект,
 *   // в данный момент привязанный к |this|
 *   // (т.е. наиболее часто встречающийся случай).
 * }
 */

function C() {
  this.a = 37;
}

var o = new C();
console.log(o.a); // logs 37

function C2() {
  this.a = 37;
  return { a: 38 };
}

o = new C2();
console.log(o.a); // logs 38
```

В последнем примере (`C2`), из-за того, что конструктор вернул объект, новый объект, к которому было привязано `this`, был просто отброшен. (Это фактически делает выражение "`this.a = 37;`" "мёртвым" кодом. Он не является буквально нерабочим, так как он выполняется, но он может быть изъят без каких-либо внешних эффектов.)

### `call` и `apply`

Когда в теле функции используется ключевое слово `this`, его значение может быть привязано к конкретному объекту в вызове при помощи методов [`call`](/ru/docs/Web/JavaScript/Reference/Global_Objects/Function/call) или [`apply`](/ru/docs/Web/JavaScript/Reference/Global_Objects/Function/apply), которые наследуются всеми функциями от `Function.prototype`.

```js
function add(c, d) {
  return this.a + this.b + c + d;
}

var o = { a: 1, b: 3 };

// Первый параметр - это объект, который следует использовать как
// 'this', последующие параметры передаются
// как аргументы при вызове функции
add.call(o, 5, 7); // 1 + 3 + 5 + 7 = 16

// Первый параметр - объект, который следует использовать как
// 'this', второй параметр - массив,
// элементы которого используются как аргументы при вызове функции
add.apply(o, [10, 20]); // 1 + 3 + 10 + 20 = 34
```

Необходимо отметить, что если методам `call` и `apply` передаётся значение с `this`, которое не является при этом объектом, будет предпринята попытка конвертировать значение в объект, используя внутреннюю операцию `ToObject`. Если переданное значение является примитивным типом, например `7` или `'foo'`, оно будет преобразовано в объект с использованием родственного конструктора, так примитив `7` преобразовывается в объект через `new Number(7),` а строка `'foo'` в объект через `new String('foo'),` и т.д.

```js
function bar() {
  console.log(Object.prototype.toString.call(this));
}

bar.call(7); // [object Number]
```

### Как обработчик событий DOM

Когда функция используется как обработчик событий, `this` присваивается элементу с которого начинается событие (некоторые браузеры не следуют этому соглашению для обработчиков, добавленных динамически с помощью всех методов, кроме `addEventListener`).

```js
// Когда вызывается как обработчик, связанный элемент становится синим
function bluify(e) {
  // Всегда true
  console.log(this === e.currentTarget);
  // true, когда currentTarget и target один объект
  console.log(this === e.target);
  this.style.backgroundColor = "#A5D9F3";
}

// Получить список каждого элемента в документе
var elements = document.getElementsByTagName("*");

// Добавить bluify как обработчика кликов, чтобы при
// нажатии на элемент он становился синим
for (var i = 0; i < elements.length; i++) {
  elements[i].addEventListener("click", bluify, false);
}
```

### В инлайновом обработчике событий

Когда код вызван из инлайнового обработчика, `this` указывает на DOM-элемент, в котором расположен код события:

```js
<button onclick="alert(this.tagName.toLowerCase());">Показать this</button>
```

Код выше выведет '`button`'. Следует отметить, что `this` будет указывать на DOM-элемент только во внешних (не вложенных) функциях:

```js
<button onclick="alert((function() {return this;} ()));">
  Показать вложенный this
</button>
```

В этом случае `this` вложенной функции не будет установлен, так что будет возвращён global/window объект.

## Спецификации

{{Specifications}}

## Совместимость

{{Compat}}

## Смотрите также

- [Строгий режим](/ru/docs/Web/JavaScript/Reference/Strict_mode)
- [All this](http://bjorn.tipling.com/all-this), статья о `this` в разном контексте
- [Краткое объяснение ключевого слова 'this' в JavaScript](https://rainsoft.io/gentle-explanation-of-this-in-javascript/)
