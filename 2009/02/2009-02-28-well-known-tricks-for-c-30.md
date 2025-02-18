Пара интересных трюков для C# 3.0
=================================

2009-02-28 .net,trick https://andir-notes.blogspot.com/2009/02/well-known-tricks-for-c-30.html

В заметке я хочу обсудить применение двух трюков, ставших возможными после введения новых синтаксических конструкций в C# 3.0, таких как анонимные типы и лямбда-выражения (анонимные функции).

Эти трюки многим, конечно же, уже известны.

#### Анонимные типы как словарь

Итак, у нас появилась возможность задавать в области действия функций локальные анонимные типы:

``` cs
var obj = new { Name = "Object", Color = "Black" };
```

Анонимный тип, по вполне понятным причинам, можно передать другой функции только нетипизировано, в виде объекта типа **System.Object**. Что делает его довольно бесполезным вне функции.

Идея состоит в том, чтобы преобразовать анонимный объект в словарь типа ключ-значение. И использовать такие объекты как альтернативный вариант для функций принимающих параметры в виде такого словаря.

Реализуем небольшой метод-расширение, который принимает произвольный объект и возвращает словарь со свойствами и их значениями из объекта.

**ObjectExtensions.cs**

``` cs
using System.Collections.Generic;

namespace Home.Andir.Examples
{
    static class ObjectExtensions
    {
        public static Dictionary<string, object> ToDictionary(this object obj)
        {
            return obj.ToDictionary<object>();
        }

        public static Dictionary<string, T> ToDictionary<T>(this object obj)
        {
            var dict = new Dictionary<string, T>();
            foreach (var prop in obj.GetType().GetProperties())
            {
                dict.Add(prop.Name, (T)prop.GetValue(obj, null));
            }
            return dict;
        }
    }
}
```

Применение этого “трюка” можно найти в ASP.Net MVC в виде экземпляра объекта **HtmlHelper**. Напишем примерную реализацию метода **HtmlHelper**._Link_, который сгенерирует содержимое произвольного тега “**a”**.

Вначале применение:

``` cs
HtmlHelper.Link("Ссылка",
    new { href \= "http://example.org/", title \= "Нажмите, чтобы перейти далее." }
    );
```

Неправда ли, немного элегантнее и чуть меньше символов, чем:

``` cs
HtmlHelper.Link("Ссылка",
    new Dictionary<string, string\>{
        { "href", "http://example.org/" },
        { "title", "Нажмите, чтобы перейти далее." } }
    );
```

А теперь реализация:

**HtmlHelper.cs**

``` cs
using System;
using System.Collections.Generic;
using System.Linq;

namespace Home.Andir.Examples
{
    class HtmlHelper
    {
        public static string Tag(string name, string text, object attrs)
        {
            return Tag(name, text, attrs.ToDictionary<string>());
        }

        public static string Tag(string name, string text,  Dictionary<string, string> attrs)
        {
            var attrString = String.Join(" ",
                attrs.Select(x => String.Format("{0}=\"{1}\"", x.Key, x.Value))
                     .ToArray()
                    );
            return String.Format(
                "<{0} {1}>{2}</{0}>", name, attrString, text);
        }

        public static string Link(string text, object attrs)
        {
            return Tag("a", text, attrs);
        }

        public static string Link(string text, Dictionary<string, string> attrs)
        {
            return Tag("a", text, attrs);
        }
    }
}
```

#### Статически-типизированный Reflection

Основная проблема, которая возникает при использовании механизма отражения (Reflection) – это использование строк в качестве аргументов при извлечении данных. Это сильно мешает использованию инструментов для автоматического рефакторинга.

Рассмотрим объект:

``` cs
class NamedObject
{
    [DisplayName("Имя")]
    public string Name { get; set; }
}
```

Пусть, у нас имеется нетипизированный экземпляр этого объекта и требуется узнать значение атрибута “DisplayName”.

**ComponentHelper.cs**

``` cs
using System;
using System.ComponentModel;
using System.Reflection;
namespace Home.Andir.Examples
{
    public static class ComponentHelper
    {
        public static string GetPropertyDisplayName(Type type, string propName)
        {
            return GetDisplayName(type.GetProperty(propName));
        }

        private static string GetDisplayName(PropertyInfo propInfo)
        {
            var displayName = propInfo.Name;

            foreach (var attr in propInfo.GetCustomAttributes(true))
                if (attr is DisplayNameAttribute)
                    displayName = (attr as DisplayNameAttribute).DisplayName;

            return displayName;
        }
    }
}
```

Теперь вызываем:

**Program.cs**

``` cs
using System;

namespace Home.Andir.Examples
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("DisplayName = \"{0}\"",
                ComponentHelper.GetPropertyDisplayName(typeof(NamedObject), "Name")
                );
            Console.ReadKey();
        }
    }
}
```

Вот тут мы и обнаруживаем проблему. Если использовать автоматический рефакторинг и переименовать свойство **NamedObject**._Name_, например, в **NamedObject**._Name1_, придётся ещё вручную переименовывать и имя свойства в вызове метода _GetPropertyDisplayName_.

Вот тут нам приходят на помощь, как это не странно, лямбда выражения, с их помощью предыдущий вызов можно переписать так:

**Program.cs**

``` cs
using System;

namespace Home.Andir.Examples
{
    class Program
    {
        static void Main(string[] args)
        {

            Console.WriteLine("DisplayName = \"{0}\"",
                ComponentHelper.GetPropertyDisplayName<NamedObject\>(x \=> x.Name)
                );
            Console.ReadKey();
        }
    }
}
```

Как видно, теперь у нас нет использования строки с именем свойства, и автоматический рефакторинг будет работать безопасно.

Реализация такого варианта следующая:

**ComponentHelper.cs**

``` cs
using System;
using System.ComponentModel;
using System.Reflection;
using System.Linq.Expressions;

namespace Home.Andir.Examples
{
    public static class ComponentHelper
    {
        public static string GetPropertyDisplayName<T>(
            Expression<Func<T, object\>> propertyExpression
            )
        {
            return GetDisplayName(
                StaticReflection.GetPropertyInfo<T>(propertyExpression));
        }

        public static string GetPropertyDisplayName(Type type, string propName)
        {
            return GetDisplayName(type.GetProperty(propName));
        }

        private static string GetDisplayName(PropertyInfo propInfo)
        {
            var displayName = propInfo.Name;

            foreach (var attr in propInfo.GetCustomAttributes(true))
                if (attr is DisplayNameAttribute)
                    displayName \= (attr as DisplayNameAttribute).DisplayName;

            return displayName;
        }
    }
}
```

**StaticReflection.cs**

``` cs
using System;
using System.Linq.Expressions;
using System.Reflection;

namespace Home.Andir.Examples
{
    public static class StaticReflection
    {
        public static PropertyInfo GetPropertyInfo<T>(
            Expression<Func<T, object>> propertyExpression)
        {
            var memberExpression = propertyExpression.Body as MemberExpression;
            if (memberExpression == null)
                throw new InvalidOperationException(
                    "Only member access allowed in lambda expression.");

            var propInfo = memberExpression.Member as PropertyInfo;
            if (propInfo == null)
                throw new InvalidOperationException(
                    "Only property access allowed in lambda expression.");
            return propInfo;
        }
    }
}
```

#### Эпилог

Применение этих трюков можно найти в [ASP.Net MVC](http://asp.net/mvc/ "ASP.Net MVC"), [Rhino Mocks](http://ayende.com/projects/rhino-mocks.aspx), [Autofac](http://code.google.com/p/autofac/), [AutoMapper](http://www.codeplex.com/AutoMapper) и многих других фреймворках. Уверен, что можно найти применение им и в своём коде ;-).