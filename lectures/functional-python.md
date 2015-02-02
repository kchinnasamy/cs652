# Functional programming in Python

Collected lots of little bits and pieces from around the Web to cobble this together.

## Resources

Notes taken from the following resources with my own thoughts intermingled:

* http://docs.python.org/howto/functional.html
* http://www.slideshare.net/adambyrtek/functional-programming-with-python-516744
* http://www.ibm.com/developerworks/library/l-pycon/index.html
* http://www.slideshare.net/akyrola/sulake-func-presis

## History

From wikipedia history of functional programming:

* 40s? lambda calculus
* 50s: Lisp
* 70s: ML for theorem proving (not pure functional), Scheme (dialect of Lisp)
* 1977 John Backus presents FP about his ideas to get away from imperative programming
* 1987 Haskell (pure functional)
* 1986 Erlang
* 2003 Scala
* 2005 F#

## Characteristics

* Functional programs evaluate expressions instead of executing sequences instructions.
* Focused on what  to compute not how
* In pure functional languages, there are no variables, just labeled values
  * Implies you can't iterate through an array by moving index.
  * Can't update data structures as we go along
```
val x = 1;
x = 2; // INVALID
```
* There is no predefined order evaluation because it doesn't matter--values can be computed on demand. lazy evaluation supported
* Want functions to operate on parameters and return values. that's it. can only depend on parameters:
  * No local state maintained
  * Can't change global variables
```
y = f(x)
y' = f(x)
=> y = y'
```
* No side effects such as print statements, writing to the disk, launching missiles. Pure functional languages hide these things in creatures called monads.
* Inverse of OO programming that sends messages (functions) between objects (state). Functional languages send state between functions as parameters and return values
* Functions are first-class objects (C function pointers don't count).  Moreover, these first-class objects are **closures** and can capture parameters and other stuff; they are not not just a function address. We can create new functions on-the-fly whereas everything is fixed/static in C.
* Support for higher-order functions (functions can take functions as parameters and return functions)
* Reliance on recursion instead of looping


## Advantages

While we *should* write small methods and object ordered languages, I've seen and written functions that are huge.  Functional programming makes it more difficult to write very large functions; everything is done with lots of little helper functions and then combining the results.

Because state is not being changed all over the place, it's often easier to debug functional programs. All we care about is the parameters and return values. We can treat each function in isolation. Every function can be a unit test.

Functional programs tend to make it very easy to separate out boilerplate code into utility functions such as map, filter, reduce. To make this happen, we need to be able to pass around functions or other snippets of code.

Functional programs are more abstract; We don't have to worry about null pointers and memory issues etc. The productivity boost of Java over C is an order magnitude. The productivity boost of functional languages over Java his equally large, if you can fit your problem into its world.

Pure functional programming languages easily take advantage of concurrency because no 2 threads can be modifying the same memory. A single thread can't even modify its own memory space.

## Disadvantages

Functional programming isn't used by mainstream programmers; harder to hire people, nobody knows it, completely different way of programming

Time/space performance is an issue

Lack of good IDEs as we have for imperative programming

Adding some functional flavor to your imperative programming, though, can be extremely effective. You can gain some of the advantages listed above by incorporating functional ideas.

Lack of side effects makes it pretty hard to do GUI programming.

## Summary of functional elements from Python

 * lambdas
 * iterators
 * map, reduce, filter
 * list comprehensions
 * generator expressions
 * generators
 * higher-order functions / closures

### Lambdas

```python
def dub(x):
    return 2*x
print dub(5)
```
vs
```python
dub = lambda x: 2*x
print dub(5)
```

Can't have multi line lambdas in Python and they have to be expressions; e.g., you cannot use `print`.

### Iterators

An iterator is any object that answers `next(self)`, but they don't seem useful unless they also implement `__iter__(self)`.  `next()` "consumes" elements:
```python
>>> a,b = it.next(),it.next()
>>> a
1
>>> b
2
```

Iterators can represent infinite streams.

Raise a `StopIteration` exception when the sequence has reached its logical conclusion.

Some locations automatically convert to iterators like for each loop.  Can convert an object to an iterator by calling `iter()` manually. If that object doesn't iterate, raise `TypeError`

```python
>>> L = [1,2,3]
>>> it = iter(L)
>>> it.next()
1
>>> it.next()
2
 
>>> for x in iter(L): # iter() unnecessary
...     print x
... 
1
2
3
```

We can also create tuples, lists, multiple assignments, do mix/max with iterators:

```python
>>> tuple(iter(L))
(1, 2, 3)
>>> a,b,c = iter(L)
```

Common: file, set, dict, string, tuples support iteration

```python
for line in file: ...
for c in 'hi': ...
for c in set((1,2)): ...
```

Sample iterator:

```python
class Forever:
    n = 0
    def __iter__(self):
        return self
    def next(self):
        self.n += 1
        return self.n
```

### Map, reduce, filter

Don't you hate doing this in Java?

```java
int sum = 0;
for (int i=0; i<data.size(); i++) {
    sum += data.get(i);
}
```

You could make a function that but the idea of iterating through a list performing operation that gets a value happens constantly and we should not have to make a function for each one of those. This is the boilerplate code talking about. If I want to change that `+=` to `*=`, I have to make an entirely new loop. These are automated for us with a few keystrokes in an IDE usually but he gives us code bloat and breaks single point change rule.

One of the most common operations for me is to walk a list or other data structure to get a new list where the new elements are a function of the old or I have filtered out some of the elements.  Python provides a number of functions from functional programming that make this really easy.

To apply a function to each element of the list or sequence, we use 
`map()`:

```python
L = [1,2,3]
print map(lambda x:2*x, L)     # [2, 4, 6]
names = ['parrt', 'ksb', 'afedosov']
map(lambda x : len(x), names)  # [5, 3, 8]
map(lambda line : line.split(), file('document')) # list of word lists
```

Select values that fit a criterion we use filter():

```python
filter(lambda x:x < 3, L)      # [1, 2]
```

To combine the elements of a list we use `reduce()`:

```python
reduce(lambda x,y : x + y, L)  # 6
reduce(lambda x,y : x + y, [1,2,3,4,5]) # 15
```

We often use the term map-reduced because they are so often used in combination. These are also very easy to parallelize because we can split part the list into n chunks for n processors or machines and do them in parallel. We can reduce the elements within each chunk and then do a final reduce on the partial results from the n processors to get a single value.

### list comprehensions and generator expressions

List comprehensions are generally much more efficient than the equivalent for-loops in Python.
Syntax:

[ *expr* for *var* in *sequence*]

[ *expr* for *var* in *sequence* if *condition*]

They are like a loop that adds to the list:

```python
a = []
for var in sequence :
    a.append( expr )
```

Filter-like example:

```python
[x for x in L if x<3]
[len(x) for x in names]
compare to: map(lambda x : len(x), names)
```

Get 10 random numbers:

```python
import random
samples = [ random.random() for x in range(10) ]
```

Create list of tuples from a map:

```python
dict2list = lambda dic: [(k, v) for (k, v) in dic.iteritems()]
dict2list( {"a":3, "b":4} )  # [('a', 3), ('b', 4)]
```

Make 10 0s:

```python
[ 0 for i in range(10) ]
```

Use the IF part to filter

```python
[x for x in range(10) if x%2 == 0]  # [0, 2, 4, 6, 8]
```

How to run out of memory because list comprehensions are not lazy in Python:

```python
[str(n) for n in range(10**100)]
```

But you can use generator expressions for the same purpose I am getting lazy version:

```python
(str(n) for n in range(10**100))
```

Think of list comprehension as generator expressions in list constructors. lets us avoid creating a list and can be used generally where list comprehensions can:

```python
sum( 2*x for x in range(10) ) # 90
```

Used xrange() instead of range(), which gives you a list of elements instead of an iterator.

`any()` returns true if any element of the sequence is true; it returns false if the sequence is empty. all() requires all elements in the sequence to be true.

```python
any(x%2==0 for x in [1,2,3])  # True. Asks "is there a non-zero value"
any(x>3 for x in [1,2,3])  # False. Asks "is zero value greater than 3"

# http://theglenbot.com/all-any-and-reduce-in-python/
if a and b and c and d:
    print 'All true' 

if all([a,b,c,d]):
    print 'All true'
 
from operator import and_
if reduce(and_,[a,b,c,d]):
    print 'All true'

if a or b or c or d:
    print 'One was true'
if any([a,b,c,d]):
    print 'One was true'
 
# a cool one that checks to see if certain parameters exist
# within a URL encoded parameter list
query_string = '&a=1&b=2&c=3'
if any(map(query_string.find,['a','b'])):
    print 'It exists!'
```

These are cool because they short-circuit the generator like C && and || operators that avoid evaluating operands unnecessarily. They can cut out of an iteration when they can decide the answer.

### Generators

Generators are just continuations. Every method call has a stack activation record that holds locals and parameters. Return instruction normally pops this record off the stack but yield does not. Moreover, this makes the method special as the virtual machine recognizes it differently. Entry into this method the next time starts where it left off the last time. in other words, the pc register doesn't start at the first location in the method, it starts where the pc left off the last time. It's like a resume.

java has no capabilities to do this.

Calling a generator function actually returns a generator object implementing the iterator protocol that knows how to jump in and out of the code, treating it like a resume. It's a way to build an iterator implicitly without having to define a class and next(). If this were like an actor in a thread, we would treat it like a thread yield that could continue later.

```python
def generate_ints(N):
    for i in range(N):
        yield i
```

A return (with no value) means stop the generator. no more in next().

Generators are good for 2 reasons:

* You can avoid constructing a list object to make large or infinite streams.
* It's often much easier to build a generator than an iterator for recursive data structures such as trees. it remembers where in the recursion stack we were and can resume from there.

python doc: "You could achieve the effect of generators manually by writing your own class and storing all the local variables of the generator as instance variables."

From [stackoverflow.com answer](http://stackoverflow.com/questions/2776829/difference-between-python-generators-vs-iterators):

<blockquote>
iterator is a more general concept: any object whose class has a next method (__next__ in Python 3) and an __iter__ method that does return self.

Every generator is an iterator, but not vice versa.
</blockquote>

Another way to walk a recursive data structure is to pass in the functionality to execute rather turning the data structure into stream. The problem is, sometimes we want to treat the nodes in the tree like a stream because we get to use all of the great Python function such as map, reduce, filter etc. on them.

A recursive generator that generates tree nodes in preorder:

```python
class Tree:
    def __init__(self, payload):
        self.payload = payload
        self.children = []
    def addChild(self, c):
        self.children.append(c)
        return self

def preorder(t):
    yield t
    for c in t.children:
        for y in preorder(c):
            yield y
```

My first attempt look like this:

```python
def preorder(t):
    yield t
    for c in t.children:
        preorder(c) # whoops!
```

Then we can create a tree and walked the nodes like this:

```python
x = Tree("c").addChild(Tree("d").addChild(Tree("e")))
for node in preorder(x):
    print node.payload
```

### Higher-order functions / closures

In Java &lt; 8, these are methods wrapped in objects such as anonymous inner classes. We can pass functionality to a function by passing an object wrapped around that method. ugly. ugly. ugly. The advantage is that the function can carry along some data in the wrapper object.
We do this in Java as callbacks for GUIs or for passing in comparator functions to sorting algorithms.
We can return anonymous and class objects from functions in Java, but we never do. In Python we can do this with more utility. For example, here's a function factory that returns functions computing linear curves where 'a' is the slope and 'b' is the offset:

```python
def linear(a, b):
    def result(x):         # nested function returned
        return a*x + b
    return result
angle45_at_origin = linear(1,0)
print angle45_at_origin(0)
0
>>> print angle45_at_origin(1)
1
>>> print angle45_at_origin(2)
2
>>> f = linear(2,-1)
>>> [f(x) for x in range(0,4)]
[-1, 1, 3, 5]
```

When a function captures some arguments in another function and returns it, we call it currying. Basically, we are taking a function of multiple arguments and breaking it down into functions with fewer arguments. Note that we can fix values in the function using parameters from another function as we've done here in linear(). From the Wikipedia page: Given function f(x,y) = y/x, we can create a new function g(y) = 2/x which is just like calling f(2,y). So g(y)=f(2,y)=y/2. g(y) is a function we can return from f. (Some languages only allow functions with single arguments and so they use currying to create functions with multiple arguments.) In Python, it looks like this

```python
def f(x):
     def g(y):
             return float(y)/x
     return g 
>>> f(2)
<function g at 0x100465de8>
>>> g = f(2)
>>> g(8)
4.0
>>> g(2)
1.0
```

This kind of stuff is great for creating new functionality by composing other functions. For example, imagine that we have a general sort routine that takes a comparator function. We could create another function that return of function with a reverse sort function already put in it. Imagine that we have built-in `sort()` and `reverse_cmp()` functions. We get tired of having to combine a manually, which is a bit of boilerplate code. Let's create a function that gives us a handy shortcut function that combines sorted and `reverse_cmp`:

```python
def reverse_cmp(x,y):
    return y-x
def reverse_sorter():
    def sorter(data):
        return sorted(data, reverse_cmp)
    return sorter
rsort = reverse_sorter()
print rsort([1,2,3])
[3, 2, 1]
```

The key is that we're not statically defining new methods. They get created as needed on the fly.

### Function composition

Because Python has higher-order functions, we can compose them to make more complicated functions like we did informally with the function objects in the previous section. Composition just means applying a function to the result of calling another function.  Given functions f and g, composing the two means calling `f(g(x))` or in sequence `y = g(x)` then `z = f(y). z=f(g(x))`. Note that because of the nesting, we actually compute the 2nd function first. we often write in functional stuff new composed function fg as:

`fg` = `f` • `g`

I had to manually install the functional package:

```bash
$ pip install functional
```

But then I can use the compose() method.

```python
from functional import compose
def dub(x):
    return 2*x
times8 = compose(dub, compose(dub, dub))
>>> times8(1)
8
>>> times8(2)
16
>>> times8(10)
80
```

From [Python functional doc](http://docs.python.org/release/2.7.2/howto/functional.html):

<blockquote>
`compose(outer, inner, unpack=False)`

The compose() function implements function composition. In other words, it returns a wrapper around the outerand inner callables, such that the return value from inner is fed directly to outer.
</blockquote>

They give this example:

```python
def add(a, b):
... return a + b
...
>>> def double(a):
... return 2 * a
...
>>> compose(double, add)(5, 6)
22
```

Also note that sequence `f(); g(;` is the same as `compose(g, f)` because that's `g(f())`, which would call `f` first.  Weird but correct.

### Computing prime number example

Copied more or less from [Byrtek slides](http://www.slideshare.net/adambyrtek/functional-programming-with-python-516744) with tweaks by me. Here is the imperative version:

```python
def is_prime(n):
    k = 2
    while k<n:
        if n%k == 0: return False
        k += 1
    return True
```

With `filter()`, we ask whether the list of non-1 divisors is empty.

```python
def is_prime(n):
    def divisible(x):
        return n%x==0
    return len(filter(divisible, range(2,n))) == 0

def primes(m):
    return filter(is_prime, range(1,m))

print primes(20) # [1, 2, 3, 5, 7, 11, 13, 17, 19]
```

Using list comprehensions now instead of the built-in operations, we can simplify even more.

```python
def is_prime(n):
    return True not in [n%k==0 for k in range(2,n)]
def primes(m):
    return [n for n in range(1,m) if is_prime(n)]
```

Problem is that the `is_prime()` method does the entire range even if  it could stop early. That means that we can replace the comprehension in `is_prime()` with `(n%k==0 for k in xrange(2,n))`. It will terminate right away for `is_prime(1000000)` even though that is a big number; the previous one would take a while. note that we are using xrange() not `range()` which does not create a list either.

Finally, using `any()`, we get a very English like saying that says a number, n, is prime if there are not any numbers that divide cleanly from 2..n-1.

```python
def is_prime(n):
    return not any(n%k==0 for k in xrange(2,n))
Compute phone number word possibilities
letters=[
    [' '],            # 0
    [' '],            # 1
    ['a','b','c'],    # 2
    ['d','e','f'],    # 3
    ['g','h','i'],    # 4
    ['j','k','l'],
    ['m','n','o'],
    ['p','q','r','s'],
    ['t','u','v'],
    ['w','x','y','z']
]
phone = '1234'
phone_digits = [int(c) for c in phone]
combos = [a+b+c+d
     for a in letters[phone_digits[0]-int('0')]
    for b in letters[phone_digits[1]-int('0')]
    for c in letters[phone_digits[2]-int('0')]
    for d in letters[phone_digits[3]-int('0')]
]
 
-> [' adg', ' adh', ' adi', ' aeg', ' aeh', ' aei', ' afg', ' afh',
 ' afi', ' bdg', ' bdh', ' bdi', ' beg', ' beh', ' bei', ' bfg', 
' bfh', ' bfi', ' cdg', ' cdh', ' cdi', ' ceg', ' ceh', ' cei', 
' cfg', ' cfh', ' cfi']
```

## A world without statements

Resources:
http://www.ibm.com/developerworks/linux/library/l-prog/index.html

How are we going to avoid using statements like if-then-else or even sequences of statements?  let's start with looping.

### Walk a list recursively

```python
def walk(a):
    if len(a)==0: return
    x = a[0]
    print x
    walk(a[1:])

walk([1,2,3])
# gives:
1
2
3
```

### Replace if-then-else

```python
if cond1: f1() -> cond1 and f1()
if cond1: f1() else: f2() -> (cond1 and f1()) or f2()  # uses short-circuiting of expressions
```

### Loops

```python
for x in seq: f(x) -> map(f, seq)
for x in seq: f(x) -> (f(x) for x in seq)
```

Find all elements of outer product of 2 vectors == 0

```python
A=[1,2,3]
B=[0,1,5]
zeroes=[]
for x in A:
    for y in B:
        if x*y==0:
            zeroes.append((x,y))
print zeroes # [(1, 0), (2, 0), (3, 0)]
print [(x,y) for x in A for y in B if x*y==0]
```