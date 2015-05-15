---
---

# Temporary Settings in Tests

The ReSharper test environment will use default values for settings. If your test needs particular values in the settings, you need to set them yourself in the test, and they should be reset at the end of the test. This is important because ReSharper will try to reuse the current project and solution for future tests.

The `BaseTest` class provides two methods to help with this:

```csharp
public class BaseTest : BaseTestNoShell
{
  public void ExecuteWithinSettingsTransaction( Action<IContextBoundSettingsStore> action);
  public ChangeSettingsTemporarilyCore ChangeSettingsTemporarily(Lifetime lifetime);

  // […snip…]
}
```

Normally, you would use the `ExecuteWithinSettingsTransaction` method. This takes an action that receives an instance of `IContextBoundSettingsStore`, and your code will use this settings store to temporarily change the settings. Once the action is complete, the changes to the settings are automatically undone.

```csharp
[Test]
public void MyTest()
{
  ExecuteWithinSettingsTransaction(store => {

    // Update settings
    store.SetValue((CSharpLanguageProjectSettings s) => {
      s.LanguageLevel = CSharpLanguageLevel.CSharp50
    });

    // Do test
    // […snip…]
  });
}
```

## Advanced Usage

The `ChangeSettingsTemporarily` method does all of the work. It creates an instance of `ChangeSettingsTemporarilyCore` using the `Lifetime` you pass in. It creates a new in-memory settings layer with a very high priority, which means it overrides all other layers (project, solution, global), and any writes against the settings store will go to this temporary layer. The layer is created with the given `Lifetime` instance, and when the `Lifetime` terminates, the layer is automatically removed and cleaned up. Should you need it, you can get access to the layer's storage from the returned instance of `ChangeSettingsTemporarilyCore`.

The `ExecuteWithinSettingsTransaction` method makes this an easier API to consume. It creates a new `Lifetime` instance, calls `ChangeSettingsTemporarily` and then gets a bound settings store accessor which it uses in a call to the passed in action. Once the action completes, the `Lifetime` is terminated, which cleans up the store accessor, and the instance of `ChangeSettingsTemporarilyCore`, and therefore removes the temporary settings layer.
