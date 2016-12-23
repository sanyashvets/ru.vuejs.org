---
title: Компоненты
type: guide
order: 11
---

## Что такое компоненты?

Компоненты — это одна из самых мощных возможностей Vue. Компоненты расширяют базовые HTML-элементы, позволяя инкапсулировать повторно используемый код. Не вдаваясь в подробности, можно сказать, что компоненты — это пользовательские элементы, к которым компилятор Vue привязывает определенное поведение. В некоторых случаях компоненты также можно задать с помощью нативных элементов, расширенных специальным атрибутом `is`.

## Использование компонентов

### Регистрация

В предыдущих разделах мы научились создавать инстансы Vue:

``` js
new Vue({
  el: '#some-element',
  // опции
})
```

Зарегистрировать глобальный компонент можно с помощью `Vue.component(tagName, options)`:

``` js
Vue.component('my-component', {
  // опции
})
```

<p class="tip">Обратите внимание, что Vue не требует соблюдения [правил W3C](http://www.w3.org/TR/custom-elements/#concepts) для пользовательских имён тегов (таких как требования использования только нижнего регистра и применения дефисов), хотя следование этим соглашениям считается хорошей практикой.</p>

Зарегистрированный компонент можно использовать в шаблоне инстанса как пользовательский элемент `<my-component></my-component>`. Компонент обязательно должен быть зарегистрирован **до создания корневого инстанса Vue**. Вот полный пример:

``` html
<div id="example">
  <my-component></my-component>
</div>
```

``` js
// регистрация
Vue.component('my-component', {
  template: '<div>Пользовательский компонент!</div>'
})

// создание корневого инстанса
new Vue({
  el: '#example'
})
```

Результатом рендеринга будет:

``` html
<div id="example">
  <div>Пользовательский компонент!</div>
</div>
```

{% raw %}
<div id="example" class="demo">
  <my-component></my-component>
</div>
<script>
Vue.component('my-component', {
  template: '<div>Пользовательский компонент!</div>'
})
new Vue({ el: '#example' })
</script>
{% endraw %}

### Локальные компоненты

Необязательно регистрировать все компоненты глобально. Можно сделать компонент доступным только в области видимости другого инстанса или компонента, зарегистрировав его через опцию `components`:

``` js
var Child = {
  template: '<div>Пользовательский компонент!</div>'
}

new Vue({
  // ...
  components: {
    // <my-component> будет доступен только в шаблоне родителя
    'my-component': Child
  }
})
```

Аналогичные принципы инкапсуляции применимы и для всех остальных регистрируемых пользовательских расширений Vue, например для директив.

### Особенности парсинга DOM-шаблона

Если в качестве шаблона используется DOM (то есть в опции `el` указана точка монтирования, уже содержащая контент), это накладывает определенные ограничения, обусловленные самим языком HTML. Тогда содержимое шаблона поступает в Vue только **после** того, как браузер распарсит и нормализует HTML-страницу. При этом определенные сочетания элементов могут не соответствовать нормам языка HTML. Например, есть ограничение по тому, какие элементы могут находиться внутри элементов `<ul>`, `<ol>`, `<table>` и `<select>`. Для некоторых других элементов, например для `<option>`, подобным же образом ограничен список допустимых родительских элементов.

С такими элементами пользовательские компоненты могут работать некорректно. Рассмотрим пример:

``` html
<table>
  <my-row>...</my-row>
</table>
```

Пользовательский компонент `<my-row>` будет отброшен браузером как некорректный, что в конечном итоге приведет к ошибке во время рендеринга. Обойти эту проблему можно используя специальный атрибут `is`:

``` html
<table>
  <tr is="my-row"></tr>
</table>
```

**Стоит заметить, что эти ограничения не действуют, если в качестве шаблонов используются следующие источники**:

- `<script type="text/x-template">`
- inline-строки JavaScript
- `.vue`-компоненты

Поэтому мы советуем, по возможности, всегда использовать строковые шаблоны.

### Опция `data` должна быть функцией

Большую часть опций, которые можно передавать в конструктор Vue, допускается использовать и в компонентах. Есть одно важное исключение: опция `data` должна быть функцией. Попытаемся выполнить следующий код:

``` js
Vue.component('my-component', {
  template: '<span>{{ message }}</span>',
  data: {
    message: 'привет!'
  }
})
```

Vue остановится и выведет в консоль предупреждение, что опция `data` в компонентах должна быть функцией. Тем не менее, неплохо бы понимать, почему существуют такие правила — так что давайте немного схитрим:

``` html
<div id="example-2">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
```

``` js
var data = { counter: 0 }

Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  // технически data является функцией, так что Vue
  // не будет жаловаться, но при каждом вызове эта функция
  // возвращает ссылку на один и тот же внешний объект
  data: function () {
    return data
  }
})

new Vue({
  el: '#example-2'
})
```

{% raw %}
<div id="example-2" class="demo">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
<script>
var data = { counter: 0 }
Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () {
    return data
  }
})
new Vue({
  el: '#example-2'
})
</script>
{% endraw %}

Какая неожиданность! Инкрементирование одного из счётчиков также инкрементирует и остальные два, поскольку все они используют один и тот же объект `data`. Исправим ошибку: пусть функция при каждом вызове возвращает вновь созданный объект data.

``` js
data: function () {
  return {
    counter: 0
  }
}
```

Теперь у каждого счётчика есть свое собственное внутреннее состояние:

{% raw %}
<div id="example-2-5" class="demo">
  <my-component></my-component>
  <my-component></my-component>
  <my-component></my-component>
</div>
<script>
Vue.component('my-component', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  }
})
new Vue({
  el: '#example-2-5'
})
</script>
{% endraw %}

### Композиция компонентов

Компоненты обычно используются совместно, в основном в рамках иерархических отношений, когда компонент-родитель A ссылается на компонент-потомок B в своём собственном шаблоне. Для этого нужно обеспечить коммуникацию компонентов друг с другом. Например, родитель может передавать данные потомку, а потомок, в свою очередь, может уведомлять родителя о произошедших событиях. С помощью четко заданного интерфейса взаимодействие между компонентами сводится к необходимому минимуму. Благодаря такому подходу можно писать и анализировать код каждого компонента в условиях относительной изоляции. Это упрощает поддержку и потенциально облегчает повторное использование компонентов.

Во Vue.js иерархические отношения подчиняются следующему принципу: **"входные параметры — вниз, события — вверх" ("props down, events up")**. Родитель передаёт данные потомку через **входные параметры (props)**, а потомок посылает сообщения родителю посредством **событий (events)**. Давайте посмотрим как это работает.

<p style="text-align: center">
  <img style="width:300px" src="/images/props-events.png" alt="props down, events up">
</p>

## Входные параметры

### Передача данных через входные параметры

Каждый инстанс компонента имеет свою собственную **изолированную область видимости**. Поэтому напрямую обращаться к данным родительского компонента из шаблона компонента-потомка невозможно (да это и не требуется). Вместо этого данные передаются вниз по цепочке иерархии с помощью **входных параметров**.

Входной параметр — это пользовательский атрибут для передачи информации из родительского компонента. Ожидаемые входные параметры нужно явно определить в потомке с помощью [опции `props`](../api/#props):

``` js
Vue.component('child', {
  // определяем входной параметр
  props: ['message'],
  // как и другие данные, входной параметр можно использовать
  // внутри шаблонов (а также и в методах, обращаясь через this.message)
  template: '<span>{{ message }}</span>'
})
```

Мы можем передать в компонент строку, например так:

``` html
<child message="привет!"></child>
```

Результатом будет:

{% raw %}
<div id="prop-example-1" class="demo">
  <child message="привет!"></child>
</div>
<script>
new Vue({
  el: '#prop-example-1',
  components: {
    child: {
      props: ['message'],
      template: '<span>{{ message }}</span>'
    }
  }
})
</script>
{% endraw %}

### camelCase против kebab-case

Атрибуты HTML являются регистронезависимыми, так что **при использовании в DOM в качестве шаблона** вместо camelCase-версий имён входных параметров приходится применять их kebab-case эквиваленты (разделять слова дефисом):

``` js
Vue.component('child', {
  // camelCase в JavaScript
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})
```

``` html
<!-- kebab-case в HTML -->
<child my-message="привет!"></child>
```

Впрочем, строковые шаблоны не накладывают и этого ограничения.

### Динамические входные параметры

С помощью директивы `v-bind` входные параметры можно динамически связывать с данными родительского компонента аналогично тому, как  обычные атрибуты связываются с выражениями. Любое обновление данных в родителе в этом случае будет передано и в компонент-потомок:

``` html
<div>
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
```

Зачастую проще использовать для `v-bind` сокращённую запись:

``` html
<child :my-message="parentMsg"></child>
```

Результат:

{% raw %}
<div id="demo-2" class="demo">
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
<script>
new Vue({
  el: '#demo-2',
  data: {
    parentMsg: 'Сообщение из родителя'
  },
  components: {
    child: {
      props: ['myMessage'],
      template: '<span>{{myMessage}}</span>'
    }
  }
})
</script>
{% endraw %}

### Различие между литералами и динамическими параметрами

Часто встречается ошибка, когда число передается компоненту в виде литеральной константы:

``` html
<!-- при такой записи в компонент будет передана строка "1" -->
<comp some-prop="1"></comp>
```

Так как в качестве параметра передан литерал, компонент получит не число, а строку `"1"`. Для передачи числа нужно использовать директиву `v-bind`, поскольку её значение вычисляется как выражение JavaScript:

``` html
<!-- этот синтаксис позволит передать в компонент число -->
<comp v-bind:some-prop="1"></comp>
```

### Однонаправленный поток данных

Входные параметры обеспечивают **однонаправленный** поток данных от родительского компонента к потомкам. Если свойство компонента-родителя изменилось, это изменение передается потомку, но не наоборот. Если бы потомки могли произвольно изменять состояние родителя, понять структуру потоков данных внутри приложения было бы намного труднее. Благодаря однонаправленным потокам такая ситуация исключается.

Кроме того, при любом обновлении родительского компонента каждый входной параметр потомка обновляется до актуального значения. Поэтому **не следует изменять значения** входных параметров внутри компонента и расчитывать на их сохранность. При попытке изменить входной параметр Vue выведет предупреждение в консоли.

Новички обычно пытаются изменить значение входного параметра в двух случаях:

1. Если параметр нужен лишь для передачи потомку начального значения, после чего планируется использовать эту переменную как локальную.

2. Если значению, которое передается как параметр, требуется дальнейшая обработка.

Правильное решение этих задач следующее:

1. Объявить локальную переменную, которая принимает значение входного параметра при инициализации:

  ``` js
  props: ['initialCounter'],
  data: function () {
    return { counter: this.initialCounter }
  }
  ```

2. Определить вычисляемое свойство, основанное на значении входного параметра:

  ``` js
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

<p class="tip">Обратите внимание, что объекты и массивы в JavaScript передаются по ссылке, так что если входным параметром является объект или массив, его изменение внутри потомка **повлияет** на состояние родительского компонента.</p>

### Валидация входных параметров

В компонентах можно не только указать список ожидаемых параметров, но и предъявить к этим параметрам определённые требования. Если полученные параметры не удовлетворяют требованиям, Vue выведет предупреждение. Эта возможность особенно полезна при создании компонентов для внешнего использования.

Вместо задания списка параметров с помощью массива строк, можно использовать объект с правилами валидации:

``` js
Vue.component('example', {
  props: {
    // простая проверка типа (`null` означает допустимость любого типа)
    propA: Number,
    // несколько допустимых типов
    propB: [String, Number],
    // обязательное значение строкового типа
    propC: {
      type: String,
      required: true
    },
    // число со значением по умолчанию
    propD: {
      type: Number,
      default: 100
    },
    // значения по умолчанию для объектов и массивов
    // должны задаваться через функцию
    propE: {
      type: Object,
      default: function () {
        return { message: 'привет!' }
      }
    },
    // пользовательская функция для валидации
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

В качестве параметра `type` используется один из нижеперечисленных нативных конструкторов:

- String
- Number
- Boolean
- Function
- Object
- Array

Кроме того, `type` может быть и пользовательской функцией-конструктором. При этом проверка соответствия выполняется с помощью `instanceof`.

На ошибки валидации Vue реагирует, выводя предупреждения в консоль (при использовании development-сборки).

## Пользовательские события

Мы узнали, что компонент-родитель может передавать данные потомкам через входные параметры. Но как организовать связь в обратном направлении? Самое время поговорить о системе пользовательских событий Vue.

### Использование `v-on` с пользовательскими событиями

Каждый инстанс Vue поддерживает [интерфейс событий](../api/#Instance-Methods-Events), позволяющий:

- Отслеживать события, используя `$on(eventName)`
- Порождать события, используя `$emit(eventName)`

<p class="tip">Обратите внимание, что система событий Vue отделена от [EventTarget API](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget) браузера. Хотя они и похожи, `$on` и `$emit` — это не псевдонимы `addEventListener` и `dispatchEvent`.</p>

Кроме того, родительский компонент может зарегистрировать подписчика событий, используя директиву `v-on` непосредственно в шаблоне при создании компонента-потомка.

Вот пример:

``` html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```

``` js
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

{% raw %}
<div id="counter-event-example" class="demo">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
{% endraw %}

Важно отметить, что потомок остаётся полностью независимым от всего происходящего снаружи. Он всего лишь уведомляет внешний мир о происходящем с ним, на случай если родительскому компоненту это будет интересно.

#### Подписка на нативные события в компонентах

Иногда нужно подписаться на нативные события браузера в корневом элементе компонента. В таких случаях можно применить `v-on` с модификатором `.native`, например так:

``` html
<my-component v-on:click.native="doTheThing"></my-component>
```

### Поля ввода форм с использованием пользовательских событий

С помощью пользовательских событий можно также создавать пользовательские поля ввода с поддержкой директивы `v-model`. Вспомните, что

``` html
<input v-model="something">
```

это всего лишь синтаксический сахар для:

``` html
<input v-bind:value="something" v-on:input="something = $event.target.value">
```

При использовании с компонентом, запись упрощается до:

``` html
<custom-input v-bind:value="something" v-on:input="something = arguments[0]"></custom-input>
```

Таким образом, чтобы иметь возможность работать с `v-model`, компонент должен:

- принимать входной параметр `value`
- порождать событие `input` с новым значением

Давайте разберём в качестве примера простое поле ввода денежной суммы с точкой в качестве десятичного разделителя:

``` html
<currency-input v-model="price"></currency-input>
```

``` js
Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
  props: ['value'],
  methods: {
    // Вместо того, чтобы обновлять значение напрямую,
    // в этом методе мы выполняем нормализацию и форматирование
    // введенного значения, а затем порождаем событие,
    // уведомляющее родительский компонент об изменениях
    updateValue: function (value) {
      var formattedValue = value
        // Удалить пробелы с обеих сторон
        .trim()
        // Сократить до 2 знаков после запятой
        .slice(0, value.indexOf('.') + 3)
      // Если значение не нормализовано — нормализуем вручную
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      // Порождаем событие с обновлённым значением поля ввода
      this.$emit('input', Number(formattedValue))
    }
  }
})
```

{% raw %}
<div id="currency-input-example" class="demo">
  <currency-input v-model="price"></currency-input>
</div>
<script>
Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
  props: ['value'],
  methods: {
    updateValue: function (value) {
      var formattedValue = value
        .trim()
        .slice(0, value.indexOf('.') + 3)
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      this.$emit('input', Number(formattedValue))
    }
  }
})
new Vue({ el: '#currency-input-example' })
</script>
{% endraw %}

Очевидно, что наша реализация не лишена недостатков. Например, пользователь может ввести несколько десятичных точек, а кое-где и буквы вместо цифр. Для тех, кого интересует пример менее тривиальной и более надёжной реализации, вот он:

<iframe width="100%" height="300" src="https://jsfiddle.net/chrisvfritz/1oqjojjx/embedded/result,html,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Интерфейс событий может быть использован не только для связи с полями ввода форм внутри компонентов, но и для создания более необычных полей ввода. К примеру, представьте себе следующие возможности:

``` html
<voice-recognizer v-model="question"></voice-recognizer>
<webcam-gesture-reader v-model="gesture"></webcam-gesture-reader>
<webcam-retinal-scanner v-model="retinalImage"></webcam-retinal-scanner>
```

### Коммуникация между компонентами, не связанными иерархически

Иногда нужно обеспечить обмен информацией между компонентами, которые не состоят в отношениях родитель-потомок. В простых случаях можно использовать пустой инстанс Vue в качестве централизованной шины событий:

``` js
var bus = new Vue()
```
``` js
// в методе компонента A
bus.$emit('id-selected', 1)
```
``` js
// в обработчике created компонента B
bus.$on('id-selected', function (id) {
  // ...
})
```

Для более сложных случаев подойдет специализированный [паттерн управления состоянием](state-management.html).

## Дистрибьюция контента через слоты

Нередко хочется вкладывать компоненты друг в друга следующим образом:

``` html
<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>
```

Здесь стоит обратить внимание на две вещи:

1. Компонент `<app>` не знает, какой контент он будет содержать после монтирования. Это определяется родительским компонентом, использующим `<app>`.

2. Скорее всего, у компонента `<app>` есть собственный шаблон.

Чтобы такая композиция работала, необходим метод "переплетения" шаблона компонента и внутреннего содержимого, указанного при его использовании в родительском контексте. Этот процесс называется **дистрибьюцией контента**, или, в терминах Angular, "включением" ("transclusion"). Во Vue.js реализован API дистрибьюции контента, примерно соответствующий текущему [черновику спецификации Web Components](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md). В нем используется специальный элемент `<slot>`, служащий "точкой выхода" для исходного контента.

### Область видимости при компиляции

Перед тем как углубиться в рассмотрение API слотов, давайте сперва разберёмся, в какой области видимости компилируется содержимое шаблонов. Представим такой шаблон:

``` html
<child-component>
  {{ message }}
</child-component>
```

Из какого контекста берется переменная `message`, контекста родителя или контекста потомка? Правильный ответ — из контекста родителя. Действует простое правило:

> Всё в шаблоне родительского компонента компилируется в области видимости родителя; всё в шаблоне потомка — в области видимости потомка.

Часто встречается ошибка, когда в шаблоне родителя указывается связывание со свойством компонента-потомка:

``` html
<!-- НЕ сработает -->
<child-component v-show="someChildProperty"></child-component>
```

Если `someChildProperty` является свойством потомка, вышеприведённый пример работать не будет. Шаблон родителя не имеет никакого представления о состоянии компонента-потомка.

Если все-таки нужно привязать директивы из области видимости компонента-потомка к корневому элементу, это необходимо сделать в его же шаблоне:

``` js
Vue.component('child-component', {
  // такой вариант сработает, поскольку мы находимся
  // в правильной области видимости
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      someChildProperty: true
    }
  }
})
```

Контент, полученный через систему дистрибьюции, также будет компилироваться в родительской области видимости.

### Вариант с единственным слотом

Родительский контент будет **отброшен**, если в шаблоне компонента-потомка компонента нет хотя бы одного элемента `<slot>`. В случае, если слот всего один и не содержит атрибутов, всё содержимое родительского элемента будет помещено в DOM на место слота, замещая его собой.

Изначальное содержимое тега `<slot>` считается **резервным контентом**. Оно компилируется в области видимости компонента-потомка и отображается только в том случае, если родительский элемент пуст и не содержит никакого контента для передачи потомку.

Предположим, у нас есть компонент `my-component`, с таким шаблоном:

``` html
<div>
  <h2>Заголовок компонента-потомка</h2>
  <slot>
    Этот текст будет отображён только если
    не будет передано контента для дистрибьюции.
  </slot>
</div>
```

И родитель, использующий этот компонент:

``` html
<div>
  <h1>Заголовок компонента-родителя</h1>
  <my-component>
    <p>Немного оригинального контента</p>
    <p>И ещё немного</p>
  </my-component>
</div>
```

Результатом рендеринга будет:

``` html
<div>
  <h1>Заголовок компонента-родителя</h1>
  <div>
    <h2>Заголовок компонента-потомка</h2>
    <p>Немного оригинального контента</p>
    <p>И ещё немного</p>
  </div>
</div>
```

### Именованные слоты

Для элементов `<slot>` можно указать специальный атрибут `name`, который используется для ещё более гибкой дистрибьюции контента. Можно создать несколько слотов с различными именами. Именованный слот получит весь контент, находящийся в элементе с соответствующим значением атрибута `slot`.

Если одному из слотов не задать имя, он станет **слотом по умолчанию**, в который попадёт весь контент, для которого имя слота не указано. В случае отсутствия безымянного слота, такой контент будет попросту отброшен.

Для примера, предположим что у нас есть компонент `app-layout` с таким шаблоном:

``` html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

Шаблон родителя:

``` html
<app-layout>
  <h1 slot="header">Здесь мог бы быть заголовок страницы</h1>

  <p>Абзац основного контента.</p>
  <p>И ещё один.</p>

  <p slot="footer">Вот контактная информация</p>
</app-layout>
```

Результатом рендеринга будет:

``` html
<div class="container">
  <header>
    <h1>Здесь мог бы быть заголовок страницы</h1>
  </header>
  <main>
    <p>Абзац основного контента.</p>
    <p>И ещё один.</p>
  </main>
  <footer>
    <p>Вот контактная информация</p>
  </footer>
</div>
```

API дистрибьюции контента — это очень полезный механизм для создания компонентов, которые будут использоваться совместно.

### Слоты с ограниченной областью видимости

> Добавлены в 2.1.0

Слот с ограниченной областью видимости — это особый тип слота, который применяется как повторно используемый шаблон, то есть шаблон, в который передются данные, а не уже отрендеренные элементы.

В компоненте-потомке нужно просто передать данные в слот, так же, как входные параметры передаются в компонент:

``` html
<div class="child">
  <slot text="сообщение от потомка"></slot>
</div>
```

В родителе, элемент `<template>` с особым атрибутом `scope` указывает, что это шаблон для именованного слота. Значение `scope` — это имя временной переменной, содержащей входные параметры, переданные от потомка:

``` html
<div class="parent">
  <child>
    <template scope="props">
      <span>сообщение от родителя</span>
      <span>{{ props.text }}</span>
    </template>
  </child>
</div>
```

Результатом рендеринга кода выше будет:

``` html
<div class="parent">
  <div class="child">
    <span>сообщение от родителя</span>
    <span>сообщение от потомка</span>
  </div>
</div>
```

Более характерное применение для слотов с ограниченной областью видимости — это компонент, выводящий список элементов, в котором пользователь может переопределить вид элемента:

``` html
<my-awesome-list :items="items">
  <!-- слот с ограниченной областью видимости может быть и именованным -->
  <template slot="item" scope="props">
    <li class="my-fancy-item">{{ props.text }}</li>
  </template>
</my-awesome-list>
```

И шаблон самого компонента списка:

``` html
<ul>
  <slot name="item"
    v-for="item in items"
    :text="item.text">
    <!-- здесь — контент для резервного отображения -->
  </slot>
</ul>
```

## Динамическое переключение компонентов

Можно подключить несколько компонентов к одной и той же точке монтирования, а затем динамически переключаться между ними. Для этого используется псевдоэлемент `<component>` и динамическое связывание его атрибута `is`:


``` js
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```

``` html
<component v-bind:is="currentView">
  <!-- изменяя vm.currentView можно переключаться между компонентами -->
</component>
```

При желании можно связываться с объектами компонентов и напрямую:

``` js
var Home = {
  template: '<p>Добро пожаловать домой!</p>'
}

var vm = new Vue({
  el: '#example',
  data: {
    currentView: Home
  }
})
```

### `keep-alive`

Иногда бывает выгодно хранить отключенные компоненты в памяти, чтобы не терять их состояния и не выполнять их повторный рендеринг. Для этого нужно обернуть динамический компонент в псевдоэлемент `<keep-alive>`:

``` html
<keep-alive>
  <component :is="currentView">
    <!-- неактивные компоненты будут закешированы! -->
  </component>
</keep-alive>
```

Более детально `<keep-alive>` рассмотрен в [справочнике по API](../api/#keep-alive).

## Разное

### Создание компонентов для повторного использования

Создавая компоненты, неплохо понимать, планируется ли использовать их где-то ещё в будущем. Если компоненты одноразовые, они могут быть и сильно связанными. В компонентах, предназначенных для повторного использования, следует определить четкий публичный интерфейс, в котором нет излишних предположений о контексте использования компонента.

API компонентов Vue состоит из трёх частей: входных параметров, событий и слотов:

- **Входные параметры** позволяют передавать в компонент данные извне.

- **События** позволяют компонентам воздействовать на внешнее окружение.

- **Слоты** позволяют внешнему окружению дополнять компоненты новым контентом.

Благодаря специальному сокращённому синтаксису `v-bind` и `v-on`, назначение компонента можно коротко и ясно выразить в шаблоне:

``` html
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="doThis"
  @event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Привет!</p>
</my-component>
```

### Ссылки на компоненты-потомки

Несмотря на существование таких средств как входные параметры и события, иногда всё же возникает необходимость обратиться к компонентам-потомкам в JavaScript напрямую. Для этих целей можно при помощи атрибута `ref` назначить компоненту идентификатор. Например:

``` html
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>
```

``` js
var parent = new Vue({ el: '#parent' })
// получаем инстанс компонента-потомка
var child = parent.$refs.profile
```

Если `ref` используется вместе с директивой `v-for`, будет возвращен массив или объект со ссылками на инстансы, структурно повторяющий исходные данные.

<p class="tip">Объект `$refs` заполняется только после рендеринга компонента и не является реактивным. Считайте, что это крайнее средство для непосредственного вмешательства в работу компонента-потомка, и не используйте `$refs` в шаблонах и вычисляемых свойствах.</p>

### Асинхронные компоненты

Иногда бывает удобно разделить крупное приложение на части и подгружать компоненты с сервера только тогда, когда в них возникнет потребность. Для этого Vue позволяет определить компонент как функцию-фабрику, асинхронно возвращающую определение компонента. Vue вызовет фабричную функцию только тогда, когда компонент действительно понадобится, и закеширует результат для дальнейшего использования. Например:

``` js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // Передаем шаблон компонента в функцию обратного вызова resolve
    resolve({
      template: '<div>Я — асинхронный!</div>'
    })
  }, 1000)
})
```

Функция-фабрика принимает параметр `resolve` — функцию обратного вызова, которая вызыватся после того, как определение компонента получено от сервера. Кроме того, можно вызвать `reject(reason)`, если загрузка по какой-либо причине не удалась. Мы используем `setTimeout` исключительно в демонстрационных целях; как именно получать компонент в реальной ситуации — решать только вам самим. Один из удачных подходов — это использовать асинхронные компоненты в связке с [функциями Webpack по разделению кода](http://webpack.github.io/docs/code-splitting.html):

``` js
Vue.component('async-webpack-example', function (resolve) {
  // специальный синтаксис require укажет Webpack
  // автоматически разделить сборку на части
  // для последующей асинхронной загрузки
  require(['./my-async-component'], resolve)
})
```

В функции resolve можно также вернуть промис, так что используя Webpack 2 и синтаксис ES2015 можно сделать так:

``` js
Vue.component(
  'async-webpack-example',
  () => System.import('./my-async-component')
)
```

<p class="tip">Если вы используете <strong>Browserify</strong> и также хотите реализовать асинхронную загрузку компонентов, нам, к сожалению, придётся вас огорчить. Это невозможно, и вряд ли будет возможно когда-либо, так как сам создатель Browserify [прояснил](https://github.com/substack/node-browserify/issues/58#issuecomment-21978224), что асинхронная загрузка "не является функцией, которую Browserify когда-либо будет поддерживать". По крайней мере, такова официальная позиция. Сообщество Browserify обнаружило возможные [обходные пути](https://github.com/vuejs/vuejs.org/issues/620), которые могут быть полезны в уже существующих сложных приложениях. Но в целом мы советуем использовать Webpack, обладающий полноценной встроенной поддержкой асинхронной загрузки частей сборки.</p>

### Соглашения по именованию компонентов

При регистрации компонентов (или входных параметров), можно использовать kebab-case, camelCase или TitleCase. Это не имеет никакого значения для Vue.

``` js
// при определении компонента
components: {
  // регистрация с использованием kebab-case
  'kebab-cased-component': { /* ... */ },
  // регистрация с использованием camelCase
  'camelCasedComponent': { /* ... */ },
  // регистрация с использованием TitleCase
  'TitleCasedComponent': { /* ... */ }
}
```

В HTML-шаблонах, однако, придётся использовать эквивалентный kebab-case:

``` html
<!-- всегда используйте kebab-case в HTML-шаблонах -->
<kebab-cased-component></kebab-cased-component>
<camel-cased-component></camel-cased-component>
<title-cased-component></title-cased-component>
```

При использовании **строковых** шаблонов ограничения регистронезависимости HTML не действуют. Это значит, что даже в шаблоне можно указывать компоненты и входные параметры как в camelCase, так и в TitleCase или kebab-case:

``` html
<!-- в строковых шаблонах вы вольны использовать любой подход! -->
<my-component></my-component>
<myComponent></myComponent>
<MyComponent></MyComponent>
```

Если компонент не содержит слотов, его можно даже сделать самозакрывающимся, указав `/` после имени:

``` html
<my-component/>
```

Ещё раз заметим, что это возможно **только при использовании строковых шаблонов**, поскольку самозакрывающие пользовательские элементы не соответствуют нормам языка HTML, и нативные парсеры браузеров такую запись не поймут.

### Рекурсивные компоненты

Компоненты могут рекурсивно вызывать самих себя в своих шаблонах. Однако, эта возможность доступна только при указании опции `name`:

``` js
name: 'unique-name-of-my-component'
```

Если компонент регистрируется глобально с помощью `Vue.component`, то опция `name` компонента автоматически становится равной его глобальному ID:

``` js
Vue.component('unique-name-of-my-component', {
  // ...
})
```

Если не соблюдать осторожность, рекурсивные компоненты могут привести к появлению бесконечных циклов:

``` js
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```

Использование такого компонента приведет к ошибке переполнения стека, поэтому следите, чтобы рекурсивный вызов был условным (т.е. чтобы в нем была директива `v-if`, которая рано или поздно станет ложной).

### Циклические ссылки между компонентами

Предположим, вы проектируете каталог файлов в виде дерева, похожий на  Finder или Проводник. Представьте себе, что для этого используется компонент `tree-folder` с таким шаблоном:

``` html
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>
```

Затем компонент `tree-folder-contents` с таким шаблоном:

``` html
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```

Присмотритесь к примеру. Парадоксально, но каждый из этих компонентов **одновременно** является и потомком, и родителем другого компонента! При глобальной регистрации компонентов с использованием `Vue.component`, данный парадокс будет разрешен автоматически. Если это ваш случай, то можете дальше не читать.

С другой стороны, если компоненты импортируются с помощью **модульного сборщика**, такого как Webpack или Browserify, возникнет ошибка.

```
Failed to mount component: template or render function not defined.
```

Чтобы объяснить это явление, давайте назовем наши компоненты A и B. Модульный сборщик видит, что ему нужен компонент A, но A сперва нужен B, но B нужен A, и т.д. Сборщик застревает в цикле, не зная как полностью разрешить оба компонента. Чтобы это исправить, нам нужно указать сборщику точку в которой он сможет сказать: "Рано или поздно для разрешения A нужно разрешить B, но нет необходимости разрешать B прямо сейчас."

Для нашего случая, мы сделаем такой точкой компонент `tree-folder`. Мы знаем, что компонент-потомок, порождающий парадокс — это `tree-folder-contents`. Поэтому мы не будем его регистрировать, пока не наступит событие жизненного цикла `beforeCreate`.

``` js
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue')
}
```

Проблема решена!

### Inline-шаблоны

Если у компонента-потомка присутствует специальный атрибут `inline-template`, содержимое элемента будет использовано не для дистрибьюции контента, а в качестве шаблона этого компонента. Это позволяет более гибко использовать шаблоны.

``` html
<my-component inline-template>
  <div>
    <p>Этот шаблон будет скомпилирован в области видимости компонента-потомка.</p>
    <p>Доступа к данным родителя нет.</p>
  </div>
</my-component>
```

С другой стороны, использование `inline-template` затрудняет понимание происходящего в шаблонах. Поэтому желательно задавать шаблоны внутри компонента с помощью опции `template` или в элементе `template` файла `.vue`.

### Определение шаблонов через X-Template

Еще один способ задания шаблонов — это специальные элементы `script` с типом `text/x-template` и идентификатором, на который можно сослаться при регистрации шаблона. Например:

``` html
<script type="text/x-template" id="hello-world-template">
  <p>Привет привет привет</p>
</script>
```

``` js
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

Эта возможность бывает полезной для демонстрационных приложений с большими шаблонами или для очень маленьких приложений. Тем не менее, в целом такого подхода следует избегать, так как шаблоны отделяются от остальных частей определения компонента.

### "Дешёвые" статические компоненты с использованием `v-once`

Рендеринг простых элементов HTML во Vue происходит достаточно быстро, но иногда встречаются компоненты в которых статических данных **очень** много. В таком случае можно указать в корневом элементе директиву `v-once`. С этой директивой компонент будет вычислен только в первый раз, а дальнейшая работа будет происходить с закешированной версией. Например:

``` js
Vue.component('terms-of-service', {
  template: '\
    <div v-once>\
      <h1>Условия Использования</h1>\
      ... много-много статического контента ...\
    </div>\
  '
})
```