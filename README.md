# .NET Concepts

## IEnumerator, IEnumerable, ICollection, IList, List
- IEnumerator
    -  has one method to enumerate the next object.
    -  Supports simple iteration over a collection.
    -  Key members: current, MoveNext(), Reset()
    -  Usage: Typically used to implement custom iteration over a collection. Usually implemented along with IEnumerable.
- IEnumerable
    - can only loop through data and no manipulation of collection, Count the records, GetEnumerator() method
    - Purpose: Exposes an enumerator that provides iteration over a collection.
    - Namespace: System.Collections for non-generic, System.Collections.Generic for generic.
    - Key Members: GetEnumerator(): Returns an enumerator that iterates through a collection.
    - Usage: Provides a simple way to iterate over a collection using a foreach loop. Base interface for all non-generic collections that can be enumerated.
- ICollection
    - can do add, remove
    - Purpose: Defines size, enumerators, and synchronization methods for all nongeneric collections.
    - Namespace: System.Collections for non-generic, System.Collections.Generic for generic.
    - Key Members:
        - Count: Gets the number of elements in the collection.
        - IsReadOnly: Gets a value indicating whether the collection is read-only.
        - Add(T item), Remove(T item), Clear(): Methods to modify the collection.
    - Usage: Represents collections like lists, queues, and sets that can be enumerated and manipulated.
- IList
    - add, insert, remove, IndexOf (have option to modify collection)
    - Purpose: Represents a collection of objects that can be individually accessed by index.
    - Namespace: System.Collections for non-generic, System.Collections.Generic for generic.
    - Key Members:
        - Inherits all members of ICollection.
        - this[int index]: Gets or sets the element at the specified index.
        - IndexOf(T item): Returns the index of a specific item.
        - Insert(int index, T item): Inserts an item at the specified index.
        - RemoveAt(int index): Removes the item at the specified index.
    - Usage: Provides additional functionality over ICollection by allowing indexed access.
- List
    - Add, RemoveRange, InsertAt, Remove
    - .NET is a versatile, dynamic data structure provided by the System.Collections.Generic namespace.
    - offers a range of methods and properties for working with collections of objects.
    - Supports type safety by allowing you to specify the type of elements it can hold.
    - Automatically resizes as elements are added or removed.
    - Elements can be accessed by their zero-based index.
    - Provides a range of methods for adding, removing, searching, and sorting elements.

# TASK PROGRAMMING
- Task is Unit of Work
- .net way of grouping work together
- tells scheduler to execute this on separate thread

## Ways to create Task
- Ist Way: use
```csharp
Task.Factory.StartNew(action) => here we are creating a task and starting it also
```
- 2nd Way: explicitly make instance of class
```csharp
var t = new Task(action) => we are creating a task only
t.Start() => We need to start it as and when required.
```
**Note:** thing will run concurrently
- Task.CurrentId: will give id of current task

## Ways to create Task which will return a value
- Ist Way: use
```csharp
Task.Factory.StartNew<datatype of result>(action, parameter)
```
**OR** 
```csharp
Task<datatype of result> test = Task.Factory.StartNew(action, parameter)
```
- @nd Way: explicitly make instance of class
```csharp
Task<datatype of result> t = new Task<datatype of result>(action, parameter) 
t.Start()
Console.WriteLine(t.result)
```
**Note:** here since we need result, so we are waiting for task to complete or we blocking a task to complete

## Cancel a Task
- there are 2 Ways
    - Use CancellationTokenSource
    ```csharp
    var cts = new CancellationTokenSource();
    var token = cts.token; // took token from cts.
    
    var t= new Task(() =>{
        int i = 0; 
        While(true){
            if(token.IsCancellationReuested) // checking if cancellation was requested or not
                break;
            else
                Console.WriteLine($"{i++\t"});
        }
    }, token); // pass cts token to method
    t.Start();
    
    Console.ReadKey();
    cts.Cancel(); // request cancellation on some action, here it is key press
    ```
    - Ths is example of Soft exist to task.
    
    - Use Exceptions -> Recommended
    ```csharp
    var cts = new CancellationTokenSource();
    var token = cts.token; // took token from cts.
    
    var t= new Task(() =>{
        int i = 0; 
        While(true){
            if(token.IsCancellationReuested) // checking if cancellation was requested or not
            throw new OperationCanceledException() // throw exception. This will not cause exeception at high level
            else
                Console.WriteLine($"{i++\t"});
        }
    }, token); // pass cts token to method
    t.Start();
    
    Console.ReadKey();
    cts.Cancel(); // request cancellation on some action, here it is key press
    ```
    **OR**
    ```csharp                  
    var cts = new CancellationTokenSource();
    var token = cts.token; // took token from cts.
    
    var t= new Task(() =>{
      int i = 0; 
      While(true){
              token.ThrowIfCancellationRequest(); // here task will know that the cancellation has been done. it will not notify about the cancellation
              Console.WriteLine($"{i++\t"});
      }
    }, token); // pass cts token to method
    t.Start();
    
    Console.ReadKey();
    cts.Cancel(); // request cancellation on some action, here it is key press
    ```

## Ways to get notification if task got IsCancellationReuested
- subscribe to token as below
```csharp
var cts = new CancellationTokenSource();
var token = cts.token; // took token from cts.
token.Register(() => { Console.WriteLine("Cancellation requested")});
```
- so whenever any one calls "cts.Cancel()", they will see "Cancellation requested"" message.

- wait on cancellation waithandler.
```csharp
Task.Factory.StartNew(() =>{
  token.WaitHandle.WaitOne(); // wait on the token until someone cancels the token
  Console.Writeline("Wait Handle releases");
})
```

- Composite Cancellation Tokens
    - we have say multple CancellationTokenSource
    - Here we can create new CTS which can be a link to multiple cTS's and can then call cancel on new CTS token
```csharp
var cts1 = new CancellationTokenSource();
var cts2 = new CancellationTokenSource();
var cts3 = new CancellationTokenSource();

var cts = CancellationTokenSource.CreateLinkedTokenSource(cts1, cts2, cts3); //creatd linked CTS

var t= new Task(() =>{
    int i = 0; 
    While(true){
            cts.Token.ThrowIfCancellationRequest(); // throwing error using linked cts token
            Console.WriteLine($"{i++\t"});
    }
}, cts.Token); // pass cts token to method
t.Start();

Console.ReadKey();
cts2.Cancel(); // requested cancellation by say cts2
```

## Different ways to sleep the thread 
- Thread.Sleep() => tells scheduler that we don't need procesor time. Scheduler can execute another thread for meanwhile
- SpinWait.SpinUntil() -> here we are not releasing the waiting time/ not giving our execution period and ltting CPU waist its time till we are waitig for the thread to continue
- Waiting on a Cancellation Token CancellationTokenSource  
```csharp
var cts = new CancellationTokenSource();
var token = cts.Token;
var t = new Task(() =>{
    Console.WriteLine("Wait for 5 secs");
    bool cancelled = token.WaitHandle.WaitOne(5000);
    Console.WriteLine(cancelled? "Bomb Disarmed" : "BOOM!!!!");
}, token);

Console.ReadKey();
cts.Cancel(); // requested cancellation by say cts2
```
   
- t.Wait(Token) - will wait for the specified task to complete
- Task.WaitAll(task1, task2); //wait for all the tasks to complete.
- Task.WaitAny(task1, task2); //will wait for either of the task to compelete
- Task.WaitAny(new[] {task1, task2}, 4000, token); // it will wait for given time i.e. 4 secs

## Way to handle Multiple exceptions in task
- try catch
- catch AggregateExeception
- U can handle all at one place or can propogate some to upper level by handleing selected execeptions u want at that place using  Handl().

## DATA SHARING AND SYNCHRONIZATION
- Atomic
  - if any operation cannot be interrupted that that is atomic.
  - x=1 is atomic
  - x++ is not atomic
  - Atomic operations   
    - refernce assignments
    - read and writes to value type <= 32 or if it is 64 bit

  - Apporaches to make threads atomic
    - Ist way: To make tasks inside thread atomic, we need to use LOCK().
    - Example - we are running 2 tasks inside one loop -> deposit and withdraw. to make them atomic, write Lock() in the logic of each depoosit and withdraw

    - 2nd way: interlock operation on primitive data types.
    ```csharp
    private int balance; // backing property to use with ref in interlocked
    public int Balance{
        get{return balance;}
        private set{ balance = value;}
    }

    public int depsit(int amount){
        Interlocked.Add(ref balance, amount)
    }

     public int wihthdraw(int amount){
        Interlocked.Add(ref balance,-amount)
    }
    ```
    
- MemoryBarrier
  - some times CPU reorders the execution of tasks. Memory barrier tells CPU that no instructions before barrier can execute after MemoryBarrier
  ```csharp
  //1
  //2
  Thread.MemeoryBarrier
  //3

  here 1,2 cannot execute after barrier
  ```
  
- Interlocked.Exechange() and ComapreExchange()
  - used for thread safe assignments or compare vaiues
  - ComapreExchange - compares 2 values if they are same then replaces that.

- Spin lock
```csharp
SpinLock sl = new SpinLock(); // define varaible  
sl.Enter(bool variable); // apply lock
sl.Exit(); //release a lock
```
- **disadvantage** of Spin lock
  - there can be lock recursion situation (means we already have lock and we are trying to take a lock again). This situation is difficult to handle with spin lock

- Mutex
  - method of locking
  ```csharp
  Mutex mu = new Mutex;
  mutex.WaitOn(); // acquire
  mutex.ReleaseMutex(); // release
  ```
  - **Advantages** - can be global construct, we can have mutex share between different processes

- Reader/Writer Lock
  - allows you to read the data from several threads concurrenlty but controls writing on several different threads
  - ReaderWriterLockSlim class

## CONCURRENT COLLECTIONS
- uses TryXxx() methods: returns a bool indicating success.
- Optimized for multithreaded use: some ops eg count can block and make collection slow.
- Collections
  - ConcurrentDictionary - it adds the data concurrently and helps to control whether we are able to do or not.
  - ConcurrentQueue - implements IProducerConsumerCollection
  - ConcurrentStack - implements IProducerConsumerCollection
  - ConcurrentBag - no ordering data structure. Order of data exit can be dynamic
  - BlockingCollection
    - block the elements we are retiriving from concurent collections. 
    - It is wrapper around one of the IProducerConsumerCollection
    - Provides blocking and bounding capabilities

## TASK CORDINATION
- Continuation
  - Ist way: we can use ContinueWith() which will take previous task
  ```csharp
  var task = Task.Factory.StartNew(() =>{
      console.writeline('test');
  });
  var task2 = task.ContinueWith(t => {
      Console.WriteLoine("Conpleted Task");
  });
  task2.Wait();
  ```
  - 2nd Way: ComntinueWhenAll() -> continue next task when all are done in the array
  ```csharp
  var task = Task.Factory.StartNew(() =>"Task 1");
  var task2 = Task.Factory.StartNew(() =>"Task 2");

  var task3 = Task.Factory.ComntinueWhenAll(new[] {task, task2}, tasks =>{
      console.writeline("Tasks completed");
      foreach(var t in tasks){
          Console.WriteLine(t.result);
      }
  });
  task3.Wait();
  ```
  - ComntinueWhenAny(): continue when any task in array gets done.
  ```csharp
  var task = Task.Factory.StartNew(() =>"Task 1");
  var task2 = Task.Factory.StartNew(() =>"Task 2");

  var task3 = Task.Factory.ComntinueWhenAll(new[] {task, task2}, t =>{
      console.writeline("Tasks completed");
          Console.WriteLine(t.result);
  });
  task3.Wait();
  ```

## Child Tasks
- we need to build a relation between parent and child so that child task will wait completeion if we are waiting for parent task to complete
- that can be done using various options like one is TaskCreationOptions
```csharp
var parent = new Task(() =>{
    var child = new Task(() =>{
        console.writeline("started");
        Thread.Sleep(3000);
        console.writeline("completed");
    }, TaskCreationOptions.AttachedToParent);
    child.Start();
});
parent.Start();
try{
    parent.Wait();
}
catch(AggregateException ae){
    ae.handle(e => true);
}
```
- We can also link tasks in Continuation using TaskContinuationOptions as below
```csharp
var parent = new Task(() =>{
  var child = new Task(() =>{
      console.writeline("started");
      Thread.Sleep(3000);
      console.writeline("completed");
  }, TaskCreationOptions.AttachedToParent);

  var completionHandler = child.ContinueWith(t=>{
      Console.WriteLine("completed");
  }, TaskContinuationOptions.AttachedToParent | TaskContinuationOptions.OnlyRanToCompletion);

   var failHandler = child.ContinueWith(t=>{
      Console.WriteLine("failed");
  }, TaskContinuationOptions.AttachedToParent | TaskContinuationOptions.OnlyOnFaulted);

  child.Start();
});
parent.Start();
try{
  parent.Wait();
}
catch(AggregateException ae){
  ae.handle(e => true);
}
```

## Barrier
- suppose we have various worker threads that need to work in stages ie. all the threads first need to compelete stage 1 then need to move to stage 2. Here barrier comes into picture

## ManualResetEventSlim
- reset event at one place and wait at other
- single set() is ok for all wait() or waitone();

## AutoResetEvent 
- default it is set to false
- as soon as we are done waiting, it sets to true
- so for each waitone(), there should be corresponding set().

## Semaphore
- we can specify the number of requests and max number of requests.
- SemaphoreSlim()
- it causes a clock when we wait on Semaphore (semaphore.WaitOne() OR semaphore.Wait())
- unblock, when we release certain number of execution slots. (semaphore.Release())

# PARALLEL LOOPS
- Parallel Invoke 
    - takes set of functions/actions and runs them concurrently.
    - This is particularly useful when you have several independent tasks that can run in parallel, improving the performance of your application by utilizing multiple CPU cores.
    - Benefits
      - Concurrency: Utilizes multiple CPU cores to run tasks in parallel, improving performance for CPU-bound operations.
      - Simplicity: Easy to use for running multiple independent tasks without the need for complex threading code.
    - Considerations
      - Order of Execution: Parallel.Invoke does not guarantee the order in which tasks will run.
      - Exception Handling: If any task throws an exception, Parallel.Invoke will aggregate these exceptions and throw an AggregateException.
- Parallel For
    - Used for parallel execution of a traditional for loop.
    - Provides more control over the loop index and allows you to manipulate the state of the loop.
    - Scenarios where you need to iterate over a range of integers and perform operations based on the index, such as processing arrays or performing calculations.
- Parallel foreach
    - Used for parallel execution of a foreach loop over a collection.
    - Automatically handles partitioning of the collection and scheduling of tasks.
    - Scenarios where you need to iterate over a collection of items and perform operations on each item like lists, arrays, or any IEnumerable<T>
```csharp
public IEnumerable<int> Range (int start, int end, int step)
{
    for (int i = start; i < end; i += step)
    {
        yield return i;
    }
}
public void Demo()
{
    var a = new Action(() => Console.WriteLine($"First {Task.CurrentId}"));
    var b = new Action(() => Console.WriteLine($"First {Task.CurrentId}"));
    var c = new Action(() => Console.WriteLine($"First {Task.CurrentId}"));

    Parallel.Invoke(a, b, c);

    //here it will go from 1-> 10 as end is exclusive.
    Parallel.For(1, 11, i => 
    {
        Console.WriteLine($"{i * i}\t");
    });

    string[] words = { "oh", "what", "a", "program" };
    Parallel.ForEach(words, word =>
    {
        Console.WriteLine($"{word}'s length is {word.Length}");
    });

    Parallel.ForEach(Range(1, 20, 3),Console.WriteLine);

}
```

## Stopping, cancelling a parallel loop and how to handle exceptions in that
- ParallelLoopState.Stop()
- ParallelLoopState.Break()
- CancellationTokenSource.
```csharp
var cts = new CancellationTokenSource();
ParallelOptions po = new ParallelOptions();
ParallelLoopResult result;

result = Parallel.For(0,20, po, (int x, ParallelLoopState state) => {
    if(x == 20)
    {
        //we can check whether loop has been successfully completed or not by using IsCompleted from result of loop 
        //state.Stop(); 
       // state.Break(); // we can check whether break has been done by using result of this for loop and checking value of LowestBreakPoint in result.
        cts.Cancel(); // please handle OperationCancelledException in try catch for this
    }
})
```

## Optimization technique 
- Thread local storage: to speed up the actual calculations that we do in loops
- we create loacal storage for each task, update the things in that thread local storage and at end, update the main variable using interlocked by summing up values from all the local storages.
```csharp
int sum = 0; // main variable that needs to get add up in each task
Paralle.For(1, 1001,  // loop from 1-> 1000
    () => 0, // default value fo each thread local storage
    (x, state, tls) => { // update operation on thread local storage  with x being counter, state being parallelloopstate, and tls being thread local storage that needs to be updated
        tls += x;
        return tls;
    },
    partialSum => { // here finally we are summing up all thread local stoarge values into our actual sum variable using Interlocked
        Interlocked.Add(ref sum, partialSum);
    }
);
```
- this will speed up the loop processing.

## partitioning
- partition the loops
- we will benchmark the chunck of code in loop alsong with partitioning
- Nuget: BenchmarkDOTNET
```csharp
// not efficient
[Benchmark]
public void SquareEachValue(){
    const int count = 100000;
    var values = Enumerable.Range(0,count);
    var results = new int[count];

    Parallel.foreach(values, x =>{ results[x] = Math.Pow(x,2);}) // here it is creating a delagate for each task which is not efficient, to avoid that, we should use Benchmark
}

//partioning 
[Benchmark]
public void SquareEachValueChunked(){
    const int count = 100000;
    var values = Enumerable.Range(0,count);
    var results = new int[count];
    var part = Partitioner.Create(0, count, 10000);
    Parallel.foreach(part, range =>{
        for(int i=range.Item1; i < range.Item2; i++){
            results[i] = (int) Math.Pow(i,2);
        }
    });
}

Main(strin[] args){
    var summary = BenchmarkRunner.Run<classname>();
    Console.WriteLine(summary);
}
```

# PARALLEL LINQ(PLINQ)
- LINQ runs by default on single thread, i.e. its execution is sequential
- it is TPL's counterpart for parallel linq.

- AsParallel() and ParallelQuery
  - AsParallel().AsOrdererd() => will process elements parallel in order.

- Cancellation & Exception
  - CancellationTokenSource for cancellation -> items.WithCancellation(cts.token).Select();/ here items can be ParallelEnumerable
  - execpetions - Aggregate exception.

- Merge options
  - executing parallel linq query and then consuming result -> Parallel producer consumer Pattern
  - in the above pattern, it might be possible that producer is going on and i  between we started getting comsuming records. 
  - to merge that we can use "WithMergeOptions()"
  ```csharp
  numbers.AsParallel()
                .WithMergeOptions(buffering options)
                .Select();
  ```

- Custom aggregation
  - sequential aggregate
  ```csharp
  var sum = Enemerable.Range(1, 1000)
              .Aggregate(0, (i, acc) => i + acc);
  ```
  - above aggregate is equivalent to general sum()  =>  var sum = Enemerable.Range(1, 1000).Sum();

- Parallel Aggregate
```csharp
var sum = ParallelEnumerable.Range(1, 1000)
      .Aggregate(0,
           (partialSum, i) => partialSum += i,
           (total, subTotal) => total += subTotal,
           i => i);
```              

# ASYNCHRONOUS PORGRAMMING
- yield -> allow temporary suspension of execution
- async await 
  - allows to release current thread immediately, 
  - you start doing things on new thread and 
  - you have these contiuations one after another

## What does async do?
- it is a hint to compiler.
- enables the use of await keyword
    
## What does await do?
- Register continuation with the async operation ( code that follwos me is a continuation like ContinueWith())
- Gives up the current thread.
  - the call heppens on a thread from TPL thread pool.
- If SynchronizationContext.Current != null, ensures the contiuation is also posted there eg on UI thread.
  - If null, continuation sceheduled using current task scheduler.
  - this behavior can be customizd on the call.
- Coerces the result of the ASYNCHRONOUS operation.
- can be used as the language equivalent f Unwrap().
- use of double await can retur in rather than TAsk<int> in Task.Run() example below. 
- When we use async await functionality, we are actually building STATE MACHINE.

## STATE MACHINE    
- suppose we get any exception while implementing async await, this state machine helps us to maintain states and return back on the appropriate state

## TASK.Run()   
- shortcut that calls taskfatcory.startnew() and unwrap()s the result.
- utility method inside the task class.
- eqquivalent to Task.Factory.StartNew(something, CancellationToken.None, TaskCreationOptions.DenyChildAttach, TaskScheduler.Default);
- 8 overloads to support below combinations
  - Task VS Task<T>
  - Cancelable VS Non-Cancelable
  - synchronous vs asynchronous delegate or lambda operation'
  - suppose we have below code
  ```csharp
  var t = Task.Factory.StartNew(async() =>{
      await Task.Deloay(5000);
      return 123;
  })
  ```
 - **This will return Task<Task<Int>> but we want Task<int> so we can use 2 ways**
  - use unwrap() as below
  ```csharp
   var t = Task.Factory.StartNEw(async() =>{
      await Task.Deloay(5000);
      return 123;
   }).Unwrap();
  ```
  - Use Task.Run()
  ```chsrp
  int result = await Task.Run(async() =>{
       await Task.Deloay(5000);
          return 123;
  })
  ```

- Task.WhenAny() - creates a task that will complete when any of the supplied tasks has completed.
- Tasks.WhenAll() - creates a task that will complete when all of the supplied tasks have completed.


## Async Factory method
- we cannot use async and await in constructors. 
- to indicate that class requires asynchronous initialization
- To overcome that we need to create a factory method

## Steps to create Factory Method
- Create proviate constructor of class
- Create private method that will perform async await functionality and return class
```csharp
private async Task<ClassName> InitAsync(){
    await Task.Delay(1000);
    return this;
}
```
- Create aysnc factory method
```csharp
public static Task<ClassName> CreateAsync(){
    var result = new Foo();
    return result.InitAsync();
}

public static void async  Task Main(string[] args)
{
    ClassName abc = await ClassName.CreateAsync();
}
```

## Async Initialization Pattern 
- Another way to indicate that some classed require Asynchronous intialization
- introduce an interface
- then detect that interface and perform initialization
```csharp
public interface IAsyncInit{
    Task InitTask {get;}
}

public class MyClass : IAsyncInit
{
    public MyClass(){
        InitTask = InitAsync();
    }

    public Task InitTask{get;}

    private async Task InitAsync(){
        await Task.Delay(1000);
    }
}

public class Demo{
    public static async Task Main(string[] args){
        var c =  new MyClass();
        if(c is IAsyncInit ai)
        await ai.InitTask;
    }
}
```
## Asynchronous Lazy Initialization
- AsyncLazy<int> wrapper fo aysnc lazy initialization

## ValueTask<T>
- lightweight alternative to a Taks<T>
- ingtroduced to help improve async performance with decreased allocation overhead.
- it is a struct not a class

## Task 
- promise of eventual completion of some operation.
- initiate an operation, get a task for it
- task will complete when op completes
  - synchronous completion
  - async completion
- use ConfigureAwait(false) to avoid deadlocks and improve performance. ConfigureAwait(false) prevents the continuation from capturing the current synchronization context, which can help avoid deadlocks and improve performance, especially in library code or non-UI contexts

## First async await method's output is used by another async await method
- When data from one async/await operation is consumed by another async/await operation, the overall process remains asynchronous. Here’s a breakdown of how this works:
- Asynchronous Flow
  - First Async Operation: The first async method runs and completes its task asynchronously.
  - Awaiting the Result: The result of the first async operation is awaited, meaning the calling method pauses until the task completes without blocking the thread.
  - Second Async Operation: Once the first task completes, its result is passed to the second async method, which then runs asynchronously.
  - Example
    ```charp
    public async Task<string> FirstOperationAsync()
    {
        await Task.Delay(1000); // Simulate async work
        return "Data from first operation";
    }
    
    public async Task<string> SecondOperationAsync(string input)
    {
        await Task.Delay(1000); // Simulate async work
        return $"Processed {input}";
    }
    
    public async Task<string> CombinedOperationsAsync()
    {
        var firstResult = await FirstOperationAsync();
        var secondResult = await SecondOperationAsync(firstResult);
        return secondResult;
    }
    ```
    - **Explanation**
    - FirstOperationAsync: Runs asynchronously and returns a result after a delay.
    - SecondOperationAsync: Takes the result of the first operation and processes it asynchronously.
    - CombinedOperationsAsync: Coordinates the two operations, awaiting the first before starting the second.
      
    - **Asynchronous Nature**
    - Non-blocking: Each await keyword ensures that the method does not block the calling thread while waiting for the task to complete.
    - Concurrency: If there are other tasks to run, they can proceed while the awaited tasks are in progress.
    
    - **Synchronous vs. Asynchronous**
    - Synchronous Call: If you were to call the second operation immediately after the first without await, it would be a synchronous call, blocking the thread until both operations complete.
    - Asynchronous Call: Using await ensures that each operation completes asynchronously, allowing other tasks to run concurrently.
    - In summary, even though the second async operation depends on the result of the first, the overall process remains asynchronous due to the use of await.


## What happens if the first operation fails or is canceled?
- If the first asynchronous operation fails or is canceled, the subsequent operations that depend on its result will not proceed. Here’s how you can handle such scenarios:
  - Handling Failures : When the first operation fails, you can catch the exception and decide how to handle it. This might involve logging the error, retrying the operation, or providing a fallback.
  - Handling Cancellations: If the first operation is canceled, you can check for cancellation and handle it appropriately, often by cleaning up resources or notifying the user.
  - Example with Exception and Cancellation Handling: Here’s an example that demonstrates how to handle both exceptions and cancellations:
  ```csharp
    public async Task<string> FirstOperationAsync(CancellationToken token)
    {
        try
        {
            await Task.Delay(1000, token); // Simulate async work
            return "Data from first operation";
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("First operation was canceled.");
            throw;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"First operation failed: {ex.Message}");
            throw;
        }
    }

    public async Task<string> SecondOperationAsync(string input, CancellationToken token)
    {
        await Task.Delay(1000, token); // Simulate async work
        return $"Processed {input}";
    }

    public async Task<string> CombinedOperationsAsync(CancellationToken token)
    {
        try
        {
            var firstResult = await FirstOperationAsync(token);
            var secondResult = await SecondOperationAsync(firstResult, token);
            return secondResult;
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("Combined operations were canceled.");
            return "Operation canceled";
        }
        catch (Exception ex)
        {
            Console.WriteLine($"An error occurred: {ex.Message}");
            return "Operation failed";
        }
    }
    ```
    - **Explanation**
    - FirstOperationAsync: Handles both exceptions and cancellations. If an exception occurs, it logs the error and rethrows it. If the operation is canceled, it catches the OperationCanceledException and rethrows it.
    - SecondOperationAsync: Also supports cancellation.
    - CombinedOperationsAsync: Coordinates the two operations. If the first operation fails or is canceled, it catches the exception and handles it accordingly.
    - 
    - **Key Points**
    - Exception Handling: Use try-catch blocks to handle exceptions and provide meaningful error messages or fallback logic.
    - Cancellation Handling: Use CancellationToken to support cancellation and handle OperationCanceledException to clean up resources or notify the user.
    - Flow Control: If the first operation fails or is canceled, the second operation will not proceed, ensuring that dependent operations are not executed with invalid or incomplete data.
    - By handling exceptions and cancellations properly, you can make your asynchronous code more robust and user-friendly.

## What if the second operation fails after the first one succeeds?
- If the second operation fails after the first one succeeds, you need to handle the failure gracefully and decide whether to retry the second operation, roll back the first operation, or take some other action. Here’s how you can manage this scenario:
- Example with Exception Handling and Rollback
  - Define your asynchronous operations:
  ```csharp
  public async Task<string> FirstOperationAsync()
  {
      await Task.Delay(1000); // Simulate async work
      return "Data from first operation";
  }
  
  public async Task<string> SecondOperationAsync(string input)
  {
      await Task.Delay(1000); // Simulate async work
      throw new InvalidOperationException("Second operation failed"); // Simulate failure
  }
  ```
  - Create a method to handle the operations and rollback logic:
  ```csharp
  public async Task HandleOperationsWithRollbackAsync()
  {
      string firstResult = null;
      try
      {
          firstResult = await FirstOperationAsync();
          var secondResult = await SecondOperationAsync(firstResult);
          Console.WriteLine("Both operations completed successfully.");
          Console.WriteLine(secondResult);
      }
      catch (Exception ex)
      {
          Console.WriteLine($"An error occurred: {ex.Message}");
          if (firstResult != null)
          {
              await RollbackFirstOperationAsync(firstResult);
          }
      }
  }
  
  private async Task RollbackFirstOperationAsync(string result)
  {
      // Implement your rollback logic here
      Console.WriteLine($"Rolling back first operation with result: {result}");
      await Task.Delay(500); // Simulate rollback work
  }
  ```
- **Explanation**
- FirstOperationAsync: Runs asynchronously and returns a result.
- SecondOperationAsync: Takes the result of the first operation and processes it, but simulates a failure.
- HandleOperationsWithRollbackAsync: Coordinates the two operations. If the second operation fails, it catches the exception and calls RollbackFirstOperationAsync to undo the first operation.
- RollbackFirstOperationAsync: Contains the logic to roll back the first operation.
  
- **Key Points**
- Exception Handling: Use try-catch blocks to catch exceptions and handle them appropriately.
- Rollback Logic: Implement rollback logic to undo the effects of the first operation if the second operation fails.
- Logging and Monitoring: Log errors and rollback actions to help with debugging and monitoring.
  
- **Retry Logic**
- If you want to retry the second operation before rolling back, you can incorporate retry logic using Polly or a similar library:
```csharp
private static readonly AsyncRetryPolicy retryPolicy = Policy
    .Handle<Exception>()
    .WaitAndRetryAsync(3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));

public async Task HandleOperationsWithRetryAndRollbackAsync()
{
    string firstResult = null;
    try
    {
        firstResult = await FirstOperationAsync();
        var secondResult = await retryPolicy.ExecuteAsync(() => SecondOperationAsync(firstResult));
        Console.WriteLine("Both operations completed successfully.");
        Console.WriteLine(secondResult);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"An error occurred: {ex.Message}");
        if (firstResult != null)
        {
            await RollbackFirstOperationAsync(firstResult);
        }
    }
}
```
- In this example, the second operation is retried up to three times before rolling back the first operation if it still fails.
- By handling exceptions, implementing rollback logic, and optionally adding retry mechanisms, you can ensure that your application remains robust and can recover gracefully from failures.

## What if the first operation modifies external resources (e.g., a database)?
- When the first operation modifies external resources, such as a database, and the second operation fails, it’s crucial to ensure data consistency. This often involves implementing a rollback mechanism or using transactions to maintain atomicity. Here are some strategies to handle this scenario:
  - Using Transactions: Transactions ensure that a series of operations either all succeed or all fail, maintaining data integrity. In C#, you can use the TransactionScope class to manage transactions.
  - Example with TransactionScope
  ```csharp
    using System;
    using System.Data.SqlClient;
    using System.Transactions;
    using System.Threading.Tasks;

    public async Task HandleDatabaseOperationsAsync()
    {
        using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
        {
            try
            {
                await FirstDatabaseOperationAsync();
                await SecondDatabaseOperationAsync();
                scope.Complete();
                Console.WriteLine("Both operations completed successfully.");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"An error occurred: {ex.Message}");
                // Transaction will be rolled back automatically
            }
        }
    }

    public async Task FirstDatabaseOperationAsync()
    {
        // Simulate database operation
        await Task.Delay(1000);
        Console.WriteLine("First operation completed.");
    }

    public async Task SecondDatabaseOperationAsync()
    {
        // Simulate database operation
        await Task.Delay(1000);
        throw new InvalidOperationException("Second operation failed.");
    }
    ```
    - **Explanation**
    - TransactionScope: Ensures that both operations are part of a single transaction. If any operation fails, the transaction is rolled back.
    - TransactionScopeAsyncFlowOption.Enabled: Allows the transaction scope to flow across asynchronous calls.
      
    - **Rollback Logic**
    - If you prefer not to use transactions, you can implement custom rollback logic to undo changes made by the first operation if the second operation fails.
    - Example with Custom Rollback
    ```csharp
    public async Task HandleOperationsWithRollbackAsync()
    {
        string firstResult = null;
        try
        {
            firstResult = await FirstOperationAsync();
            await SecondOperationAsync(firstResult);
            Console.WriteLine("Both operations completed successfully.");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"An error occurred: {ex.Message}");
            if (firstResult != null)
            {
                await RollbackFirstOperationAsync(firstResult);
            }
        }
    }

    public async Task<string> FirstOperationAsync()
    {
        // Simulate modifying an external resource
        await Task.Delay(1000);
        return "Data from first operation";
    }

    public async Task SecondOperationAsync(string input)
    {
        // Simulate a failure
        await Task.Delay(1000);
        throw new InvalidOperationException("Second operation failed.");
    }

    public async Task RollbackFirstOperationAsync(string result)
    {
        // Implement your rollback logic here
        Console.WriteLine($"Rolling back first operation with result: {result}");
        await Task.Delay(500); // Simulate rollback work
    }
    ```
    - **Explanation**
    - FirstOperationAsync: Modifies an external resource and returns a result.
    - SecondOperationAsync: Fails, triggering the rollback logic.
    - RollbackFirstOperationAsync: Contains the logic to undo the changes made by the first operation.
      
    - **Best Practices**
    - Use Transactions: Prefer transactions for database operations to ensure atomicity and consistency.
    - Implement Robust Rollback Logic: If transactions are not feasible, ensure your rollback logic is comprehensive and tested.
    - Log and Monitor: Always log operations and errors to help with debugging and monitoring.
    - By using transactions or implementing rollback logic, you can ensure that your application maintains data consistency even when asynchronous operations fail



## How can I handle resource cleanup in case of failure?
- Handling resource cleanup in case of failure is crucial to ensure that your application remains stable and does not leak resources. Here are some strategies to manage resource cleanup effectively in C#:
  - Using finally Blocks: The finally block is used to ensure that cleanup code runs regardless of whether an exception is thrown. This is particularly useful for releasing resources like file handles, database connections, or network sockets.
  ```csharp
  public async Task PerformOperationAsync()
  {
      FileStream file = null;
      try
      {
          file = new FileStream("example.txt", FileMode.OpenOrCreate);
          // Perform operations with the file
          await Task.Delay(1000); // Simulate async work
      }
      catch (Exception ex)
      {
          Console.WriteLine($"An error occurred: {ex.Message}");
      }
      finally
      {
          file?.Close();
          Console.WriteLine("File closed.");
      }
  }
  ```
  
  - Using using Statements: The using statement ensures that IDisposable objects are disposed of correctly, even if an exception occurs. This is a preferred method for managing resources that implement IDisposable.
  ```csharp
  public async Task PerformOperationWithUsingAsync()
  {
      try
      {
          using (var file = new FileStream("example.txt", FileMode.OpenOrCreate))
          {
              // Perform operations with the file
              await Task.Delay(1000); // Simulate async work
          }
          Console.WriteLine("File closed automatically by using statement.");
      }
      catch (Exception ex)
      {
          Console.WriteLine($"An error occurred: {ex.Message}");
      }
  }
  ```
  
  - Implementing IDisposable: For custom classes that manage resources, implement the IDisposable interface to ensure proper cleanup.
  ```csharp
  public class ResourceHandler : IDisposable
  {
      private FileStream _file;
  
      public ResourceHandler(string filePath)
      {
          _file = new FileStream(filePath, FileMode.OpenOrCreate);
      }
  
      public async Task PerformOperationAsync()
      {
          // Perform operations with the file
          await Task.Delay(1000); // Simulate async work
      }
  
      public void Dispose()
      {
          _file?.Dispose();
          Console.WriteLine("Resources cleaned up.");
      }
  }
  
  public async Task UseResourceHandlerAsync()
  {
      try
      {
          using (var handler = new ResourceHandler("example.txt"))
          {
              await handler.PerformOperationAsync();
          }
      }
      catch (Exception ex)
      {
          Console.WriteLine($"An error occurred: {ex.Message}");
      }
  }
  ```
  
  - Combining try-catch-finally with using: In some cases, you might need to combine try-catch-finally with using to handle more complex scenarios.
  ```csharp
  public async Task ComplexOperationAsync()
  {
      try
      {
          using (var file = new FileStream("example.txt", FileMode.OpenOrCreate))
          {
              // Perform operations with the file
              await Task.Delay(1000); // Simulate async work
          }
      }
      catch (Exception ex)
      {
          Console.WriteLine($"An error occurred: {ex.Message}");
      }
      finally
      {
          // Additional cleanup if needed
          Console.WriteLine("Additional cleanup completed.");
      }
  }
  ```
- **Best Practices**
- Prefer using Statements: Use using statements for objects that implement IDisposable to ensure deterministic cleanup.
- Use finally for Non-Disposable Resources: Use finally blocks to clean up resources that do not implement IDisposable.
- Log Exceptions and Cleanup Actions: Always log exceptions and cleanup actions to help with debugging and monitoring.
- Avoid Throwing Exceptions in finally Blocks: Ensure that finally blocks do not throw exceptions, as this can obscure the original exception.

## Synchronous call vs asynchronous call
- Understanding the difference between synchronous and asynchronous calls is crucial for designing efficient and responsive applications. Here’s a breakdown of both concepts:

- **Synchronous Calls**
- Definition: In a synchronous call, the caller waits for the operation to complete before moving on to the next task.
- Blocking: The calling thread is blocked until the operation finishes.
- Use Case: Suitable for simple, short-duration tasks where waiting is acceptable.
```csharp
public void SynchronousMethod()
{
    var result = LongRunningOperation();
    Console.WriteLine(result);
}

public string LongRunningOperation()
{
    // Simulate a long-running task
    Thread.Sleep(5000);
    return "Operation Complete";
}
```
- **Asynchronous Calls**
- Definition: In an asynchronous call, the caller can continue executing other tasks while waiting for the operation to complete.
- Non-blocking: The calling thread is not blocked; it can perform other operations.
- Use Case: Ideal for I/O-bound operations, such as file access, network requests, or database queries, where waiting can be avoided.
```csharp
public async Task AsynchronousMethod()
{
    var result = await LongRunningOperationAsync();
    Console.WriteLine(result);
}

public async Task<string> LongRunningOperationAsync()
{
    // Simulate a long-running task
    await Task.Delay(5000);
    return "Operation Complete";
}
```

## Key Differences
- **Execution Flow:**
- Synchronous: The flow of execution is linear and sequential.
- Asynchronous: The flow of execution can continue independently of the long-running operation.
- **Performance:**
- Synchronous: Can lead to performance bottlenecks, especially in I/O-bound operations.
- Asynchronous: Improves responsiveness and scalability by allowing other tasks to run concurrently.
- **Complexity:**
- Synchronous: Simpler to implement and understand.
- Asynchronous: Requires more complex error handling and state management.
- **Practical Considerations**
- UI Applications: Asynchronous calls are essential to keep the UI responsive. For example, in a Windows Forms or WPF application, using async/await prevents the UI from freezing during long operations.
- Web Applications: Asynchronous calls can handle more concurrent requests, improving the scalability of web applications.
- **Example Scenario**
- Imagine a web application that fetches data from a remote server. Using synchronous calls, each request would block the server thread, reducing the number of concurrent users the server can handle. With asynchronous calls, the server can handle multiple requests simultaneously, improving performance and user experience.

## Thread VS Task
- Thread
    - It represents the actual OS thead.
    - we have higher control over thread
    - It cna be suspended, aborted, resume etc.
    -  we can observe the state of thread, can set thread level properties
- Threadpool is the wrapper over pool of threads and is maintained by CLR
-Task
    - class from TPL offers the of both words.
    - It does not create OS thread
    - Executed by Task Scheduler. The default scheduler simply runs on the ThreadPool
    - Task allows u to find out when it finishes to return a result.

## What is the difference between Concurrency and Parallelism?
- Concurrency is when two or more tasks can start, run, and complete in overlapping time periods. It doesn’t necessarily mean they’re running at the same instant. 
- Parallelism is when tasks literally run at the same time, like in a multicore processor.

## CORS error (Corss-Origin Resource sharing)
- In program.cs
```csharp
builder.Services.AddCors();
```
- use middleware as below before UseAuthentication and authorization. 
```csharp
app.UseCors(opt =>{
  opt.AllowAnyHeader().AllowAnyMethod().WithOrigins("http://localhost:3000"); // path of client app.
});
```

# Interview Questions on various technologies

## DENSE_RANK()
- The DENSE_RANK() function in SQL is used to assign ranks to rows within a partition of a result set, with no gaps in the ranking values. This means that if two or more rows have the same rank, the next rank will be the immediate next integer, without skipping any numbers.
```sql
DENSE_RANK() OVER (PARTITION BY column1 ORDER BY column2)
```
- PARTITION BY: Divides the result set into partitions to which the DENSE_RANK() function is applied.
- ORDER BY: Specifies the order of rows in each partition.
- Example
- Suppose you have a table employees with columns department and salary. You want to rank employees within each department based on their salary:
```sql
SELECT 
    employee_id, 
    department, 
    salary, 
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM 
    employees;
```
In this example, employees within each department are ranked by their salary in descending order. If two employees have the same salary, they will receive the same rank, and the next rank will be the next consecutive number.

## CTE Example
- A Common Table Expression (CTE) in SQL is a temporary result set that you can reference within a SELECT, INSERT, UPDATE, or DELETE statement. CTEs make complex queries easier to read and maintain.
- Example
- Suppose you have a table employees with columns employee_id, department, and salary. You want to find the average salary for each department and list employees with their salaries and the department's average salary.
```sql
WITH avg_salary AS (
    SELECT 
        department, 
        AVG(salary) AS average_salary
    FROM 
        employees
    GROUP BY 
        department
)
SELECT 
    e.employee_id, 
    e.department, 
    e.salary, 
    a.average_salary
FROM 
    employees e
JOIN 
    avg_salary a
ON 
    e.department = a.department;
```
- In this example:
- The CTE avg_salary calculates the average salary for each department.
- The main SELECT statement joins the employees table with the CTE to display each employee's details along with the average salary of their department


## Recursive CTE
- A recursive Common Table Expression (CTE) in SQL is a CTE that references itself. This allows you to perform recursive operations, which are particularly useful for querying hierarchical data, such as organizational charts or tree structures.

- Structure of a Recursive CTE
- A recursive CTE typically consists of three parts:
- Anchor Member: The initial query that provides the base result set.
- Recursive Member: The query that references the CTE itself, allowing it to repeatedly execute.
- Termination Condition: A condition that stops the recursion.
- Syntax
```sql
WITH RECURSIVE cte_name (column_list) AS (
    -- Anchor member
    initial_query
    UNION ALL
    -- Recursive member
    recursive_query
)
SELECT * FROM cte_name;
```
- Example
- Let's say you have a table employees with columns employee_id, employee_name, and manager_id. You want to list all employees and their hierarchical levels in the organization.
```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor member: Select the top-level managers (those without a manager)
    SELECT 
        employee_id, 
        employee_name, 
        manager_id, 
        0 AS level
    FROM 
        employees
    WHERE 
        manager_id IS NULL
    
    UNION ALL
    
    -- Recursive member: Select employees and increment their level
    SELECT 
        e.employee_id, 
        e.employee_name, 
        e.manager_id, 
        eh.level + 1
    FROM 
        employees e
    INNER JOIN 
        employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT 
    employee_id, 
    employee_name, 
    manager_id, 
    level
FROM 
    employee_hierarchy;
```
- In this example:
- The anchor member selects top-level managers (those with NULL in manager_id).
- The recursive member joins the employees table with the CTE itself to find employees reporting to the managers selected in the previous step, incrementing their hierarchical level.
- The recursion continues until all employees are processed

 ## How to optimize SQL Server SPRO
- Optimizing SQL Server stored procedures (sprocs) can significantly improve performance and efficiency. Here are some key tips and techniques:

1. Use SET NOCOUNT ON
- This prevents the message indicating the number of rows affected by a T-SQL statement from being returned after each statement, reducing network traffic.
```sql
CREATE PROCEDURE YourProcedure
AS
BEGIN
    SET NOCOUNT ON;
    -- Your code here
END
```
2. Avoid Using SELECT *
- Specify only the columns you need. This reduces the amount of data transferred and can improve performance.
```sql
SELECT column1, column2 FROM YourTable;
```

3. Use Proper Indexing
- Ensure that your tables have appropriate indexes to speed up data retrieval. Use the INDEX hint if necessary.

4. Use EXISTS Instead of COUNT
- When checking for the existence of rows, use EXISTS rather than COUNT(*).
```sql
IF EXISTS (SELECT 1 FROM YourTable WHERE condition)
BEGIN
    -- Your code here
END
```

5. Avoid Cursors
- Cursors can be slow and resource-intensive. Use set-based operations instead.

6. Optimize Joins
- Ensure that joins are optimized by indexing the columns used in the join conditions. Also, prefer inner joins over outer joins when possible.

7. Use sp_executesql for Dynamic SQL
- sp_executesql allows for parameterized queries, which can improve performance by reusing execution plans and reducing SQL injection risks.
```sql
DECLARE @sql NVARCHAR(MAX);
SET @sql = N'SELECT * FROM YourTable WHERE column = @value';
EXEC sp_executesql @sql, N'@value INT', @value = 123;
```

8. Keep Transactions Short
- Long transactions can lead to locking and blocking issues. Keep transactions as short as possible to improve concurrency.

9. Use Schema Name
- Always use the schema name when referencing objects to avoid unnecessary lookups and improve plan reuse.
```sql
SELECT column1 FROM dbo.YourTable;
```

10. Avoid Using sp_ Prefix
- Avoid using the sp_ prefix for user-defined stored procedures as SQL Server first looks for these in the master database.

11. Monitor and Analyze Performance
- Use SQL Server Profiler, Execution Plans, and Dynamic Management Views (DMVs) to monitor and analyze the performance of your stored procedures

## can we create clustered index on unique key column
- Yes, you can create a clustered index on a unique key column. When you create a UNIQUE constraint on a column, SQL Server typically creates a unique nonclustered index by default. However, you can specify that the unique index should be clustered if no other clustered index exists on the table

## how to optimikze SQL queries
- Optimizing SQL queries can significantly improve the performance of your database operations. Here are some key techniques to help you get started:

1. Use Indexes Wisely:
- Add Missing Indexes: Ensure that frequently queried columns are indexed.
- Check for Unused Indexes: Remove indexes that are not being used to save resources.

2. Optimize Query Structure:
- **Avoid SELECT ***: Specify only the columns you need2.
- Use Joins Efficiently: Prefer INNER JOIN over WHERE for joining tables.
- Limit the Use of Subqueries: Use joins instead of subqueries when possible.

3. Filter Data Effectively:
- Use WHERE Clauses: Filter data as early as possible in your query.
- Avoid Wildcards at the Beginning: Use wildcards at the end of a phrase only.

4. Monitor and Analyze Performance:
- Execution Plans: Analyze execution plans to understand how queries are being processed1.
- Statistics and Metrics: Use tools to monitor query performance and identify bottlenecks.

5. Optimize Data Retrieval:
- Limit Rows Returned: Use LIMIT or TOP to restrict the number of rows returned4.
- Avoid Redundant Data: Ensure you are not retrieving unnecessary data

## .NET async await
- The async and await keywords in .NET are used to simplify asynchronous programming. Here's a brief overview of how they work:

1. async Keyword:
- Marks a method as asynchronous.
- Allows the method to run in the background while other code continues to execute.
- The method returns a Task or Task<T>, which represents the ongoing operation.

2. await Keyword:
- Used inside an async method to pause its execution until the awaited task completes.
- The method resumes execution once the task is finished, allowing for non-blocking operations.
- Example
```csharp
public async Task<string> GetDataAsync()
{
    // Simulate an asynchronous operation
    await Task.Delay(1000);
    return "Data retrieved";
}

public async Task MainAsync()
{
    string result = await GetDataAsync();
    Console.WriteLine(result);
}
```
- In this example:
- GetDataAsync is an asynchronous method that simulates a delay and then returns a string.
- MainAsync calls GetDataAsync and waits for it to complete using await.
- Benefits
    - Improved Responsiveness: Keeps the UI responsive in client applications.
    - Better Resource Utilization: Frees up threads to handle other tasks in server applications.
 
## How async await works
- The async and await keywords in .NET are used to simplify asynchronous programming by allowing you to write code that looks synchronous but runs asynchronously. Here's a detailed look at how they work:

- **How async and await Work**
1. async Keyword:
- When you mark a method with the async keyword, it allows you to use the await keyword inside that method.
- An async method typically returns a Task or Task<T>, which represents the ongoing operation.

2. await Keyword:
- The await keyword is used to pause the execution of an async method until the awaited task completes.
- It allows other operations to run while waiting for the task to finish, making the application more responsive.
- Example
```csharp
public async Task<string> GetDataAsync()
{
    // Simulate an asynchronous operation
    await Task.Delay(1000);
    return "Data retrieved";
}

public async Task MainAsync()
{
    string result = await GetDataAsync();
    Console.WriteLine(result);
}
```
- In this example:
- GetDataAsync is an asynchronous method that simulates a delay and then returns a string.
- MainAsync calls GetDataAsync and waits for it to complete using await.
- **How It Works Under the Hood**
- When the compiler encounters the await keyword, it transforms the method into a state machine. This state machine manages the execution of the method, allowing it to pause and resume at the await points. Here's a simplified breakdown:
    - State Machine Creation: The compiler generates a state machine for the async method.
    - Task Continuation: When the await keyword is used, the method is paused, and control is returned to the caller. The state machine registers a continuation that will resume the method once the awaited task completes.
    - Resuming Execution: Once the awaited task completes, the state machine resumes the method from where it left off.
- Benefits
    - Improved Responsiveness: Keeps the UI responsive in client applications.
    - Better Resource Utilization: Frees up threads to handle other tasks in server applications

## Explain the state machine concept.
- A state machine, also known as a finite-state machine (FSM), is a mathematical model used to design algorithms and describe the behavior of systems. Here's a breakdown of the concept:

- **Key Components**
- States: Distinct modes or conditions in which the system can exist.
- Transitions: Rules that define how the system moves from one state to another based on inputs or events.
- Inputs: External factors or events that trigger transitions between states.
- Actions: Operations that occur during transitions or while in a particular state.

- **How It Works**
- A state machine starts in an initial state and transitions between states based on inputs. Each state represents a specific condition of the system, and transitions define how the system responds to different inputs.

- Example
- States: Locked, Unlocked
- Inputs: Coin (inserts a coin), Push (pushes the turnstile)
- Transitions:
    - From Locked to Unlocked when a coin is inserted.
    - From Unlocked to Locked when the turnstile is pushed.
      
- **State Diagram**
- A state diagram visually represents the states and transitions. For the turnstile example, it would look like this:

- [Locked] --(Coin)--> [Unlocked]
- [Unlocked] --(Push)--> [Locked]

- **Applications**
- State machines are used in various fields, including:
    - Computer Science: Designing algorithms, compilers, and network protocols.
    - Engineering: Modeling digital circuits and control systems.
    - Software Development: Managing user interfaces and game logic.
      
- **Benefits**
    - Clarity: Provides a clear and structured way to model complex systems.
    - Predictability: Ensures predictable behavior by defining explicit states and transitions.

 ## explain the code execution of async await
 - The async and await keywords in .NET are used to handle asynchronous operations in a way that makes the code easier to read and maintain. Here's a detailed explanation of how the code execution works:

- **How async and await Work**
1. Marking a Method as async:
- When you mark a method with the async keyword, it allows you to use the await keyword inside that method.
- An async method typically returns a Task or Task<T>, which represents the ongoing operation.

2. Using the await Keyword:
- The await keyword is used to pause the execution of an async method until the awaited task completes.
- It allows other operations to run while waiting for the task to finish, making the application more responsive.
- Example
```csharp
public async Task<string> GetDataAsync()
{
    // Simulate an asynchronous operation
    await Task.Delay(1000);
    return "Data retrieved";
}

public async Task MainAsync()
{
    string result = await GetDataAsync();
    Console.WriteLine(result);
}
```
- In this example:
- GetDataAsync is an asynchronous method that simulates a delay and then returns a string.
- MainAsync calls GetDataAsync and waits for it to complete using await.
- Code Execution Flow

- **State Machine Creation:**
- When the compiler encounters the await keyword, it transforms the method into a state machine. This state machine manages the execution of the method, allowing it to pause and resume at the await points.

- **Task Continuation:**
- When the await keyword is used, the method is paused, and control is returned to the caller. The state machine registers a continuation that will resume the method once the awaited task completes.

- **Resuming Execution:**
- Once the awaited task completes, the state machine resumes the method from where it left off.

- **Benefits**
- Improved Responsiveness: Keeps the UI responsive in client applications.
- Better Resource Utilization: Frees up threads to handle other tasks in server applications.

- **Detailed Example**
```csharp
public async Task<string> GetDataAsync()
{
    // Step 1: The method is marked as async, allowing the use of await.
    await Task.Delay(1000); // Step 2: Execution is paused here, and control is returned to the caller.
    return "Data retrieved"; // Step 3: Once the delay completes, execution resumes here.
}

public async Task MainAsync()
{
    // Step 4: The method calls GetDataAsync and awaits its completion.
    string result = await GetDataAsync(); // Step 5: Execution is paused here until GetDataAsync completes.
    Console.WriteLine(result); // Step 6: Once GetDataAsync completes, execution resumes here.
}
```
- In this flow:
- The await Task.Delay(1000) line pauses the execution of GetDataAsync for 1 second.
- During this pause, control is returned to the caller (MainAsync), which also pauses at await GetDataAsync().
- After 1 second, GetDataAsync resumes, returns the string "Data retrieved", and MainAsync continues to print the result.
- This mechanism ensures that the application remains responsive and can handle other tasks while waiting for asynchronous operations to complete.

## What happens if I don't use await?
- If you don't use await when calling an async method, several things can happen:

- **Task Runs Asynchronously:**
- The async method will still run asynchronously, but the calling method will not wait for it to complete. This means the calling method will continue executing the next lines of code immediately.

- **Unobserved Exceptions:**
- If the async method throws an exception, it will not be caught by the calling method unless you explicitly handle it. This can lead to unobserved exceptions, which might cause your application to crash or behave unexpectedly1.

- **Task Not Completed:**
- The calling method might complete before the async method finishes its execution. This can lead to incomplete operations or unexpected results, especially if the calling method depends on the result of the async method2.

- Example
```csharp
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);
    return "Data retrieved";
}

public void MainMethod()
{
    var task = GetDataAsync();
    Console.WriteLine("Task started");
    // No await here, so MainMethod continues immediately
    Console.WriteLine("Task not awaited");
}
```
- In this example:
- GetDataAsync starts running asynchronously.
- MainMethod continues executing without waiting for GetDataAsync to complete.
- "Task started" and "Task not awaited" are printed immediately, while "Data retrieved" is printed after a delay.

- **Best Practices**
- Always Await: As a best practice, you should always await the async method to ensure proper execution flow and exception handling3.
- Handle Exceptions: If you choose not to await, make sure to handle potential exceptions and check the task's status.

## Why gudelines have been defined for RESTful api
- Guidelines for RESTful APIs have been defined to ensure consistency, reliability, and ease of use across different systems and applications. Here are some key reasons why these guidelines are important:

1. Consistency
- Standardization: Following common guidelines ensures that APIs are designed in a consistent manner, making them easier to understand and use for developers.
- Predictability: Consistent design patterns reduce the learning curve for developers, as they can predict how the API will behave based on familiar conventions.
2. Interoperability
- Seamless Integration: Standardized APIs facilitate communication between different systems, allowing them to work together more effectively.
- Cross-Platform Compatibility: Ensures that APIs can be used across various platforms and technologies without compatibility issues.
3. Scalability
- Statelessness: RESTful APIs are designed to be stateless, meaning each request from a client contains all the information needed to process it. This enhances scalability by allowing servers to handle requests independently.
- Caching: Proper use of caching mechanisms can improve performance and reduce server load.
4. Security
- Best Practices: Adhering to security guidelines helps protect sensitive data and prevent unauthorized access.
- Error Handling: Standardized error responses make it easier to diagnose and fix issues, improving the overall security and reliability of the API.
5. Maintainability
- Clear Documentation: Well-defined guidelines lead to better documentation, making it easier to maintain and update the API.
- Versioning: Proper versioning practices ensure that changes to the API do not break existing clients.
6. User Experience
- Ease of Use: APIs designed with clear and consistent guidelines are easier for developers to use, leading to a better overall user experience.
- Error Handling: Consistent error handling and messaging improve the developer experience by providing clear guidance on how to resolve issues.
  
- **Note:** By following these guidelines, developers can create RESTful APIs that are robust, scalable, and easy to use, ultimately leading to more efficient and effective software development

## Lazy in singleton pattern
- In the Singleton pattern, lazy initialization is a technique where the instance of the singleton class is not created until it is needed for the first time. This can be beneficial in terms of resource management and performance, especially if the initialization of the instance is expensive or if the instance might not be used at all during the application's lifetime.

- **Lazy Initialization in Singleton Pattern**
- Here's how lazy initialization can be implemented in the Singleton pattern:

1. **Basic Lazy Initialization:**
- The instance is created only when it is requested for the first time.
- This approach is not thread-safe.
```csharp
public sealed class Singleton
{
    private static Singleton instance = null;
    private Singleton() { }

    public static Singleton Instance
    {
        get
        {
            if (instance == null)
            {
                instance = new Singleton();
            }
            return instance;
        }
    }
}
```

2. **Thread-Safe Lazy Initialization:**
- Ensures that the instance is created in a thread-safe manner.
- Uses locking to prevent multiple threads from creating multiple instances.
```csharp
public sealed class Singleton
{
    private static Singleton instance = null;
    private static readonly object padlock = new object();
    private Singleton() { }

    public static Singleton Instance
    {
        get
        {
            lock (padlock)
            {
                if (instance == null)
                {
                    instance = new Singleton();
                }
                return instance;
            }
        }
    }
}
```

3. **Lazy<T> Initialization:**
- Utilizes the Lazy<T> type provided by .NET, which handles lazy initialization and thread safety internally.
```csharp
public sealed class Singleton
{
    private static readonly Lazy<Singleton> lazyInstance = new Lazy<Singleton>(() => new Singleton());
    private Singleton() { }

    public static Singleton Instance
    {
        get
        {
            return lazyInstance.Value;
        }
    }
}
```

- **Benefits of Lazy Initialization**
- Resource Efficiency: The instance is created only when needed, saving resources if the instance is never used.
- Performance: Reduces the initial load time of the application by deferring the creation of the instance.
- Thread Safety: When implemented correctly, it ensures that only one instance is created even in a multithreaded environmen

## can we declare action method as private in .net core
- In ASP.NET Core, action methods in controllers must be public. If you declare an action method as private, it won't be accessible via routing, and the framework won't recognize it as an action method.

- However, if you want to prevent a method from being treated as an action, you can use the [NonAction] attribute. This attribute can be applied to public methods to exclude them from being considered as actions by the framework34.

- Here's an example:
```csharp
public class MyController : Controller
{
    [NonAction]
    public IActionResult HelperMethod()
    {
        // This method won't be accessible as an action.
        return View();
    }

    public IActionResult Index()
    {
        // This is a public action method.
        return View();
    }
}
```
In this example, HelperMethod is marked with [NonAction], so it won't be accessible as an action method, even though it's public.

## conditional implementation of middleware, .net core
- In ASP.NET Core, you can conditionally implement middleware using the UseWhen and MapWhen methods. These methods allow you to execute middleware based on specific conditions.

- **UseWhen**
- The UseWhen method executes middleware only when a specified condition is met. This is useful when you want most of your middleware to run for all requests but have some middleware that should only run for certain requests.

- Here's an example:
```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseMiddleware<CommonMiddleware>();

    app.UseWhen(context => context.Request.Path.StartsWithSegments("/api"), appBuilder =>
    {
        appBuilder.UseMiddleware<ApiSpecificMiddleware>();
    });

    app.UseMiddleware<OtherMiddleware>();
}
```
- In this example, ApiSpecificMiddleware will only run for requests where the path starts with /api12.

- **MapWhen**
- The MapWhen method splits the middleware pipeline into two branches based on a condition. This is useful when you want to create completely separate middleware pipelines for different types of requests.

- Here's an example:
```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseMiddleware<CommonMiddleware>();

    app.MapWhen(context => context.Request.Path.StartsWithSegments("/admin"), appBuilder =>
    {
        appBuilder.UseMiddleware<AdminMiddleware>();
    });

    app.UseMiddleware<OtherMiddleware>();
}
```
- In this example, requests with paths starting with /admin will be handled by AdminMiddleware, and other requests will continue through the rest of the pipeline34.

- **Environment-Based Middleware**
- You can also conditionally add middleware based on the environment (e.g., Development, Production):
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
    }

    app.UseMiddleware<CommonMiddleware>();
}
```
- In this example, the DeveloperExceptionPage middleware is only used in the Development environment

## map() and use() in .core
- In ASP.NET Core, the Map and Use methods are used to configure middleware in the request processing pipeline. Here's a brief overview of each:

- **Use**
- The Use method adds middleware to the pipeline. Middleware added with Use can perform operations both before and after the next middleware in the pipeline. It can also decide whether to pass the request to the next middleware.

- Here's an example:
```csharp
public void Configure(IApplicationBuilder app)
{
    app.Use(async (context, next) =>
    {
        // Do work before the next middleware
        await context.Response.WriteAsync("Before next middleware\n");
        await next.Invoke();
        // Do work after the next middleware
        await context.Response.WriteAsync("After next middleware\n");
    });

    app.Run(async context =>
    {
        await context.Response.WriteAsync("Hello from terminal middleware\n");
    });
}
```
- In this example, the first middleware writes a message before and after calling the next middleware1.

- **Map**
- The Map method branches the middleware pipeline based on the request path. It allows you to specify a branch of middleware to handle requests that match a certain path prefix.

- Here's an example:
```csharp
public void Configure(IApplicationBuilder app)
{
    app.Map("/branch", branchApp =>
    {
        branchApp.Use(async (context, next) =>
        {
            await context.Response.WriteAsync("Branch middleware\n");
            await next.Invoke();
        });
    });

    app.Run(async context =>
    {
        await context.Response.WriteAsync("Hello from main pipeline\n");
    });
}
```
- In this example, requests with paths starting with /branch will be handled by the middleware in the branchApp pipeline23.

- **Differences**
- Use: Adds middleware to the main pipeline and can decide whether to pass the request to the next middleware.
- Map: Branches the pipeline based on the request path, creating a separate pipeline for specific paths.

## when to use angular and react
- Choosing between Angular and React depends on the specific needs of your project and your team's expertise. Here's a comparison to help you decide:

- **When to Use Angular**
- Enterprise-Level Applications: Angular is a full-fledged framework with a lot of built-in features, making it ideal for large-scale applications that require a robust structure and maintainability.
- Two-Way Data Binding: If your application needs two-way data binding, Angular's built-in support for this feature can simplify development.
- Standardization: Angular enforces a strict project structure and coding standards, which can be beneficial for large teams and long-term projects.
- Comprehensive Tooling: Angular comes with a complete set of tools for testing, routing, and form handling, reducing the need for third-party libraries12.
- **When to Use React**
- Dynamic and Interactive UIs: React is excellent for building highly dynamic and interactive user interfaces due to its efficient virtual DOM and component-based architecture.
- Flexibility: React is a library, not a full framework, which gives you the flexibility to choose your own tools and libraries for routing, state management, etc.
- Learning Curve: React has a simpler learning curve compared to Angular, making it easier for new developers to get started quickly.
- Incremental Adoption: React can be integrated into existing projects incrementally, allowing you to adopt it without a complete rewrite13.
- **Considerations**
- Existing Projects: If you are adding to an existing project, React might be more suitable due to its flexibility and ease of integration2.
- New Projects: For new projects, Angular can provide a more comprehensive solution out of the box, especially if you need a lot of built-in functionality2.
- Ultimately, the choice between Angular and React should be based on your project's requirements, your team's familiarity with the technologies, and the specific features you need.

## How to decide whether to opt for Angular or react for buildingproject
- Deciding between Angular and React for your project involves considering several factors. Here are some key points to help you make an informed decision:

1. Project Requirements
- Complexity and Scale: For large-scale, enterprise-level applications with complex requirements, Angular might be more suitable due to its comprehensive framework and built-in features.
- Dynamic and Interactive UIs: If your project requires highly dynamic and interactive user interfaces, React's efficient virtual DOM and component-based architecture can be advantageous.
2. Development Team Expertise
- Familiarity: Consider the expertise and experience of your development team. If your team is more familiar with one technology, it might be more efficient to use that.
- Learning Curve: React generally has a simpler learning curve compared to Angular, which can be beneficial if your team is new to these technologies.
3. Flexibility and Ecosystem
- Flexibility: React offers more flexibility as it is a library rather than a full framework. This allows you to choose your own tools and libraries for routing, state management, etc.
- Comprehensive Solution: Angular provides a complete solution out of the box, including tools for testing, routing, and form handling, which can reduce the need for third-party libraries.
4. Performance Considerations
- Rendering Performance: React's virtual DOM can offer better performance for applications with a lot of dynamic content and frequent updates.
- Initial Load Time: Angular applications might have a larger initial load time due to the framework's size, but this can be mitigated with techniques like lazy loading.
5. Community and Support
- Community Size: Both Angular and React have large, active communities, but React's community is slightly larger, which can be beneficial for finding resources, libraries, and support.
- Corporate Backing: Angular is maintained by Google, while React is maintained by Facebook (Meta), ensuring both have strong backing and continuous updates.
6. Long-Term Maintenance
- Standardization: Angular enforces a strict project structure and coding standards, which can be beneficial for long-term maintenance and scalability.
- Incremental Adoption: React allows for incremental adoption, making it easier to integrate into existing projects without a complete rewrite.
7. Specific Use Cases
- Single-Page Applications (SPAs): Both frameworks are well-suited for SPAs, but Angular's built-in features might give it an edge for more complex SPAs.
- Mobile Development: If you plan to extend your web application to mobile, React Native can be a strong advantage for React, allowing code reuse across web and mobile platforms.
- **Conclusion**
- Ultimately, the choice between Angular and React should be based on your project's specific needs, your team's expertise, and the features you require. Both technologies are powerful and widely used, so either choice can lead to a successful project

## create restore points in multiple async await calls, c#

## How can I handle exceptions in async methods?

## can we save output of successful async await operations and roll back failure, c#

## How can I handle retries for failed operations?

## What if the retry logic itself fails?

## data of one async await is consumed by another async await, is this synchronous call or asynchronous call

## What happens if the first operation fails or is canceled?

## What if the second operation fails after the first one succeeds?

## What if the first operation modifies external resources (e.g., a database)?

## What if the second operation modifies external resources instead?

## How can I handle resource cleanup in case of failure?

## Synchronous call vs asynchronous call

## Can you explain more about the async/await pattern?

## Paralle.For VS Parallel.Foreach

## Parallel.Invoke

## what actually async do in c#

## after 7 pm service will return rejection else will handle request

## after 7 pm service will return rejection else will handle request in .net core

## after 7 pm service will return rejection else will handle request in .net core services

## after 7 pm service will return rejection else will handle request in .net core without using middleware

## mutex, semaphore

## semaphore slim

## Can you explain more about the differences between Semaphore and SemaphoreSlim?

## steps to migrate .NET app to .NET core

## steps to deploy React app and http api in Azure

## Backend API is .net core, front end is ReactJS, how to deploy it via CI/CD pipleline with Azure

## how to implement security in reactjs

## how to secure the data in transit in web api

## Encryption and hashing techniques for data security

## how to implement multiple language functionality in react

## how to perform security scans in CI/CD pipeline

## What tools can help identify code smells in my legacy application?

## What are some common code smells in legacy applications?

## UseSerialLog middleware example

## How can I configure Serilog to log to a file?

## .net pillars

## How u handled vulnerability in .net   

## What can be scope of DI for database in .net

## How to secure web api in .net

## Show me how to create and validate JWT in C#.

## How do I handle token expiration with JWT?

## How do you handle security and authentication in a microservices architecture?

## How do I implement OAuth 2.0 in my application?

## How do I handle OAuth token expiration and refresh?

## How do I handle token revocation?

## How to monitor microservices

## how will you scale your microservices application  vertically and why you should scale it?

## which dicovery service can be used in microservices for .ent

## CQRS design pattern for microsevices

## IEnumerable vs IList

## asp.net core CORS




