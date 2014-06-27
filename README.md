# Scala's Try-Success-Failure for Java 8 #

This API is a Java 8 implementation of <a href="http://www.scala-lang.org/api/current/#scala.util.Try">Scala Try API</a>,
originally implemented by the guys at <a href="https://twitter.com/">Twitter</a> and later added to the Scala Standard Library.

The `Try` type represents a computation that may fail. If the computation is successful returns
the value wrapped in a `Try.Success` otherwise returns the
`java.lang.Exception` wrapped in a `Try.Failure`.

To use `Try` you need to call the `Try.apply(FailableSupplier)` method passing in a lambda with
the same signature used for a common `java.util.function.Supplier`.
Indeed `FailableSupplier` is just a `java.util.function.Supplier` with a
`throws Exception` added to its `get` method.

Note that I'm not saying here that the `try-catch` approach must be abandoned in favour of `Try-Success-Failure`.
Indeed there are cases where you would use the traditional `try-catch` pattern but, in general, I think this API provides
a more *fluent* interface to deal with exceptions.

## Build ##
This is a [Maven](http://maven.apache.org/) project. So it can be built with the usual way:

```
$ git clone https://github.com/typesafely/try.git
$ cd try
$ mvn package
```

You'll find the `jar` under the usual `target` directory.

## Running the examples in source code ##
Using [Maven](http://maven.apache.org/) and the [exec-maven-plugin](http://mojo.codehaus.org/exec-maven-plugin/) 
you can run the main classes representing the examples for this project. For example, to run the `ReadFileLines` main
class you can use:

```
$ mvn exec:java -Dexec.mainClass="org.typesafely.example.ReadFileLines"
```

The changing part is the full path to the main class you intend to run.

## Examples ##
In order to get you acquainted with this API each example will be provided using both the same old `try-catch` pattern
and the new `Try` API.

### Read a file line by line ###
As a first example consider the code you need to implement a method to read a file line by line in Java 8. 
#### Using the traditional `try-catch block ####
```java
public static List<String> readFile(String file) {

    List<String> lines;
    try {
        lines = Files.readAllLines(new File(file).toPath());
    } catch (IOException e) {
        lines = Arrays.asList("Could not read the file: " + file);
    }

    return lines;
}
```

`readFile` reads the content of a file, line by line, into a `List<String>`. In case of exception the method
returns a `List<String>` with just one line: *"Could not read..."*. 
#### Using the Try API ####
```java
public static List<String> readFile(String file) {

    return Try.apply(() -> Files.readAllLines(new File(file).toPath()))
            .getOrElse(Arrays.asList("Could not read the file: " + file));

}
```

Which version do you like more? It may be a matter of taste or just because I'm used to it but I prefer the latter--also
because otherwise I wouldn't have written this API! :-)

### Integer division ###
This is an interesting one because it shows another peculiarity of the `Try` API. You may already know that Java
has both checked and unchecked exception. For checked exceptions the compiler will refuse to compile your code
if you forget to handle them. However unchecked exceptions such as `NullPointerException`, `IllegalArgumentException`,
`RuntimeException` and so on are not notified by the compiler if you don't handle them. Consider the following code
snippet:
```java
System.out.println("Enter the dividend press Return and then enter the divisor: ");
Scanner dividend = new Scanner(System.in);
Scanner divisor = new Scanner(System.in);

int num = dividend.nextInt();
int denum = divisor.nextInt();
String res = "The result of division is: " + (num / denum);
System.out.println(res);
```

The previous code asks the user to enter two integers and then performs their division. The problem is that it could
throw two types of unchecked exceptions and the compiler of course didn't tell you so. You are required to know it 
yourself. The two unchecked exceptions I'm talking about are `java.util.InputMismatchException` and 
`java.lang.ArithmeticException` if the user enter a non-integer or zero as the second number, respectively. Now,
if you have a decent mathematical knowledge you know you can't divide by zero and yopu can also imagine that
`Scanner`'s `nextInt` method may throw some type of exeption if you enter a non-integer. However, in both cases
you have to look up the type of exception. Yes, you can use a general `catch(Exception e)` and catch them all if you're
not interested in the specific type or you could avoid using `try-catch` at all thanks to `Try`. Here are both 
implementations.
#### Using the traditional `try-catch block ####
```java
public static void divideWithoutTry() {

    System.out.println("Enter the dividend press Return and then enter the divisor: ");
    Scanner dividend = new Scanner(System.in);
    Scanner divisor = new Scanner(System.in);

    String res;
    try {
        res = "The result of division is: " + (dividend.nextInt() / divisor.nextInt());
    } catch(InputMismatchException|ArithmeticException e) {
        res = "The integers you entered are not valid or the divisor is zero.";
    }

    System.out.println(res);
}
```

#### Using the Try API ####
```java
public static void divideWithTry() {

    System.out.println("Enter the dividend press Return and then enter the divisor: ");
    Scanner dividend = new Scanner(System.in);
    Scanner divisor = new Scanner(System.in);

    String res = Try.apply(() -> dividend.nextInt() / divisor.nextInt())
            .map(division -> "The result of division is: " + division)
            .getOrElse("The integers you entered are not valid or the divisor is zero.");

    System.out.println(res);
}
```

Again it's a matter of taste/getting used to it.

### Integer sum ###
An important property of `Try` is its ability
to *pipeline* (*chain* if you prefer)  operations, catching exceptions along the way thanks to its `flatMap` method.
If you are not a seasoned functional programmer concepts such as `flatMap/map` might not be easy to grasp
at first. However you'll get used to them and, in the end, you'll love them. Moreover you're going to encounter
these methods more and more often since some important Java 8 classes already implement them
(e.g. `java.util.Optional` and `java.util.stream.Stream`. Anyway for the moment just take for
granted that to pipeline more than two operations, say N, you just need to chain them by using N - 1
`flatMap` calls and a last call to `map`. An example follows.
Suppose you have 3 variables (x, y and z) being
of type `Try<Integer>` and you just want to sum them up. Here is the code you need:

```java
x.flatMap(a -> y.flatMap(b -> z.map(c -> a + b + c)))
```

Apart from the methods seen in these examples, such as `map`, `flatMap` and `getOrElse`, `Try` has many other useful methods. See the `TryTest`
class for a thorough coverage of all methods.

## Javadoc ##
<a href="http://typesafely.github.io/try/apidocs/">API documentation</a> for this project.

## Bugs and Feedback ##

For bugs, questions and discussions please use the [Github Issues](https://github.com/typesafely/try/issues).
