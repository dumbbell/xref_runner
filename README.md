![](http://www.mindsmart.com.br/wp-content/uploads/2014/07/paulie-juno-mocinhos.jpg)

# Xref Runner
Erlang Xref Runner (inspired in rebar's rebar_xref)

## Contact Us
For **questions** or **general comments** regarding the use of this library,
please use our public [hipchat room](https://www.hipchat.com/gpBpW3SsT).

If you find any **bugs** or have a **problem** while using this library, please
[open an issue](https://github.com/inaka/elvis/issues/new) in this repo
(or a pull request :)).

And you can check all of our open-source projects at [inaka.github.io](http://inaka.github.io).

## Why Xref Runner?
So, Erlang/OTP comes with an excellent tool to check your code and detect bugs: [Xref](http://www.erlang.org/doc/apps/tools/xref_chapter.html).
The problem lies in its not-extremely-simple interface. Using `xref` out of the box requires you to start a process, set up parameters, add directories, etc. before you can actually just run the checks you want to run.
To mitigate that, [rebar](http://github.com/rebar/rebar) comes with a handy command line tool (`rebar xref`) that prints xref generated warnings on your console.
But, sometimes, you don't want to use rebar or you want to get the warnings as erlang terms and not just printed out in the console.
That's when xref_runner comes along.

## How to Use it?
Just make sure it's in your code path and call `xref_runner:check/2` with the proper parameters.

The first parameter is one of the available checks provided by `xref`:
```erlang
-type check() :: undefined_function_calls
               | undefined_functions
               | locals_not_used
               | exports_not_used
               | deprecated_function_calls
               | deprecated_functions.
```

The second paramter is a configuration map. All of its fields are optional. The allowed fields are:

* **extra_paths**: Directories to be added to the xref code path. _default value:_ `[]`
* **xref_defaults**: Default values to configure xref (check `xref:set_default`). _default value:_ `[]`
* **dirs**: Directories to be scanned with `xref`. _default value:_ `["ebin"]`

Using that function will give you a list of _warnings_ as its result. Warnings are also maps with the following fields:

* **filename**: The name of the file for which the warning is reported
* **line**: The line number where the warning is reported. `0` means it's a module-level warning (like an undefined function). _Note:_ In case of warnings with source and target, line numbers refer to the line in which the source function is _defined_, not the line where the target function is _used_.
* **source**: Module, function and argument number where the warning is found
* **target** _(optional)_: For `undefined_function_calls` and `deprecated_function_calls`, function call that generates the warning.

## Examples
Using the modules in the [examples](test/examples) folder, these are some of the results generated by the tests:

```erlang
> xref_runner:check(undefined_function_calls, #{dirs => ["test"]}).
[#{filename => "test/examples/undefined_function_calls.erl",
   line => 5,
   source => {undefined_function_calls,bad,0},
   target => {undefined_function_calls,undefined_here,0}},
 #{filename => "test/examples/undefined_function_calls.erl",
   line => 9,
   source => {undefined_function_calls,bad,1},
   target => {other_module,undefined_somewhere_else,1}},
 #{filename => "test/examples/undefined_function_calls.erl",
   line => 9,
   source => {undefined_function_calls,bad,1},
   target => {undefined_functions,undefined_there,0}},
 …
]
```

```erlang
> xref_runner:check(undefined_functions, #{dirs => ["test"],
                                           xref_defaults => []}).
[#{filename => [],line => 0,source => {other_module,undefined_somewhere_else,0}},
 #{filename => [],line => 0,source => {other_module,undefined_somewhere_else,1}},
 #{filename => "test/examples/undefined_function_calls.erl",
   line => 0,
   source => {undefined_function_calls,undefined_here,0}},
 …
]
```

```erlang
> xref_runner:check(locals_not_used, #{dirs => ["test"]}).
[#{filename => "test/examples/locals_not_used.erl",
   line => 9,
   source => {locals_not_used,local_not,1}}]
```

```erlang
> xref_runner:check(exports_not_used, #{dirs => ["test"]}).
[#{filename => "test/examples/deprecated_function_calls.erl",
   line => 7,
   source => {deprecated_function_calls,bad,0}},
 #{filename => "test/examples/deprecated_function_calls.erl",
   line => 10,
   source => {deprecated_function_calls,bad,1}},
 #{filename => "test/examples/deprecated_function_calls.erl",
   line => 14,
   source => {deprecated_function_calls,good,0}},
…
]
```

```erlang
> xref_runner:check(deprecated_function_calls, #{dirs => ["test"]}).
[#{filename => "test/examples/deprecated_function_calls.erl",
   line => 7,
   source => {deprecated_function_calls,bad,0},
   target => {deprecated_functions,deprecated,0}},
 #{filename => "test/examples/deprecated_function_calls.erl",
   line => 10,
   source => {deprecated_function_calls,bad,1},
   target => {deprecated_function_calls,internal,0}},
 #{filename => "test/examples/deprecated_function_calls.erl",
   line => 10,
   source => {deprecated_function_calls,bad,1},
   target => {deprecated_functions,deprecated,1}},
 …
]
```

```erlang
> xref_runner:check(deprecated_functions, #{dirs => ["test"]}).
[#{filename => "test/examples/deprecated_function_calls.erl",
   line => 17,
   source => {deprecated_function_calls,internal,0}},
 #{filename => "test/examples/deprecated_functions.erl",
   line => 8,
   source => {deprecated_functions,deprecated,0}},
 #{filename => "test/examples/deprecated_functions.erl",
   line => 10,
   source => {deprecated_functions,deprecated,1}},
 #{filename => "test/examples/ignore_xref.erl",
   line => 19,
   source => {ignore_xref,internal,0}}]
```

```erlang
2015-02-09 15:57:42.499
> xref_runner:check(deprecated_function_calls, #{dirs => ["test"]}).
[#{filename => "test/examples/deprecated_function_calls.erl",
   line => 7,
   source => {deprecated_function_calls,bad,0},
   target => {deprecated_functions,deprecated,0}},
 #{filename => "test/examples/deprecated_function_calls.erl",
   line => 10,
   source => {deprecated_function_calls,bad,1},
   target => {deprecated_function_calls,internal,0}},
 #{filename => "test/examples/deprecated_function_calls.erl",
   line => 10,
   source => {deprecated_function_calls,bad,1},
   target => {deprecated_functions,deprecated,1}},
 …
]
```
