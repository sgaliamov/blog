# How to run asynchronous tasks in batch

In this post I will explore different ways to batch Tasks.

## The problem

довольно часто нужно обработать элементы в коллекции асинхронно.
это может быть отправка сообщений в очередь, запрос данных из бд, или выполнение http запросов.
не всегда есть возможность поменять существующий api и приходится использовать методы, которые работают по одной сущности.

для конкретики давайте представим, что у нас есть список id продуктов (`productsIds`), и мы хотим запросить данные по ним.
то, как может выглядеть синхронный код:

``` c#
var products = productsIds
    .Select(id => _productsService.GetById(id)) // GetById returns Task<Product>
    .ToArray();
```

синхронное решение работает хорошо пока у вас нет много `productsIds`.
и когда их становиться действительно много, вы можете захотеть призвать на помощь многопоточность.

## Naive solution

предположим, у вас есть асинхронная версия метода, и чтобы сэкономить время вы решили задачу в лоб:

``` c#
var productTasks = productsIds.Select(id => _productsService.GetByIdAsync(id));

var products = await Task.WhenAll(productTasks);
```

нет ничего плохого в этом коде.
до тех пор пока это ваш pet project, или у вас нет реальной нагрузки.

по законам мерфи, если что-то плохое может случится, оно случится.
если элементов в коллекции много, то возникает 2 проблемы.

все таски инстанциируются в памяти, и это может быть очень много.
таска это сложный объект, у нее много внутренних полей.
даже если мы используем `IEnumerable`, .net все равно внутри [делает](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Private.CoreLib/src/System/Threading/Tasks/Task.cs#L5561):

``` c#
InternalWhenAll(taskList.ToArray());
```

все таски начнут исполняться асинхронно.
так как на время исполнения IO операции поток начинает исполнять следующую таску, высока вероятность того, что в конечном счете запустятся все таски.
особенно, если они требуют какого-то времени на исполнение.
`Task.WhenAll` будет просто ждать завершения всех IO операций.

да, TPL manage количество одновременно запущенных потоков, но количество исполняемых тасок может быть гораздо большим.
Task.WhanAll and async/await manage thread pool itself. we don't have to manage it ourselves.
amount of tasks !== amount of threads. concurrency !== parallelism.

в случае high-load очень легко может возникнуть ситуация snap port exhaustion или excide tcp connections limit.
даже если в вашей реализации нет сетевого трафика, все равно на исполнение задачи требуются какие-то ресурсы, а они могут в любой момент кончиться.
так же вы можете хотеть не ddos ваш ресурс и как-то контролировать количество запросов.

проблема может усугубиться если мы захотим выполнить несколько асинхронных действий:

``` c#
var productTasks = productsIds
    .Select(async id => {
        var product = await _productsService.GetByIdAsync(id);
        await _otherService.DoSomething(product);

        return product;
    })
    .ToArray();

var products = await Task.WhenAll(productTasks);
```

в этом случае таски начнут исполняться до того как они попадут в `Task.WhenAll`.

по сути, `Task.WhenAll` не отвечает за старт тасок, он только ждет результат.

по-этому простое решение может не подойти.

## Other solutions

мы можем попытаться сделать следующий helper:

``` c#
public static async Task ProcessInBatch<T>(IEnumerable<T> items, Func<T, Task> func, int batchSize)
{
    var itemsList = items.ToList();
    while (itemsList.Any())
    {
        var batch = itemsList.Take(batchSize).ToList();
        var tasks = batch.Select(func).ToList();
        await Task.WhenAll(tasks);
        itemsList = itemsList.Skip(batchSize).ToList();
    }
}
```

работает хорошо до тех пор пока разработчики не используют больше значение в `batchSize`.
клиент не должен знать деталей реализации, по-этому рано или поздно кто-то захочет "оптимизировать" свое решение и передаст туда например 1000.
и получите 1000 одновременных запросов на ваш ресурс.

другой потенциальной проблемой этого метода является `items.ToList()`.
если объекты тяжелые, то это так же потребляем память.

что бы решить это можно воспользоваться `Enumerator`:

``` c#
            using var enumerator = tasks.GetEnumerator();
            var part = new Task[batch];
            do
            {
                while (index < batch)
                {
                    if (enumerator.MoveNext())
                    {
                        part[index] = enumerator.Current;
                    }
                    else
                    {
                        part[index] = Task.CompletedTask;
                    }

                    index++;
                    counter++;
                }

                Task.WaitAll(part);
                index = 0;
            } while (counter < Count);
```

проблема этого кода в том, что он сложный.
нет гарантии что здесь нет какой-то ошибки.

еще можно использовать `SemaphoreSlim`:

``` c#
TBD
```

но это не решает проблему сложности кода и нам нужно менеджить количество ресурсов.

## Optimal solution

использовать `AsParallel` и стандартные средства.
не изобретайте велосипедов.
create dynamic partitioner.

## Other suggestions

1. Долгие Task создавать с флагом LongRunning. Не будет уменьшаться пул потоков.
1. Всегда задавать в конфиге `<configuration><runtime><ThrowUnobservedTaskExceptions enabled="true" /></runtime></configuration>`. При завершении приложения все не отловленные исключения в потоках будут проброшены.
1. Use ContinueAwait(false).

## Summary

Good book to read [Pro Asynchronous Programming with .NET](https://www.amazon.com/Asynchronous-Programming-NET-Richard-Blewett/dp/1430259205).

``` c#
public static async Task<IEnumerable<TResult>> ProcessInBatch<T, TResult>(IEnumerable<T> items, Func<T, Task<TResult>> func, int batchSize)
{
    var result = new List<TResult>();
    var itemsList = items.ToList();
    while (itemsList.Any())
    {
        var batch = itemsList.Take(batchSize).ToList();
        var tasks = batch.Select(func).ToList();
        var batchResult = await Task.WhenAll(tasks);
        result.AddRange(batchResult);
        itemsList = itemsList.Skip(batchSize).ToList();
    }

    return result;
}
```

ProcessInBatch is a malicious method.
the problem with it, it does var itemsList = items.ToList();
that means that it will create all task in memory anyway.

and more funny thing, once you did ToList, you materialize the collection. all tasks actually are created at that moment, and most probably are started. tasks are not started just after await Task.WhenAll(tasks) they can be started immediately. so ProcessInBatch is just pretending that it's doing batching!

``` c#
public static class Program
{
    private static void Main()
    {
        var tasks = Enumerable.Range(0, 1000).Select(async x => await Task.Run(() => Console.Write(x)));

        var list = tasks.ToArray();

        Task.WaitAll(list);
    }
}
```

yes, sometimes we may want to limit concurrent execution.
it this case we can use ParallelOptions, for example. or Partitioner
non need to invent wheels.

if we want just to not waste memory we can do

``` c#
products.AsParallel().ForAll(x => direct
   ? _indexingService.IndexProductAsync(new ProductReference(x.BrandId, x.ProductId))
   : _productService.IndexProductAsync(x.BrandId, x.ProductId));
```



https://www.codeproject.com/Articles/5289661/Batch-Parallel
https://blog.bitscry.com/2017/10/22/batching-and-running-tasks-in-parallel/
https://stackoverflow.com/questions/42511104/run-asynchronous-tasks-in-batch
