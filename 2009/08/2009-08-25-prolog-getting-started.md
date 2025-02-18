Пролог: Изучение с нуля
=======================

    published: 2009-08-25 
    tags: prolog 
    permalink: https://andir-notes.blogspot.com/2009/08/prolog-getting-started.html

Добрался таки до пролога. Чтобы не потерять, все источники собираю в одном месте.

**Реализации**:

*   [SWI-Prolog](http://www.swi-prolog.org/),
*   [GNU Prolog](http://www.gprolog.org/),
*   [B Prolog](http://www.probp.com/),
*   [SICStus Prolog](http://www.sics.se/isl/sicstuswww/site/index.html) (платный, 30 дневный trial выдаётся по запросу),
*   [Strawberry Prolog](http://www.dobrev.com/) (Light версия распространяется бесплатно).

**Редакторы кода**:

*   PceEmacs (встроенный Emacs для SWI-Prolog),
*   [Emacs](http://www.gnu.org/software/emacs/) ([prolog-mode](http://www.swi-prolog.org/FAQ/GnuEmacs.html)),
*   [Visual Prolog](http://www.visual-prolog.com/),
*   [Prolog Development Tool for Eclipse](http://sewiki.iai.uni-bonn.de/research/pdt/start).

**Источники первичной информации**:

*   [Wikipedia: Prolog](http://en.wikipedia.org/wiki/Prolog),

       Большая статья из которой можно почерпнуть практически всю доступную информацию о языке Prolog.

*   [Wikibooks: Prolog](http://en.wikibooks.org/wiki/Prolog),

        Очень хорошее начало, но, к сожалению, содержит пропуски в некоторых частях, для полноценного изучения не годится.

*   Prolog ISO/IEC 13211-1:1995 - стандарт языка,

       В 1995 году был принят стандарт языка, но к сожалению в свободном доступе его обнаружить не удалось, есть только [черновик](http://www.cs.cmu.edu/afs/cs/project/ai-repository/ai/lang/prolog/doc/standard/) в PostScript формате.

*   [Prolog: ISO Standard documents](http://pauillac.inria.fr/~deransar/prolog/docs.html).

        Набор документов и программ использованных при разработке стандарта языка.

**Туториалы**:

*   [Prolog Tutorial](http://www.csupomona.edu/~jrfisher/www/prolog_tutorial/intro.html),
*   [A Prolog Introduction for Hackers](http://www.kuro5hin.org/print/2004/2/25/124713/784),

**Книги**:

*   "Программирование на языке Пролог", У. Клоксин К. Меллиш ("Programming in Prolog" by W.F. Clocksin and C.S. Mellish),
*   ["Декларативное программирование"](http://www.softcraft.ru/paradigm/dp/index.shtml), И.А.Дехтяренко,
*   Программирование на языке Пролог для искусственного интеллекта, И. Братко,
*   [Learn Prolog Now](http://www.learnprolognow.org) by Patrick Blackburn, Johan Bos and Kristina Striegnitz  ([бесплатная онлайн-версия](http://cs.union.edu/~striegnk/learn-prolog-now/html/index.html)).

**Задачи**:

*   [99 Prolog problems](https://prof.ti.bfh.ch/hew1/informatik3/prolog/p-99/),

**Другое**:

*   [Free stuff for programming in Prolog](http://personnel.univ-reunion.fr/fred/Enseignement/Prolog/freestuff.html),
*   [The Prolog Dictionary](http://www.cse.unsw.edu.au/~billw/prologdict.html),
*   [Higher-order logic programming in Prolog](http://www.cs.mu.oz.au/~lee/papers/ho/) (статья о программировании с правилами высшего порядка, сама статья в pdf лежит [отдельно](http://www.cs.umbc.edu/771/papers/mu_96_02.pdf)).

#### Очень краткое введение

Пролог – это декларативный логический язык программирования. Основными его конструкциями являются логические предложения (например: human('Socrat') :- true.), на основании которых пролог строит некоторые логические выводы. Предложения объединяются в правила с помощью операции and, которая обозначается знаком ','. Предложение при проверке может создавать побочные эффекты, что используется для ввода/вывода.

#### Краткий путеводитель по синтаксису

``` prolog
% Это строчный комментарий
/* А это блочный комментарий */

goal(X, Y) :-   % goal - это цель, для которой мы собираем правила,
                % имя цели всегда начинается с маленькой буквы.
                % результат у цели - это всегда true или false.
                % X, Y - параметры цели.

    X > Y       % Первое утверждение, которое утверждает что X больше Y.

    ,           % Разделитель между утверждениями, означает логическое И.

    X + 1 == Y  % Второе утверждение, которое может быть верным,
                % только если Y = X + 1.

    .           % Точка означает конец описания цели.

/*
 * Суммирую:
 * Имя цели всегда с маленькой буквы,
 * Параметры цели, и все другие переменные с большой буквы,
 * Утверждения разделяются запятой,
 * Описание цели всегда заканчивается точкой.
 */

prolog('logic language').
% Так записываются некоторые факты,
% и по сути являются сокращениями для цели,
% которая всегда возвращает true.
% в данном случае эквивалент выглядел бы так:
% prolog('logic language') :- true.

% Цели могут быть рекурсивными.
% Посчитаем сумму от 0 до X.
sum(0, 0).   % факт используется для остановки рекурсии.
sum(X, Result) :-
    Y is X - 1,             % is - это по сути присвоение значения.
    sum(Y, SubResult),      % Рекурсивный вызов.
    Result is X + SubResult.


% Цель, которая будет использоваться при запуске программы,
% имя 'main' выбрано мною произвольно.
main :-
    % запрашиваем значение факта
    prolog(Which),
    % выводим значение на экран
    format('prolog is a ~a.', [Which]),

    % newline
    nl,

    % задаём значение X
    X is 100,
    % считаем сумму от 0 до 100
    sum(X, Result),
    % выводим результат с помощью printf-like функции
    format('sum(0, ~a) = ~a.', [X, Result]),

    % newline.
    nl.

/*
 * Вывод программы при запуске:
 * prolog is a logic language.
 * sum(0, 100) = 5050.
 */
```

#### Типы данных

**Атом**: атом или 'атом', начинается с маленькой буквы или обрамляется одинарными кавычками.

**Строка**: "Строка", обрамляется двойными кавычками, по факту представляет список из чисел – кодов ASCII символов строки.

**Число**: 10, целые (-1,0,1), плавающая запятая (0.123, 123.456).

**Список**: \[0, 1, 2\], набор элементов перечисленных через запятую и обрамлённых квадратными скобками. Элементы могут быть разнородными (разных типов). Списки можно деструктурировать через синтаксис \[Head | Tail\].

#### Первые результаты

Теперь, с помощью новоприобретённых знаний, можно написать несколько известных примеров:

**HelloWorld.pro**:

``` prolog
main :- writeln('Hello, Prolog World!'). 
```

**Factorial.pro**:

``` prolog
% Factorial implementation

factorial(1, 1).
factorial(X, Result) :-
    X > 0,
    X1 is X - 1,
    factorial(X1, Result1),
    Result is X * Result1. 

main :-
    factorial(7, X),
    writeln(X). 
```

**Fibonachi.pro**:

``` prolog
% Fibbonachi sequence implementation

fib(0, 1).
fib(1, 1).
fib(X, Result) :-
    X1 is X - 1,
    X2 is X - 2,
    fib(X1, Result1),
    fib(X2, Result2),
    Result is Result1 + Result2.

main :-
    fib(7, X),
    writeln(X). 
```

**QuickSort.pro:**

``` prolog
% QuickSort algorithm implementation

qsort([], []).
qsort([X], [X]).
qsort([Head | Tail], Result) :-
    partition(Head, Tail, Left, Right),
    qsort(Left, LeftSorted),
    qsort(Right, RightSorted),
    append(LeftSorted, [Head | RightSorted], Result). 

partition(_, [], [], []).
partition(Pivot, [Head | Tail], [Head | Left], Right) :-
    Head =< Pivot,
    partition(Pivot, Tail, Left, Right).
partition(Pivot, [Head | Tail], Left, [Head | Right]) :-
    Head > Pivot,
    partition(Pivot, Tail, Left, Right). 

append([], Right, Right).
append([Head | Left], Right, [Head | Result]) :-
    append(Left, Right, Result). 

main :-
    qsort([1, 5, 4, 6, 3, 2], X),
    writeln(X). 
```

#### Какие ещё есть логические языки программирования?

Путём [нехитрых](http://google.com/search?q=programming+logical+language) поисков в источниках близким к прологу собрал такой список:

*   [Curry Language](http://www.curry-language.org/), "A Truly Integrated Functional Logic Language":

        Расширение языка Haskell для реализации логического программирования.

*   [Mercury Project](http://www.cs.mu.oz.au/research/mercury/index.html),

       Mercury is a new logic/functional programming language, which combines the clarity and expressiveness of declarative programming with advanced static analysis and error detection features.

*   [Oz Language](http://www.mozart-oz.org/),

        Мультипарадигменный язык, который включает в себя подмножество логического программирования.

*   [ALF – Algebraic logical functional language](http://www.informatik.uni-kiel.de/~mh/systems/ALF.html),

        Язык, комбинирующий функциональное и логическое программирование. По описанию сильно похоже, что к базе логического программирования добавлены функциональные возможности (функции, операторы, типы, структуры и т.п.).