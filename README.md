
This is a short introduction to Julia (valid for versions >= 1.2 and at least until version 1.6) made for people who are already familiar with programming. Please consider using the html file for a better graphical rendering of this introduction.

## Basics

### Before Starting
Where to look for help:
- Documentation: [https://docs.julialang.org/](https://docs.julialang.org/) (documentation can also be accessed offline from the REPL, see below)
- Ask for help: [https://discourse.julialang.org/](https://discourse.julialang.org/) is the dedicated forum for discussing issues around Julia. StackOverflow can also be helpful sometimes.

Integrated Development Environment:
- Visual Studio Code with Julia extension is the most commonly used. The Julia extension is well maintained and updated. It has a good integration with the Julia REPL, a variable explorer, access to the documentation, a plot navigator, a debugger, etc.

### Julia REPL
To start using the julia REPL in VS Code, you can either:
* use the command palette (Cmd+Shift+P on Mac / ctrl+Shift+P on PC) and type "Julia: Start REPL" (the command will appear while typing it).
* Run a line/block of code using Shift+Enter or Ctrl+Enter. The difference between the two is that Shift+Enter places the cursor at the beginning of the next code block, and Ctrl+Enter doesn't.

Besides running the code, the REPL has convenient functionalities:
* Access the documentation, by pressing "?". Search for a term between quotes to return all the entries matching the term, otherwise without quotes for a something already defined.
* Access the package manager, by pressing "]".
* Access the shell, by pressing ";".

More on the REPL: [https://docs.julialang.org/en/v1/stdlib/REPL/](https://docs.julialang.org/en/v1/stdlib/REPL/).

### Basic Syntax

#### Variable declaration and assignment

```julia
a = 1.0 ## A floating number

arr = [1.0, 3.0, 4.0] ### A one dimensional array

d = Dict(1 => 'a', 5 => 'c') ### A dictionary with integers as keys and characters as values

```

#### Loops, functions and code blocks
The basic syntax for loops and (conditional) blocks follows the pattern:
```
*declaration_keyword* [argument]
    *instructions*
end
```
For example, a `for` loop:
```julia
for i in 1:100
    #instructions
end
```
More detailed information about control flow: [https://docs.julialang.org/en/v1/manual/control-flow/](https://docs.julialang.org/en/v1/manual/control-flow/)

Functions are declared similarly:
```julia
### A function named "simple" that takes u and v as arguments
function simple(u, v)
    return (u+2v)^4
end
```
Simple functions can also be declared without creating a dedicated code bloc: `simple(u,v) = (u+2v)^4`.

#### Convention for mutating functions

When naming functions in Julia, the convention is to add an `!` at the end of a function name if it mutates at least one of its arguments. 
This convention is not enforced by the compiler so it is possible to use another one (or none), but all native functions in Julia and in the packages built around it use it. For example:

```julia
##Non mutating function
function add(a::Vector{Float64}, b::Vector{Float64})
    result = zeros(length(a)) ##We create a new vector that will be the result
    for i=1:length(a)
        result[i] = a[i] + b[i]
    end
    return result
end

###Mutating function: stores the result in an existing vector "res" that is given as an argument
function add!(res::Vector{Float64}, a::Vector{Float64}, b::Vector{Float64})
    for i=1:length(a)
        res[i] = a[i] + b[i]
    end
    return res
end
```
Note here that the returned value isn't related to the mutated object, we could have returned `a` or `b` without any impact on the mutation or not of `res`.

## Types and multiple dispatch
In Julia, every instance has a type: floating point numbers are of type `Float64`, `Float32`, etc. depending on their precision. Integers are of type `Int64`, `Int32`, etc.
single characters are of type `Char`. Types can have abstract types as an umbrella type, abstract types cannot be instantiated. For example `Int64` and `Int32` have the same supertype `Signed`.
Then signed and unsigned numbers have the same supertype `Integer` and so on...all the way up to `Any` that is the abstract supertype of every type.

### Multiple Dispatch
Besides helping to organize the code, types are one of the backbones of Julia since they are 
central to the multiple dispatch system (close to the notion of overloading in statically typed languages).

For example if we define the following function `p` with arguments `a`and `b` without declaring their types:
```julia
p(a, b) = a * b
```
Julia is going to store the **function** without compiling it. Then the first time we call `p` in the code with two `Float64` as arguments (say `p(1.0, 3.0)`), the compiler creates a **method**
`p` specialized on the types of `1.0` and `3.0`, i.e. on two `Float64` and apply it every time it encounters the function `p` applied over two `Float64`. This specialization of functions based on
the types of their arguments is called multiple dispatch. The fact that a specialized method of the function is compiled for each type of argument is essentially what makes Julia faster 
than interpreted languages like Python. As a consequence, the easier it is for the compiler to guess the types of the variables used within a function, the more likely it will be able to efficiently 
specialize it. As a rule of thumb, if the type of the function output is entirely determined by the type of its inputs, and if the types of variables within a function don't change, it can be efficiently compiled.

More information about performance: [https://docs.julialang.org/en/v1/manual/performance-tips/](https://docs.julialang.org/en/v1/manual/performance-tips/).

In case there are ambiguous definitions of functions (2 or more functions with the same name), which would prevent the compiler to dispatch correctly to the right method,
it is possible to define a function with constraints on the arguments types.
```julia
p(a::Float64, b::Float64) = a * b ##This is a function that corresponds to only one method
```
we can also be more general, by declaring supertypes instead to types:
```julia
p(a::Real, b::Real) = a * b
```

### Predefined Types

#### Primitive types
From the [documentation](https://docs.julialang.org/en/v1/manual/types/#Primitive-Types), the list of defined primitive predefined types and their direct (abstract) supertypes:
```julia
######### Floating Numbers
primitive type Float16 <: AbstractFloat 16 end  
primitive type Float32 <: AbstractFloat 32 end
primitive type Float64 <: AbstractFloat 64 end
######### Boolean: true and false
primitive type Bool <: Integer 8 end
###Unique characters: declared between simple quotes, e.g. 'a'
primitive type Char <: AbstractChar 32 end
######### Integers:
primitive type Int8    <: Signed   8 end
primitive type UInt8   <: Unsigned 8 end
primitive type Int16   <: Signed   16 end
primitive type UInt16  <: Unsigned 16 end
primitive type Int32   <: Signed   32 end
primitive type UInt32  <: Unsigned 32 end
primitive type Int64   <: Signed   64 end
primitive type UInt64  <: Unsigned 64 end
primitive type Int128  <: Signed   128 end
primitive type UInt128 <: Unsigned 128 end
```
#### Composite types
Composite types are analogous to objects in object oriented programming. They are encapsulations of one or several primitive or composite types.
They are declared by the keyword `struct`:
```
[mutable] struct SomeStruct
    field_1::Type_field_1
    field_2::Type_field_2
    ...
end
```
Structs have default constructors of this form: `SomeStruct(f_1, f_2)`. Calling `SomeStruct(f_1, f_2)` will create a struct with `field_1`
having value `f_1` and `field_2` having value `f_2`. Further constructors based on the default constructor can also be created.

If necessary, it is possible to override the default constructor by defining an [inner constructor](https://docs.julialang.org/en/v1/manual/constructors/#man-inner-constructor-methods) taking the same arguments.

The different fields of a struct can be accessed using `s.field_1` where `s` is an instance of a certain `struct`. It is also possible to use `getfield(s, :field_1)`, this can be helpful when [broadcasting](https://docs.julialang.org/en/v1/manual/arrays/#Broadcasting) or when doing [function composition or piping](https://docs.julialang.org/en/v1/manual/functions/#Function-composition-and-piping). 

Only the fields of a mutable struct can be changed, but in counterpart, mutable structs are slower. In terms of performance, structs are the fastest composite data type if the number and type of elements to be used is known in advance and fixed.

Structs can be declared with fields having abstract types, they are however much slower. If the type of a certain field is not known in advance but is nonetheless assumed to be invariable, it is possible to declare a parametric struct (or parametric type). For example:
```julia
struct SomeStruct{T}
    field_1::T
end
```
will create a parametric struct that will only become a "concrete one" once the constructor is used with a concrete type. For example:
`SomeStruct(1.0)` will create a struct having the concrete type `SomeStruct{Float64}`. This forces each instance of `SomeStruct` to have a definite type and allows the compiler to specialize the code for that particular type, 
avoiding the performance penalty linked to abstract types.

#### Important non-primitive types

##### Arrays
[`Arrays`](https://docs.julialang.org/en/v1/base/arrays) are lists of objects indexed by an element in $\{1, ..., n_1\} \times \{1, ..., n_2\} \times \dots \times \{1, ..., n_d\}$, where $d$ is the dimension of the array. `Array` is a parametric type taking as arguments
the type of objects contained (`Float64`, `String`, `Bool`, etc.) and a positive integer corresponding to the dimension of the array, e.g. a three dimensional array containing only `Float64` has type `Array{Float64, 3}`. The type of the contained
objects can be abstract (e.g. `Real` for any kind of number, or `Any` if the contained elements are heterogeneous), but this impacts negatively the performance of the code.

Perhaps the most commonly used array types are:
* The one dimensional array (or vector): `Array{type, 1}` (e.g. `Array{Float64, 1}`) which is also designated by the type alias `Vector{type}`, (e.g. `Vector{Float64}`).
* The two dimensional array (or matrix): `Array{type, 2}` (e.g. `Array{Float64, 2}`) which is also designated by the type alias `Matrix{type}`, (e.g. `Matrix{Float64}`).

Common ways to generate arrays:
* Arrays containing uninitialized objects: `Array{type, d}(undef, n_1, n_2, ..., n_d)`, or `Vector{type}(undef, size)` for vectors (works similarly for matrices), e.g. `Array{Int64, 3}(undef, 2, 3, 3)`. These arrays are the fastest to create if their elements are to be determined using indexing.
* Arrays filled with the same object: `fill(object_variable, n_1, n_2, ..., n_d)`. 
* Arrays filled with ones or zeros: `zeros(n_1, n_2, ..., n_d)`, `ones(n_1, n_2, ..., n_d)`. The type of ones or zeros of the array can be chosen by specifying type as a first argument. The constructors return arrays of `Float64` if the type is not specified.

###### Array access and modification
Array elements can be accessed and modified using `arr[ind_1, ind_2, ..., ind_d]`, e.g. `arr[ind_1, ind_2, ..., ind_d] = new_value`. This notation is equivalent to calling the functions `getindex(arr, ind_1, ind_2, ..., ind_d)` for access without modification and to `setindex!(arr, new_value, ind_1, ind_2, ..., ind_d)` for array modification.
It is also possible to access an entire subset of an array using a [variety of manners](https://docs.julialang.org/en/v1/manual/arrays/#man-array-indexing) (logical indexing, cartesian indexing, etc.) that also allow to reshape the subset of the array in the same function call. We show here how to return whole array slices, which is the simplest way of returning a subset of an array. Array slicing is done by using "`:`" instead of an integer index when indexing. The colon corresponds to all indices at that dimension, for example `arr[:, ind_2]`
corresponds to the (whole) column `ind_2` of the array. When subsetting (including slicing), it is important to remember that accessing a subset of an array without modification (i.e. not followed by "`=`") returns a copy of the subset. This has two consequences:
* A copy of the subset is created every time the subset is accessed this way, which [*can*](https://docs.julialang.org/en/v1/manual/performance-tips/#Copying-data-is-not-always-bad) negatively impact performance.
* The original array will not be mutated if the subset of the array is given as an argument of a function that is meant to mutate it (the function will mutate the copy!).

The solution to the issues above is to use a [view](https://docs.julialang.org/en/v1/base/arrays/#Views-(SubArrays-and-other-view-types)). Views can be seen as pointers to a subset of an array, thus modifying a view modifies the array they point at. Views are of the abstract type `AbstractArray`, they can thus be used like normal arrays.
To return a view of an array, use: `view(arr, index_subset_1, ...., index_subset_d)`, e.g. for array slicing: `view(arr, :, ind_2)`, it is also possible to use the macros [@view](https://docs.julialang.org/en/v1/base/arrays/#Base.@view) and [@views](https://docs.julialang.org/en/v1/base/arrays/#Base.@views) to manipulate views.

!!! note "<b>Note</b>"
    Due to the Julia variable assignment system, views are not attached to variables but to the arrays underlying them. In the following example:
    ```julia
    a = zeros(5, 3) ##Assign an array to "a"
    a_view = view(a, :, 2) ##Create a view of the array that "a" is attached to
    a = ones(3, 4) ##Assign a new array to "a"
    ```
    `a_view` is still a view of the array `zeros(5, 3)` despite the reassignment of `a` in the third line. This is because the **variable assignment operation** only 
    creates a reference to an object through a variable name. The operation in the third line thus only creates a new reference to `array(3, 4)` through `a`, leaving `array(5, 3)`
    "unreachable" directly through a variable name, but still existing in memory as long as `a_view` is still referring to it.



!!! note "<b>Performance of array access and modification</b>"
    Arrays in Julia are stored in column major order, i.e. two slots of an array (of immutables) are contiguous in memory if their indices are consecutive following the **co**lexicographic order.
    This means, in the case of a matrix, that two slots are contiguous in memory if they are consecutive within a same column or if they are the last and first elements of two
    consecutive columns. Accessing slots in consecutive order can greatly improve performance since it reduces unnecessary movement along the memory.

###### Arrays of structs

Consider using the package [StructArrays](https://github.com/JuliaArrays/StructArrays.jl) that provides an efficient implementation of arrays of which elements are structs.

##### Broadcasting
In Julia, it is possible to vectorize operations on scalars (or [arrays of dimension 0](https://docs.julialang.org/en/v1/manual/faq/#What-are-the-differences-between-zero-dimensional-arrays-and-scalars?)) through [broadcasting](https://docs.julialang.org/en/v1/manual/functions/#man-vectorized). 
The easiest way to do this is by using the dot syntax, i.e. by adding a dot "." when calling a function or an operator. For example, the function `p(a::Float64, b::Float64) = a * b` 
only takes scalars as arguments (two `Float64`), but can however be applied elementwise over two arrays by using the dot syntax: 
```julia
p(a::Float64, b::Float64) = a * b

a_s = 3.0
b_s = 2.0
p(a_s, b_s) ## Apply p over scalars

a_arr = collect(1:1.0:10)
b_arr = collect(10:-1.0:1)
p.(a_arr, b_arr) ## Apply p elementwise over the elements of a_arr and b_arr
```
Broadcasting operations is closely equivalent (but actually more [complex](https://docs.julialang.org/en/v1/manual/arrays/#Implementation)) to doing them over the array elements using a `for` loop.
The arrays involved in the broadcasting operations must be of [compatible dimensions](https://docs.julialang.org/en/v1/manual/arrays/#Broadcasting).

!!! note "Note: <b>broadcasting assignment operations and performance</b>"
    Broadcasting assignment operations over arrays avoid useless memory allocations, which generally improves performance. For example, compare the two operations **yielding the same result** over
    a vector `v = ones(10)`:
    ```julia
    v .= (v .+ v) .^2
    ```
    and:
    ```julia
    v = (v .+ v) .^2
    ```
    The only syntactical difference between both expressions is the dot before the equal sign, but the resulting compiled instructions differ greatly in terms of memory allocation.
    The first expression will result in instructions that are similar to the following `for` loop:
    ```julia
    for i in eachindex(v) #eachindex returns the most efficient iterator over the indices of v
        v[i] = (v[i] + v[i])^2
    end
    ```
    while the second will result in a supplementary memory allocation corresponding to the result of `(v .+ v) .^2` (and a possible (silent) garbage collection operation):
    ```julia
    result = (v .+ v) .^2 ##Allocate a new vector that is the result
    v = result ##Change the reference of the variable `v` to `result`
    ##Also: the garbage collector will have to clean (at some point) the array that was 
    ##previously referenced by v if the array isn't referenced by any other variable
    ```
    In terms of function calls, the first expression calls the non allocating function `broadcast!` with `v` as the perallocated vector used to contain the result, and the second
    expression calls `broadcast` that allocates a new vector that will contain the result.

##### Other important data structures

* [Dictionaries](https://docs.julialang.org/en/v1/base/collections/#Dictionaries)
* [Sets](https://docs.julialang.org/en/v1/base/collections/#Base.Set)
* [Iterators](https://docs.julialang.org/en/v1/base/collections/#lib-collections-iteration)
* [Generators](https://docs.julialang.org/en/v1/manual/arrays/#Generator-Expressions-1)

Other useful data structures can be found in the [DataStructures](https://juliacollections.github.io/DataStructures.jl/latest/) package.

## Overview of Julia's Organization
Julia libraries are organized into [*modules*](https://docs.julialang.org/en/v1/manual/modules/) that contain the definitions of variables, functions (and macros) to be used. Each module defines a separate namespace, meaning that different modules can use the same objects names without conflict. 
Modules or objects within modules can be loaded and renamed this way:
```
using ModuleName[: function_or_sturct_name] [as rename_function_or_struct], [other_function_or_sturct_name] [as rename_other_function_or_struct], ....
```
Example: we load the (pseudo)random function generating exponentially distributed samples from the `Random` module and rename it "rexp":
```julia
using Random: randexp as rexp
```
`using ModuleName` alone imports all the definitions in the module/package named `ModuleName`.

### Essential Modules
At the heart of the language lie the `Core` module and the `Base` module that are always automatically imported. For example the *Core* module contains the definitions of primitive types, IO oprations, etc. The `Base` module contains several basic functions like `sin`, `cos`, `sort`, etc. Finally there is a module called `Main` where all the definitions go by default if they are not explicitely attributed to a module.

More information about modules: [https://docs.julialang.org/en/v1/manual/modules/](https://docs.julialang.org/en/v1/manual/modules/).

### Packages
A package is roughly some coherent Julia code organized into modules and following a certain standard (with metadata, version, etc.) that makes it directly downloadable and usable as is. Packages are downloadable using the package manager that is accessible:
- In the REPL: press "]" to call the package manager, then: add PackageName.
- Within the code:
```julia
using Pkg ##Load the package manager (which is itself a package)
Pkg.add("PackageName")
```

Some useful packages:
* Plots: plotting, rather basic but still can do many things
* Gadfly: plotting, similar to ggplot2
* DataStructures (part of [JuliaCollections](https://github.com/JuliaCollections))
* StatsBase (part of [JuliaStats](https://github.com/JuliaStats)): statistics
* Distributions (part of [JuliaStats](https://github.com/JuliaStats)): probabilistic distributions
* DataFrames (part of [JuliaData](https://github.com/JuliaData))
* CSV (part of [JuliaData](https://github.com/JuliaData)): for reading delimited files
* DifferentialEquations (part of [SciML](https://github.com/SciML/)): differential equations and (much) more
* SpecialFunctions (part of [JuliaMath](https://github.com/JuliaMath)): special functions such as gamma, hypergeometric, etc.
* Agents (part of [JuliaDynamics](https://github.com/JuliaDynamics)): agent-based modeling framework
* StructArrays: Efficient arrays of `structs` 
