
Страшновато выглядит, да? Работает так (по строкам):
<ol>
<li>Вызов `bind` сохраняет дополнительные аргументы `args` (они идут со 2го номера) в массив `bindArgs`.</li>
<li>... и возвращает обертку `wrapper`.</li>
<li>Эта обёртка делает из `arguments` массив `args` и затем, используя метод [concat](http://javascript.ru/Array/concat), прибавляет их к аргументам `bindArgs` (карринг).</li>
<li>Затем передаёт вызов `func` с контекстом и общим массивом аргументов.</li>
</ol>
