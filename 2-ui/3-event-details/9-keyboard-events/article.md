# Клавиатура: keyup, keydown, keypress

Здесь мы рассмотрим основные "клавиатурные" события и работу с ними.
[cut]

## Тестовый стенд [#keyboard-test-stand]

Для того, чтобы лучше понять, как работают события клавиатуры, можно использовать тестовый стенд.

Попробуйте различные варианты нажатия клавиш в текстовом поле.

[codetabs src="keyboard-dump" height=480]

По мере чтения, если возникнут вопросы -- возвращайтесь к этому стенду.

## События keydown и keyup

События `keydown/keyup` происходят при нажатии/отпускании клавиши и позволяют получить её *скан-код* в свойстве `keyCode`.

Скан-код клавиши одинаков в любой раскладке и в любом регистре. Например, клавиша [key z] может означать символ `"z"`, `"Z"` или `"я"`, `"Я"` в русской раскладке, но её *скан-код* будет всегда одинаков: `90`.

[online]
В действии:

```html
<input onkeydown="this.nextSibling.innerHTML = event.keyCode"> <b></b>
```

<input size="40" placeholder="Нажмите клавишу, скан-код будет справа" onkeydown="this.nextElementSibling.innerHTML = event.keyCode"> <b></b>
[/online]

### Скан-коды

Для буквенно-цифровых клавиш есть очень простое правило: скан-код будет равен коду соответствующей заглавной английской буквы/цифры.

Например, при нажатии клавиши [key S] (не важно, каков регистр и раскладка) её скан-код будет равен `"S".charCodeAt(0)`.

Для других символов, в частности, знаков пунктуации, есть  таблица кодов, которую можно взять, например, из статьи Джона Уолтера: <a href="http://unixpapa.com/js/key.html">JavaScript Madness: Keyboard Events</a>, или же можно нажать на нужную клавишу на [тестовом стенде](#keyboard-test-stand) и получить код.

Когда-то в этих кодах была масса кросс-браузерных несовместимостей. Сейчас всё проще -- таблицы кодов в различных браузерах почти полностью совпадают. Но некоторые несовместимости, всё же, остались. Вы можете увидеть их в таблице ниже. Слева -- клавиша с символом, а справа -- скан-коды в различных браузерах.

Таблица несовместимостей:
<table>
<thead>
<tr><th>Клавиша</th><th>Firefox</th><th>Остальные браузеры</th></tr>
</thead>
<tbody>
<tr>
<td>[key ;]</td>
<td>59</td>
<td>186</td>
</tr>
<tr>
<td>[key =]</td>
<td>107</td>
<td>187</td>
</tr>
<tr>
<td>[key -]</td>
<td>109</td>
<td>189</td>
</tr>
</tbody>
</table>

Остальные коды одинаковы, код для нужного символа будет в тестовом стенде.

## Событие keypress

Событие `keypress` возникает сразу после `keydown`, если нажата *символьная* клавиша, т.е. нажатие приводит к появлению символа.

Любые буквы, цифры генерируют `keypress`. Управляющие клавиши, такие как [key Ctrl], [key Shift], [key F1], [key F2].. -- `keypress` не генерируют.

Событие `keypress` позволяет получить *код символа*. В отличие от скан-кода, он специфичен именно для символа и различен для `"z"` и `"я"`. 

Код символа хранится в свойствах: `charCode` и `which`. Здесь скрывается целое "гнездо" кросс-браузерных несовместимостей, разбираться с которыми нет никакого смысла -- запомнить сложно, а на практике нужна лишь одна "правильная" функция, позволяющая получить код везде.

### Получение символа в keypress [#getChar]

Кросс-браузерная функция для получения символа из события `keypress`:

```js
//+ autorun
// event.type должен быть keypress
function getChar(event) {
  if (event.which == null) { // IE
    if (event.keyCode < 32) return null; // спец. символ
    return String.fromCharCode(event.keyCode)
  }

  if (event.which != 0 && event.charCode != 0) { // все кроме IE
    if (event.which < 32) return null; // спец. символ
    return String.fromCharCode(event.which); // остальные
  }

  return null; // спец. символ
}
```

Для общей информации -- вот основные браузерные особенности, учтённые в `getChar(event)`:

<ol>
<li>Во всех браузерах, кроме IE, у события `keypress` есть свойство `charCode`, которое содержит код символа.</li>
<li>Браузер IE для `keypress` не устанавливает `charCode`, а вместо этого он записывает код символа в `keyCode` (в `keydown/keyup` там хранится скан-код).</li>
<li>Также в функции выше используется проверка `if(event.which!=0)`, а не более короткая `if(event.which)`. Это не случайно! При `event.which=null` первое сравнение даст `true`, а второе -- `false`.</li>
</ol>

[online]
В действии:

```html
<input onkeypress="this.nextSibling.innerHTML = getChar(event)+''"><b></b>
```

<input size="40" placeholder="Наберите символ, он будет справа" onkeypress="this.nextElementSibling.innerHTML = getChar(event)+''"> <b></b>
[/online]

[warn header="Неправильный `getChar`"]
В сети вы можете найти другую функцию того же назначения:

```js
function getChar(event) {
  return String.fromCharCode(event.keyCode || event.charCode);
}
```

Она работает неверно для многих специальных клавиш, потому что не фильтрует их. Например, она возвращает символ амперсанда `"&"`, когда нажата клавиша 'Стрелка Вверх'. Лучше использовать ту, что приведена выше.
[/warn]

Как и у других событий, связанных с пользовательским вводом, поддерживаются свойства `shiftKey`, `ctrlKey`, `altKey` и `metaKey`.

Они установлены в `true`, если нажаты клавиши-модификаторы -- соответственно, [key Shift], [key Ctrl], [key Alt] и [key Cmd] для Mac.

## Отмена пользовательского ввода   

Появление символа можно предотвратить, если отменить действие браузера на  `keydown/keypress`:

```html
Попробуйте что-нибудь ввести в этих полях:
<input *!*onkeydown="return false"*/!* type="text" size="30">
<input *!*onkeypress="return false"*/!* type="text" size="30">
```

[online]
Попробуйте что-нибудь ввести в этих полях (не получится):

<input onkeydown="return false" type="text" size="30">

<input onkeypress="return false" type="text" size="30">
[/online]

При тестировании на стенде вы можете заметить, что отмена действия браузера при `keydown` также предотвращает само событие `keypress`.
 
[warn header="При `keydown/keypress` значение ещё старое"]
На момент срабатывания `keydown/keypress` *клавиша ещё не обработана браузером*. 

Поэтому в обработчике значение `input.value` -- старое, т.е. до ввода. Это можно увидеть в примере ниже. Вводите символы `abcd..`, а справа будет текущее `input.value`: `abc..`

<input onkeydown="this.nextSibling.innerHTML=this.value" type="text" placeholder="Вводите символы"><b></b>

А что, если мы хотим обработать `input.value` именно после ввода? Самое простое решение -- использовать событие `keyup`, либо запланировать обрабочик через `setTimeout(..,0)`.
[/warn]



### Отмена любых действий  

Отменять можно не только символ, а любое действие клавиш.

Например:
<ul>
<li>При отмене [key Backspace] -- символ не удалится.</li>
<li>При отмене [key PageDown] -- страница не прокрутится.</li>
<li>При отмене [key Tab] -- курсор не перейдёт на следующее поле.</li>
</ul>

Конечно же, есть действия, которые в принципе нельзя отменить, в первую очередь -- те, которые происходят на уровне операционной системы. Комбинация Alt+F4 инициирует закрытие браузера в Windows, что бы мы ни делали в JavaScript. 

### Демо: перевод символа в верхний регистр

В примере ниже действие браузера отменяется с помощью `return false`, а вместо него в `input` добавляется значение в верхнем регистре:

```html
<input id="only-upper" type="text" size="2">
<script>
  document.getElementById('only-upper').onkeypress = function(e) {
    // спец. сочетание - не обрабатываем  
    if (e.ctrlKey || e.altKey || e.metaKey) return;

    var char = getChar(e);

    if (!char) return; // спец. символ - не обрабатываем

    this.value = char.toUpperCase();

    return false;
  };
</script>
```

[online]
В действии: <input id="only-upper" type="text" size="2">
<script>
document.getElementById('only-upper').onkeypress = function(e) {
  var char = getChar(e);  
  
  // спец. сочетание - не обрабатываем  
  if (e.ctrlKey || e.altKey || e.metaKey) return;
  if (!char) return; // спец. символ - не обрабатываем
  
  this.value = char.toUpperCase();
  
  return false;
}
</script>
[/online]

## Невосместимости [#keyboard-events-order]

Некоторые несовместимости в порядке срабатывания клавиатурных событий (когда что) ещё существуют.

Стоит иметь в виду три основных категории клавиш, работа с которыми отличается.

<table>
<thead>
<tr>
  <th>Категория</th>
  <th>События</th>
  <th>Описание</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Печатные клавиши [key S] [key 1] [key ,]</td>
  <td>`keydown`
`keypress`
`keyup`</td>
<td>Нажатие  вызывает `keydown` и `keypress`. 
Когда клавишу отпускают, срабатывает `keyup`. 

Исключение -- CapsLock под MacOS, с ним есть проблемы:
<ul>
<li>В Safari/Chrome/Opera: при включении только `keydown`, при отключении только `keyup`.</li>
<li>В Firefox: при включении и отключении только `keydown`.</li>
</ul>
</td>
 
<tr>
  <td>Специальные клавиши [key Alt] [key Esc] [key ⇧]</td>
  <td>`keydown`
`keyup`</td>
<td>Нажатие  вызывает `keydown`. 
Когда клавишу отпускают, срабатывает `keyup`. 

Некоторые браузеры могут дополнительно генерировать и `keypress`, например IE для [key Esc]. 

На практике это не доставляет проблем, так как для специальных клавиш мы всегда используем `keydown/keyup`.
 </td>
</tr>


<tr>
  <td>Сочетания с печатной клавишей 
 [key Alt+E]
 [key Ctrl+У]
 [key Cmd+1]
</td>
  <td>`keydown`
`keypress?`
`keyup`</td>
<td>
Браузеры под Windows -- не генерируют `keypress`, браузеры под MacOS -- генерируют. 

Кроме того, если сочетание вызвало браузерное действие или диалог ("Сохранить файл", "Открыть" и т.п., ряд диалогов можно отменить при `keydown`), то может быть только `keydown`.
</td>
</tr>
</tbody>
</table>

Общий вывод можно сделать такой:
<ul>
<li>Обычные символы работают везде корректно.</li>
<li>CapsLock под MacOS ведёт себя плохо, не стоит ставить на него обработчики вообще.</li>
<li>Для других специальных клавиш и сочетаний с ними следует использовать только `keydown`.</li>
</ul>

## Автоповтор

При долгом нажатии клавиши возникает *автоповтор*. По стандарту, должны генерироваться многократные события `keydown (+keypress)`, и вдобавок стоять свойство [repeat=true](http://www.w3.org/TR/DOM-Level-3-Events/#events-KeyboardEvent-repeat) у объекта события.

То есть поток событий должен быть такой:

```
keydown
keypress
keydown
keypress
..повторяется, пока клавиша не отжата...
keyup
```

Однако в реальности на это полагаться нельзя. На момент написания статьи, под Firefox(Linux) генерируется и `keyup`:

```
keydown
keypress
keyup
keydown
keypress
keyup
..повторяется, пока клавиша не отжата...
keyup
```

...А Chrome под MacOS не генерирует `keypress`. В общем, "зоопарк". 

Полагаться можно только на `keydown` при каждом автонажатии и `keyup` по отпусканию клавиши. 

## Итого 

Ряд рецептов по итогу этой главы:

<ol>
<li>Для реализации горячих клавиш, включая сочетания -- используем `keydown`. Скан-код будет в `keyCode`, почти все скан-коды кросс-браузерны, кроме нескольких пунктуационных, перечисленных в таблице выше.</li>
<li>Если нужен именно символ -- используем `keypress`. При этом функция `getChar` позволит получить символ и отфильтровать лишние срабатывания. Гарантированно получать символ можно только при нажатии обычных клавиш, если речь о сочетаниях с модификаторами, то `keypress` не всегда генерируется.</li>
<li>Ловля CapsLock глючит под MacOS. Её можно организовать при помощи проверки `navigator.userAgent` и `navigator.platform`, а лучше вообще не трогать эту клавишу.</li>
</ol>

Распространённая ошибка -- использовать события клавиатуры для работы с полями ввода в формах.

Это нежелательно. События клавиатуры предназначены именно для работы с клавиатурой. Да, их можно использовать для проверки ввода в `<input>`, но будут недочёты. Например, текст может быть вставлен мышкой, при помощи правого клика и меню, без единого нажатия клавиши. И как нам помогут события клавиатуры?

Некоторые мобильные устройства также не генерируют `keypress/keydown`, а сразу вставляют текст в поле. Обработать ввод на них при помощи клавиатурных событий нельзя.

Далее мы разберём [события для элементов форм](/events-change), которые позволяют работать с вводом в формы правильно.

Их можно использовать как отдельно от событий клавиатуры, так и вместе с ними.
 
