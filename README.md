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

 
