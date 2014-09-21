## Template Metaprogramming with Modern C++: Templates in depth

[The last time]() we learnt what metaprogramming is, how metaprogramming in C++ via templates works, and the functional spirit of the embbeded language that C++ template metaprogramming is.

In this post we will learn C++ templates in depth: Class and function templates, template parameters, variadic templates, and then some useful template idioms like SFINAE.

## The template system: Function and class templates

As we seen in the first post, the C++ template system generates C++ types and functions from an specification written by the programmer, what we call *template*. And of course generating a type means generating code too, since normally C++ types are coupled to code (Thats the concept of a class). 

Lets see how those templates work. First a simple template function:

``` cpp
template<typename T>
T identity( const T& e )
{
	return e;
}
```

This simple template declares a family of functions that take a value of any type `T` and returns it untouched. Its the runtime version of the metafunction we seen in the first post.  
When the programmer uses that template:

``` cpp
int i = identity(0);
```

the compiler instantiates the template using the correct parameters, an `int` type parameter in this case. 
*Note how template parameters are inferred from the function argumments passed to the template function. This is wy when using function templates is not neccessary nor a good practice to pass template parameters explicitly. Only is needed in some cases when a parameter could not be inferred from the function argumments, see [`std::make_shared()`](http://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared) for an example.*

Exactly the same occurs for class templates: The compiler generates one type (class) and its corresponding code for each combination of template parameters.

There is one point to be noted: Its true that the compiler generates one instantation for each combination of parameters, but **modern C++ compilers are smart enough to not generate executable code for templates that are not actually used in the program.** Also, modern compilers perform memoization for template instantation, which increases the performance of the template system. Both optimizations make invalid the old argumment saying that C++ templates increase executable size. **Thats not completely true, since the compiler only generates code for the things that are actually used**, after optimizations like inlining, etc.

See for example the classic fibonnacci metafunction:

``` cpp
template<int n>
struct fibonacci
{
	static constexpr int value = fibonacci<n-1>::value + fibonacci<n-2>::value;
};

template<>
struct fibonacc<0>
{
	static constexpr value = 0;
};

template<>
struct fibonacci<1>
{
	static conexpr value = 1;
};
```
