[TOC]

# Офис

### Ссылки

https://habr.com/ru/articles/319036/

https://habr.com/ru/articles/459514/

**Потоки выполнения**

- [ ] https://learn.microsoft.com/ru-ru/windows/win32/procthread/processes-and-threads
- [ ] https://habr.com/ru/articles/40227/
- [ ] [Параллельные вычисления, контекст синхронизации](https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext)
- [ ] [Порядок выполнения, поток управления](https://ru.m.wikipedia.org/wiki/%D0%9F%D0%BE%D1%80%D1%8F%D0%B4%D0%BE%D0%BA_%D0%B2%D1%8B%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F)
- [ ] [Чувак, это база!](https://learn.microsoft.com/ru-ru/dotnet/standard/threading/managed-threading-basics)
- [ ] [управление памятью](https://learn.microsoft.com/ru-ru/dotnet/standard/managed-code)

**Threads**

- [ ] https://code-maze.com/csharp-new-thread/
- [ ] https://blog.stephencleary.com/2013/11/there-is-no-thread.html
- [ ] https://learn.microsoft.com/en-us/dotnet/api/system.threading.threadpool?view=net-6.0#remarks

Асинхронность

- [ ] https://learn.microsoft.com/ru-ru/dotnet/csharp/asynchronous-programming/

**CodeMaze**

- [x] https://code-maze.com/csharp-locking-mechanism/
- [ ] https://code-maze.com/csharp-volatile-interlocked-lock/
- [ ] https://code-maze.com/csharp-tasks-vs-threads/
- [ ] https://code-maze.com/csharp-difference-between-await-and-task-wait/
- [ ] https://code-maze.com/charp-difference-between-returning-and-awaiting-a-task/
- [ ] https://code-maze.com/csharp-avoid-await-in-loop/

**Другое**
- [ ] https://dev.to/rasulhsn/introduction-to-monitor-class-in-c-422
- [ ] https://dev.to/rasulhsn/deep-dive-into-monitor-class-as-hybrid-synchronization-construct-in-net-cn8
- [ ] https://www.youtube.com/watch?v=3eegNyisX2I
- [ ] https://www.youtube.com/watch?v=PEHQjSwp01k

**ParalellForeach**

- [ ] https://www.c-sharpcorner.com/UploadFile/efa3cf/parallel-foreach-vs-foreach-loop-in-C-Sharp/

### **План в офис**

##### Теория

1. Операционка
   - [ ] CAS, Kernel
   - [ ] ...
   - [ ] ...
2. Вся теория по потокам, контекстам, комментарии F12
   - [x] База MSDN
   - [x] CodeMaze
   - [ ] комментарии F12
   - [ ] Две лекции Сидра
   - [ ] https://professorweb.ru/my/csharp/thread_and_files/1/thread_index.php

**Практика**

- [x] Monitor, Mutex, Semaphore
- [ ] [SpinLock](https://learn.microsoft.com/ru-ru/dotnet/standard/threading/how-to-use-spinlock-for-low-level-synchronization)
- [ ] [Отслеживание потоков SpinLock](https://learn.microsoft.com/ru-ru/dotnet/standard/threading/how-to-enable-thread-tracking-mode-in-spinlock)
- [ ] ParallelForeach()
- [ ] Task
- [ ] async/await
- [ ] [Визуализатор параллелизма](https://learn.microsoft.com/ru-ru/visualstudio/profiling/concurrency-visualizer?view=vs-2022)

------

##### Проект Егора

```
//PASSPORTS.SSV 1 млн
Приходит каждый день
Какие паспорта устарели, сравнивать с текущей датой
Приходит каждый день
Своя БД
Тесты, запускающие в CI/CD
Docker
```

# Глава 1. Основные определения

​	*Конкурентность* - выполнение сразу нескольких действий в одно и то же время.

​	*Многопоточность* - форма конкурентности, использующая несколько программных потоков выполнения.

​	*Параллельная обработка* - выполнение большого объема работы за счет распределения ее между несколькими потоками, выполняемыми одновременно. Использует многопоточность для максимально эффективного использования много­ядерных процессоров.

[^Примечание ]: Параллельная обработка является одной из разновидностей *многопоточ­ности*, а многопоточность является одной из разновидностей *конкурент­ности*.

​	*Асинхронное программирование* - разновидность конкурентности, использующая *обещания* или обратные вызовы для предотвращения создания лишних потоков, не блокируя основной поток (набор операторов выполняется независимо от основного потока программирования).

##### Разница между асинхронным программированием и многопоточностью в C#

​	*Многопоточность* – это метод программирования для выполнения операций, запущенных в нескольких потоках (также называемых рабочими), где мы используем разные потоки и блокируем их до тех пор, пока работа не будет выполнена.

​	*Асинхронное программирование* – это одновременное выполнение нескольких задач (здесь назначенный поток возвращается обратно в пул потоков, как только в методе достигается ключевое слово `await`).



# Глава 2. Потоки выполнения

​	*Процесс* — это исполнение программы. Операционная система использует процессы для разделения исполняемых приложений. 

​	Поток — это основная единица, которой операционная система выделяет время процессора.   В контексте процесса могут выполняться несколько потоков.

​	Фоновые потоки:

- фоновый поток не поддерживает работу управляемой среды выполнения
- все `new Thread`
- входящие в пул управляемых потоков (`IsThreadPoolThread` = `true`*)

## 2.1. Использование потоков и работа с потоками

#### Извлечение данных из потоков с помощью методов обратного вызова

https://learn.microsoft.com/ru-ru/dotnet/standard/threading/creating-threads-and-passing-data-at-start-time



[^Примечание]: После того, как в управляемом процессе (где управляемой сборкой является файл EXE) остановятся все основные потоки, система принудительно останавливает все фоновые потоки и завершает работу процесса.



> [!WARNING]
>
> Не блокируйте методы экземпляра, используя `this` !



#### Отмена потоков

 [`System.Threading.CancellationToken`](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.cancellationtoken)

​	Хотя класс [`System.Threading.Thread`](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.thread) не обеспечивает встроенную поддержку токенов отмены, токен можно передать процедуре потока с помощью конструктора [`Thread`](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.thread) , принимающего делегат [`ParameterizedThreadStart`](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.parameterizedthreadstart) .



#### Взаимоблокировки

Взаимоблокировка происходит, когда каждый из двух потоков пытается заблокировать ресурс, уже заблокированный другим потоком. Ни один из потоков не может продолжить работу.

#### Состояние гонки

​	Это ошибка, которая возникает, когда результат программы зависит от того, какой из двух или более потоков первым достигнет определенного блока кода.

[^]: [Environment.ProcessorCount](https://learn.microsoft.com/ru-ru/dotnet/api/system.environment.processorcount#system-environment-processorcount), чтобы определить количество процессоров, доступных во время выполнения.

#### Общие рекомендации

Вместо оператора `lock`  для простого изменения состояния лучше использовать методы класса [Interlocked](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.interlocked). Оператор `lock` — хороший универсальный инструмент, но класс [Interlocked](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.interlocked) обеспечивает высокую производительность для обновлений, которые должны быть атомарными. Если конкуренции нет, он выполняет внутри единственный префикс lock. При проверке кода ищите код, похожий на показанный в следующих примерах. В первом примере увеличивается переменная состояния:

```c#
lock(lockObject)
{  
    myField++;  
}
```

​	Вы можете повысить производительность, применяя метод [Increment](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.interlocked.increment) вместо оператора `lock`, как показано ниже.

```c#
System.Threading.Interlocked.Increment(myField);
```



## 2.2. Пул управляемых потоков

Потоки из пула являются [фоновыми](https://learn.microsoft.com/ru-ru/dotnet/standard/threading/foreground-and-background-threads). Для каждого потока используется размер стека по умолчанию, поток запускается с приоритетом по умолчанию и находится в многопотоковом подразделении. 

​	Когда поток в пуле завершает свою задачу, он возвращается в очередь потоков в состоянии ожидания. С этого момента его можно использовать вновь. Повторное использование позволяет приложениям избежать дополнительных затрат на создание новых потоков для каждой задачи. Для каждого процесса существует только один пул потоков.

#### Пул потоков не подходит:

- Необходим основной поток.
- Поток должен иметь определенный приоритет.
- Имеются задачи, которые приводят к блокировке потока на длительное время (большое число заблокированных потоков в пуле может препятствовать запуску задач).
- Необходимо иметь назначить конкретный поток конкретной задаче.



------

# Глава 3. Примитивы синхронизации

#### Monitor

Класс [`System.Threading.Monitor`](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.monitor) предоставляет монопольный доступ к общему ресурсу, блокируя или разблокируя объект, определяющий ресурс. 

​	Во время блокировки поток, удерживающий блокировку, может снова поставить и снять блокировку. Любой другой поток не может получить блокировку, и метод [Monitor.Enter](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.monitor.enter) ожидает снятия блокировки.

> [!IMPORTANT]
>
> `lock` реализуется с помощью методов [Enter](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.monitor.enter), [Exit](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.monitor.exit) и блока `try…finally`, обеспечивающих постоянное снятие полученной блокировки.

#### Mutex

Как и [Monitor](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.monitor), предоставляет монопольный доступ к общему ресурсу. В отличие от [Monitor](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.monitor), класс [Mutex](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.mutex) может использоваться для межпроцессной синхронизации. Для этого нужно использовать именованный мьютекс, который виден в операционной системе.

#### Структуры SpinLock и SpinWait

Как и [Monitor](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.monitor), предоставляет монопольный доступ к общему ресурсу на основе доступности блокировки. Когда [SpinLock](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.spinlock) пытается получить блокировку, которая недоступна, этот примитив будет ожидать в цикле, постоянно проверяя возможность получения блокировки.

#### Классы Semaphore и SemaphoreSlim

Ограничивают число потоков, которые могут одновременно обращаться к ресурсу или пулу ресурсов. Потоки, запрашивающие ресурс, ожидают освобождения семафора любым из потоков.

[SemaphoreSlim](https://learn.microsoft.com/ru-ru/dotnet/api/system.threading.semaphoreslim) —  можно использовать для синхронизации в рамках одного процесса.

#### Interlocked - класс

### Классы и методы

#### Класс Thread

##### `ThreadState`

##### `Thread.Yield()`

##### `Thread.IsBackground`()

##### `Thread.Join`

​	Используйте метод, чтобы вызывающий поток ждал завершения останавливаемого потока.

#### Класс ThreadPool

#### Класс Interlocked

​	Мы можем использовать `Interlocked`класс для переменных, которые совместно используются несколькими потоками. Этот класс не может заблокировать весь блок кода, но может обеспечить потокобезопасную простую операцию с одной переменной. Он не блокирует другие потоки.

#### Класс Monitor

![Механизм блокировки с классом Monitor](https://code-maze.com/wp-content/uploads/2023/07/MonitorClass.png)

##### `Pulse All(Object)`

​	Поток, который в данный момент владеет блокировкой указанного объекта, вызывает этот метод, чтобы подать сигнал всем потокам, ожидающим получения блокировки объекта. После отправки сигнала ожидающие потоки перемещаются в очередь готовности. Когда вызвавший поток `PulseAll`снимает блокировку, следующий поток в очереди готовности получает блокировку.

#### Класс Mutex

> **На одном компьютере может работать только один экземпляр класса Mutex**!

#### Класс SpinLock 

В отличие от `lock` и `Monitor`  не блокирует поток.

`SpinLock` вращается в цикле, ожидая получения блокировки в критические секунды, предполагая, что она скоро станет доступной.`SpinLock`следует использовать только для коротких частых методов.

```c#
public static class SpinLockClass
{
    private static bool lockAquired;
    private static int _counter;
    private static SpinLock spinLock = new();

    public static int Execute()
    {
        var thread1 = new Thread(IncrementCounter);
        var thread2 = new Thread(IncrementCounter);
        var thread3 = new Thread(IncrementCounter);

        thread1.Start();
        thread2.Start();
        thread3.Start();

        thread1.Join();
        thread2.Join();
        thread3.Join();

        return _counter;
    }

    private static void IncrementCounter()
    {
        try
        {
            if (lockAquired)
            {
                Thread.Sleep(100);
            }

            spinLock.Enter(ref lockAquired);
            for (var cnt = 0; cnt < 100000; cnt++)
            {
                _counter++;
            }
        }
        finally
        {
            if (lockAquired)
            {
                lockAquired = false;
                spinLock.Exit();
            }
        }
    }
}
```

Класс `SpinLock` использует `boolean` переданную переменную `by ref` и устанавливает ее `true`, если блокировка была успешно получена.

Независимо от получения блокировки, мы сбрасываем переменную в `finally` блоке и снимаем блокировку с помощью `SpinLock.Exit()` метода.

#### Класс Semaphore 

Мы используем `Semaphore`  для ограничения количества потоков, обращающихся к ограниченному пулу ресурсов.

# Глава 4.  Асинхронность

### Что происходит на внутреннем уровне

С точки зрения C#, компилятор преобразовывает код в конечный автомат, который контролирует такие моменты, как передача выполнения при достижении `await` и возобновление выполнения после завершения фонового задания.

https://learn.microsoft.com/ru-ru/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap

Перед написанием любого кода нужно ответить на два вопроса.

1. Будет ли код "ожидать" чего-либо, например данных из базы данных?

   Если ответ утвердительный, то ваша задача **ограничена производительностью ввода-вывода**.

2. Будет ли код выполнять сложные вычисления?

   Если ответ утвердительный, то задача **ограничена ресурсами процессора**.

Если ваша задача **ограничена производительностью ввода-вывода**, используйте `async` и `await` *без* `Task.Run`. Библиотеку параллельных задач использовать *не следует*.

Если ваша задача **ограничена ресурсами процессора** и вам важна скорость реагирования, используйте `async` и `await`, но перенесите выполнение задачи в дополнительный поток, у которого *есть* `Task.Run`. Если к задаче применим параллелизм, рассмотрите возможность использования [библиотеки параллельных задач](https://learn.microsoft.com/ru-ru/dotnet/standard/parallel-programming/task-parallel-library-tpl).

#### Task

##### `Task.Run()`

Ставит указанную работу в очередь для выполнения в `ThreadPool` и возвращает задачу.

##### `ConfigureAwait(bool continueOnCapturedContext)`

Этот метод позволяет нам указать [контекст синхронизации](https://docs.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext) , в котором должен быть возобновлен код после `await`.

Проще говоря, он позволяет разработчикам явно указать, хотят ли они продолжить работу после `await`в том же контексте синхронизации, который инициировал `await`.

