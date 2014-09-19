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
* Note how template parameters are inferred from the function argumments passed to the template function. This is wy when using function templates is not neccessary nor a good practice to pass template parameters explicitly. Only is needed in some cases when a parameter could not be inferred from the function argumments, see [`std::make_shared()`]() for an example.
