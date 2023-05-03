Download Link: https://assignmentchef.com/product/solved-cs2030s-mini-project-infinite-list
<br>
<table width="683">
<tbody>
<tr>
<td width="683">In the lectures, we have implemented an immutable list that is finite. Now, you are going to implement an infinite version, which is recursive and makes heavy use of the laziness of lambda expressions. Here are some essential properties on how our infinite list should behave:An infinite list needs to be as lazy as possible and only generate the elements from the data source when necessaryAn infinite list pipeline needs to be immutableWhen an element is generated, it should not be generated again. We do this by caching a copy of the value the first time an element is generated. Subsequent probing of the same value will result in the cached copy being returned. <em>You will see the need for this once we go stateful</em>.<strong>The Task</strong>You are to design an implementation of an InfiniteList&lt;T&gt; that supports the following operations:<strong>Data sources:</strong>public static &lt;T&gt; InfiniteList&lt;T&gt; generate(Supplier&lt;? extends T&gt; s) public static &lt;T&gt; InfiniteList&lt;T&gt; iterate(T seed, UnaryOperator&lt;T&gt; next)<strong>Terminal operations:</strong>public long count();public &lt;U&gt; U reduce(U identity, BiFunction&lt;U, ? super T, U&gt; accumulator);public Object[] toArray(); public void forEach(Consumer&lt;? super T&gt; action);<strong>Stateless intermediate operations:</strong>public InfiniteList&lt;T&gt; filter(Predicate&lt;? super T&gt; predicate); public &lt;R&gt; InfiniteList&lt;R&gt; map(Function&lt;? super T, ? extends R&gt; mapper);<strong>Stateful intermediate operations:</strong></td>
</tr>
</tbody>
</table>
<table width="609">
<tbody>
<tr>
<td width="609"><strong>jshell&gt; /open Lazy.java</strong> <strong>jshell&gt; Lazy.ofNullable(4)</strong>$.. ==&gt; Lazy[4]<strong>jshell&gt; Lazy.ofNullable(4).get()</strong>$.. ==&gt; Optional[4]<strong>jshell&gt; Lazy.ofNullable(4).map(x -&gt; x + 4)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.ofNullable(4).filter(x -&gt; x &gt; 2)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.ofNullable(4).map(x -&gt; 1).get()</strong>$.. ==&gt; Optional[1]<strong>jshell&gt; Lazy.ofNullable(4).filter(x -&gt; true).get()</strong>$.. ==&gt; Optional[4]<strong>jshell&gt; Lazy.ofNullable(4).filter(x -&gt; false).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy.ofNullable(4).map(x -&gt; 1).filter(x -&gt; false).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy.ofNullable(4).filter(x -&gt; false).map(x -&gt; 1).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy.ofNullable(4).filter(x -&gt; true).map(x -&gt; 1).get()</strong>$.. ==&gt; Optional[1]<strong>jshell&gt; Lazy.ofNullable(4).filter(x -&gt; false).filter(x -&gt; true).get()</strong>$.. ==&gt; Optional.empty <strong>jshell&gt; </strong><strong>jshell&gt; Lazy.ofNullable(null)</strong>$.. ==&gt; Lazy[null]<strong>jshell&gt; Lazy.ofNullable(null).get()</strong>$.. ==&gt; Optional.empty</td>
</tr>
</tbody>
</table>
public InfiniteList&lt;T&gt; limit(long n); public InfiniteList&lt;T&gt; takeWhile(Predicate&lt;? super T&gt; predicate);

Since InfiniteList is similar to Java’s Stream, you are <strong>not allowed</strong> to import packages from java.util.stream.

The InfiniteList <strong>interface</strong> has also been provided for you with most of the methods commented out. It can be downloaded <a href="https://codecrunch.comp.nus.edu.sg/taskfile.php/5103/InfiniteList.java">here</a>. You can uncomment individual methods as you proceed through the levels. Note that an uncommented version of this same interface will be used when testing in CodeCrunch.

This task is divided into several levels. Read through all the levels to see how the different levels are related.

Remember to:

always compile your program files first before either using jshell to test your program, or using java to run your program run checkstyle on your code
<h2>Level 1: Getting Lazy</h2>
We will start by developing a building block for our InfiniteList: an abstraction for an object of which the value is not computed until it is needed. Let’s build a standalone generic class Lazy&lt;T&gt; (an extension of the one that was discussed in lecture) that encapsulates a value of type T, with the following operations and properties:

static &lt;T&gt; Lazy&lt;T&gt; ofNullable(T v) creates a lazy object with a given value (precomputed) v. Note that the Lazy object can contain a null value.

static &lt;T&gt; Lazy&lt;T&gt; of(Supplier&lt;? extends T&gt; supplier) creates a lazy object with a supplier to produce a value.

Optional&lt;T&gt; get() returns an Optional of the contained value if it is not null; returns Optional.empty otherwise. If necessary (i.e. when the value is not already cached), get() triggers the computation of the value encapsulated in the Lazy instance (invoking the chain of supplier, predicates, and functions).

&lt;R&gt; Lazy&lt;R&gt; map(Function&lt;? super T, ? extends R&gt; mapper) applies the function mapper lazily (i.e., only when needed) to the content of this calling Lazy instance and returns the new Lazy instance. If, upon evaluation, the content is null, then a lazy object containing null is returned.

Lazy&lt;T&gt; filter(Predicate&lt;? super T&gt; predicate) applies the given predicate lazily to test the content of the calling Lazy instance. If the predicate returns true, the content is retained, otherwise, the content becomes null. If the content is already null, then the calling object is returned.

String toString() returns “Lazy[?]” if the value has not been computed; the string representation of the value enclosed in Lazy[..] otherwise.

As Lazy is the context that handles null values and caching

You are allowed to use the null value in the Lazy class,

Object properties need not be declared final; however they should still be private.

Implementing Lazy and understanding how Lazy works is going to be very useful in later levels.
<table width="609">
<tbody>
<tr>
<td width="609"><strong>jshell&gt; Lazy.ofNullable(null).map(x -&gt; 1)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.ofNullable(null).filter(x -&gt; true)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.ofNullable(null).filter(x -&gt; false)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.ofNullable(null).map(x -&gt; 1).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy.ofNullable(null).filter(x -&gt; true).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy.ofNullable(null).filter(x -&gt; false).get()</strong>$.. ==&gt; Optional.empty <strong>jshell&gt; </strong><strong>jshell&gt; Lazy.of(() -&gt; 4)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.of(() -&gt; 4).get()</strong>$.. ==&gt; Optional[4]<strong>jshell&gt; Lazy.of(() -&gt; 4).map(x -&gt; 1)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.of(() -&gt; 4).filter(x -&gt; true)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.of(() -&gt; 4).map(x -&gt; 1).get()</strong>$.. ==&gt; Optional[1]<strong>jshell&gt; Lazy.of(() -&gt; 4).filter(x -&gt; true).get()</strong>$.. ==&gt; Optional[4]<strong>jshell&gt; Lazy.of(() -&gt; 4).filter(x -&gt; false).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy&lt;Integer&gt; lazy = Lazy.of(() -&gt; 4) </strong><strong>jshell&gt; lazy </strong>lazy ==&gt; Lazy[?] <strong>jshell&gt; lazy.get()</strong> $.. ==&gt; Optional[4] <strong>jshell&gt; lazy </strong>lazy ==&gt; Lazy[4]<strong>jshell&gt; </strong><strong>jshell&gt; Lazy.of(() -&gt; null)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.of(() -&gt; null).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy.of(() -&gt; null).map(x -&gt; 1)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.of(() -&gt; null).filter(x -&gt; false)</strong>$.. ==&gt; Lazy[?]<strong>jshell&gt; Lazy.of(() -&gt; null).map(x -&gt; 1).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy.of(() -&gt; null).filter(x -&gt; true).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy.of(() -&gt; null).filter(x -&gt; false).get()</strong>$.. ==&gt; Optional.empty<strong>jshell&gt; Lazy&lt;Integer&gt; lazy = Lazy.of(() -&gt; null) </strong> <strong>jshell&gt; lazy </strong>lazy ==&gt; Lazy[?] <strong>jshell&gt; lazy.get()</strong> $.. ==&gt; Optional.empty <strong>jshell&gt; lazy </strong>lazy ==&gt; Lazy[null]</td>
</tr>
</tbody>
</table>
Compile your program by running the following on the command line:

$ javac -Xlint:rawtypes *.java
<h2>Level 2: To Infinity and Beyond</h2>
Define the InfiniteListImpl class that implements the InfiniteList interface. Write the static generate and iterate methods that initiate the initial list pipeline. You are encouraged to adapt the implementation demonstrated in class using the head and tail suppliers. If you do it right, you need only have these two declared private final and nothing else. That said, if you feel compelled to introduce other fields to aid in your implementation, feel free to do so. But note that the implementation demonstrated in class does not support caching of computed values. To solve this lab, you will need to use the type Lazy&lt;T&gt; for the head to take advantage of its laziness, instead of Supplier&lt;T&gt;.
<table width="609">
<tbody>
<tr>
<td width="609"><strong>jshell&gt; InfiniteList&lt;Integer&gt; list</strong><strong>jshell&gt; InfiniteList.generate(() -&gt; 1) instanceof InfiniteListImpl</strong>$.. ==&gt; true<strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1) instanceof InfiniteListImpl</strong>$.. ==&gt; true<strong>jshell&gt; list = InfiniteListImpl.generate(() -&gt; 1).peek()</strong>1<strong>jshell&gt; list = InfiniteListImpl.iterate(1, x -&gt; x + 1).peek()</strong>1<strong>jshell&gt; list = InfiniteListImpl.iterate(1, x -&gt; x + 1).peek().peek()</strong>1 2<strong>jshell&gt; InfiniteList&lt;Integer&gt; list2 = list.peek()</strong>3<strong>jshell&gt; list != list2</strong>$.. ==&gt; true<strong>jshell&gt; InfiniteList&lt;String&gt; list = InfiniteListImpl.iterate(“A”, x -&gt; x + “Z”).peek().pee</strong>AAZ AZZ <strong>jshell&gt; </strong><strong>jshell&gt; UnaryOperator&lt;Integer&gt; op = x -&gt; { System.out.printf(“iterate: %d -&gt; %d
”, x, x + jshell&gt; list2 = InfiniteList.iterate(1, op).peek().peek()</strong>1iterate: 1 -&gt; 22iterate: 2 -&gt; 3 <strong>jshell&gt; </strong> <strong>jshell&gt; /exit</strong></td>
</tr>
</tbody>
</table>
<table width="609">
<tbody>
<tr>
<td width="609"><strong>jshell&gt; InfiniteList&lt;Integer&gt; list, list2</strong><strong>jshell&gt; list = InfiniteList.generate(() -&gt; 1).map(x -&gt; x * 2)</strong> <strong>jshell&gt; list2 = list.peek()</strong>2<strong>jshell&gt; list2 = list.peek()</strong>2<strong>jshell&gt; InfiniteList.generate(() -&gt; 1).map(x -&gt; x * 2) instanceof InfiniteListImpl</strong>$.. ==&gt; true<strong>jshell&gt; list = InfiniteList.generate(() -&gt; 1).map(x -&gt; x * 2).peek()</strong>2<strong>jshell&gt; list = InfiniteList.generate(() -&gt; 1).map(x -&gt; x * 2).peek().peek()</strong>2 2<strong>jshell&gt; list = InfiniteList.iterate(1, x -&gt; x + 1).map(x -&gt; x * 2).peek().peek()</strong>2 4 <strong>jshell&gt; </strong><strong>jshell&gt; Supplier&lt;Integer&gt; generator = () -&gt; { System.out.println(“generate: 1”); return 1 jshell&gt; Function&lt;Integer,Integer&gt; doubler = x -&gt; { System.out.printf(“map: %d -&gt; %d
”, x jshell&gt; Function&lt;Integer,Integer&gt; oneLess = x -&gt; { System.out.printf(“map: %d -&gt; %d
”, x</strong><strong>jshell&gt; list = InfiniteList.generate(generator).map(doubler).peek().peek()</strong> generate: 1 map: 1 -&gt; 22</td>
</tr>
</tbody>
</table>
For debugging purposes, include a method InfiniteList&lt;T&gt; peek() within your implementation that prints the first element of the infinite list to the standard output and returns the rest of the list as an InfiniteList. This enables the chaining of peek() methods to produce an output of a sequence of stream elements. Note that peek() is solely for the purpose of debugging, until a terminal operation is introduced at a later level.

Compile your program by running the following on the command line:

$ javac -Xlint:rawtypes *.java

$ jshell -q Lazy.java InfiniteList.java InfiniteListImpl.java &lt; level2.jsh

Check your styling by issuing the following

$ checkstyle *.java
<h2>Level 3: Map and Filter</h2>
Now implement the map and filter operations. In particular, if an element from an upstream operation does not pass through filter, an Optional.empty() will be generated. peek() does not print anything if the element is Optional.empty().
<table width="609">
<tbody>
<tr>
<td width="609">generate: 1 map: 1 -&gt; 22<strong>jshell&gt; list = InfiniteList.generate(generator).map(doubler).map(oneLess).peek().peek() </strong>generate: 1 map: 1 -&gt; 2 map: 2 -&gt; 11 generate: 1 map: 1 -&gt; 2 map: 2 -&gt; 11 <strong>jshell&gt; </strong><strong>jshell&gt; list = InfiniteList.iterate(1, x -&gt; x + 1).filter(x -&gt; x % 2 == 0)</strong> <strong>jshell&gt; list2 = list.peek()</strong> <strong>jshell&gt; list2 = list.peek()</strong><strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).filter(x -&gt; x % 2 == 0) instanceof InfiniteLis</strong>$.. ==&gt; true<strong>jshell&gt; list = InfiniteList.iterate(1, x -&gt; x + 1).filter(x -&gt; x % 2 == 0).peek().peek()</strong>2<strong>jshell&gt; list = InfiniteList.iterate(1, x -&gt; x + 1).filter(x -&gt; x % 2 == 0).filter(x -&gt; x &lt;</strong>2 <strong>jshell&gt; </strong><strong>jshell&gt; Predicate&lt;Integer&gt; isEven = x -&gt; { System.out.printf(“filter: %d -&gt; %b
”, x, x % jshell&gt; Predicate&lt;Integer&gt; lessThan10 = x -&gt; { System.out.printf(“filter: %d -&gt; %b
”, x, jshell&gt; UnaryOperator&lt;Integer&gt; op = x -&gt; { System.out.printf(“iterate: %d -&gt; %d
”, x, x +</strong><strong>jshell&gt; list = InfiniteList.iterate(1, op).filter(isEven).peek().peek()</strong> filter: 1 -&gt; false iterate: 1 -&gt; 2 filter: 2 -&gt; true2iterate: 2 -&gt; 3<strong>jshell&gt; list = InfiniteList.iterate(1, op).filter(isEven).filter(lessThan10).peek().peek( </strong>filter: 1 -&gt; false iterate: 1 -&gt; 2 filter: 2 -&gt; true filter: 2 -&gt; true2iterate: 2 -&gt; 3 <strong>jshell&gt; </strong><strong>jshell&gt; list = InfiniteList.iterate(1, op).map(doubler).filter(isEven).filter(lessThan10) </strong>map: 1 -&gt; 2 filter: 2 -&gt; true filter: 2 -&gt; true2iterate: 1 -&gt; 2 map: 2 -&gt; 4 filter: 4 -&gt; true filter: 4 -&gt; true4iterate: 2 -&gt; 3<strong>jshell&gt; list = InfiniteList.iterate(1, op).filter(isEven).map(doubler).filter(lessThan10) </strong>filter: 1 -&gt; false iterate: 1 -&gt; 2 filter: 2 -&gt; true map: 2 -&gt; 4 filter: 4 -&gt; true4iterate: 2 -&gt; 3<strong>jshell&gt; list = InfiniteList.iterate(1, op).filter(isEven).filter(lessThan10).map(doubler) </strong>filter: 1 -&gt; false iterate: 1 -&gt; 2 filter: 2 -&gt; true filter: 2 -&gt; true map: 2 -&gt; 44iterate: 2 -&gt; 3 <strong>jshell&gt; /exit</strong></td>
</tr>
</tbody>
</table>
Compile your program by running the following on the command line:

$ javac -Xlint:rawtypes *.java

$ jshell -q &lt;your java files&gt; &lt; level3.jsh
<table width="609">
<tbody>
<tr>
<td width="609"><strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; InfiniteList.generate(() -&gt; 2).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; InfiniteList.generate(() -&gt; 2).filter(x -&gt; x % 3 == 0).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).map(x -&gt; 2).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).filter(x -&gt; x % 2 == 0).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; new EmptyList&lt;&gt;().isEmpty()</strong>$.. ==&gt; true<strong>jshell&gt; new EmptyList&lt;&gt;().map(x -&gt; 2).isEmpty()</strong>$.. ==&gt; true<strong>jshell&gt; new EmptyList&lt;&gt;().filter(x -&gt; true).isEmpty()</strong>$.. ==&gt; true<strong>jshell&gt; new EmptyList&lt;&gt;().filter(x -&gt; false).isEmpty()</strong>$.. ==&gt; true <strong>jshell&gt; </strong><strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).limit(0).isEmpty()</strong>$.. ==&gt; true<strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).limit(1).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).limit(-1).isEmpty()</strong>$.. ==&gt; true <strong>jshell&gt; </strong><strong>jshell&gt; UnaryOperator&lt;Integer&gt; op = x -&gt; { System.out.printf(“iterate: %d -&gt; %d
”, x, x +</strong><strong>jshell&gt; InfiniteList.iterate(1, op).limit(0).isEmpty()</strong>$.. ==&gt; true<strong>jshell&gt; InfiniteList.iterate(1, op).limit(1).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; InfiniteList.iterate(1, op).limit(2).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; InfiniteList&lt;Integer&gt; list;</strong><strong>jshell&gt; list = InfiniteList.iterate(1, op).limit(0).peek()</strong> <strong>jshell&gt; list = InfiniteList.iterate(1, op).limit(1).peek()</strong>1<strong>jshell&gt; list = InfiniteList.iterate(1, op).limit(1).peek().peek()</strong>1<strong>jshell&gt; list = InfiniteList.iterate(1, op).limit(2).peek().peek().peek()</strong>1iterate: 1 -&gt; 22<strong>jshell&gt; list = InfiniteList.iterate(1, op).limit(2).limit(1).peek().peek()</strong>1<strong>jshell&gt; list = InfiniteList.iterate(1, op).limit(1).limit(2).peek().peek()</strong>1 <strong>jshell&gt; </strong><strong>jshell&gt; InfiniteList.iterate(1, op).takeWhile(x -&gt; x &lt; 0).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; InfiniteList.iterate(1, op).takeWhile(x -&gt; x &lt; 2).isEmpty()</strong>$.. ==&gt; false<strong>jshell&gt; list = InfiniteList.iterate(1, op).takeWhile(x -&gt; x &lt; 0).peek()</strong></td>
</tr>
</tbody>
</table>
Check your styling by issuing the following

$ checkstyle *.java
<h2>Level 4: Emptiness and Limitations</h2>
Now, create a subtype of InfiniteList that represents an empty list called EmptyList. The EmptyList should return true when isEmpty() is called and it should return itself for intermediate operations and appropriate values for terminal operations.

Then, implement the limit method. There is now a need to differentiate between an Optional.empty() produced from filter and the end of the stream in limit. Make use of EmptyList to indicate the end of the stream. When dealing with limit, you will need to decide if the upstream element

produces an empty list; produces an Optional.empty and ignored by limit; or produces a stream element and accounted for by limit

The other operation to truncate an infinite list is the takeWhile operator. The same considerations that you have given to limit would probably apply here.
<table width="609">
<tbody>
<tr>
<td width="609"><strong>jshell&gt; list = InfiniteList.iterate(1, op).takeWhile(x -&gt; x &lt; 2).peek().peek()</strong>1iterate: 1 -&gt; 2<strong>jshell&gt; list = InfiniteList.iterate(1, op).takeWhile(x -&gt; x &lt; 2).takeWhile(x -&gt; x &lt; 0).pee jshell&gt; list = InfiniteList.iterate(1, op).takeWhile(x -&gt; x &lt; 0).takeWhile(x -&gt; x &lt; 2).pee jshell&gt; list = InfiniteList.iterate(1, op).takeWhile(x -&gt; x &lt; 5).takeWhile(x -&gt; x &lt; 2).pee</strong>1iterate: 1 -&gt; 2 <strong>jshell&gt; </strong><strong>jshell&gt; Predicate&lt;Integer&gt; lessThan5 = x -&gt; { System.out.printf(“takeWhile: %d -&gt; %b
”, x</strong><strong>jshell&gt; list = InfiniteList.iterate(1, op).takeWhile(lessThan5).peek().peek()</strong> takeWhile: 1 -&gt; true1iterate: 1 -&gt; 2 takeWhile: 2 -&gt; true2iterate: 2 -&gt; 3 <strong>jshell&gt; </strong> <strong>jshell&gt; /exit</strong></td>
</tr>
</tbody>
</table>
<table width="609">
<tbody>
<tr>
<td width="609"><strong>jshell&gt; new EmptyList&lt;&gt;().toArray()</strong>$.. ==&gt; Object[0] { }<strong>jshell&gt; InfiniteList.iterate(0, i -&gt; i + 1).limit(10).limit(3).toArray()</strong>$.. ==&gt; Object[3] { 0, 1, 2 }<strong>jshell&gt; InfiniteList.iterate(0, i -&gt; i + 1).limit(3).limit(100).toArray()</strong>$.. ==&gt; Object[3] { 0, 1, 2 }<strong>jshell&gt; InfiniteList.generate(() -&gt; 1).limit(0).toArray()</strong>$.. ==&gt; Object[0] { }<strong>jshell&gt; InfiniteList.generate(() -&gt; 1).limit(2).toArray()</strong>$.. ==&gt; Object[2] { 1, 1 } <strong>jshell&gt; Random rnd = new Random(1)</strong><strong>jshell&gt; InfiniteList.generate(() -&gt; rnd.nextInt() % 100).limit(4).toArray();</strong>$.. ==&gt; Object[4] { -25, 76, 95, 26 }<strong>jshell&gt; InfiniteList.generate(() -&gt; “A”).map(x -&gt; x + “-“).map(str -&gt; str.length()).limit</strong>$.. ==&gt; Object[4] { 2, 2, 2, 2 }<strong>jshell&gt; InfiniteList.generate(() -&gt; “A”).limit(4).map(x -&gt; x + “-“).map(str -&gt; str.length</strong>$.. ==&gt; Object[4] { 2, 2, 2, 2 }<strong>jshell&gt; InfiniteList.generate(() -&gt; “A”).map(x -&gt; x + “-“).limit(4).map(str -&gt; str.length</strong>$.. ==&gt; Object[4] { 2, 2, 2, 2 }<strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).limit(4).filter(x -&gt; x % 2 == 0).toArray()</strong>$.. ==&gt; Object[2] { 2, 4 }<strong>jshell&gt; InfiniteList.iterate(1, x -&gt; x + 1).filter(x -&gt; x % 2 == 0).limit(4).toArray()</strong>$.. ==&gt; Object[4] { 2, 4, 6, 8 }<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).filter(x -&gt; x &gt; 10).filter(x -&gt; x &lt; 20).limit</strong>$.. ==&gt; Object[5] { 11, 12, 13, 14, 15 }<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).filter(x -&gt; x &gt; 10).map(x -&gt; x.hashCode() % 30</strong>$.. ==&gt; Object[5] { 11, 12, 13, 14, 15 }<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).takeWhile(x -&gt; x &lt; 3).toArray()</strong>$.. ==&gt; Object[3] { 0, 1, 2 }<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).takeWhile(x -&gt; x &lt; 0).toArray()</strong>$.. ==&gt; Object[0] { }<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).takeWhile(x -&gt; x &lt; 10).takeWhile(x -&gt; x &lt; 5).t</strong>$.. ==&gt; Object[5] { 0, 1, 2, 3, 4 }<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).map(x -&gt; x).takeWhile(x -&gt; x &lt; 4).limit(1).toA</strong>$.. ==&gt; Object[1] { 0 }<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(4).takeWhile(x -&gt; x &lt; 2).toArray()</strong>$.. ==&gt; Object[2] { 0, 1 }<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).map(x -&gt; x * x).filter(x -&gt; x &gt; 10).limit(1).t</strong>$.. ==&gt; Object[1] { 16 }</td>
</tr>
</tbody>
</table>
Compile your program by running the following on the command line:

$ javac -Xlint:rawtypes *.java

$ jshell -q Lazy.java InfiniteList.java InfiniteListImpl.java &lt; level4.jsh

Check your styling by issuing the following

$ checkstyle *.java
<h2>Level 5: Terminals</h2>
Now, we are going to implement the terminal operations: forEach, count, reduce, and toArray.

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).filter(x -&gt; x &gt; 10).map(x -&gt; x * x).limit(1).t</strong>

$.. ==&gt; Object[1] { 121 } <strong>jshell&gt; Random rnd = new Random(1)</strong>

<strong>jshell&gt; InfiniteList.generate(() -&gt; rnd.nextInt() % 100).filter(x -&gt; x &gt; 10).limit(4).toAr</strong>

$.. ==&gt; Object[4] { 76, 95, 26, 69 } <strong>jshell&gt; </strong>

<strong>jshell&gt; new EmptyList&lt;&gt;().count()</strong>

$.. ==&gt; 0

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(0).count()</strong>

$.. ==&gt; 0

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(1).count()</strong>

$.. ==&gt; 1

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).filter(x -&gt; x % 2 == 1).limit(10).count()</strong>

$.. ==&gt; 10

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(10).filter(x -&gt; x % 2 == 1).count()</strong>

$.. ==&gt; 5

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).takeWhile(x -&gt; x &lt; 10).count()</strong>

$.. ==&gt; 10

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).takeWhile(x -&gt; x &lt; 10).filter(x -&gt; x % 2 == 0</strong>

$.. ==&gt; 5

<strong>jshell&gt; Random rnd = new Random(1)</strong>

<strong>jshell&gt; InfiniteList.generate(() -&gt; Math.abs(rnd.nextInt()) % 100).takeWhile(x -&gt; x &gt; 5).c</strong>

$.. ==&gt; 9 <strong>jshell&gt; </strong>

<strong>jshell&gt; new EmptyList&lt;Integer&gt;().reduce(100, (x,y) -&gt; x*y)</strong>

$.. ==&gt; 100

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(5).reduce(0, (x, y) -&gt; x + y)</strong>

$.. ==&gt; 10

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(0).reduce(0, (x, y) -&gt; x + y)</strong>

$.. ==&gt; 0

<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).map(x -&gt; x * x).limit(5).reduce(1, (x, y) -&gt; x</strong>

$.. ==&gt; 0

<strong>jshell&gt; Random rnd = new Random(1)</strong>

<strong>jshell&gt; InfiniteList.generate(() -&gt; rnd.nextInt() % 100).filter(x -&gt; x &gt; 0).limit(10).redu</strong>

$.. ==&gt; 95 <strong>jshell&gt; </strong>

<strong>jshell&gt; UnaryOperator&lt;Integer&gt; op = x -&gt; { System.out.printf(“iterate: %d -&gt; %d
”, x, x + jshell&gt; Supplier&lt;Integer&gt; generator = () -&gt; { System.out.println(“generate: 1”); return 1 jshell&gt; Function&lt;Integer,Integer&gt; doubler = x -&gt; { System.out.printf(“map: %d -&gt; %d
”, x jshell&gt; Function&lt;Integer,Integer&gt; oneLess = x -&gt; { System.out.printf(“map: %d -&gt; %d
”, x jshell&gt; Predicate&lt;Integer&gt; lessThan100 = x -&gt; { System.out.printf(“takeWhile: %d -&gt; %b
” jshell&gt; Predicate&lt;Integer&gt; moreThan10 = x -&gt; { System.out.printf(“filter: %d -&gt; %b
”, x, </strong>

<strong>jshell&gt; </strong>

<strong>jshell&gt; InfiniteList.iterate(0, op).filter(lessThan100).map(doubler).takeWhile(lessThan100 </strong>takeWhile: 0 -&gt; true map: 0 -&gt; 0 takeWhile: 0 -&gt; true map: 0 -&gt; -1 iterate: 0 -&gt; 1 takeWhile: 1 -&gt; true map: 1 -&gt; 2 takeWhile: 2 -&gt; true map: 2 -&gt; 1 iterate: 1 -&gt; 2 takeWhile: 2 -&gt; true map: 2 -&gt; 4

takeWhile: 4 -&gt; true

<a href="#_Toc22806">map: 4 -&gt; 3 </a>

<a href="#_Toc22807">iterate: 2 -&gt; 3 </a>

<a href="#_Toc22808">map: 3 -&gt; 6 takeWhile: 6 -&gt; true map: 6 -&gt; 5 </a>

takeWhile: 3 -&gt; true

iterate: 3 -&gt; 4

takeWhile: 4 -&gt; true

map: 4 -&gt; 8

takeWhile: 8 -&gt; true map: 8 -&gt; 7

$.. ==&gt; Object[5] { -1, 1, 3, 5, 7 }

<strong>jshell&gt; InfiniteList.generate(generator).filter(lessThan100).map(doubler).takeWhile(lessTh </strong>generate: 1 takeWhile: 1 -&gt; true map: 1 -&gt; 2 takeWhile: 2 -&gt; true
<table width="683">
<tbody>
<tr>
<td width="37"></td>
<td width="609">map: 2 -&gt; 1 generate: 1 takeWhile: 1 -&gt; true map: 1 -&gt; 2 takeWhile: 2 -&gt; true map: 2 -&gt; 1 generate: 1 takeWhile: 1 -&gt; true map: 1 -&gt; 2 takeWhile: 2 -&gt; true map: 2 -&gt; 1 generate: 1 takeWhile: 1 -&gt; true map: 1 -&gt; 2 takeWhile: 2 -&gt; true map: 2 -&gt; 1 generate: 1 takeWhile: 1 -&gt; true map: 1 -&gt; 2 takeWhile: 2 -&gt; true map: 2 -&gt; 1$.. ==&gt; Object[5] { 1, 1, 1, 1, 1 } <strong>jshell&gt; </strong><strong>jshell&gt; new EmptyList&lt;&gt;().forEach(System.out::println)</strong><strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(0).forEach(System.out::println)</strong> <strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(1).forEach(System.out::println)</strong>0<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).filter(x -&gt; x % 2 == 1).limit(10).forEach(Syst</strong>1357911131517 19<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).limit(10).filter(x -&gt; x % 2 == 1).forEach(Syst</strong>1357 9<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).takeWhile(x -&gt; x &lt; 10).forEach(System.out::pri</strong></td>
<td width="37"></td>
</tr>
<tr>
<td rowspan="2" width="37"></td>
<td width="609">012345678 9<strong>jshell&gt; InfiniteList.iterate(0, x -&gt; x + 1).takeWhile(x -&gt; x &lt; 10).filter(x -&gt; x % 2</strong>0246 8<strong>jshell&gt; /exit</strong></td>
<td rowspan="2" width="37"><strong> == 0</strong></td>
</tr>
<tr>
<td width="609">Compile your program by running the following on the command line:
<table width="609">
<tbody>
<tr>
<td width="609">$ javac -Xlint:rawtypes *.java</td>
</tr>
</tbody>
</table>
You should try to test your implementation as exhaustively as you can before submitting to CodeCrunch. We shall be using another client class to test your implementation.</td>
</tr>
</tbody>
</table>