# Класс "расширенные часы"

[importance 5]

Есть реализация часиков на прототипах. Создайте класс, расширяющий её, добавляющий поддержку параметра `precision`, который будет задавать частоту тика в `setInterval`. Значение по умолчанию: `1000`.



<ul>
<li>Для этого класс `Clock` надо унаследовать. Пишите ваш новый код в файле `extended-clock.js`.</li>
<li>Исходный класс `Clock` менять нельзя.</li>
<li>Пусть конструктор потомка вызывает конструктор родителя. Это позволит избежать проблем при расширении `Clock` новыми опциями.</li>
</ul>

P.S. Часики тикают в браузерной консоли (надо открыть её, чтобы увидеть).