[//]: # (title: Run Actions from Code)

**What you should know beforehand:**
* [Component model](ObtainComponentsInRuntime.md)
* [Lifetime](WorkWithLifetime.md)
* Shell locks

**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [RunActionCommand.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Actions/RunActionCommand.cs)
 
Running an action is done via the `ActionManager component`. Synchronously:
``` 
var actionManager = Shell.Instance.GetComponent<IActionManager>();
actionManager.ExecuteAction<ActionShowMessageBox>();
``` 
Or asynchronously. In this case, you have to pass a lifetime as well (if the lifetime is terminated before the action is executed, the action is removed from the queue):
``` 
actionManager.ExecuteActionAsync<ActionShowMessageBox>(_lifetime);
```
To have more options on how to execute the action (synchronously or asynchronously with various types of locks), you may decide to wrap a synchronous `ExecuteAction` call with one of the `ShellLocks` methods:
```
var shellLocks = Shell.Instance.GetComponent<IShellLocks>();
shellLocks.ExecuteOrQueue(_lifetime, "ExecuteSampleAction", () =>
    {
        actionManager.ExecuteAction<ActionShowMessageBox>();
    });
```