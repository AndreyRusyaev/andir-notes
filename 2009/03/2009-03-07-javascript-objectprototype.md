JavaScript: Никогда не переопределяйте Object.prototype!!
=========================================================

    published: 2009-03-07 
    tags: wtf,javascript 
    permalink: https://andir-notes.blogspot.com/2009/03/javascript-objectprototype.html

Вот уже в который раз наблюдаю использование в коде сторонней библиотеки попытки сделать _всё красиво_ за счёт добавления функций в **Object.prototype**.

``` js
Object.prototype.Foo = function()
{
    // Что-то делаем здесь.
}
```

На первый взгляд может показаться, что нет ничего страшного в таком коде. Но, стоит только вспомнить один из самых распространённых вариантов использования объектов в JavaScript: “Object as hash-tables” и:

``` js
var myHashTable = { "key1" : "value1", "key2" : "value2" };

for (var key in myHashTable)
{
    // используется функция логирования от расширения Firebug (http://getfirebug.org/).

    console.log("key = '" + key + "', value = '" + myHashTable[key] + "';");
}

// Вывод:
// key = 'key1', value = 'value1';
// key = 'key2', value = 'value2';
// key = 'Foo', value = 'function () { }'; // WTF???
```

Упс. В импровизированной хэш-табличке, внезапно, появился новый забавный элемент.

Неприятно обнаруживать, что ошибки в твоём коде обусловлены использованием чужой библиотекой. Да и избавиться от такого бага довольно непросто: он заставляет либо отказаться от использования “Объект как хэш-таблица” (подменив на свою реализацию), либо от использования сторонней библиотеки (в некоторых случаях гораздо проще).

Чтобы обнаружить этот баг на ранней стадии, имеет смысл добавить в свои unit-тесты следующий тест:

``` js
testObjectAsHashTableAreEmptyByDefault = function()
{
    var obj = {};

    var keysCount = 0;
    for (var key in obj) { keysCount++; }

    assertEquals(keysCount, 0);
}
```

Разработчикам же, библиотек для JavaScript нелишне будет запомнить, что **Object.prototype** – это тайна за семью печатями (sealed).