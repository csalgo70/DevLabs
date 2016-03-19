# DevLabs - by Kannan Sundararajan

###Task Runner

ITaskRunner interface so that we can unit test easily 

```C#

public interface ITaskRunner
{
    Task RunTask<T>(Func<T> task, Action<T> onTaskComplete, Action<IList<Exception>> onException);

    Task RunTask<T, TResult>(Func<T, TResult> task, T taskParameter, Action<TResult> onTaskComplete,
            Action<IList<Exception>> onException);
}
```
Concrete implementation of the Task runner

```C#
public class TaskRunner : ITaskRunner
{
    public  Task RunTask<T, TResult>(Func<T, TResult> task, T taskParameter, Action<TResult> onTaskComplete, Action<IList<Exception>> onException)
    {
        return ContinueTask(Task.Factory.StartNew(() => task(taskParameter)), onTaskComplete, onException);
    }

    public Task RunTask<T>(Func<T> task, Action<T> onTaskComplete, Action<IList<Exception>> onException)
    {
       return ContinueTask(Task.Factory.StartNew(task), onTaskComplete, onException);
    }

    private static Task ContinueTask<T>(Task<T> task, Action<T> onTaskComplete, Action<IList<Exception>> onException)
    {
        var t = task.ContinueWith(
            completedTask =>
            {
                if (completedTask.Exception != null)
                {
                    onException(completedTask.Exception.Flatten().InnerExceptions);
                }
                else
                {
                    try
                    {
                        onTaskComplete(completedTask.Result);
                    }
                    catch (Exception exception)
                    {
                       //TODO: Exclude exceptions that should not be handled by user code
                       //TODO: Maybe we can add extension method to Exception to filter here 
                       //TODO: and re-throw if shouldn't be handled
        
                        onException(new[] { exception });
                    }
                }
            },
            GetSynchronizationContext());
        return t;
    }
    
    private static TaskScheduler GetSynchronizationContext()
    {
        try
        {
            return TaskScheduler.FromCurrentSynchronizationContext();
        }
        catch (InvalidOperationException)
        {
            return TaskScheduler.Default;
        }
    }

}
```
