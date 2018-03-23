# From Java To Clojure
Your Cheat Sheet For Clojurizing Java Syntax

## Printing out

#### Java

```java
System.out.print("Amit Shekhar");
System.out.println("Amit Shekhar");
```

#### Clojure

```clojure
(prn "Amit Shekhar")
(println "Amit Shekhar")
```

## Defining

#### Java

```java
String name = "Amit Shekhar";
final String name = "Amit Shekhar";
```

#### Clojure

```clojure
(def ^:private name "Amit Shekhar")
(def name "Amit Shekhar")
```

---

#### Java

```java
String otherName;
otherName = null;
```

#### Clojure

```clojure
(declare other-name)
(def other-name nil)
```
Normally one wouldn't use `declare` except when doing something like creating mutually referent functions, which make forward declarations necessary.

But if we're creating this var as `nil` because we expect it to change, we should think twice. More likely, we should either rethink the structure of our program to use an atom or be less imperative:

```clojure
(def other-name (atom nil))
```

...and if `other-name` needs to mutate, we do so with `reset!` or `swap!`.

## Conditionals

#### Java

```java
if (text != null) {
  int length = text.length();
}
```

#### Clojure
It's somewhat unusual to conditionally define a var in Clojure because it's quite imperative. To be Clojure-idiomatic, we'd just inline it and omit the name `length`.

So technically a straight port would be:
```clojure
(def length
  (if (nil? text)
    nil
    (count text)))
```
...but something like this, inline in some other statement, is more likely:
```clojure
(when text
  (count text))
```

---

#### Java

```java
String text = x > 5 ? "x > 5" : "x <= 5";
```

#### Clojure

```clojure
(def text
  (if (> x 5) 
    "x > 5" 
    "x <= 5"))
```

---

#### Java

```java
if (score >= 0 && score <= 300) { foo(); }
```

#### Clojure

```clojure
(if (and (>= score 0)
         (<= score 300))
  (foo))
```
---

#### Java

```java
int score = // some score;
String grade;
switch (score) {
	case 10:
	case 9:
		grade = "Excellent";
		break;
	case 8:
	case 7:
	case 6:
		grade = "Good";
		break;
	case 5:
	case 4:
		grade = "Ok";
		break;
	case 3:
	case 2:
	case 1:
		grade = "Fail";
		break;
	default:
	    grade = "Fail";				
}
```

#### Clojure

```clojure
(let [score _] ;; replace underscore with actual score
  (cond
    (#{9 10} score) "Excellent"
    (>= 6 8 score) "Good"
    (#{4 5} score) "Ok"
    (>= 1 3 score) "Fail"
    :default "Fail"))
```


## Strings

#### Java

```java
String firstName = "Amit";
String lastName = "Shekhar";
String message = "My name is: " + firstName + " " + lastName;
```

#### Clojure
Translating this one is a bit weird because the original has these variable names that come from nowhere. We'd probably make it a function rather than a series of defined names.
```clojure
(defn message [first-name last-name]
  (str "My name is: " first-name " " last-name))
  
(message "Amit" "Shekhar")
```

---

#### Java

```java
String text = "First Line\n" +
              "Second Line\n" +
              "Third Line";
```

#### Clojure

```clojure
(def text 
  (str "First Line\n" 
       "Second Line\n" 
       "Third Line"))
```

## Data Literals

#### Java

```java
final List<Integer> listOfNumber = Arrays.asList(1, 2, 3, 4);

final Map<Integer, String> keyValue = new HashMap<Integer, String>();
map.put(1, "Amit");
map.put(2, "Ali");
map.put(3, "Mindorks");

// Java 9
final List<Integer> listOfNumber = List.of(1, 2, 3, 4);

final Map<Integer, String> keyValue = Map.of(1, "Amit",
                                             2, "Ali",
                                             3, "Mindorks");
```

#### Clojure

```clojure
(def list-of-number [1 2 3 4])

(def key-value {1 "Amit" 
                2 "Ali" 
                3 "Mindorks"})
```

## What to do with Java loops

#### Java

```java
for (int i = 1; i <= 10 ; i++) { }

for (int i = 1; i < 10 ; i++) { }

for (int i = 10; i >= 0 ; i--) { }

for (int i = 1; i <= 10 ; i+=2) { }

for (int i = 10; i >= 0 ; i-=2) { }

for (String item : collection) { }

for (Map.Entry<String, String> entry: map.entrySet()) { }
```

#### Clojure
Translating a Java loop to Clojure has a lot of options, and choosing between them depends on *why* you're looping in the first place.

Recursion is handy:
```clojure
(loop [x 10]
  (when-not (= x 0)
    (recur (- x 2))
```

You may want to calculate a value *for* every value in some collection(s):
```clojure
(for [x coll])
  (calculate-y x))
```

You can [destructure](https://clojure.org/guides/destructuring) each value in a collection as you iterate over it:
```clojure
(for [[k v] map]
  (somefunc k v))
```

You can iterate over multiple collections, similar to nested loops in Java:
```clojure
(for [x ['a 'b 'c] 
      y [1 2 3]]
  (foo x y))
```

If you just need to produce a side effect some number of times, [repeatedly](http://clojuredocs.org/clojure.core/repeatedly) is your jam:
``` clojure
(repeatedly 10 some-fn)
```

If you need to produce a side effect for each value in a collection, try [doseq](http://clojuredocs.org/clojure.core/doseq):
```clojure
(doseq [x coll]
  (do-some-side-effect! x))
```

You can destructure, just like in `for` above:
```clojure
(doseq [[k v] {:one 1}]
  (do-some-side-effect! (inc v)))
```

If you need to produce a side effect for a range of integers, you *could* use `doseq` like this:
``` clojure
(doseq [x (range 10)] 
  (do-something! x))
```
...but [dotimes](http://clojuredocs.org/clojure.core/dotimes) is like `doseq` with a built-in [range](http://clojuredocs.org/clojure.core/range):
``` clojure
(dotimes [x 9] 
  (do-something! x))
```


## Bitwise Manipulation

#### Java

```java
final int andResult  = a & b;
final int orResult   = a | b;
final int xorResult  = a ^ b;
final int rightShift = a >> 2;
final int leftShift  = a << 2;
```

#### Clojure

```clojure
(def and-result (bit-and a b))
(def or-result (bit-or a b)
(def xor-result (bit-xor a b)
(def right-shift (bit-shift-right a 2))
(def left-shift (bit-shift-left a 2))
```

## Objects

#### Java

```java
if (object instanceof Car) {
}
Car car = (Car) object;
```

#### Clojure

```clojure
(if (= (type object) Car))

(def car (cast object Car)
```

---

#### Java

```java
if (object instanceof Car) {
   Car car = (Car) object;
}
```

#### Clojure

```clojure
; There isn't really an equivalent for this since Clojure is dynamically typed and not object-oriented
```

---

#### Java

```java
// Java 7 and below
for (Car car : cars) {
  System.out.println(car.speed);
}

// Java 8+
cars.forEach(car -> System.out.println(car.speed));

// Java 7 and below
for (Car car : cars) {
  if (car.speed > 100) {
    System.out.println(car.speed);
  }
}

// Java 8+
cars.stream().filter(car -> car.speed > 100).forEach(car -> System.out.println(car.speed));
```

#### Clojure

```clojure
(doseq [car cars]
  (println (:speed car)))

(map (comp println :speed)
  (filter #(> (:speed %) 100) cars))
```

---

#### Java

```java
void doSomething() {
   // logic here
}
```

#### Clojure
Idiomatic Clojure naming uses an exclamation point suffix to denote that the function is impure (it "does something" elsewhere rather than returning a value). Use of side effects is minimized in the functional style.

```clojure
(defn do-something! []
  ; side effects here
  )
```

---

#### Java

```java
void doSomething(int... numbers) {
   // logic here
}
```

#### Clojure

```clojure
(defn do-something! [& xs]
  ; side effects here)
```

---

#### Java

```java
int getScore() {
   // logic here
   return score;
}
```

#### Clojure
Idiomatic functional naming usually elides "get-" prefixes:

```clojure
(defn score []
  ;; logic here
  ;; no need for a separate name to "return" because the last value just gets returned
  )
```

---

#### Java

```java
public class Utils {

    private Utils() { 
      // This utility class is not publicly instantiable 
    }
    
    public static int getScore(int value) {
        return 2 * value;
    }
    
}
```

#### Clojure

```clojure
(defn score [value] (* 2 value))
```

---

#### Java

```java
public class Developer {

    private String name;
    private int age;

    public Developer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Developer developer = (Developer) o;

        if (age != developer.age) return false;
        return name != null ? name.equals(developer.name) : developer.name == null;

    }

    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        return result;
    }

    @Override
    public String toString() {
        return "Developer{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

#### Clojure

```clojure
(defrecord Developer [name age])

```

---

#### Java

```java
public class Developer implements Cloneable {

    private String name;
    private int age;

    public Developer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return (Developer)super.clone();
    }
}

// cloning or copying 
Developer dev = new Developer("Max", 30);
try {
     Developer dev2 = (Developer) dev.clone();
} catch (CloneNotSupportedException e) {
     // Handle Exception
}

```

#### Clojure

```clojure
(defrecord Developer [name age])

;; cloning or copying
(def dev (->Developer "Max" 30))
(def dev2 dev)
;; in case you only want to copy selected properties
(def dev3 (map->Developer {:age 25})

```

---

#### Java

```java
public class Utils {

    private Utils() { 
      // This utility class is not publicly instantiable 
    }
    
    public static int triple(int value) {
        return 3 * value;
    }
    
}

int result = Utils.triple(3);

```

#### Clojure

```clojure
(defn triple [n] (* 3 n)

(def result (triple 3))
```

---

#### Java

```java
ImageView imageView;
```

#### Clojure

```clojure
(ImageView.)
```
