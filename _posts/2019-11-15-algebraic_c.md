---
title: Sum algebraic data types in C
tags: c haskell functional
---

Algebraic data types (ADTs) are the most basic types in Haskell and in many functional programming languages, they are
so used that even some imperative (and multi-paradigm) programming languages have them (e.g. C++, Swift, Rust, ...). 
They come from _type theory_ (where they correspond to an [_initial algebra_](https://en.wikipedia.org/wiki/Initial_algebra))
and from _domain theory_. They are called "algebraic" becase ADTs are created using algebraic operations, these 
operations are:

<!--more-->

- $$+$$ (sum): represents sum types or disjoint unions, for example 
```haskell
data Bool = True | False
```

- $$\cdot$$ (product): represents product types such as tuples or structs. They are analogous to the cartesian product
  for sets.
  Example:
```haskell
data Pair = P Int Doubl
```

- $$X$$ (singleton): represents a singleton type like
```haskell
data X a = X a
```

- $$1$$ (unit): the unit type `()`;

- $$\mu$$: the least-fixed point operator, used for building recursive types.

These operators are also used in domain and type theory to express the denotational semantics of programming languages, 
for more details see [1].

An ADT is therefore defined by two pieces of data: 

1. a name for the type used to represents its values;
2. a set of constructors (and destructors) used to create new values (and for pattern matching). These constructors
may have arguments that hold values of the given type.

For example let's consider the [Peano numbers](https://wiki.haskell.org/Peano_numbers) defined as
```haskell
data Peano = Zero | Succ Peano
```
where `Peano` is the name of the ADT, `Zero` and `Succ` are value constructors. The destructor of the sum data type is
the `case` construct.

Since the majority of programming languages have product types already, in the following we will focus on sum types.

## Sum types
As we've already seen, sum types denote some kind of alternation or choice (`A | B`, meaning `A` or `B` but not both).
They are particularly useful when paired with pattern matching because they provide a great level of type safety, they
have an efficient implementation, and they provide programmers the tools to describe different possibilities.

One can say that sum types feel a lot like a combination of unions and enumerations, and that's true, in fact we'll 
see how to implement various (sum) algebraic data types in C.

### Example: Peano numbers
As a first example we'll implement Peano numbers in C and a function to add two (Peano) numbers. As we've seen in the
example above, Peano numbers are a way to represent natural numbers using only the value zero and a successor 
function, for example 4 is represented as `S S S S 0` where `S` stands for Successor.

To represent the (value) constructors we can use an enum
```c
enum peano_const {ZERO, SUCC};
```
this enumeration will be used for building peano numbers and for pattern matching.

To actually represent the data type we could use the following struct:
```c
struct peano {
        enum peano_const type;
        union {
                struct peano *data;
        };
};
```
Where `*data` will be used to point to the preceding peano number, for example if the number is `S S S 0`, `data` will
point to `S S 0`.  
Note that anonymous unions (and structs) are valid only from  C11 or when using GNU extensions.

Since only the Succ constructor holds a value we can move the struct outside the union:
```c
struct peano {
        enum peano_const type;
        struct peano *data;
};
```

To implement the data constructor we will define two functions, one for each of them:

```c
struct peano *
make_zero(struct peano *num)
{
        if (num) {
                num->type = ZERO;
                num->data = 0;
        }

        return num;
}
```
and
```c
struct peano *
succ(struct peano *num)
{
        if (num) {
                struct peano *res = malloc(sizeof(struct peano));

                if (res) {
                        res->type = SUCC;
                        res->data = num;
                        num = res;
                }
        }

        return num;
}
```
The first just initializes a peano number as zero and sets the pointer to the data to 0 or `NULL`, while the second
allocates space for a new number and inserts it at the beginning, marking it as a `SUCC` and adjusts the pointers.

Next we will see how to add two numbers, to do that we will use a recursive function that emulates pattern matching: if
the first argument $$n$$ is `Zero` we can return the second argument. However, if the first argument is a `Succ` of another 
number, let's say $$a$$, we will call `add` recursively on $$a$$ and on the second argument, let's call it $$m$$ and 
increment by one the result. Effectively we are decreasing the first argument by one on each recursive call and 
increasing the result by one, at the end we will reach the base case ($$a$$ is `Zero`) and we just have to walk back the
stack of recursive calls and compute the successor at each step. At the end we would have called the successor function
$$n$$ time on the number $$m$$, thus adding $$n$$ to $$m$$.
```c
struct peano *
add(struct peano *n, struct peano *m)
{
        /* Emulates pattern matching */
        switch (n->type) {
        case ZERO:
                return m;
        case SUCC:
                return succ(add(n->data, m));
        default:
                fprintf(stderr, "Non-exhaustive patterns in function %s\n", 
                    __func__);
                return 0;
        }
}
```

### Example: lazy list
As our last example we'll look at something more involved: lazy lists
```haskell
data List = Empty | Cons Int List
```
This type is a bit more complex because it combines sum types with product types (see the `Cons` constructor). In type
theory this will correspond to $$\mu\beta.1 + Int \times \beta$$, where $$\mu\beta$$ means that this is a recursive
type called $$\beta$$, and $$1$$ is the unit type used to indicate `Empty`, while $$Int \times \beta$$ is the `Cons`
constructor.

In our encoding this will translate to:
```c
/* Data constructor names */
enum list_const {EMPTY, CONS};

/* Actual sum data type */
struct list {
        enum list_const type;
        struct data {
            int num;
            struct list *next;
        } data;
};
```
Where the inner struct represents the product type $$Int \times \beta$$. The implementation of the two constuctors is
similar to the ones in the previous example:
```c
/* Empty list constructor */
struct list *
make_empty(struct list *lst)
{
        lst->type = EMPTY;
        lst->data = (struct data) {0};

        return lst;
}

/* Cons constructor */
struct list *
cons(int a, struct list *lst)
{
        if (lst) {
                struct list *new = malloc(sizeof(struct list));

                if (new) {
                        new->type = CONS;
                        new->data.num = a;
                        new->data.next = lst;
                        lst = new;
                }
        }

        return lst;
}
```

Computing the length of the list is quite easy, we just have to traverse it until we reach the "empty" element and sum
1 at each call. We could have made it using a loop, but this showcases our version of pattern matching:
```c
int
length(struct list *lst)
{
        if (lst) {
                switch (lst->type) {
                case EMPTY:
                        return 0;
                case CONS:
                        /* Could use tail recursion */
                        return 1 + length(lst->data.next);
                default:
                        fprintf(stderr, "Non-exhaustive patterns in function "
                            "%s\n", __func__);
                }
        }

        return -1;
}
```

The complete sources of the examples are available [here](https://git.sr.ht/~spidernet/algebraiC/).

### Caveats
Consider a slightly more complex sum data type in which more that one constructor carries a value, for example
```haskell
data Pair = PI Int | PD Double
```
in our implementation it would correspond to:
```c
/* Data constructor names */
enum pair_const {PI, PD};

/* Actual sum data type */
struct pair {
        enum pair_const type;
        union data {
                int i;
                double d;
        } data;
};
```
This is not safe, nothing prevents us from accessing `.d` even if `type` is `PI`, because the fields of the union are
always accessible. This is what distinguishes our implementation from classic sum types in Haskell for example, in which
they are safe.

## Summary
Sum types are one of my favourite features of Haskell and once you've used them it's difficult to come back
to a language without them, they are a useful tool that provides excellent safety and expressive power. They're so
useful that more and more languages are adopting them and it's even possibile to implement them in C, with some
caveats. As a side note, in C++ they could be implemented using a class with virtual members, by using the visitor
pattern, using variant types or by using a templated class.

### Reading material
If you're interested in reading more about algebraic data types and pattern matching you can read their wikipedia
pages [2,3], the pages in the haskell wiki [4,5] and the haskell tutorial [6].

---
#### References

[1]\: The Formal Semantics of Programming Languages: An Introduction, Glynn Winskel, MIT Press, 1993.   
[2]\: [Wikipedia: algebraic data type](https://en.wikipedia.org/wiki/Algebraic_data_type).  
[3]\: [Wikipedia: pattern matching](https://en.wikipedia.org/wiki/Pattern_matching).  
[4]\: [wiki.haskell: algebraic data type](https://wiki.haskell.org/Algebraic_data_type).  
[5]\: [wikibooks: pattern matching](https://en.wikibooks.org/wiki/Haskell/Pattern_matching).  
[6]\: [haskell tutorial: pattern matching](https://www.haskell.org/tutorial/patterns.html).
