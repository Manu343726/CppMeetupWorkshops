# Template Metaprogramming with Modern C++: Introduction	

> *"Any sufficiently complex C++ code is indistingishable from trolling"*   
> Arthur C. Clarke

## Preface
Template metaprogramming is one of such things that makes C++ that complex, poor known, and sometimes horrible, language. However its power and expresiveness makes it to be one of the  best features of C++.  
Extensible and fully generic C++ libraries aren't possible without template metapogramming. Even the Standard Library implementations hide many template metaprogramming tricks to make standard containers and algorithms that generic, high level, and efficient tools we use everyday.  
The fact that tmp is a powerful tool could be seen in the evolution of the language, which now has features designed to improve metaprogramming, see C++11 `<type_traits>`, C++11 variadic templates, C++14 variable templates, C++14 `std::integer_sequence`, etc.

But C++ template metaprogramming power comes at a high cost: Its really hard to do and understand. The template system was not originally designed to do such things, and thats reflected primarily in the cumbersome syntax involved and the criptical error messages one get when something fails. That are the reasons why people is usually scared by tmp, and most of us doesn't even try to use it. 

These posts try to introduce template metaprogramming to the average C++ programmer, showing how it works, what can do, and finally leading with its problems trying to make it easier than in the old days of C++98, thanks to C++11 and C++14 language improvements.

---
## But, what is *metaprogramming*?

From Wikipedia:

> Metaprogramming is the writing of computer programs that write or manipulate other programs (or themselves) as their data, or that do part of the work at compile time that would otherwise be done at runtime. 

So instead of writing code that is compiled and does something at runtime (i.e. represents some actions to be done at runtime), we write code (*metacode?*) that generates code. Let me show a simple example:

    #define MIN(x,y) (((x) > (y)) ? (x) : (y))

C parametrized macros could be viewed as metaprogramming functions, *metafunctions*. That is, a function that takes some parameters and generates C code. If you use that macro:

    int main()
    {
    	int a , b , c = MIN(a,b);
    }
 *Please ignore the UB, is just an example.*
 
 The C preprocessor parses that macro, interprets its argumments, and returns the code `(((a) > (b)) ? (a) : (b))`, so the resulting code becomes:
 
     int main()
     {
     	int a , b , c = (((a) < (b)) ? (a) : (b));
     }
 
 Reflection, the ability of some programming languages to inspect type and code information at runtime and modify it, could be another type of metaprogramming.
 ---
## C++ Template Metaprogramming

Template metaprogramming, sometimes shorted to *tmp*, consists in **using the C++ template system to generate C++ types, and C++ code in the process**. 

Consider what a C++ template is: As the name says, **its only a template**. A template function is not a function at all, is **a template to generate functions**, and the same for class templates.  
That wonderful thing all we love, `std::vector`, is not a class. Is a template  designed to generate a correct vector class for each type. When we ***instance*** a template, like `std::vector<int>`, then the compiler generates the code for a vector of ints, following the template the Standard Library developer provided.

So if we write a template `foo` parametrized with a type parameter:

    template<typename T>
    struct foo
    {
    	T elem;
    };

and then that template is instanced:

    typedef foo<int> fooint;
    typedef foo<char> foochar;

what the compiler does is to generate different versions of the `foo` struct, **one for each different combinations of template parameters**:

    struct foo_int
    {
    	int elem;
    };
    
    struct foo_char
    {
    	char elem;
    };
    
    typedef foo_int fooint;
    typedef foo_char foochar;
*Note that the generated classes `foo_int` and `foo_char` are not written in your source file at all, like what the C preprocessor does. The template instantation is managed internally by the compiler. I wrote them in that way to make the example clear.* 

As you can see, the C++ template system actually generates code. We, as C++ *metaprogrammers*, explode this to generate some code automatically.

---

## Metafunctions

In the C preprocessor example, we introduced the concept of *metafunction*. In general a metafunction is a function working in the specific metaprogramming domain we are. In the case of C preprocessor, we manipulate C sourcecode explicitly, so its metafunctions (macros) take and manipulate C source. 

In C++ template metaprogramming we work with types, so a metafunction is a function working with types. 
C++ templates could take non-type parameters too, but its hard to be generic using heterogeneous cathegories of template parameters. Instead, we will work with type parameters only whenever possible. 

    template<typename T>
    struct identity
    {
    	using type = T;
    };

The template `identity` is a metafunction representing the identity function: Takes a value (Actually a type, since we work with types) and returns itself untouched.

We can *"call"* that metafunction referencing its memeber type `type`:

    using t = typename identity<int>::type; // t is int

Also, its a good practice to use a template alias to reference metafunctions, to avoid the `typename ::type` construction:

    template<typename T>
    using identity_t = typename identity<T>::type;

But you should be carefull with that pattern, since template aliases are not templates nor types,  so we can't apply partial specialization on them, among other problems we will discuss later in other posts. What I usually do is to declare the metafunction in an `impl`ementation nested namespace, and the result alias in the main namespace, using the same name for both entities.

Such C++ metafunctions are not a new thing. Even the Standard Library provides many, we usually call them *type traits*:

    using t = typename std::remove_reference<int&>::type; //t is int

As you can see that templates provided since C++11 are, from the tmp point of view,  metafunctions.   
 ***You were doing template metaprogramming without knowing it!***

---

## A Haskell-like language inside C++

Since we work with the C++ type system, using types as values for our computations, tmp works like a functional programming language; because metafunctions have no side effects: **We can only create types, not to modify existing ones**.  
And like in a functional language, one of the pilars of tmp is **recursion**. In this case ***recursive template  instantations*** (Remember that name). 

    template<typename T>
    struct throw_stars
    {
    	using type = T;
    }; 
    
    template<typename T>
    struct throw_stars<T*>
    {
    	using type = typename throw_stars<T>::type;
    };

I think the classic factorial/Fibonnaci metafunctions examples are so boring. Here is something more interesting: The template `throw_stars` is a metafunction that takes a type and throws away all the *"stars"*.

    using t = typename throw_stars<int********>::type; //t is int 

The template specialization acts as the recursive case, and the main template as the base case.   Note how C++ template specialization behaves like pattern matching.

Another example could be traversing of C++11 variadic packs:

    template<typename HEAD , typename... TAIL>
    struct last
    {
    	using type = typename last<TAIL...>::type;
    };
    
    template<typename T>
    struct last<T>
    {
    	using type = T;
    };
    
    using t = typename last<int,char,bool,double>::type; //t is double

which is a great example of a `head:tail` approach for list traversing common in functional languages.

--- 

## Summary

In this first approach to C++ template metaprogramming we have seen that:

 - ***Metaprogramming*** is the process of writting code to generate code, that is, automatice code generation.
 - **C++ template metaprogramming uses the template system to generate types, and code in the process**: We generate types using templates, and we actually use those types to do computations or to generate the desired code.
 - **The basic unit of metaprogramming is the** ***metafunction***, as in common programming the basic unit is the function. Metafunctions manipulate entities of their specific metaprogramming domain. In C++ template metaprogramming, those entities are types, and metafunctions are represented through templates.
 - **Template metaprogramming is like a functional language embbeded into C++ itself**. That  "*language*" has no side effects (We cannot modify an existing type, only create new ones), so we use the same patterns as in a functional programming language such as Haskell or F#.

Now we have a good overview of what C++ template metaprogramming is, but we need some C++ knowledge before getting into it.  
The next time we will learn C++ templates in depth: Template parameters, template specialization, SFINAE, etc; to make sure we all have and understand the neccesary tools to do proper metaprogramming in Modern  C++.




