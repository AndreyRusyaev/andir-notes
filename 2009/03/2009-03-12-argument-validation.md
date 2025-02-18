Упрощённая валидация аргументов в .Net
======================================

        published: 2009-03-12 
        tags: .net,aop,validation 
        permalink: https://andir-notes.blogspot.com/2009/03/argument-validation.html

Для валидации аргументов методов в .Net присутствует набор исключений, которые бросаются при нарушении контракта метода:

*   **ArgumentException** – обобщённое исключение предназначенное для наложения произвольных ограничений на аргументы метода,
*   **ArgumentNullException** – исключение, указывающее что значение аргумента оказалось равным _null_,
*   **ArgumentOutOfRangeException** – исключение, указывающее что значение аргумента выходит за пределы интервала разрешённых значений.

Одним из параметров конструктора этого типа исключений является имя аргумента, для которого проводится валидация:

``` cs
void Foo(string fooArg)
{
    if (fooArg == null)
        throw new ArgumentNullException("fooArg");

    Console.WriteLine("Foo({0})", fooArg);
}

void Bar(int barArg)
{
    if (barArg < 0 || barArg \> 10)
        throw new ArgumentOutOfRangeException("barArg");

    Console.WriteLine("Bar({0})", barArg);
}
```

Собственно, это всё довольно известно и часто используется.

Однако в таком коде присутствуют и свои, довольно очевидные, проблемы.

*   Отсутствует декларативность (явно используемые проверки с помощью if),
*   Использование строковых констант в качестве имён аргументов (мешает использованию автоматического рефакторинга _Rename_).

Попробуем избавится от этих проблем с помощью типизированного варианта получения имени аргумента. Для этого воспользуемся возможностями Linq Expressions. У меня получился следующий вариант:

**Program.cs**

``` cs
using System;

namespace Home.Andir.Examples
{
    class Program
    {
        static void Main(string[] args)
        {
            var p = new Program();
            p.Foo(null);
            p.Bar(-1);
        }

        void Foo(string fooArg)
        {
            Argument.NotNull(() => fooArg);
            Console.WriteLine("Foo({0})", fooArg);
        }

        void Bar(int barArg)
        {
            Argument.InRange(() => barArg,
                new Range<int> { Begin = 0, End = 10 });

            Console.WriteLine("Bar({0})", barArg);
        }
    }
}
```

В коде явно прибавилось декларативности и исчезли проблемы со строковым именем проверяемого аргумента. А итоговая функциональность кода совсем не изменилась. Отлично.

Посмотрим на реализацию класса **Argument**.

**Argument.cs**

``` cs
using System;

using System.Linq.Expressions;

namespace Home.Andir.Examples
{
    public static class Argument
    {
        private static string GetArgumentName<T>(Expression<Func<T>> argumentExpression)
        {
            var memberExpression = argumentExpression.Body as MemberExpression;

            if (memberExpression == null)
                throw new ArgumentException(
                    "Only MemberExpression allowed for expression body.",
                    "argumentExpression");

            return memberExpression.Member.Name;
        }

        private static T GetArgumentValue<T>(Expression<Func<T>> argumentExpression)
        {
            var compiledExpr = argumentExpression.Compile();
            return compiledExpr.Invoke();
        }

        public static void NotNull<T>(
            Expression<Func<T\>> argumentExpression)
        {
            var argName = GetArgumentName(argumentExpression);
            var argValue = GetArgumentValue(argumentExpression);
            if (argValue == null)
                throw new ArgumentNullException(argName);
        }

        public static void InRange<T>(
            Expression<Func<T>> argumentExpression,
            Range<T> range)
            where T : IComparable<T>
        {
            var argName = GetArgumentName(argumentExpression);
            var argValue = GetArgumentValue(argumentExpression);
            if (argValue.CompareTo(range.Begin) < 0
                || argValue.CompareTo(range.End) > 0)
                throw new ArgumentOutOfRangeException(argName);
        }
    }
}
```

И небольшой класс для задания интервала **Range**.

**Range.cs**

``` cs
using System;

namespace Home.Andir.Examples
{
    public class Range<T> where T : IComparable<T>
    {
        public T Begin { get; set; }
        public T End { get; set; }
    }
}
```

Реализация, конечно, получилась чисто игрушечная, но её вполне достаточно чтобы показать основную идею.

Такой вариант, конечно, не является идеальным, но позволяет используя только средства языка C# 3.0 упростить и улучшить механизм валидации аргументов.

А более красивый и практически идеальный вариант можно получить с помощью [AOP](http://ru.wikipedia.org/wiki/AOP "Википедия: Аспектно-ориентированное программирование"). Для моего примера можно предположить такую реализацию:

``` cs
void Foo([NotNull] string fooArg)
{
    Console.WriteLine("Foo({0})", fooArg);
}

void Bar([InRange(0, 10)] int barArg)
{
    Console.WriteLine("Bar({0})", barArg);
}
```

Здесь условия на значения аргументов накладываются с помощью атрибутов. Максимально декларативно и без потери читабельности.

На сегодняшний день подобный вариант можно организовать, например, с помощью [PostSharp](http://www.postsharp.org/ "PostSharp: Bringing AOP to .Net"). В частности, стоит обратить внимание на проект [ValidationAspects](http://www.codeplex.com/ValidationAspects "CodePlex: проект ValidationAspects"), который также использует PostSharp для реализации аспектов.

Похожий вариант валидации аргументов возможно будет организовывать и в будущей версии языка C# 4.0 (прочитать об этом можно [здесь](http://www.codinginstinct.com/2008/05/argument-validation-using-attributes.html "Coding Instinct: Argument validation using attributes and method interception")).