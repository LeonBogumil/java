# The evolution of Java

![](img/gosling.jpg)

## James Gosling: The Feel of Java (June 1997)

- Blue Collar Language
  - Distributed objects on the web
  - Thin clients
  - Architecture neutral
- Java Virtual Machine
  - Compile-time checking
  - Garbage collection
  - Pointer restrictions
  - Exception handling
- Object-oriented extensibility
  - Dynamic linking
- Performance

> Java evolved out of a Sun research project started six years ago to look into distributed control of **consumer electronics devices**.
> It was not an academic research project studying programming languages: Doing language research was actively an antigoal.

> In the consumer electronics business, there are **dozens of different CPU types** and good reasons for all of them in their individual contexts.
> But developing software for a dozen different platforms just doesn't scale, and it was this desire for architecture neutrality that broke the C++ mold — not so much C++ the language, but the standard way people built C++ compilers. 

> We use a very old technique where the compiler generates some **bytecoded instructions for this abstract virtual machine** that's based largely on work from Smalltalk and Pascal-P machines.
> I put a lot of effort into making it very easy to interpret and verify byte-code before it was compiled into machine code, using both an interpreter and a machine code generator to make sure that generating machine code was pretty straightforward.

## Bytecode

```java
public class Pythagoras {
    public static double distance(double x1, double y1,
                                  double x2, double y2) {
        double dx = x1 - x2;
        double dy = y1 - y2;
        return Math.sqrt(dx * dx + dy * dy);
    }
}
```

- `javac` turns source code into bytecode
- `javap` prints bytecode in human-readable format

```
$ javac Pythagoras.java

$ javap -c Pythagoras
Compiled from "Pythagoras.java"
public class Pythagoras {
  public Pythagoras();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static double distance(double, double, double, double);
    Code:
       0: dload_0
       1: dload         4
       3: dsub
       4: dstore        8
       6: dload_2
       7: dload         6
       9: dsub
      10: dstore        10
      12: dload         8
      14: dload         8
      16: dmul
      17: dload         10
      19: dload         10
      21: dmul
      22: dadd
      23: invokestatic  #7                  // Method java/lang/Math.sqrt:(D)D
      26: dreturn
}
```

## Case study: GUI code

### Java 1.0 (1996)

```java
public class GUI {
    public static void main(String[] args) {
        Frame frame = new Frame("Close me!");
        frame.addWindowListener(new FrameCloser(frame));

        Button button = new Button("Click me to see the current date!");
        button.addActionListener(new ButtonUpdater(button));
        frame.add(button);

        frame.pack();
        frame.show();
    }
}

class FrameCloser extends WindowAdapter {
    private final Frame frame;

    FrameCloser(Frame frame) {
        this.frame = frame;
    }

    public void windowClosing(WindowEvent event) {
        frame.dispose();
    }
}

class ButtonUpdater implements ActionListener {
    private final Button button;

    ButtonUpdater(Button button) {
        this.button = button;
    }

    public void actionPerformed(ActionEvent event) {
        button.setLabel(new Date().toString());
    }
}
```

- `GUI.java`
  - `GUI.class`
- `FrameCloser.java`
  - `FrameCloser.class`
- `ButtonUpdater.java`
  - `ButtonUpdater.class`

### Java 1.1 (1997)

> Closures were left out of Java initially more because of time pressures than anything else. Closures, as a concept, are tried and true – well past the days of being PhD topics. The arguments are in the details, not the broad concepts.
>
> In the early days of Java the **lack of closures** was pretty painful, and so **inner classes** were born: an uncomfortable compromise that attempted to avoid a number of hard issues. But as is normal in so many design issues, the simplifications didn't really solve any problems, they just moved them. We should have gone all the way back then. [James Gosling, 2008]

```java
public class GUI {
    public static void main(String[] args) {
        final Frame frame = new Frame("Close me!");
        frame.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent event) {
                frame.dispose();
            }
        });

        final Button button = new Button("Click me to see the current date!");
        button.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent event) {
                button.setLabel(new Date().toString());
            }
        });
        frame.add(button);

        frame.pack();
        frame.show();
    }
}
```

- `GUI.java`
  - `GUI.class`
  - `GUI$1.class`
  - `GUI$2.class`

## Java 1.2 (1998)

```java
public class GUI {
    public static void main(String[] args) {
        JFrame frame = new JFrame("Close me!");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        final JButton button = new JButton("Click me to see the current date!");
        button.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent event) {
                button.setText(new Date().toString());
            }
        });
        frame.add(button);

        frame.pack();
        frame.show();
    }
}
```

- `GUI.java`
  - `GUI.class`
  - `GUI$1.class`

## Java 8 (2014)

Lambdas are more concise than anonymous inner classes:

```java
button.addActionListener((ActionEvent event) -> {
    button.setText(new Date().toString());
});
```

Lambda parameter types can be inferred by the compiler:

```java
button.addActionListener((event) -> {
    button.setText(new Date().toString());
});
```

Parentheses around a single lambda parameter name are optional:

```java
button.addActionListener(event -> {
    button.setText(new Date().toString());
});
```

The curly braces around a single statement are optional:

```java
button.addActionListener(event -> button.setText(new Date().toString()));
```

- `GUI.java`
  - `GUI.class`
- [Lambdas in Java: A Peek under the Hood](https://www.youtube.com/watch?v=MLksirK9nnE)

## Case study: Collections

### Java 1.4

```java
public static List adultDomains(List persons) {
    Set domains = new HashSet();
    for (Iterator it = persons.iterator(); it.hasNext(); ) {
        Person person = (Person) it.next();
        if (person.isAdult()) {
            domains.add(person.getEmail().getDomain());
        }
    }
    List list = new ArrayList(domains);
    Collections.sort(list);
    return list;
}
```

### Java 5

```java
public static List<String> adultDomains(List<Person> persons) {
    Set<String> domains = new HashSet<String>();
    for (Person person : persons) {
        if (person.isAdult()) {
            domains.add(person.getEmail().getDomain());
        }
    }
    List<String> list = new ArrayList<String>(domains);
    Collections.sort(list);
    return list;
}
```

### Java 7

```java
public static List<String> adultDomains(List<Person> persons) {
    Set<String> domains = new HashSet<>();
    for (Person person : persons) {
        if (person.isAdult()) {
            domains.add(person.getEmail().getDomain());
        }
    }
    List<String> list = new ArrayList<>(domains);
    Collections.sort(list);
    return list;
}
```

### Java 8

```java
public static List<String> adultDomains(List<Person> persons) {
            // source:
    return persons.stream()
            // intermediate operations:
            .filter(person -> person.isAdult())
            .map(person -> person.getEmail().getDomain())
            .sorted()
            .distinct()
            // terminal operation:
            .collect(Collectors.toList());
}
```

Compressed overview of `Stream<T>` JavaDoc:

> - A sequence of elements supporting sequential and parallel aggregate operations.
> - In addition to Stream, which is a stream of object references, there are **primitive specializations** for `IntStream`, `LongStream`, and `DoubleStream`.
> - To perform a computation, stream operations are composed into a **stream pipeline**. A stream pipeline consists of
>   - a **source**,
>   - zero or more **intermediate operations** (which transform a stream into another stream),
>   - and a **terminal operation** (which produces a result or side-effect).
> - Streams are **lazy**; computation on the source data is only performed when the terminal operation is initiated, and source elements are consumed only as needed.
> - A stream should be operated on **only once**. This rules out multiple traversals of the same stream.
> - Streams have a `close()` method and implement `AutoCloseable`. Generally, only streams whose source is an IO channel will require closing.

```java
public interface Stream<T> {

    // source

    static <T> Stream<T> empty();
    static <T> Stream<T> of(T t);
    static <T> Stream<T> ofNullable(T t);
    static <T> Stream<T> of(T... values);

    static <T> Stream<T> generate(Supplier<? extends T> s);

    static <T> Stream<T> iterate(T seed, UnaryOperator<T> next);
    static <T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next);

    static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b);

    // intermediate operations

    Stream<T> filter(Predicate<? super T> predicate);

    <R> Stream<R> map(Function<? super T, ? extends R> mapper);

    <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);

    Stream<T> limit(long maxSize);
    Stream<T> skip (long n);

    Stream<T> takeWhile(Predicate<? super T> predicate);
    Stream<T> dropWhile(Predicate<? super T> predicate);

    Stream<T> peek(Consumer<? super T> action);

    Stream<T> distinct();

    Stream<T> sorted();
    Stream<T> sorted(Comparator<? super T> comparator);

    // terminal operations

    void forEach       (Consumer<? super T> action);
    void forEachOrdered(Consumer<? super T> action);

    long count();

    boolean  anyMatch(Predicate<? super T> predicate);
    boolean  allMatch(Predicate<? super T> predicate);
    boolean noneMatch(Predicate<? super T> predicate);

    Optional<T> findFirst();
    Optional<T> findAny();

    Optional<T> min(Comparator<? super T> comparator);
    Optional<T> max(Comparator<? super T> comparator);

    Object[] toArray();
    <A>  A[] toArray(IntFunction<A[]> generator);

    <R, A> R collect(Collector<? super T, A, R> collector);
    <R>    R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner);

    T           reduce(T identity, BinaryOperator<T> accumulator);
    Optional<T> reduce(BinaryOperator<T> accumulator);
    <U> U       reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);
}
```

The package `java.util.function` contains 43 functional interfaces:

- Suppliers
  - `Supplier<T>`
  - `BooleanSupplier`
  - `IntSupplier`
  - `LongSupplier`
  - `DoubleSupplier`
- Consumers
  - `Consumer<T>`
  - `IntConsumer`
  - `LongConsumer`
  - `DoubleConsumer`
- BiConsumers
  - `BiConsumer<T, U>`
  - `ObjIntConsumer<T>`
  - `ObjLongConsumer<T>`
  - `ObjDoubleConsumer<T>`
- UnaryOperators
  - `UnaryOperator<T>`
  - `IntUnaryOperator`
  - `LongUnaryOperator`
  - `DoubleUnaryOperator`
- BinaryOperators
  - `BinaryOperator<T>`
  - `IntBinaryOperator`
  - `LongBinaryOperator`
  - `DoubleBinaryOperator`
- Functions
  - `Function<T, R>`
  - `ToIntFunction<T>`
  - `ToLongFunction<T>`
  - `ToDoubleFunction<T>`
  - `IntFunction<R>`
  - `IntToLongFunction`
  - `IntToDoubleFunction`
  - `LongFunction<R>`
  - `LongToIntFunction`
  - `LongToDoubleFunction`
  - `DoubleFunction<R>`
  - `DoubleToIntFunction`
  - `DoubleToLongFunction`
- BiFunctions
  - `BiFunction<T, U, R>`
  - `ToIntBiFunction<T, U>`
  - `ToLongBiFunction<T, U>`
  - `ToDoubleBiFunction<T, U>`
- Predicates
  - `Predicate<T>`
  - `IntPredicate`
  - `LongPredicate`
  - `DoublePredicate`
  - `BiPredicate<T, U>`

Stream sources are not limited to collections:

```java
public static long countVowels1(String s) {
    return s.chars()
            .filter(ch -> "AEIOUaeiou".indexOf(ch) != -1)
            .count();
}


public static long countVowels2(String s) {
    return VOWELS.matcher(s).results()
            .count();
}

private static Pattern VOWELS = Pattern.compile("[aeiou]", Pattern.CASE_INSENSITIVE);
```

Simple lambdas can often be replaced with method references:

```java
public static List<String> adultDomains(List<Person> persons) {
    return persons.stream()
            .filter(Person::isAdult)
            .map(Person::getEmail)
            .map(Email::getDomain)
            .sorted()
            .distinct()
            .toList(); // since Java 16
}


public static Map<String, List<Email>> emailsByDomain(List<Person> persons) {
    return persons.stream()
            .map(Person::getEmail)
            .collect(Collectors.groupingBy(Email::getDomain));
}
```

> | Method Ref Type   | Example                  | Lambda Equivalent               |
> | ----------------- | ------------------------ | ------------------------------- |
> | Static            | `Integer::parseInt`      | `str -> Integer.parseInt(str)`  |
> | Bound             | `Instant.now()::isAfter` | `Instant then = Instant.now();` |
> |                   |                          | `t -> then.isAfter(t)`          |
> | Unbound           | `String::toLowerCase`    | `str -> str.toLowerCase()`      |
> | Class Constructor | `TreeMap<K, V>::new`     | `() -> new TreeMap<K, V>()`     |
> | Array Constructor | `int[]::new`             | `len -> new int[len]`           |
> [EJ3 44]

**In vivo:** [pangit](https://github.com/fredoverflow/pangit/blob/master/src/main/java/GitBlob.java#L32)
```java
public static Stream<GitBlob> findGitBlobs(Path root, Consumer<GitBlob> gitBlobConsumer) throws IOException {
    return Files.walk(root)
            .filter(GitBlob::isGitObject)
            .map(GitBlob::gitBlobOrNull)
            .filter(Objects::nonNull)
            .peek(gitBlobConsumer)
            .sorted();
}
```

> **Exercise:** Write a method `public static int diceRollSum(int numDice)` which rolls `numDice` dice once and computes the sum (between 2 and 12).

> **Exercise:** Write a method `public static Map<Integer, Long> twinDiceRollSumDistribution(int numRolls)` which rolls 2 dice `numRolls` times and returns the distribution, e.g. `{2=3, 3=11, 4=5, 5=12, 6=14, 7=14, 8=12, 9=14, 10=5, 11=6, 12=4}`

## Case study: Nullable references

![](img/hoare.jpg)

> **Tony Hoare:** I call it my billion-dollar mistake.
> It was the invention of the null reference in 1965.
> At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W).
> My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler.
> But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement.
> This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

### Java 7

Language lawyer: "Every reference type is a supertype of the null type."

Java programmer: "Every reference can be null."

```java
public Person findFirstPersonWithAge(int age);
```

What if there is no person with the given age? Should the method...
- ...return null?
- ...return a "default Person"?
- ...throw an exception?

One approach is to return null and let the caller deal with the situation:

```java
public Person findFirstPersonWithAge(int age) {
    for (Person person : persons) {
        if (person.getAge() == age) {
            return person;
        }
    }
    return null;
}
```

But what if the caller is not aware of the potential null reference?

```java
String name = party.findFirstPersonWithAge(42).getName();
                                           // ^ ticking time bomb
```

If the caller is aware of the potential null reference, they can deal with it as they see fit:

```java
Person person = party.findFirstPersonWithAge(42);
String name;
if (person != null) {
    name = person.getName();
} else {
    // null name:
    name = null;

    // default name:
    name = "";

    // exception:
    throw new Exception("...");
}
```

### Java 8

`Optional<Person>` makes it clear that `findPersonWithAge` might return no Person:

```java
public Optional<Person> findFirstPersonWithAge(int age) {
    return persons.stream().filter(person -> person.getAge() == age).findFirst();
}
```

Now the caller is forced to deal with the situation:

```java
String name = party.findFirstPersonWithAge(42)
                   .map(Person::getName)

                   // null name:
                   .orElse(null);

                   // default name:
                   .orElse("");

                   // exception:
                   .orElseThrow(() -> new Exception("..."));
```

Compressed overview of `Optional<T>` JavaDoc:

> - A container object which may or may not contain a non-null value.
> - This is a **value-based class**; use of identity-sensitive operations (including reference equality (==), identity hash code, or synchronization on instances of Optional may have unpredictable results and should be avoided.
> - Optional is primarily intended for use as a **method return type** where there is a clear need to represent "no result," and where using null is likely to cause errors.
> - A variable whose type is Optional should **never itself be null**; it should always point to an Optional instance.

```java
public final class Optional<T> {

    private final T value;

    // Factory methods

    public static <T> Optional<T> empty();

    public static <T> Optional<T> of(T value);

    public static <T> Optional<T> ofNullable(T value);

    // Intermediate methods

    public Optional<T> filter(Predicate<? super T> predicate);

    public <U> Optional<U> map(Function<? super T, ? extends U> mapper);

    public <U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper);

    public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier); // @since 9

    // Conditional methods

    public void ifPresent(Consumer<? super T> action);

    public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction); // @since 9

    // Extractor methods

    public T orElse(T other);

    public T orElseGet(Supplier<? extends T> supplier);

    public T orElseThrow(); // @since 10, throws NoSuchElementException

    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X;

    // Interoperability

    public Stream<T> stream(); // @since 9

    public boolean isEmpty(); // @since 11

    public boolean isPresent();

    public T get(); // throws NoSuchElementException

    // equals, hashCode, toString
}
```

## Case study: Comparator

### Java 7

```java
private static final Comparator<Person> compareAgeDescendingNameEmail = new Comparator<>() {
    @Override
    public int compare(Person a, Person b) {
        int result = Integer.compare(b.getAge(), a.getAge());
        if (result == 0) {
            result = a.getName().compareTo(b.getName());
            if (result == 0) {
                result = a.getEmail().compareTo(b.getEmail());
            }
        }
        return result;
    }
};

public static void sortAgeDescendingNameEmail(List<Person> persons) {
    Collections.sort(persons, compareAgeDescendingNameEmail);
}
```

### Java 8

```java
private static final Comparator<Person> compareAgeDescendingNameEmail = Comparator
        .comparingInt(Person::getAge).reversed()
        .thenComparing(Person::getName)
        .thenComparing(Person::getEmail);

public static void sortAgeDescendingNameEmail(List<Person> persons) {
    persons.sort(compareAgeDescendingNameEmail);
}
```

> **Exercise:** Search for `new Comparator` and/or `implements Comparator` in your project.
> Identify candidates for replacement with `Comparator.comparing`!

## java.time

- Designed around ISO 8601 calendar system
- Heavily inspired by Joda Time
- Replacement for
  - `java.util.Date`
  - `java.util.Calendar`
  - `java.util.TimeZone`
  - `java.util.DateFormat`
- Plethora of immutable types for managing everything time-related:

| Type            | Zone         | Year | Month | Day | Hour | Minute | Second | Nanosecond |
| --------------- | ------------ | ---- | ----- | --- | ---- | ------ | ------ | ---------- |
| `Instant`       | UTC          |      |       |     |      |        | ✓      | ✓          |
| `LocalDate`     |              | ✓    | ✓     | ✓   |      |        |        |            |
| `LocalTime`     |              |      |       |     | ✓    | ✓      | ✓      | ✓          |
| `LocalDateTime` |              | ✓    | ✓     | ✓   | ✓    | ✓      | ✓      | ✓          |
| `OffsetTime`    | `ZoneOffset` |      |       |     | ✓    | ✓      | ✓      | ✓          |
| `OffsetDateTime`| `ZoneOffset` | ✓    | ✓     | ✓   | ✓    | ✓      | ✓      | ✓          |
| `ZonedDateTime` | `ZoneId`     | ✓    | ✓     | ✓   | ✓    | ✓      | ✓      | ✓          |
| `Year`          |              | ✓    |       |     |      |        |        |            |
| `YearMonth`     |              | ✓    | ✓     |     |      |        |        |            |
| `Month`         |              |      | ✓     |     |      |        |        |            |
| `MonthDay`      |              |      | ✓     | ✓   |      |        |        |            |
| `DayOfWeek`     |              |      |       | (✓) |      |        |        |            |
| `Duration`      |              |      |       |     | (✓)  | (✓)    | (✓)    | (✓)        |
| `Period`        |              | (✓)  | (✓)   | (✓) |      |        |        |            |

- (`DayOfWeek` has 7 values instead of 31)
- (`Duration` and `Period` store differences)

### How many days between two dates?

```java
LocalDate java7Release = LocalDate.of(2011, Month.JULY, 28);
LocalDate java8Release = LocalDate.of(2014, 3, 18);

Period pause = java7Release.until(java8Release);
System.out.println(pause); // P2Y7M18D
System.out.printf("There were %d years, %d months and %d days between Java 7 and Java 8.%n",
        pause.getYears(), pause.getMonths(), pause.getDays());

long days = ChronoUnit.DAYS.between(java7Release, java8Release);
System.out.printf("There were %d days between Java 7 and Java 8.%n", days);
```

### What about leap years?

```java
for (int y = LocalDate.now().getYear(), bound = y + 12; y < bound; ++y) {
    Year year = Year.of(y);
    System.out.printf("%d is %s leap year.%n", y, year.isLeap() ? "a" : "not a");

    YearMonth february = year.atMonth(Month.FEBRUARY);
    System.out.printf("February %d has %d days, %d has %d days.%n",
        y, february.lengthOfMonth(), y, february.lengthOfYear());

    System.out.printf("February %d ends at %s.%n%n", y, february.atEndOfMonth());
}
```

### What about daylight saving time?

```java
ZoneId berlin = ZoneId.of("Europe/Berlin");
ZonedDateTime saturday = ZonedDateTime.of(LocalDateTime.of(2018, Month.OCTOBER, 27, 9, 0), berlin);
ZonedDateTime sundayAtEight = saturday.plusHours(24);
ZonedDateTime sundayAtNine = saturday.plusDays(1);
```

> **Exercise:** Write a program that determines your next 5 birthdays that fall on a weekend.

## Enumerations

### Java 1.4

```java
public final class Month {
    private final int ordinal;
    private final String name;

    private final double days;

    public Month(int ordinal, String name, double days) {
        this.ordinal = ordinal;
        this.name = name;

        this.days = days;
    }

    public int ordinal() {
        return ordinal;
    }

    public String name() {
        return name;
    }

    @Override
    public String toString() {
        return name;
    }

    public double days() {
        return days;
    }

    public static final Month JAN = new Month(0, "JAN", 31);
    public static final Month FEB = new Month(1, "FEB", 28.2425);
    public static final Month MAR = new Month(2, "MAR", 31);
    public static final Month APR = new Month(3, "APR", 30);
    public static final Month MAY = new Month(4, "MAY", 31);
    public static final Month JUN = new Month(5, "JUN", 30);
    public static final Month JUL = new Month(6, "JUL", 31);
    public static final Month AUG = new Month(7, "AUG", 31);
    public static final Month SEP = new Month(8, "SEP", 30);
    public static final Month OCT = new Month(9, "OCT", 31);
    public static final Month NOV = new Month(10, "NOV", 30);
    public static final Month DEC = new Month(11, "DEC", 31);

    private static final Month[] values;
    private static final Map<String, Month> monthsByName;

    static {
        values = new Month[]{JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV, DEC};
        monthsByName = new HashMap<>(16);
        for (Month month : values) {
            monthsByName.put(month.name, month);
        }
    }

    public static Month[] values() {
        return values.clone();
    }

    public static Month valueOf(String name) {
        Month month = monthsByName.get(name);
        if (month == null) throw new IllegalArgumentException(name);
        return month;
    }
}
```

### Java 5

```java
public enum Month {
    JAN(31), FEB(28.2425), MAR(31), APR(30), MAY(31), JUN(30), JUL(31), AUG(31), SEP(30), OCT(31), NOV(30), DEC(31);

    private final double days;

    Month(double days) {
        this.days = days;
    }

    public double days() {
        return days;
    }
}
```

- `Month` implicitly inherits from `java.lang.Enum<Month>`
  - `public abstract class Enum<E extends Enum<E>>`
  - https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern
- Optimized collections using `ordinal()`:
  - `public abstract class EnumSet<E extends Enum<E>>`
    - `class RegularEnumSet`
      - `private long elements;`
    - `class JumboEnumSet`
      - `private long elements[];`
  - `public class EnumMap<K extends Enum<K>, V>`
  
**In vivo:** [pangit](https://github.com/fredoverflow/pangit/blob/master/src/main/java/CharacterEncoding.java)
```java
public enum CharacterEncoding {
    UTF16BigEndian {
        @Override
        public String decode(byte[] bytes, int start) {
            start += 2;
            return new String(bytes, start, bytes.length - start, StandardCharsets.UTF_16BE);
        }
    }, UTF16LittleEndian {
        @Override
        public String decode(byte[] bytes, int start) {
            start += 2;
            return new String(bytes, start, bytes.length - start, StandardCharsets.UTF_16LE);
        }
    }, UTF8 {
        @Override
        public String decode(byte[] bytes, int start) {
            // https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
            if (start + 2 < bytes.length &&
                    bytes[start]     == (byte) 0xEF &&
                    bytes[start + 1] == (byte) 0xBB &&
                    bytes[start + 2] == (byte) 0xBF) {
                start += 3;
            }
            return new String(bytes, start, bytes.length - start, StandardCharsets.UTF_8);
        }
    }, LATIN1 {
        @Override
        public String decode(byte[] bytes, int start) {
            return new String(bytes, start, bytes.length - start, StandardCharsets.ISO_8859_1);
        }
    }, BINARY {
        @Override
        public String decode(byte[] bytes, int start) {
            final int limit = HEXDUMP_LIMIT + start;
            if (bytes.length > limit) {
                bytes = Arrays.copyOf(bytes, limit);
            }
            return Hexdump.hexdump(bytes, start);
        }
    };

    public static final int HEXDUMP_LIMIT = 1024;

    public abstract String decode(byte[] bytes, int start);

    // ...
}
```

- Enumeration constants can subclass their containing class!

## Resource management

### Java 6

What is wrong with the following database query?

```java
Connection connection = dataSource.getConnection();
PreparedStatement statement = connection.prepareStatement("select * from user where id = ?");
statement.setLong(1, id);
ResultSet resultSet = statement.executeQuery();
if (resultSet.next()) {
    return resultSet.getString("name");
} else {
    return null;
}
```

Solution #1

```java
Connection connection = null;
PreparedStatement statement = null;
ResultSet resultSet = null;
try {
    connection = dataSource.getConnection();
    statement = connection.prepareStatement("select * from user where id = ?");
    statement.setLong(1, id);
    resultSet = statement.executeQuery();
    if (resultSet.next()) {
        return resultSet.getString("name");
    } else {
        return null;
    }
} finally {
    if (resultSet != null) {
        try {
            resultSet.close();
        } catch (SQLException ignored) {
        }
    }
    if (statement != null) {
        try {
            statement.close();
        } catch (SQLException ignored) {
        }
    }
    if (connection != null) {
        try {
            connection.close();
        } catch (SQLException ignored) {
        }
    }
}
```

Solution #2

```java
Connection connection = dataSource.getConnection();
try {
    PreparedStatement statement = connection.prepareStatement("select * from user where id = ?");
    try {
        statement.setLong(1, id);
        ResultSet resultSet = statement.executeQuery();
        try {
            if (resultSet.next()) {
                return resultSet.getString("name");
            } else {
                return null;
            }
        } finally {
            resultSet.close();
        }
    } finally {
        statement.close();
    }
} finally {
    connection.close();
}
```

### Java 7

try-with-resources

```java
try (Connection connection = dataSource.getConnection();
     PreparedStatement statement = connection.prepareStatement("select * from user where id = ?")) {
    statement.setLong(1, id);
    try (ResultSet resultSet = statement.executeQuery()) {
        if (resultSet.next()) {
            return resultSet.getString("name");
        } else {
            return null;
        }
    }
}
```

**In vivo:** [pangit](https://github.com/fredoverflow/pangit/blob/master/src/main/java/BackgroundScanner.java#L28)
```java
protected List<GitBlob> doInBackground() throws Exception {
    try (Stream<GitBlob> gitBlobs = GitBlob.findGitBlobs(selectedDirectory.toPath(), this::publish)) {
        return gitBlobs.collect(Collectors.toList());
    }
}
```

**In vivo:** [freditor](https://github.com/fredoverflow/freditor/blob/master/src/main/java/freditor/Freditor.java#L865)
```java
public void saveToFile(String pathname) throws IOException {
    try (FileOutputStream out = new FileOutputStream(pathname)) {
        if ("\n".equals(System.lineSeparator())) {
            out.write(toByteArray());
        } else {
            saveWithNativeLineSeparators(out);
        }
    }
}
```

## LoggerFactory.getLogger

### Java 6

```java
public class Foo {
    private static final Logger log = LoggerFactory.getLogger(Foo.class);

    // ...
}

public class Bar {
    private static final Logger log = LoggerFactory.getLogger(Bar.class);

    // ...
}

public class Baz {
    private static final Logger log = LoggerFactory.getLogger(Bar.class);

    // ...
}
```

- Do you see the problem?

### Java 7

```java
public class Foo {
    private static final Logger log = LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());

    // ...
}

public class Bar {
    private static final Logger log = LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());

    // ...
}

public class Baz {
    private static final Logger log = LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());

    // ...
}
```

- No *actual* method handles in the above example

> A method handle is a typed, directly executable reference to an underlying method,
> constructor, field, or similar low-level operation,
> with optional transformations of arguments or return values.
>
> Method handles are dynamically and strongly typed according to their parameter and return types.

- Mostly interesting for dynamic JVM language implementers
  - [MethodHandles Everywhere!](https://www.youtube.com/watch?v=NGnuIAd0VHY)
  - https://github.com/fredoverflow/karel/tree/second

> **Exercise:** Search for `LoggerFactory.getLogger` in your project:
> - How many `Wrong.class` bugs exist?
> - How many declarations are missing `private`, `static` or `final`?
> - How many variable spellings (`LOG`, `log`, `LOGGER`, `logger`...) are used?

## Annotations

```java
public class CompactDate {
    private final int yearMonthDay;

    public CompactDate(int year, Month month, int day) {
        // TODO sanitize inputs

        this.yearMonthDay = year << 9 | month.ordinal() << 5 | day & 31;
    }

    public int year() {
        return yearMonthDay >> 9;
    }

    public Month month() {
        return Month.values()[(yearMonthDay >> 5) & 15];
    }

    public int day() {
        return yearMonthDay & 31;
    }

    public boolean equals(CompactDate that) {
        return this.yearMonthDay == that.yearMonthDay;
    }

    public int hashcode() {
        return yearMonthDay;
    }

    public String ToString() {
        return String.format("%02d. %s %d", day(), month(), year());
    }
}
```

- There are 3 bugs; how many can you spot?

```java
public class CompactDate {
    private final int yearMonthDay;

    public CompactDate(int year, Month month, int day) {
        // TODO sanitize inputs

        this.yearMonthDay = year << 9 | month.ordinal() << 5 | day & 31;
    }

    public int year() {
        return yearMonthDay >> 9;
    }

    public Month month() {
        return Month.values()[(yearMonthDay >> 5) & 15];
    }

    public int day() {
        return yearMonthDay & 31;
    }

    @Override
    // error: method does not override or implement a method from a supertype
    public boolean equals(CompactDate that) {
        return this.yearMonthDay == that.yearMonthDay;
    }

    @Override
    // error: method does not override or implement a method from a supertype
    public int hashcode() {
        return yearMonthDay;
    }

    @Override
    // error: method does not override or implement a method from a supertype
    public String ToString() {
        return String.format("%02d. %s %d", day(), month(), year());
    }
}
```

- `@Override` is *not* required for method overriding
- Rather prevents compilation if *no* overriding happens:

```java
/**
 * Indicates that a method declaration is intended to override a
 * method declaration in a supertype. If a method is annotated with
 * this annotation type compilers are required to generate an error
 * message unless at least one of the following conditions hold:
 *
 * The method does override or implement a method declared in a supertype.
 *
 * The method has a signature that is override-equivalent to that of
 * any public method declared in Object.
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

- Similary, `@FunctionalInterface` is *not* required for functional interfaces
- Rather prevents compilation for *non*-functional interfaces:

```java
/**
 * An informative annotation type used to indicate that an interface
 * type declaration is intended to be a functional interface as
 * defined by the Java Language Specification.
 *
 * Conceptually, a functional interface has exactly one abstract method.
 *
 * If a type is annotated with this annotation type, compilers are
 * required to generate an error message unless
 * the annotated type satisfies the requirements of a functional interface.
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface FunctionalInterface {
}
```

- You may have noted the empty bodies `{}`
- Annotations are merely metadata on declarations
- Without interpretation, annotations are meaningless markers:

```java
import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TreeFalls {
}

@TreeFalls
class Wood {
}
```

- Spring defines and interprets lots of runtime annotations

**In vivo:** [spritze](https://github.com/frectures/spritze)
```java
Class<?> clazz = Class.forName(qualifiedClassName, true, classLoader);

for (Annotation annotation : clazz.getAnnotations()) {
    if (annotation.annotationType() == Komponente.class) {
        System.out.println(clazz + " component detected");
        Constructor<?> constructor = solePublicConstructor(clazz);
        System.out.println(clazz + " public constructor found");

        Class<?>[] parameterTypes = constructor.getParameterTypes();
        Object[] arguments = Arrays.stream(parameterTypes)
                .map(kontext::get)
                .toArray();
        Object instance = constructor.newInstance(arguments);
        kontext.put(clazz, instance);
        System.out.println(clazz + " instantiated");
    }
}
```

# Java 9

## jshell

Java finally has a REPL!
```
|  Welcome to JShell -- Version 17
|  For an introduction type: /help intro

jshell> /help intro
|
|                                   intro
|                                   =====
|
|  The jshell tool allows you to execute Java code, getting immediate results.
|  You can enter a Java definition (variable, method, class, etc), like:  int x = 8
|  or a Java expression, like:  x + x
|  or a Java statement or import.
|  These little chunks of Java code are called 'snippets'.
|
|  There are also the jshell tool commands that allow you to understand and
|  control what you are doing, like:  /list
|
|  For a list of commands: /help

jshell> 6 * 7
$1 ==> 42

jshell> Integer.toBinaryString($1)
$2 ==> "101010"
```

## Collection 'literals'

### Lists before Java 9

ArrayList:
```java
List<Integer> primes = new ArrayList<>();
primes.add(2);
primes.add(3);
primes.add(5);
primes.add(7);

primes.set(0, 1); // ok
primes.add(11);   // ok
```

Array view:
```java
List<Integer> primes = Arrays.asList(2, 3, 5, 7);

primes.set(0, 1); // ok
primes.add(11);   // java.lang.UnsupportedOperationException
```

Read-only view:
```java
List<Integer> primes = Collections.unmodifiableList(Arrays.asList(2, 3, 5, 7));

primes.set(0, 1); // java.lang.UnsupportedOperationException
primes.add(11);   // java.lang.UnsupportedOperationException
```

### Immutable lists

```java
List<Integer> primes = List.of(2, 3, 5, 7);

primes.set(0, 1); // java.lang.UnsupportedOperationException
primes.add(11);   // java.lang.UnsupportedOperationException
```

- `List.of` is overloaded for up to 10 arguments
  - More arguments are handled via varargs
  - 0, 1 and 2 elements are implemented without a backing array
- `List.copyOf` creates an immutable list from a source collection
  - If the source collection is already an immutable list, no copy is performed

### Immutable sets

```java
Set<Integer> primes = Set.of(2, 3, 5, 7);
```

- All `List.of` and `List.copyOf` bullet points also apply to `Set.of` and `Set.copyOf`
- The set iteration order is unpredictable and varies from run to run:

```java
|  Welcome to JShell -- Version 17
|  For an introduction type: /help intro

jshell> Set.of('a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z')
$1 ==> [a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z]
```
```java
|  Welcome to JShell -- Version 17
|  For an introduction type: /help intro

jshell> Set.of('a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z')
$1 ==> [z, y, x, w, v, u, t, s, r, q, p, o, n, m, l, k, j, i, h, g, f, e, d, c, b, a]
```

### Immutable maps

- For up to 10 mappings, pass the keys and values directly to `Map.of`:

```java
Map<String, String> capitals = Map.of(
        "China", "Peking",
        "Japan", "Tokio",
        "Kongo", "Kinshasa",
        "Russland", "Moskau",
        "Korea", "Seoul",
        "Indonesion", "Jakarta",
        "Mexiko", "Mexiko-Stadt",
        "Thailand", "Bangkok",
        "Peru", "Lima",
        "Vereinigtes Königreich", "London");
```

- For 11 or more mappings, use `Map.ofEntries`: 

```java
import static java.util.Map.entry;

Map<String, String> capitals = Map.ofEntries(
        entry("China", "Peking"),
        entry("Japan", "Tokio"),
        entry("Kongo", "Kinshasa"),
        entry("Russland", "Moskau"),
        entry("Korea", "Seoul"),
        entry("Indonesion", "Jakarta"),
        entry("Mexiko", "Mexiko-Stadt"),
        entry("Thailand", "Bangkok"),
        entry("Peru", "Lima"),
        entry("Vereinigtes Königreich", "London"),
        entry("Iran", "Teheran"));
```

- All `Set.of` and `Set.copyOf` bullet points also apply to `Map.of` and `Map.copyOf`
- The backing array contains the keys and values directly, not references to entry objects
  - Immutable maps require significantly less memory than `java.util.HashMap`

> **Exercise:** Search for `new HashMap` in your test classes.
> Replace at least 1 HashMap with `Map.of` and make sure the test still works.

# Java 10

## Local variable type inference

```java
Connection connection = dataSource.getConnection();
PreparedStatement statement = connection.prepareStatement("select * from user where id = ?");
ResultSet resultSet = statement.executeQuery();

var connection = dataSource.getConnection();
var statement = connection.prepareStatement("select * from user where id = ?");
var resultSet = statement.executeQuery();
```

- Local variables can be declared with `var` instead of a manifest type
- In that case, the compiler figures out the type from the initializer
- Generate identical bytecode:
  - `String s = "hello";`
  - `var s = "hello";`

```java
var name = "Joshua";
name = 42; // Type mismatch: cannot convert from int to String

var names = new ArrayList<String>();
names = new LinkedList<String>(); // Type mismatch: cannot convert from LinkedList<String> to ArrayList<String>

var result; // cannot use `var` on variable without initializer
```

- Type inference is *not* dynamic typing!

> Nearly all other popular statically typed "curly-brace" languages, both on the JVM and off, already support some form of local-variable type inference: C++ (`auto`), C# (`var`), Scala (`var`/`val`), Go (declaration with `:=`). Java is nearly the only popular statically typed language that has not embraced local-variable type inference; at this point, this should no longer be a controversial feature.
> [JEP 286]

### Forms of type inference before Java 10

```java
// Java 5: generic method parameters
Arrays.asList("hello", "world")
Arrays.<String>asList("hello", "world")

// Java 7: diamond operator
List<String> names = new ArrayList<>();
List<String> names = new ArrayList<String>();

// Java 8: lambda parameters
names.filter(name -> name.isEmpty())
names.filter((String name) -> name.isEmpty())

names.filter((Predicate<String>)((String name) -> name.isEmpty()))
```

# Java 11

## `var` lambda parameters

`(dx, dy) -> dx*dx + dy*dy` can now be written as `(var dx, var dy) -> dx*dx + dy*dy`

## New release cadence

> Taking inspiration from the release models used by other platforms and by various operating-system distributions, I propose that after Java 9 we adopt a strict, time-based model with a **new feature release every six months**, update releases every quarter, and a **long-term support release every three years**.
> - Feature releases can contain any type of feature, including not just new and improved APIs but also language and JVM features. New features will be merged only when they're nearly finished, so that the release currently in development is feature-complete at all times. Feature releases will ship in March and September of each year, starting in March of 2018.
> - Update releases will be strictly limited to fixes of security issues, regressions, and bugs in newer features. Each feature release will receive two updates before the next feature release. Update releases will ship quarterly in January, April, July, and October, as they do today.
> - Every three years, starting in September of 2018, the feature release will be a long-term support release. Updates for these releases will be available for at least three years and quite possibly longer, depending upon your vendor.
> https://mreinhold.org/blog/forward-faster

## Distributions

Since Java 11, all JDK distributions are based 100% on the OpenJDK source code available at https://github.com/openjdk/jdk

- https://adoptium.net (recommended default)
- https://www.azul.com/downloads (optionally with JavaFX)
- https://bell-sw.com/pages/downloads (optionally with JavaFX)
- https://aws.amazon.com/de/corretto
- https://developers.redhat.com/products/openjdk/download (requires registration)
- https://www.oracle.com/java/technologies/downloads
- https://jdk.java.net (only current & early access)

# Java 14

## `switch` expressions

```java
double daysPerMonth = 0;
switch (month) {
    case JANUARY:
    case MARCH:
    case MAY:
    case JULY:
    case AUGUST:
    case OCTOBER:
    case DECEMBER:
        daysPerMonth = 31;
        break;

    case APRIL:
    case JUNE:
    case SEPTEMBER:
    case NOVEMBER:
        daysPerMonth = 30;
        break;

    case FEBRUARY:
        daysPerMonth = 28.2425;
}
```

```java
double daysPerMonth = switch (month) {
    case JANUARY, MARCH, MAY, JULY, AUGUST, OCTOBER, DECEMBER -> 31;

    case APRIL, JUNE, SEPTEMBER, NOVEMBER -> 30;

    case FEBRUARY -> 28.2425;
}
```

# Java 15

## Text Blocks

```
String js = "setTimeout(function () {\n" +
            "    console.log(\"Hello JavaScript!\");\n" +
            "}, 1000);\n";
```

```
String js = """
            setTimeout(function () {
                console.log("Hello JavaScript!");
            }, 1000);
            """;
```

# Java 16

## Pattern Matching for `instanceof`

```java
public static int count(Object obj) {
    if (obj instanceof Collection) {
        Collection coll = (Collection) obj;
        return coll.size();
    } else if (obj instanceof Map) {
        Map map = (Map) obj;
        return map.size();
    } else if (obj instanceof CharSequence) {
        CharSequence cs = (CharSequence) obj;
        return cs.length();
    } else {
        return 0;
    }
}
```

```java
public static int count(Object obj) {
    if (obj instanceof Collection coll) {
        return coll.size();
    } else if (obj instanceof Map map) {
        return map.size();
    } else if (obj instanceof CharSequence cs) {
        return cs.length();
    } else {
        return 0;
    }
}
```

## Records

```java
class Name {
    private final String surname;
    private final String forename;

    public Name(String surname, String forename) {
        if (surname.isBlank()) throw new IllegalArgumentException();
        if (forename.isBlank()) throw new IllegalArgumentException();
        this.surname = surname;
        this.forename = forename;
    }

    public String getSurname() {
        return surname;
    }

    public String getForename() {
        return forename;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Name that)) return false;
        return this.surname.equals(that.surname) && this.forename.equals(that.forename);
    }

    @Override
    public int hashCode() {
        return (1 * 31 + surname.hashCode()) * 31 + forename.hashCode();
    }

    @Override
    public String toString() {
        return "Name[" + "surname='" + surname + '\'' + ", forename='" + forename + '\'' + ']';
    }
}
```

```java
record Name(String surname, String forename) {
    Name {
        if (surname.isBlank()) throw new IllegalArgumentException();
        if (forename.isBlank()) throw new IllegalArgumentException();
    }
}
```
