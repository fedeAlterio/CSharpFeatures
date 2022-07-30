# CSharpFeatures

## Classes
Allocate sempre nello heap
```c#
class A {}
```

## Structs
Allocate nello stack quando è possibile
```c#
struct A {}
```

## Interfaces
Insieme di signature senza implementazione (in realtà non è più vero) che possono essere implementate da classi o struct. 
```c#
interface IA
{
  void DoStuff();
}
class A : IA 
{
  void DoStuff();
}

struct A : IA 
{
  void DoStuff();
}
```

## Modificatori di accesso
![image](https://user-images.githubusercontent.com/48481385/181908083-38a55e5f-a4e6-49b3-b2e7-abda0e4a9486.png)

## Delegates
Una classe speciale con una sintassi speciale che incapsula una "funzione". Ci sono conversioni implicite tra delegati e metodi, delegati e lambda expressions etc. 
```c#
delegate void IntIntToInt(int a, int b);
IntIntToInt printSum = (a, b) => Console.WriteLine(a+b);
printSum(1, 2);
```
I delegati si possono sommare
```c#
delegate void IntIntToVoid(int a, int b);
IntIntToVoid printSum = (a, b) => Console.WriteLine(a+b);
IntIntToVoid printDfference = (a,b) => Console.WriteLine(a-b);
IntIntToVoid printSumAndDifference = printSum + printDfference;
printSum(1, 2);
```

## Events
Un evento è un un delegato cichiarato con una keyword event davanti. Solo la classe dentro cui si dichiara l'evento può invocarlo (Cioè usare l'operatore ()). Ha senso quindi solo all'interno di una classe, non di un metodo.
```c#
public delegate void NumberChangedHandler(int number);
class A
{
    public event NumberChangedHandler NumberChanged;

    private void OnNumberChanged(int number) => NumberChanged(number);  // OK
}

var a = new A();
a.NumberChanged += number => Console.WriteLine(number); // OK
a.NumberChanged(); // KO

```

## Override Operatori
![image](https://user-images.githubusercontent.com/48481385/181908629-db78f757-ed79-4e6f-ae49-c429ef745090.png)

## Extension Methods
Classi static che permettono di estendere una classe (o un interfaccia) senza modificarla 
```c#
class A
{
    public void PrintObject(object obj) => Console.WriteLine(obj);
}

static class AExtensions
{
    static void PrintInt(this A @this, int a) => @this.PrintObject(a);
}

var a = new A();
a.PrintInt(2);
```

## LINQ
Language INtegrated Query.
Listone di extension methods di un IEnumerable (l'interfaccia minima che rende possibile il foreach, quindi un iterable in java tipo) che fanno scrivere in modo dichiarativo come ottenere i dati. Ci sono 2 sintassi equivalenti:

```c#
IEnumerable<int> Numbers()
{
    int i = 0;
    while (true)
        yield return i++;
}


// Sintassi 1
var doubledOddNumbers = Numbers().Where(number => number % 2 != 0).Select(number => 2 * number); // è tutto lazy, non viene eseguito nulla finchè non serve
var firstDoubledOdd = doubledOddNumbers.FirstOrDefault(); // no problem

// Sintassi2
var doubledEvenNumbers = from number in Numbers()
                         where number % 2 == 0
                         select number * 2;
var firstDoubledEven = doubledEvenNumbers.FirstOrDefault(); // No problem 



```

## Attributes
classi speciali che si "appiccicano" (decorano) a compile time su classi, struct, properties, eventi, etc

```c#

[ApiController(route: "/")]
class HomeController : IApiHandler
{
    public void Handle()
    {
        Console.WriteLine("Home");
    }
}
```

Utile con un po' di reflection.
```c#

var route = "/";
var handler = GetHandlerFromRoute(route);
handler.Handle();

IApiHandler? GetHandlerFromRoute(string route)
{
    var possibleHandlers = from type in typeof(AssemblyPlaceholder).Assembly.DefinedTypes
        where type.IsAssignableFrom(typeof(IApiHandler))
        from attribute in type.GetCustomAttributes(true)
        where attribute.GetType() == typeof(ApiControllerAttribute)
        let attributeInstance = (ApiControllerAttribute?) Activator.CreateInstance(attribute.GetType())
        where attributeInstance.Route == route
        let apiHandler = Activator.CreateInstance(type)
        select apiHandler;
    return possibleHandlers.FirstOrDefault() as IApiHandler;
}



public class AssemblyPlaceholder
{

}

class ApiControllerAttribute : Attribute
{
    public string Route { get; }

    public ApiControllerAttribute(string route = "")
    {
        Route = route;
    }
}

public interface IApiHandler
{
    void Handle();
}

[ApiController(route: "/")]
class HomeController : IApiHandler
{
    public void Handle()
    {
        Console.WriteLine("Home");
    }
}

```

## Async-await
Permette di scrivere con una sintassi che sembra sincrona, flussi di operazioni che sono asincrone, riorganizzando a compile time il sorgente prima di compilarlo "veramente" (I thread non c'entrano nulla), è puro compilatore.
