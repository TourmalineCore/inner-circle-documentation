## Research automatic architecture testing

## ArchitectureAnalyzer (https://github.com/mao2009/ArchitectureAnalyzer)

ArchitectureAnalyzer is a Roslyn analyzer. It lets you describe layers and the directions of dependencies between them.

Below is a configuration. It describes the following rules:
- `Core` cannot depend on other layers.
- `Application` can depend on `Core`.
- `API` can depend on both `Application` and `Core`.

```json
{
    "schemaVersion": 5,
    "unclassifiedCode": "error",
    "layers": [
        {
            "name": "Api",
            "namespaceRoots": [
                "Api"
            ]
        },
        {
            "name": "Application",
            "namespaceRoots": [
                "Application"
            ]
        },
        {
            "name": "Core",
            "namespaceRoots": [
                "Core"
            ]
        }
    ],
    "forbiddenDependencies": [
        {
            "from": "Core",
            "to": "Application",
            "reason": "Core must not depend on the outer Application layer."
        },
        {
            "from": "Core",
            "to": "Api",
            "reason": "Core must not depend on the outer Api layer."
        },
        {
            "from": "Application",
            "to": "Api",
            "reason": "Application must not depend on the outer Api layer."
        }
    ]
}
```

**What it checks:**
- The direction of dependencies between layers.
- Whether a namespace exists.

**Limitations:**
- The analyzer works at the namespace level, not at the type level. If a handler and a command are in the same namespace, it is impossible to add the rule "controllers call only handlers".
- The analyzer does not check signatures or whether methods exist. So it is impossible to add rules like "every handler must have a HandleAsync method".

**Pros:**
- Easy to set up.
- Errors at compile time.

**Cons:**
- It does not cover complex rules.

## NetArchTest (https://github.com/BenMorris/NetArchTest)

It lets you create tests that make sure your rules for class design, naming, and dependencies are followed.

Below is code with two tests:
- It checks that layers do not depend on layers they should not depend on.
- It checks that every `handler` has a method called `HandleAsync`.

```C#
using System.Reflection;
using Core;
using NetArchTest.Rules;
using Xunit;

namespace Api;

[UnitTest]
public class NetArchTests
{
    [Theory]
    [InlineData("Core", "Application")]
    [InlineData("Core", "Api")]
    [InlineData("Application", "Api")]
    public void Layer_ShouldNotHaveDependencyOnOtherLayer(string from, string to)
    {
        var fromAssembly = Assembly.Load(from);
        var toAssembly = Assembly.Load(to);

        var result = Types
            .InAssembly(fromAssembly)
            .Should()
            .NotHaveDependencyOn(toAssembly.GetName().Name)
            .GetResult();

        Assert.True(
            result.IsSuccessful,
            $"{from} must not depend on the {to}. " +
            $"Violators: {string.Join(", ", result.FailingTypeNames ?? [])}"
        );
    }

    [Fact]
    public void AllHandlers_ShouldHaveHandleAsyncMethod()
    {
        var handlers = Types
            .InAssembly(Assembly.Load("Application"))
            .That()
            .HaveNameEndingWith("Handler")
            .GetTypes();

        var violations = new List<string>();

        foreach (var handler in handlers)
        {
            var method = handler.GetMethod("HandleAsync",
                BindingFlags.Public | BindingFlags.Instance);

            if (method == null)
            {
                violations.Add(handler.FullName!);
            }
        }

        Assert.True(violations.Count == 0,
            $"These handlers do not have a HandleAsync method: {string.Join(", ", violations)}");
    }
}
```

**What it checks:**
- The direction of dependencies between layers.
- Unlike ArchitectureAnalyzer, it can see individual types and check their members, so the rule about `HandleAsync` becomes possible to add.

**Limitations:**
- NetArchTest checks that a dependency between types exists, but it does not analyze the body of methods. So the rule "only a `handler` is called in a controller" cannot be added.

**Pros:**
- Integration with a test framework — xUnit, NUnit, etc.
- Flexibility (it lets you write custom rules).

**Cons:**
- No errors at compile time.
- Harder to set up.
- The last update of the package was in 2021.

## ArchUnitTest (https://github.com/TNG/ArchUnitNET)

Below is code with three tests:
- It checks that layers do not depend on layers they should not depend on.
- It checks that every `handler` has a method called `HandleAsync`.
- It checks that a controller calls only a `handler`.

```C#
using System.Reflection;
using Application;
using ArchUnitNET.Domain;
using ArchUnitNET.Fluent;
using ArchUnitNET.Loader;
using ArchUnitNET.xUnit;
using Core;
using Xunit;
using static ArchUnitNET.Fluent.ArchRuleDefinition;

namespace Api;

[UnitTest]
public class ArchNetTests
{
    private static readonly Architecture Architecture = new ArchLoader()
        .LoadAssemblies(
            System.Reflection.Assembly.Load("Core"),
            System.Reflection.Assembly.Load("Application"),
            System.Reflection.Assembly.Load("Api")
        )
        .Build();

    private static readonly IObjectProvider<IType> CoreLayer = Types()
        .That()
        .ResideInNamespace("Core")
        .As("Core Layer");

    private static readonly IObjectProvider<IType> ApplicationLayer = Types()
        .That()
        .ResideInNamespace("Application")
        .As("Application Layer");

    private static readonly IObjectProvider<IType> ApiLayer = Types()
        .That()
        .ResideInNamespace("Api")
        .As("Api Layer");

    [Theory]
    [InlineData("Core", "Application")]
    [InlineData("Core", "Api")]
    [InlineData("Application", "Api")]
    public void Layer_ShouldNotHaveDependencyOnOtherLayer(string from, string to)
    {
        var fromLayer = GetLayerByName(from);
        var toLayer = GetLayerByName(to);

        IArchRule rules = Types()
            .That()
            .Are(fromLayer)
            .Should()
            .NotDependOnAny(toLayer);

        rules.Check(Architecture);
    }

    [Fact]
    public void AllHandlers_ShouldHaveHandleAsyncMethod()
    {
        var handlers = Types()
            .That()
            .ResideInAssembly(typeof(ApplicationAssemblyMarker).Assembly)
            .And()
            .HaveNameEndingWith("Handler")
            .GetObjects(Architecture);

        var violations = new List<string>();

        foreach (var handler in handlers)
        {
            var hasMethod = handler.Members
                .OfType<MethodMember>()
                .Any(x => x.Name.StartsWith("HandleAsync", StringComparison.OrdinalIgnoreCase));

            if (!hasMethod)
            {
                violations.Add(handler.FullName);
            }
        }

        Assert.True(violations.Count == 0,
            $"These handlers do not have a HandleAsync method: {string.Join(", ", violations)}");
    }

    [Fact]
    public void Controllers_ShouldOnlyCallHandlerMethods()
    {
        var controllers = Classes()
            .That()
            .ResideInAssembly(typeof(ApiAssemblyMarker).Assembly)
            .And()
            .HaveNameEndingWith("Controller")
            .GetObjects(Architecture);

        var violations = new List<string>();

        foreach (var controller in controllers)
        {
            foreach (var method in controller.GetMethodMembers())
            {
                foreach (var call in method.GetCalledMethods())
                {
                    var isHandler = call.Name.StartsWith(
                        "HandleAsync",
                        StringComparison.OrdinalIgnoreCase
                    );

                    // Skip system and framework calls
                    var declaringType = call.DeclaringType?.FullName ?? string.Empty;

                    var isSystemCall =
                        declaringType.StartsWith("Microsoft.") ||
                        declaringType.StartsWith("System.") ||
                        declaringType.StartsWith("Api.") ||
                        call.Name == ".ctor" ||
                        call.Name == ".cctor";

                    if (isSystemCall)
                    {
                        continue;
                    }

                    if (!isHandler)
                    {
                        violations.Add(
                            $"  — {controller.Name}.{method.Name}() " +
                            $"calls {declaringType}::{call.Name}() " +
                            $"— controllers may only call HandleAsync() methods"
                        );
                    }
                }
            }
        }

        Assert.True(violations.Count == 0,
            $"Controllers must only call HandleAsync() methods from the Application layer. {Environment.NewLine}" +
            $"Found {violations.Count} forbidden call(s):{Environment.NewLine}" +
            $"{string.Join(Environment.NewLine, violations)}");
    }

    private static IObjectProvider<IType> GetLayerByName(string name)
    {
        return name switch
        {
            "Core" => CoreLayer,
            "Application" => ApplicationLayer,
            "Api" => ApiLayer,
            _ => throw new NotImplementedException(),
        };
    }
}
```

**What it checks:**
- The direction of dependencies between layers.
- Unlike ArchitectureAnalyzer, it can see individual types and check their members, so the rule about `HandleAsync` becomes possible to add.
- Calls inside methods.

**Pros:**
- Integration with a test framework — xUnit, NUnit, etc.
- Flexibility (it lets you write custom rules).
- Regular updates.

**Cons:**
- No errors at compile time.
- Harder to set up.

## Comparative Table

| Check | ArchitectureAnalyzer | NetArchTest | ArchUnitNET |
| :--- | :---: | :---: | :---: |
| **Layer dependency direction** | + | + | + |
| **Namespace existence validation** | + | + | + |
| **Type and naming conventions** | - | + | + |
| **Method/Member existence** (e.g., `HandleAsync`, `ExecuteAsync`) | - |  + | + |
| **Method body analysis** (e.g., controller can call only `handler`) | - | - | + |
| **Compile-time error reporting** | + | - | - |
| **Configuration format** | JSON | C# | C# |
| **Flexibility for custom rules** | Low | High | High |