# ImplicitEquations

[![Build Status](https://travis-ci.org/jverzani/ImplicitEquations.jl.svg?branch=master)](https://travis-ci.org/jverzani/ImplicitEquations.jl)



This paper by
[Tupper](http://www.dgp.toronto.edu/people/mooncake/papers/SIGGRAPH2001_Tupper.pdf)
details a method for graphing two-dimensional implicit equations and
inequalities involving two variables. This package gives an
implementation of the  paper's basic algorithms to allow
the `julia` user to naturally represent and easily render graphs of
implicit functions and equations.



For example, the
[Devils curve](http://www-groups.dcs.st-and.ac.uk/~history/Curves/Devils.html)
is graphed over the default region as follows:

```
using ImplicitEquations
using Winston 

a,b = -1,2
f(x,y) = y^4 - x^4 + a*y^2 + b*x^2
r = (f==0)
plot(r)
```

![DevilsCurve](http://i.imgur.com/LChTzC1.png)


The `f == 0` expression above creates a predicate that gets
graphed. For all but the case of `f==g` (for two functions) these are
generated by overloading the logical operators for functions on the
left and real values or functions on the right. As `==` can already be
used to compare functions in a different way, we use either `eq(f,g)`
for that comparison, or should infix notation be preferred, the unicode
operator `f \eqcolon<tab> g` may be used.

For example, the
[Trident of Newton](http://www-history.mcs.st-and.ac.uk/Curves/Trident.html)
can be represented in Cartesian form as follows:

```
## trident of Newton
c,d,e,h = 1,1,1,1
f(x,y) = x*y
g(x,y) = c*x^3 + d*x^2 + e*x + h
plot(eq(f,g))       ## aka f ≕ g
```

![Newton trident](http://i.imgur.com/7NxtUsK.png)


Inequalities can be graphed as well

```
f(x,y)= (y-5)*cos(4*sqrt((x-4)^2 + y^2))
g(x,y) = x*sin(2*sqrt(x^2 + y^2))
r = f < g
plot(r, -10, 10, -10, 10, W=2^9, H=2^9)
```

![Inequality](http://i.imgur.com/aEFjlTp.png)


The coloring scheme employed follows Tupper:

* white for the predicate `r` definitely not being satisfied for the pixel,
* black if the predicate is definitely satisfied somewhere in the pixel,
* and red if it is not determined.


This graph  shows the algorithm

```
f(x,y) = y - sqrt(x)
plot(f==0, W=2^4, H=2^4, offset=0)
```

![Algorithm](http://i.imgur.com/8Mtmb7v.png)

The basic algorithm is to initially break up the graphing region into
square regions. (This uses the number of pixels, which are specified
by `W` and `H` above.)  There regions are checked for the
predicate. If definitely not, the region is painted white; if
definitely yes, the region is painted black; else the square region is
subdivided into 4 smaller regions and the above is repeated until
subdivision would be below the pixel level. At which point, the
remaining 1-by-1 pixels are checked for possible solutions, for
example for equalities where continuity is know a random sample of
points is investigated with the intermediate value theorem.


This example, the
[Batman equation](http://yangkidudel.wordpress.com/2011/08/02/love-and-mathematics/),
Uses a few new things: the `screen` function is used to restrict
ranges and logical operators to combine predicates.

```
f0(x,y) = ((x/7)^2 + (y/3)^2 - 1)  *   screen(abs(x)>3) * screen(y > -3*sqrt(33)/7) 
f1(x,y) = ( abs(x/2)-(3 * sqrt(33)-7) * x^2/112 -3 +sqrt(1-(abs((abs(x)-2))-1)^2)-y)
f2(x,y) = y - (9 - 8*abs(x))       *   screen((abs(x)>= 3/4) &  (abs(x) <= 1) )
f3(x,y) = y - (3*abs(x) + 3/4)     *   screen((1/2 < abs(x)) & (abs(x) < 3/4))
f4(x,y) = y - 2.25                 *   screen(abs(x) <= 1/2) 
f5(x,y) = (6 * sqrt(10)/7 + (1.5-.5 * abs(x)) - 6 * sqrt(10)/14 * sqrt(4-(abs(x)-1)^2) -y) * screen(abs(x) >= 1)

r = (f0==0) | (f1==0) | (f2== 0) | (f3==0) | (f4==0) | (f5==0)
plot(r, -7, 7, -4, 4)
```

![Batman Curve](http://i.imgur.com/Buyd9Fb.png)

The above example illustrates a few things:

* predicates can be joined logically with `&`, `|`. Use `!` for negation.

* The `screen` function can be used to restrict values according to
  some predicate call.

* the logical comparisons such as `(abs(x)>= 3/4) & (abs(x) <= 1)`
  within `screen` are not typical in that one can't write `3/4 <=
  abs(x) <= 1`, a convenient `Julian` syntax. This is due to the fact that the "`x`s"
  being evaluated are not numbers, rather intervals via
  `ValidatedNumerics`. For intervals, values may be true, false or
  "maybe" so a different interpretation of the logical operators is
  given that doesn't lend itself to the more convenient notation.

* rendering can be slow. There are two reasons: images that require a
  lot of checking, such as the inequality above, are slow just because
  more regions must be analyzed. As well, some operations are slow,
  such as division, as adjustments for discontinuities are slow. (And
  by slow, it can means really slow. The difference between rendering
  `(1-x^2)*(2-y^2)` and `csc(1-x^2)*cot(2-y^2)` can be 10 times.)

## An "typical" application

A common calculus problem is to find the tangent line using implicit
differentiation. The difficulty here is adding a layer to a
graph. With the `Winston` graph, the `wgraph` function returns a
`FramedPlot` object which can have a curve added to it. This
illustrates:

```
f(x,y) = x^2 + y^2
p = wgraph(f == 2*3^2)       # this is Winston specific
a,b = 3,3
dydx(a,b) = -b/a             # implicit differentiate to get dy/dx =-y/x
tl(x) = b + dydx(a,b)*(x-a)  
xs = linspace(-5, 5)
add(p, Curve(xs, map(tl,xs))) # add to a Winston plot
```

An alternative would be to use another function as follows:

```
g(x,y) = y - tl(x)
r = (f == 2*3^2) | (g == 0)
plot(r)
```

## Display

The graphs can be rendered in different ways.

* The `asciigraph` function is always available and makes a simple text-based plot.
* When one of several plotting packages are loaded, their `plot` function is extended to display a graph when the first argument is a `Predicate`, as generated by the expressions above. For each package a separate plotting function is exported:
- for `Winston`, the function  `wgraph` does the work,
- for `PyPlot`, the function  `pgraph` does the work, and
- for `Gadfly`, the function  `ggraph` does the work,
* For rendering with `Cairo` and `Gtk`, a function `cgraph` is exported, but no `plot` method is introduced
* For rendering with `Patchwork`, the `pwgraph` function is exported, but no plot method is introduced

  The SVG-based solutions, useful in `IJulia`, are `Patchwork` and
  `Gadfly`. These are not work as advertised above, as there is some
  issue with the load hook provided by `Jewel.@require`.


## TODO

*LOTS*:

* add in other functions, as possible. (At least all in `ValidatedNumerics`.)
* This needs to be `@doc`ed
* The plot interface should be standardized and allow for easily adding additional layers, such as tangent lines
* Check out these graphs to see which can be done
- http://www.xamuel.com/graphs-of-implicit-equations/
- http://www.peda.com/grafeq/gallery.html
* branch cut tracking and interval sets are employed by Tupper, these could be added. (As well, Tupper sketches out how to be more rigorous with computing whether a region is black or white.)
* increase speed (could color 1-pixel regions better if so, perhaps; division checks; type stability).

