why python is bullshit:

you know that adage about 100 operations on 1 data structure or whatever? python has no _de facto_ data structures. or rather, it does, but they're abstract, and accessible through terribly verbose syntax: namely the `for` loop to iterate through dicts and sequences, both of which are generators. and somewhere in there there are sets, which support overloaded arithmetic operators.... anyway, here's a real-world scenario of me trying to use the link:https://mutagen.readthedocs.io/en/latest/api/base.html[`mutagen` library] to get tag info from a music file:

[source,python]
----------------------------------------------------------------------------------
import mutagen
x = mutagen.File("04 Hell March.mp3") // track from the game, "command & conquer", btw
----------------------------------------------------------------------------------

so far it's fine, but it's about to sour. the docs say that `x`'s type is `FileType`. fine; i look at its attributes. i see `tags`. perfect! i'd expect tags to be a dict from tag to value. instead, its type is `Tags`. i begin to wonder how the value could be complex enough to deserve its own type and begin worrying about how complex will be the code that works with the data. so i look at the `Tags` definition; there's practically no documentation: it just shows `pprint()` and says, "Tags is the base class for many of the tag objects in Mutagen. In many cases it has a dict like interface." ..._in many cases_? i remain wondering what possible other cases there could be, but it seems that i'll know that only if i come across them. it's an abstract type, which means that at best, i can use abstract forms to work with it. as it turns-out, for this given mp3 file, it _does_ support a dict interface. oh, wait—"dict_-like_"...?? wtf is that? how much or little is it _like_ a dict? let's find-out.

firstly, as the docs' example shows, simply putting `mutagen.File("../04 Hell March.mp3")` prints

-------------------------------
{'TALB': TALB(encoding=<Encoding.LATIN1: 0>, text=['Command & Conquer: Red Alert COMPLETE']), 'TPE2': TPE2(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TPE1': TPE1(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TCOM': TCOM(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TPOS': TPOS(encoding=<Encoding.LATIN1: 0>, text=['1']), 'TXXX:REPLAYGAIN_TRACK_PEAK': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_TRACK_PEAK', text=['0.958618']), 'TXXX:ENSEMBLE': TXXX(encoding=<Encoding.LATIN1: 0>, desc='ENSEMBLE', text=['Frank Klepacki']), 'TCON': TCON(encoding=<Encoding.LATIN1: 0>, text=['Game']), 'TXXX:GROUPING': TXXX(encoding=<Encoding.LATIN1: 0>, desc='GROUPING', text=['PC']), 'TXXX:RIPPER': TXXX(encoding=<Encoding.LATIN1: 0>, desc='RIPPER', text=['1337haXXor']), 'TIT2': TIT2(encoding=<Encoding.LATIN1: 0>, text=['Hell March']), 'TRCK': TRCK(encoding=<Encoding.LATIN1: 0>, text=['4']), 'TXXX:REPLAYGAIN_ALBUM_GAIN': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_ALBUM_GAIN', text=['-3.11 dB']), 'TXXX:REPLAYGAIN_TRACK_GAIN': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_TRACK_GAIN', text=['-6.05 dB']), 'TSSE': TSSE(encoding=<Encoding.LATIN1: 0>, text=['Lavf56.40.101']), 'TDRC': TDRC(encoding=<Encoding.LATIN1: 0>, text=['1996'])}
-------------------------------

which indeed looks like a dict. `mutagen.File("../04 Hell March.mp3").tags` prints the same. wtf is that? if it's the same, why make the `tags` attribute public? ...ahhhh, _unless_ it just _prints_ the same, but _isn't_ the same—you know, like when you want to know _exactly_ what something is, because you're _programming_ and you need to manipulate data precisely, rather than just being given an conceptual description of it? waaaaiiiitttt. but _that_ would imply that we'd want the printout of the object on the command line to actually show exactly how the data are stored insofar as we can access & update them! you know, seeing as those are, along with control flow, arithmetic, and removal, the _only operations that ever occur in a program_. anyway, i am just guessing, since i haven't read the source code to see the relationship between the `mutagen` class and its `tags` attribute (because i have better things to do with my life).

i try `for k,v in mutagen.File("../04 Hell March.mp3").tags: print(k,' :: ',v)` but it fails with the message, "too many values to unpack." i check these python notes and see that i need to put `.items()` afterward. ...now, in a language where anyone would consider making `.items` optional, i'd expect that the language itself would make `.items()` optional in the `for` syntax, especially since the `for` syntax is the only way to iterate through a dict. anyway, i add it in, and huzzah, it works: `for k,v in mutagen.File("../04 Hell March.mp3").tags.items(): print(k,' :: ',v)`. i can omit `.tags` and i get the same result, btw.

for fun, let's do a little test: can you tell which of these is `mutagen.File("../04 Hell March.mp3").tags.items()`, vs `mutagen.File("../04 Hell March.mp3").tags`, vs `mutagen.File("../04 Hell March.mp3").tags`?

.candidate 1
--------------------------------------------------
<bound method DictMixin.items of {'TALB': TALB(encoding=<Encoding.LATIN1: 0>, text=['Command & Conquer: Red Alert COMPLETE']), 'TPE2': TPE2(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TPE1': TPE1(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TCOM': TCOM(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TPOS': TPOS(encoding=<Encoding.LATIN1: 0>, text=['1']), 'TXXX:REPLAYGAIN_TRACK_PEAK': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_TRACK_PEAK', text=['0.958618']), 'TXXX:ENSEMBLE': TXXX(encoding=<Encoding.LATIN1: 0>, desc='ENSEMBLE', text=['Frank Klepacki']), 'TCON': TCON(encoding=<Encoding.LATIN1: 0>, text=['Game']), 'TXXX:GROUPING': TXXX(encoding=<Encoding.LATIN1: 0>, desc='GROUPING', text=['PC']), 'TXXX:RIPPER': TXXX(encoding=<Encoding.LATIN1: 0>, desc='RIPPER', text=['1337haXXor']), 'TIT2': TIT2(encoding=<Encoding.LATIN1: 0>, text=['Hell March']), 'TRCK': TRCK(encoding=<Encoding.LATIN1: 0>, text=['4']), 'TXXX:REPLAYGAIN_ALBUM_GAIN': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_ALBUM_GAIN', text=['-3.11 dB']), 'TXXX:REPLAYGAIN_TRACK_GAIN': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_TRACK_GAIN', text=['-6.05 dB']), 'TSSE': TSSE(encoding=<Encoding.LATIN1: 0>, text=['Lavf56.40.101']), 'TDRC': TDRC(encoding=<Encoding.LATIN1: 0>, text=['1996'])}>
--------------------------------------------------

.candidate 2
--------------------------------------------------
{'TALB': TALB(encoding=<Encoding.LATIN1: 0>, text=['Command & Conquer: Red Alert COMPLETE']), 'TPE2': TPE2(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TPE1': TPE1(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TCOM': TCOM(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki']), 'TPOS': TPOS(encoding=<Encoding.LATIN1: 0>, text=['1']), 'TXXX:REPLAYGAIN_TRACK_PEAK': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_TRACK_PEAK', text=['0.958618']), 'TXXX:ENSEMBLE': TXXX(encoding=<Encoding.LATIN1: 0>, desc='ENSEMBLE', text=['Frank Klepacki']), 'TCON': TCON(encoding=<Encoding.LATIN1: 0>, text=['Game']), 'TXXX:GROUPING': TXXX(encoding=<Encoding.LATIN1: 0>, desc='GROUPING', text=['PC']), 'TXXX:RIPPER': TXXX(encoding=<Encoding.LATIN1: 0>, desc='RIPPER', text=['1337haXXor']), 'TIT2': TIT2(encoding=<Encoding.LATIN1: 0>, text=['Hell March']), 'TRCK': TRCK(encoding=<Encoding.LATIN1: 0>, text=['4']), 'TXXX:REPLAYGAIN_ALBUM_GAIN': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_ALBUM_GAIN', text=['-3.11 dB']), 'TXXX:REPLAYGAIN_TRACK_GAIN': TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_TRACK_GAIN', text=['-6.05 dB']), 'TSSE': TSSE(encoding=<Encoding.LATIN1: 0>, text=['Lavf56.40.101']), 'TDRC': TDRC(encoding=<Encoding.LATIN1: 0>, text=['1996'])}
--------------------------------------------------

.candidate 3
--------------------------------------------------
[('TALB', TALB(encoding=<Encoding.LATIN1: 0>, text=['Command & Conquer: Red Alert COMPLETE'])), ('TPE2', TPE2(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki'])), ('TPE1', TPE1(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki'])), ('TCOM', TCOM(encoding=<Encoding.LATIN1: 0>, text=['Frank Klepacki'])), ('TPOS', TPOS(encoding=<Encoding.LATIN1: 0>, text=['1'])), ('TXXX:REPLAYGAIN_TRACK_PEAK', TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_TRACK_PEAK', text=['0.958618'])), ('TXXX:ENSEMBLE', TXXX(encoding=<Encoding.LATIN1: 0>, desc='ENSEMBLE', text=['Frank Klepacki'])), ('TCON', TCON(encoding=<Encoding.LATIN1: 0>, text=['Game'])), ('TXXX:GROUPING', TXXX(encoding=<Encoding.LATIN1: 0>, desc='GROUPING', text=['PC'])), ('TXXX:RIPPER', TXXX(encoding=<Encoding.LATIN1: 0>, desc='RIPPER', text=['1337haXXor'])), ('TIT2', TIT2(encoding=<Encoding.LATIN1: 0>, text=['Hell March'])), ('TRCK', TRCK(encoding=<Encoding.LATIN1: 0>, text=['4'])), ('TXXX:REPLAYGAIN_ALBUM_GAIN', TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_ALBUM_GAIN', text=['-3.11 dB'])), ('TXXX:REPLAYGAIN_TRACK_GAIN', TXXX(encoding=<Encoding.LATIN1: 0>, desc='REPLAYGAIN_TRACK_GAIN', text=['-6.05 dB'])), ('TSSE', TSSE(encoding=<Encoding.LATIN1: 0>, text=['Lavf56.40.101'])), ('TDRC', TDRC(encoding=<Encoding.LATIN1: 0>, text=['1996']))]
--------------------------------------------------

the answers are: 

. `tags.items`
. `tags`
. `tags.items()`

what's the point of having the dict type builtin if:

. its interface sucks
. people use _dict-like objects_ instead

now, let's do a similar test: can you tell which of the following is an alist, an alist, or an alist?

[source,factor]
----------------------------------------------------
{ { "TALB" "Command & Conquer: Red Alert COMPLETE" }
  { "TPE2" "Frank Klepacki" }
  { "TPE1" "Frank Klepacki" }
  { "TCOM" "Frank Klepacki" }
  { "TPOS" "1" }
  { "TXXX:REPLAYGAIN_TRACK_PEAK" "0.958618" }
  { "TXXX:ENSEMBLE" "Frank Klepacki" }
  { "TCON" "Game" }
  { "TXXX:GROUPING" "PC" }
  { "TXXX:RIPPER" "1337haXXor" }
  { "TIT2" "Hell March" }
  { "TRCK" "4" }
  { "TXXX:REPLAYGAIN_ALBUM_GAIN" "-3.11 dB" }
  { "TXXX:REPLAYGAIN_TRACK_GAIN" "-6.05 dB" }
  { "TSSE" "Lavf56.40.101" }
  { "TDRC" "1996" } }
----------------------------------------------------

there's only one data set, so the answer should be obvious. btw, the object is displayed by a syntax that is simultaneously both its prettyprint and its literal source code, and were it not for today's python debacle, i'd never imagine that anyone would try to display it any other way.

## modules & env

* `import sys` then use `sys.argv`. `argv[0]` is the program name, like in C.
* entry point is file; interpreted and executed line-by-line like lua or bash
* imports
    * qualified: `for file.py, import file`
    * unqualified/static: `from file import (def1, def2, ... | *)`. don't use `*`.
* use `if __name__ == "__main__":` for modules that may be either imported or run
* path env is `PYTHONPATH`. of course, searches `CWD` and `PATH` too
* technically this is included in sys.path, which can be reflectively modified
* packages: package.subpackage <-> package/subpackage. subpackage [directory] should contain `__init__.py` so that python knows that that's a subpackage directory
* non-`__main__` modules can use relative import statements: `from ..? import ...`
* to import `file1.py` (in path,) do `import file1`
* importing a file is like bash, lua, or C's `#include`: it runs the code
    * what's `reload()` do?

## lang

* `int() : int -> string`
* strongly, dynamic, inferred duck typing. <https://docs.python.org/3/library/typing.html>
* like in Java, transative variable assignments are by reference:value::mutable:immutable atoms
* `is` is pointer equality; `==` is value equality
* `string` is a species of `sequence`
* there is no `char` type---just unary strings. quote and double-quote both mean the same thing
* terminology: object === atom
* core data types: number, string, list, dictionary, tuple, file, set, bool, type
* dicts, sets, and lists are mutable; primitives are not.
* has null, called `None`
* all objects have a truthiness measure

## funcs

* user-defined functions have automatically-defined (.)-accessible [attrs](https://docs.python.org/3/reference/datamodel.html) (like `Function` prototypes in JavaScript.) many are writable.
* param passing and modification like Java
* like lua, func params may be specified in any order, e.g. `foo(baz="c", param1=0)`
* params may be given default values

tuple-dict multi-param funcs:

    def func(param1, param2, ..., *tupleVar, **dictVar):

and are called like

    func(..., tupleVal1, tupleVal2, ..., k1=v1, k2=v2, ...)

a non-dict variadic function is defined as same but without the **dictVar param. the asterisk param is still stored and addressed as a tuple object.

## [typing](https://docs.python.org/3/library/typing.html)

python 3.6 added optional types:

    def func(param1: type, param2: type, ..., *tupleVar, **dictVar) -> return_type: #tupleVar and dictVar are obviously already bound to their types

or `x: int = 3` just in usual imperative program context. You must `import typing`.

where type ::= str | int | Dict[type, type] | List[type] | Tuple[type, type, ...]

type alias: `Name = str`

### using typing

You'll need to use [mypy](http://mypy-lang.org/) and/or PyCharm IDE; the python interpreter does not provide typechecking itself.

## general syntax

* comments: #(like bash)
bools True and False are ONLY aliases for 1 and 0 respectively, therefore e.g. True + 2 -> 3
Lua-like ops: not x, x and y, x or y, x if y else z
DOES NOT ALSO SUPPORT &&, ||, nor !
COMPARISON OPERATORS MAY BE CHAINED! e.g. x < y < z TwT #crying real man tears
branch and conditional statements are end-delimited by ':'
uses elif
ternary: a if c else b (c ? a : b)
multiple assignment like Lua (e.g. x, y = y, x)
backquotes on a var (`var`) <=> toString(var)

## semantics

### [scoping](https://docs.python.org/3/tutorial/classes.html#scopes-and-namespaces-example)

* `global` is absolutely outermost scope. local (used when no modifier is given about a symbol) is the innermost scope; `nonlocal` is a scope one level out.

* keyword `nonlocal` prefixes variables that will be included in nested functions' lexical closures; e.g.

    def f():
        x = 1
        def g():
            nonlocal x #will use f's x
        x = 2

* variables in closures by default have local scope
* if not `global`
List comprehensions: [ f(E) for E in src if cond ] <-> [ f(E) | E <- src, cond ]
can do set comprehensions too, with {}
dict comprehension: { key(x) : val(x) for x in f(x) }
NOTE: generator/lazy comprehensions use () not []; e.g. (i for i in primes() if i < 300) is lazy, but [ i for ... ] is strict!
docstrings: an unbound triple-quoted (conventionally """) string literal that's the first line inside a module, class, or function definition is automatically bound to __doc__
lists and dicts may be unpacked via * and ** respectively; this is useful for passing tuples and dicts as params to variadic and tuple-dict functions.
lambdas: \x y -> f x y <-> lambda x, y: f(x)
"raise Exception"
keyword "with": like Haskell's bracket function. syntax: with expr [as var]: ...
  - basicaly a different way of exception handling
  - where var is an optional variable to store result of expr in
expr may be a class:
class expr: #wraps an object o
    def __init__(s, o):
        s.o = o
    def __enter__(s): #2. called after __exit__ is saved
        #save or validate o's state
        return s.o #passed to the "as" clause
    def __exit__(s, type, val, traceback): #1. with stores this in a hidden variable behind-the-scenes. Params are as in sys.exc_info. For handling exceptions.
        #do w/e you need to clean-up or smth. Like Java finally
or, more simply, as a generator: #from contextlib import contextmanager
@contextmanager
def expr(o): #what's the point of this if it doesn't catch the exception anyway?!
    enter(o)
    try:
        yield o
    finally:
        exit(o)
so clearly with statements are only good for modularity; for one-time'er's, just use an ordinary try-except block

## control structures

`else:` predicate following any loop (for, while) is followed in case of the the loop condition returning false; `else` bodies are not executed if a `break` statement prematurely exits the loop.
`for expr in sequence:`
mimic Lua loop by `for v in range(begin, end, step):`
like Java ArrayLists, `do cpy=src[:]; for i in cpy: ...; src=cpy;` #to avoid side effects to the list if its value is changed from inside the loop
python has not the for (init; cond; post-op)
generators are functions that have a yield statement; yield returns the next value, and does not have to be the last function call from within the generator function. an empty return statement tells the calling context that the loop is done

## classes & oop

* `del` for unset
* classes may be defined in any scope
* *class object* refers to the class as an untyped object (effectively a dict). `String` would be a class object in Java.
* like C++ classes with public attributes and virtual (abstract, or optionally implemented in class definition) member functions
* like Lua, self is always first parameter to instance methods (obviously not to class (static) methods, though.) `self` is only a naming convention; the object invocing the function is always the first argument.
* as a sanity check, never should you say `self.attr` if `attr` is a class attribute. python will (obviously) not check for this (since it doesn't know what "check" means,) so, as always, be careful.
* the `__class__` instance attribute returns the class/type of the object
* variables can be created on-demand for any instance. attributes may be deleted via the del keyword. instances may stray from their class definitions, like JS, not Java
* C#-style attribute accessing

    class Foo(SuperClass):
        __init__(self,...): ... #constructor
        __del__(self,...): ... #destructor
        __str__(self): ... #show/toString

* instantiate like JS: `x = ClassName()` or `ClassName('hello', 3)` if `ClassName.__init__` is defined to take such parameters.
* we may define functions that set instance-particular attributes: `def f(self, x): self.a = x`
* python's duck-typed, so no need for explicitly polymorphic class definitions
* multiple inheritence allowed
* a method of a base class may end-up calling an overriden same-signature method of the derived class
* obj.isinstance() and Class.issubclass()
* obj.__class__
* underscore prefix conventionally means that the attribute is internal and subject to change w/o notice
* class attrs beginning with `__` are mangled internally so that functions defined in terms of them do not change if it's overridden in a derived class. they must still be defined:

    class Mapping:
        def __init__(self, iterable):
            self.items_list = []
            self.__update(iterable)

        def update(self, iterable):
            for item in iterable:
                self.items_list.append(item)

        __update = update   # private copy of original update() method

    class MappingSubclass(Mapping):

        def update(self, keys, values):
            # provides new signature for update()
            # but does not break __init__()
            for item in zip(keys, values):
                self.items_list.append(item)

* empty class (just gives type/sets `__class__`: `class Name: pass`

## strings

3 types: `str` for `String`/`Text`, `bytes` for binary, and `bytearray` for mutable bytes

string literal prefixes:

* `r` : don't parse escape sequences (note that raw strings still cannot have their last character a backslash)
* `b` : binary sequence literal

list(s) -> [s[0], s[1],...], effectively a mutable string
* `join` : list -> string, i.e. monadic `join` for `[String]`

## files

* pathnames are portable; forward slashes work on windows

## exceptions

    try:
      ...
    except Exception as e:
      ...
    else: #executed if no exception is thrown in try block
      ...
    finally:
      ...

## sequences

* negative indicies: `"span"[-2]` --> `'a'` `"span"[-1]` --> `'n'`
* slice/subsequence: `set[i:j]`: elements i through j-1
    * lower bound omitted: i = 0
    * upper bound omitted: j = #s
    * negative upper bound: `s[:-2] == s[0:#s-2]`
    * `s[:] == s`
* concat is `+`
* repitition is `*`
* immutable strings
* third param of slice is step, like lua. negative reverses return order
* `format` : String -> (...) -> String, like printf:
    * py2 style: `"%s %i..." % ("str", 4)`
    * py3 style: `"{} {}...".format("str", 4)`
* py3 style inferrs types; {n} may be used to explicitly specify index of tuple rather than order of "anonymous" {}'s declared

## lists, tuples, dicts

* dicts: `d = {'k':v, ...}` or `dict(k=v, k2=v2,...)`. notice that the keys are not quoted
    * or `d['k'] = v; d['k2'] = v2; ...`
* remove from list by index: `del list[index]`, or more generally a slice: `del list[2:4]`. `del list` to delete the whole thing.
* remove from list by value: `mylist.remove(value)` (deletes the first element whose value equals the provided one). if no such element, raises a `ValueError`
* `for k, v in dict.items()`

## sets

* `(-)`: difference
* `(+)`: concatenation
* `(|)`: union
* `(^)`: symmetric difference
* `(&)`: intersection
* `(<)`, `(>)`: subset, superset
* `x in y, x not in y`: membership (works for iterables in general)
* `x is y, x is not y`: identity tests
* use `Decimal` class for fixed-digit precision, e.g. `.1 + .2 - .3 == 0.0`, not `5e-21` or smth
    * what?
* python's `Fraction` type is like Haskell's `Rational`
* `{}` is an empty dict, not empty set. use `set()` for empty set
* all set members must be immutable

## iterables

* allows `for` loop over
    * `for` calls `iter()`
* define the `__iter__` method in a class. `__iter__` returns an object that has a `__next__` method. `self` usually sufficies, as classes often implement `__next__` specifically for `__iter__`. i actually can't imagine why this wouldn't *always* be the case, but w/e.
* `__next__` either returns a value or `raise`s a `StopIteration`

it's easier to use generators: their `__next__` and `__iter__` methods are created simply by using `yield`, and their index persists through invocations.

## example generator expressions

    unique_words = set(word for line in page for word in line.split())
    valedictorian = max((student.gpa, student.name) for student in graduates)
    sum(x*y for x,y in zip(xvec, yvec))


## numbers

* complex (e.g. `8-2j`), `0o3614` (octal) `0x28af`, `0b10010110`, decimals, integrals
* `1.`, `1.20`, `.8e-2`
* funcs `bin, `hex`, and `int` convert between radixes
* `(//)` for floor (rounds-down for negatives) division. in 2.6, (/) truncates iff LHS & RHS are integers

## builtins

* rc #regex
* `ord` and `chr` just like in haskell
* help(method) #like help in Bash. operates by __doc__ attrs
* dir(module) #which names a module defines
* dir() #currently loaded modules
* dir(builtins) #import builtins first
* Functional paradigm: #remember that generators are lazy eval!
* map(f, iter1, iter2, ...) -- only use if you have multiple iterators and aren't using itertools.starmap; usually just use generator expressions; you'll be much happier
* filter(predicate, iter)
* enumerate :: [a] -> Generator (Int, a)
* any :: [a] -> Bool
* all :: [a] -> Bool
* zip :: [a] -> [b] -> [(a, b)]
* itertools.count(arg=inf) = [1..arg]
* itertools.cycle([a]) = a !! 0, a !! 1, ... a !! length a, a !! 0, ...
* itertools.repeat(elem, [n]) = [elem, elem, ...] n times
* itertools.takewhile(predicate, iterable)
* itertools.dropwhile(predicate, iterable)
* functools.partial(f, args, kwarg1=v1, kwarg2=v2)
* functools.reduce(func, iterable, [init value]) #Haskell's foldl
* functools.accumulate #same as reduce(), but is a generator that returns each fold 
* use ''.join(seq) instead of sum([str1, str2,...])

## regex char classes

* `\d`: digit
* `\D`: non-digit
* `\s`: whitespace characters
* `\S`: non-whitespace
* `\w`: alnum (`[a-zA-Z0-9_]`)
* `\W`: ^alnum

Compilation flags:

* `S`: DOTALL - makes '.' match anything including newline
* `I`: ignore case
