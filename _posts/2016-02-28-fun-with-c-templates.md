---
layout: post
title: "Fun with C++ Templates"
category: programming
---

It's been long enough since my last post that I had forgotten the rake task that generates a new post and had to look it up.

I've been playing around with some C++ recently, to refresh my skills a bit. I haven't done any C++ since my first year of university,
and I liked getting back into it a lot more than I expected. I implemented some basic data structures and used them in some textbook
algorithms to solve problems from [Cracking the coding interview][1]. I found it really nice to have a compiler by my side while programming,
it really speeds up the feedback cycle of development, and I really miss it while doing JavaScript at work. Perhaps it's time for another look
at [Flow][2]?

<!-- more -->

When I was using C++ at school we never really got to use any C++11/14 features, so I just now discovered a whole new world of language features
(not all of them good, mind you). One particular feature that I really enjoyed is the addition of `std::function` and lambdas.
`std::function` is a general purpose function wrapper class, that can be used to wrap lambdas, function pointers, and any other `Callable` and
pass to a function. This enables all sorts of sexy FP goodness, and allows for writing way more general and reusable methods.

Another feature of C++ that I just discovered was the full power of the templating system. I've heard that C++ templates are Turing-complete
and so on, but I never gave this any further thought. I used to think this was an academic triviality, like the fact that Conway's game of life
is Turing-complete, and the only usage of templates I had ever come across was to genericise(is that a word?) container classes to handle
multiple datatypes. Turns out templates are a _lot_ more powerful than that.

In order to explore these new (to me) concepts I created a tiny vector implementation ([source][3]).
This implementation has some neat properties, which I will go over here.

To create a vector you supply a `size_t` to indicate it's size. This size is fixed.

{% highlight c++ %}

auto v1 = Vec<char, 3>();

{% endhighlight %}

Accessing and mutating the contents of the vector are done through the `set` and `at` methods.
{% highlight c++ %}

v1.set<0>('a');
v1.at<0>; // => 'a'

v1.set<3>('d'); // *bleep*

{% endhighlight %}
The latter of these will fail to compile, emitting a helpful error message:

    ./../vec.hpp:48:9: error: static_assert failed "Out of bounds access"

The compiler will prevent you from adding vectors of different sizes together. The definition of the `+` operator on `Vec<T, i>` is

{% highlight c++ %}
Vec<T, i> operator+(Vec<T, i>& other) {
    Vec<T, i> ret;
    for(uint32_t a = 0; a < i; a++) { ret.data[a] = other[a] + data[a]; }
    return ret;
}
{% endhighlight %}

so the operation is only defined for another vector of type `Vec<T, i>`.

I also implemented `map` over a vector. The signature of `Vec::map` is
{% highlight c++ %}
template <typename R>
Vec<R, i> map(std::function<R(T)> fn)
{% endhighlight %}

so it takes a `std::function` that takes a `T` and returns an `R` and returns a new vector of type `Vec<R, i>`.
The implementation itself is trivial but I think it's pretty awesome to be able to tell what a function does purely by
looking at it's type signature. This is not something that I've noticed in C++ before, although Haskell, OCaml and other
ML-family languages very much possess this trait.

I had lots of fun exploring the new parts of the C++ language, and even more fun trying to find ways to apply them. I realized
that something very similar to what I was doing already exists in an more powerful form in `std::tuple`, a similar container
but generalized to be able to hold heterogeneous values. This is made possible by another really cool language feature, [variadic templates][4],
that I hope to explore more in the future.

That's it for today! Please check out the code on [GitHub][3] if you're interested, and please let me know where I'm doing something
stupid and how it can be made better.

Thanks to [Hrafn Eiríksson][5] for reading a draft of this post.

[1]: http://smile.amazon.com/gp/product/0984782850?keywords=cracking%20the%20coding%20interview%206th%20edition&qid=1456668580&ref_=sr_1_1&sr=8-1
[2]: https://github.com/facebook/flow
[3]: https://github.com/kthelgason/vecpp
[4]: https://en.wikipedia.org/wiki/Variadic_template
[5]: http://hrafn.eiriksson.is
