![Elixir Logo](https://elixir-lang.org/images/logo/logo.png)
## Overview
[Elixir](https://elixir-lang.org/) is a dynamically typed, compiled, functional
programming language. You compile Elixir source code (`.ex` files) to BEAM bytecode
(`.BEAM` files) that are then run by the Erlang virtual machine, Bjorn's Erlang
Abstract Machine (BEAM).

Elixir was created primarily by Jose Valim, a former Ruby and Rails developer and
founder of the Brazilian development consultancy Plataformatec.

I've personally had a lot of fun programming in Elixir. It's a pleasure to read and
write, much like Ruby. Coming from an object-oriented background its functional style
opened my eyes to new ways of thinking about programming. And finally, it's concurrency
model is exciting because it allows you to build classes of applications that would
be extremely difficult/expensive to create in other ecosystems.

[Phoenix](http://phoenixframework.org/) is a web application development framework
written in Elixir, and is analogous to Rails in many ways. It was created by Chris
McCord, another former Ruby and Rails developer.

## Table of Contents
+ [Think Functionally](#think-functionally)
+ [High Scalability and High Performance](#high-scalability-and-high-performance)
+ [Be Explicit](#be-explicit)
+ [Phoenix way, way too briefly](#phoenix-way-way-too-briefly)
+ [Erlang Interoperability](#erlang-interoperability)
+ [Community](#community)
+ [Bonusly](#bonusly)

## Think Functionally
Functional programming emphasizes writing small, single-purpose functions, and
composing them to achieve your desired result. I've heard it described as an
assembly line, where you input some data on one end, and output some transformed
data on the other end, with a series of functions along the way that each make some
incremental transformation of the data.

There are no classes, and there are no objects. You do not combine state and behavior
(which is what an object is: instance variables and methods).

There are only modules and functions. The closest thing to an object is a struct,
which is a map (a hash in Ruby) with some guaranteed set of fields, and maybe with
some default values.

+ [The Pipe Operator](https://elixirschool.com/en/lessons/basics/pipe-operator/): elegantly chain functions together.
```elixir
  iex> "Elixir rocks" |> String.upcase |> String.split
  #["ELIXIR", "ROCKS"]
```

+ [Pattern Matching](https://elixirschool.com/en/lessons/basics/pattern-matching/): easily validate and parse even highly nested, complex data structures. Elixir School says it well _"In Elixir, the = operator is actually a match operator, comparable to the equals sign in algebra. Writing it turns the whole expression into an equation and makes Elixir match the values on the left hand with the values on the right hand. If the match succeeds, it returns the value of the equation."_
```elixir
  # Lists
  iex> list = [1, 2, 3]
  #[1, 2, 3]
  iex> [1, 2, 3] = list
  #[1, 2, 3]
  iex> [] = list
  ** (MatchError) no match of right hand side value: [1, 2, 3]

  iex> [1 | tail] = list
  #[1, 2, 3]
  iex> tail
  #[2, 3]
  iex> [2 | _] = list
  ** (MatchError) no match of right hand side value: [1, 2, 3]

  # Maps
  iex> key = "hello"
  #"hello"
  iex> %{^key => value} = %{"hello" => "world"}
  #%{"hello" => "world"}
  iex> value
  #"world"
  iex> %{^key => value} = %{:hello => "world"}
  ** (MatchError) no match of right hand side value: %{hello: "world"}
```

+ [Lazy enumeration](https://elixir-lang.org/getting-started/enumerables-and-streams.html#eager-vs-lazy): combine as many `map`, `filter`, `reduce`, etc operations as you want in a single iteration, and lazily evaluate.
Eg, define an interval from 1 to 100,000, and describe (but don't yet perform) a
series of computations to perform on each number in that interval with the `Stream` module:
```elixir
  iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?)
  #Stream<[enum: 1..100000, funs: [...]]>
```
Execute the computations described with the `Enum` module. Each entry is processed
through every computation described in the chain of `Stream` functions before the
next entry is processed. Thus a single pass can perform multiple operations:
```elixir
  iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?) |> Enum.sum
  #7500000000
```

### Immutability + Local-Only State = Programming For Dummies
Now you may say: _Errmehgod! Functional programming probably makes everything a
pain in the ass by not being able to reassign variables._

You can "reassign" a variable (Elixir folks don't use the term "assignment",
though, instead saying crazy shit like "binding the value to the name `foo`") within
a function, it's simply that *no outside functions will have access to that variable*.

This is huge! How many times have you been trying to reason about some OOP code and you
end up being bitten by some side effect. Maybe by changing the value of an instance variable
in your method you accidentally break something else that uses that instance variable. Maybe
some code outside of your method changes the instance variables your method relies on. Either way,
shitty times!

By not allowing functions to share state (no such thing as an instance variable, class variable, etc),
functions may only communicate by explicitly passing arguments to one another.

*Ruby connection: this is a useful practice many times even in OOP code. We should aim to minimize
mutation and side effects. Pure functions that take in data only via arguments, and have no side effects
are easy to reason about, compose, and test.*

Equally as important, you cannot mutate in-memory data. This concept already exists in OOP
languages like Ruby, it's just not as consistently enforced. Eg:

In Ruby you have a single value representing the integer `1`. It never changes.
Multiple variables can point to that value, but reassigning one of those variables
to a new integer does not effect the other assignments to `1` in any way.

In Ruby, if you wrote:
```ruby
  int = 1
  y = int
  x = int
  x += 1
```
You would be rightfully shocked if `y` now equaled `2`.

However, if you were to write:
```ruby
  arr = []
  x = arr
  y = arr
  x << 1
```
You may know that now both `y` and `x` will point to `[1]`, the *mutated*  version of the array pointed to by `arr`. Yuck!

This is impossible in Elixir. Data of all types is never mutated: not strings,
lists (Elixir's counterpart to arrays), nor maps (Elixir's counterpart to hashes),
nothing.

*Ruby connection: Matz is considering making strings immutable in Ruby 3*

### Persistent Data Structures
_Errmehgod! Functional programming languages probably duplicate so much data in
memory because everything is immutable._

Not so my friend, when Elixir makes a
copy of some data structure, internally it simply holds a reference to the exisiting
data. When a change is made to that copy or the original, it adds the new data (if any)
to memory, and connects all appropriate parties in a graph/tree structure. References to the
original data can safely be pointed to because it is guaranteed to never change.
Awwww, shucky ducky. These types of data structures are termed "persistent data structures",
not to be confused with persisting something in the database.

### Protocols
_Errmehgod! Functional programming probably means you can't create abstractions for
easily reasoning about your domain._

Tryna' flex on me? Don't be silly. In Elixir polymorphism akin to Ruby's duck
typing is provided via [procotols](https://elixir-lang.org/getting-started/protocols.html).
Protocols allow you to define a set of functions (including their argument signatures),
that a module or data type must implement along with the module or data type-specific
implementations.

```elixir
  defprotocol Size do
    @doc "Calculates the size (and not the length!) of a data structure"
    def size(data)
  end

  # Implementations for specific modules...
  defimpl Size, for: BitString do
    def size(string), do: byte_size(string)
  end

  defimpl Size, for: Map do
    def size(map), do: map_size(map)
  end

  defimpl Size, for: Tuple do
    def size(tuple), do: tuple_size(tuple)
  end
```

+ [Flow](https://github.com/elixir-lang/flow): makes it trivial to parallelize a function, automatically using all available CPU cores.
Eg, count all of the words in a file, in parallel:
```elixir
  File.stream!("path/to/some/file")
  |> Flow.from_enumerable()
  |> Flow.flat_map(&String.split(&1, " "))
  |> Flow.partition()
  |> Flow.reduce(fn -> %{} end, fn word, acc ->
    Map.update(acc, word, 1, & &1 + 1)
  end)
  |> Enum.to_list()
```

## High Scalability and High Performance
The Erlang VM was designed to be able to support literally millions of concurrent
users continuously for very long periods of time. This means:

+ It can't be prohibitively expensive to serve all of those users.
+ An error or performance bottleneck while responding to one user shouldn't
crash or slow down the system while it responds to other users.
+ The system should be able to easily scale horizontally, and it should be able
to survive hardware failures.
+ You should be able to inspect and alter the system while it's running, without
stopping the entire system.

### A Tale of Two Web Server Concurrency Models
Object-oriented programming languages tend to user the traditional locks-and
-threads concurrency model. In the context of a web framework this could manifest
itself in a few different ways.

To take one possible setup, maybe you spawn one OS process per core, and each of
those processes spins up some number of threads (your thread pool), and handles
requests by taking the next available thread from the pool. If all of the code in
your web framework is threadsafe (as it is in Rails) and all of *your* code is
threadsafe, you should be fine as far as race conditions, lost updates, etc.

In Ruby-land note that you can't actually execute more than one thread concurrently
unless you're running JRuby or Rubinius.

And regardless of language it is a non-trivial use of resources to spin up threads and/or OS processes.

Over in Elixir/Erlang land things look a lot different. Among Erlang's basic data types,
right next to integers, floats, string, and whatnot, is the `Process`. Not an OS process,
but one native to the Erlang VM, BEAM. BEAM processes are super lightweight, with
a memory footprint of only a few kilobytes, and taking almost no time to spawn.

It is not unusual for a large Erlang system to spawn *hundreds of thousands* of processes.
Your MacBook can spawn *millions* of these processes. Why is that cool?

### Memory Independence
Each of these processes is memory-independent of all others. They communicate like objects
in an OOP language: by sending each other messages with arguments. But there is no need
for locking via mutexes, because there is no shared, mutable state (processes cannot access
variables set in other processes).

*Ruby connection: this style of concurrency is called the Actor Model, and [some
people](https://anthonylebrun.silvrback.com/actors-vs-objects) believe it is a
sort of idealized form object-oriented programming. In OOP object's are supposed
to encapsulate their state, and NOT leak it out into other contexts. Instead
actors should communicate only by sending messages. Erlang's implementation of
the Actor Model forces you to program this way.*

```elixir
  defmodule Example do
    def listen do
      receive do
        {:ok, "hello"} -> IO.puts "World"
      end

      listen
    end
  end

  iex> pid = spawn(Example, :listen, [])
  #PID<0.108.0>

  iex> send pid, {:ok, "hello"}
  World
  #{:ok, "hello"}

  iex> send pid, :ok
  :ok
```

Further, any process crashing need not affect any other running process, unless you want
to link processes, for which there are all sorts of facilities within the language. Once
a process has crashed, Erlang provides a variety of strategies for automatically, intelligently
restarting the process (or not) in a good state. (Look up Elixir Supervisors if you're interested).

### Context Switching
Now that you know about BEAM processes, here is how they are used in practice.
Your OS will run a single BEAM process, which will automatically spawn a thread
for each CPU core on your machine. The job of these threads is to schedule work
for BEAM processes. This happens by way of lightning-fast context switching, similar
to how your OS would schedule work for different threads to handle.

There are two awesome consequences of this model:

### Responsiveness
The schedulers will give each BEAM process a tiny window with which to execute their code
before swapping out another process. This means that any process that is performing some
long-running task (maybe it's performing some intensive computation, talking to the filesystem
or network, or maybe it's stalled because of a bug or error) [will not prevent](https://youtu.be/5SbWapbXhKo?t=6m45s)
any other process from performing its job.

### Bang For Your Buck
You can, with modest hardware, quickly respond to hundreds of thousands of concurrent users.
What's App famously used Erlang to [serve millions of concurrent users](https://vimeo.com/44312354)
with a small number of engineers and relatively modest hardware.

In a traditional threaded environment the size of your thread pool might be a few dozen or
hundred threads, and so serving that number of concurrent users would require scaling
horizontally to many more servers, probably at a substantially higher cost.

### Let's Get Horizontal <3
Speaking of which, scaling horizontally is also a breeze with BEAM. If one machine isn't enough,
or you want redundancy & failover, you're in luck: BEAM processes don't care which machine other
BEAM process are on. The mechanism for communication is still the same - passing messages with
arguments.

With a few config file changes you can enable multiple BEAM processes to communicate securely
over the network via the same inter-process communication they use on a single node.

### Consistently Fast Code
Typically when diagnosing the performance of your web application it's useful to look
not only at your mean or median response time, but also at your 95th percentile response
times. These very slow response times are terrible user experiences, and often they are
caused because that request happened to trigger a run of the garbage collector.

You may have heard of "stop the world" garbage collection, where your entire Rails app
slows down because GC needs to clear out a bunch of stuff.

This doesn't happen Elixir applications, where each BEAM process's tiny memory footprint
is garbage collected individually. These GC runs take almost no time, and don't effect
other running processes. The equates to much more consistent server response times.

Phoenix server response times are typically measured in *microseconds* as opposed to
in milliseconds. An order of magnitude faster.

## Be Explicit
The Elixir community at large, including the Phoenix core team, and the authors
of most popular libraries tend to value explicitness over convenience. The
concept of "Rails magic" would be frowned upon in this community as a short-term
gain in convenience for long term declines in maintainability and ease of
comprehension.

A couple examples of this in action:

+ No autoloading of modules in Elixir and Phoenix, you must `import` the modules
and/or functions that you want to use in a given file. To me, this is an small loss
in convenience for a huge gain in simplicity. Think of all the nonsense we've gone
through in our codebase trying to figure out how Rails is autoloading various classes.

+ No polymorphic database associations in `Ecto`, the most popular Elixir ORM.
If you want a table to belong to many other tables you must explicitly add the
appropriate foreign key. No more `"commentable_id"`. Again this is a loss of
convenience, this time in exchange for increased clarity in your code and your
DB associations. It will be very obvious exactly which tables are associated to
your "polymorphic" table now, and the framework doesn't need to attempt to
autogenerate sensible names for these associations.

+ No database callbacks in Phoenix. How many times have you been asking *"What in
the ever-loving GOD is happening in my code write now?"*, only to discover that,
unbeknownst to you, some callbacks fired after you did something to a db record,
resulting in fire and brimstone descending from the heavens to wreak havoc upon
your understanding of the system.

## Phoenix way, way too briefly
Phoenix borrows heavily from Rails. Its directory structure, naming conventions,
generators and focus on developer productivity are all straight from Rails.

The seven RESTful action names are copied verbatim (however, there are, in my opinion,
simpler naming conventions when it comes to file name plurality. *Everything* is
singular. No more `DogsController` vs `Dog` model. It is `DogController` and `Dog`).

The MVC architecture is largely copied as-is, with a greater emphasis on separating concerns.
Eg, in Phoenix models generally only deal with database-related things: defining the fields, adding
validations, modeling associations, etc. Another improvement is a greater community emphasis on
modeling your domain *outside* of the MVC-related directories. AKA, your models get data in and
out of the database, your controllers take that data and pass it to modules that handle all business
logic, and the views display that data.

Where does that business logic live, if not in `/models`? In Phoenix, the convention is to have a rich
`/lib` directory with appropriately named subdirectories for modeling your business domain. The MVC directories
are encouraged to be used only for your *web layer*, only for interfacing with the outside world. The actual
logic of your business model is meant to be modeled independently in it's own world as much as possible.

I think this is a good practice that encourages you to think intelligently about how to structure the actual
meat and potatoes of your domain. Just stuffing everything into one flat directory, whatever it's called,
isn't very intention-revealing or maintainable.

Below are some examples of modules in a Phoenix project that import various modules
defined by Phoenix itself, via the Elixir `use` macro. Note the `Phoenix` namespace
before any module that is part of the framework's codebase. Think Phoenix took any
inspiration from Rails?
+ `Phoenix.Router`:
```elixir
  defmodule HelloWeb.Router do
    use HelloWeb, :router

    pipeline :browser do
      plug :accepts, ["html"]
      plug :fetch_session
      plug :fetch_flash
      plug :protect_from_forgery
      plug :put_secure_browser_headers
    end

    pipeline :api do
      plug :accepts, ["json"]
    end

    scope "/", HelloWeb do
      pipe_through :browser # Use the default browser stack

      get "/", PageController, :index
    end

    # Other scopes may use custom stacks.
    # scope "/api", HelloWeb do
    #   pipe_through :api
    # end
  end
```

+ `Phoenix.Controller`:
```elixir
  defmodule MyApp.UserController do
    use MyAppWeb, :controller

    def show(conn, %{"id" => id}) do
      user = Repo.get(User, id)
      render conn, "show.html", user: user
    end
  end
```

+ `Phoenix.Template`, note the `@name` is passed in to the template, it is not
an instance variable. Rendering a template is simply a function call; you can even:
do it in iEx, Elixir's REPL!
```elixir
  # templates/foo.html.eex
  Hello <%= @name %>

  # templates.ex
  defmodule Templates do
    use Phoenix.Template, root: "templates"
  end
```

+ `Ecto.Migration` (the Ecto project is separate from Phoenix):
```elixir
  defmodule Hello.Repo.Migrations.CreateHello.User do
    use Ecto.Migration

    def change do
      create table(:users) do
        add :name, :string
        add :email, :string
        add :bio, :string
        add :number_of_pets, :integer

        timestamps()
      end
    end
  end
```

+ `Ecto.Schema` describes data that lives in the database, and `Ecto.Changeset`
describes how that data should be transformed and validated when moving between
database and application.

```elixir
  defmodule Hello.User do
    use Ecto.Schema
    import Ecto.Changeset
    alias Hello.User


    schema "users" do
      field :bio, :string
      field :email, :string
      field :name, :string
      field :number_of_pets, :integer

      timestamps()
    end

    @doc false
    def changeset(%User{} = user, attrs) do
      user
      |> cast(attrs, [:name, :email, :bio, :number_of_pets])
      |> validate_required([:name, :email, :bio, :number_of_pets])
    end
  end
```

### Metaprogramming
Elixir metaprogramming is apparently very powerful, and uses macros (derived from LISP macros). Phoenix
and even Elixir itself is written largely with macros. I have very little experience with them so I can't
comment much further, except to say that if you love Ruby for its metaprogramming capabilities you should
have similar superpowers in the Elixir world. One thing I keep hearing is how because Elixir is compiled
its metaprogramming generated code doesn't slow things down. Again, don't really know much about it.

+ [Macros](https://elixirschool.com/en/lessons/advanced/metaprogramming/#macros): metaprogram to your heart's content.
Eg, routing in Phoenix makes heavy use of macros such as `use` and `plug`:
```elixir
  defmodule OurMacro do
    defmacro unless(expr, do: block) do
      quote do
        if !unquote(expr), do: unquote(block)
      end
    end
  end

  iex> require OurMacro
  #nil
  iex> OurMacro.unless true, do: "Hi"
  #nil
  iex> OurMacro.unless false, do: "Hi"
  #"Hi"
```

## Erlang Interoperability
Elixir is fully interoperable with Erlang. You can directly call any Erlang module/function
from your Elixir code. That's a 20+ year old ecosystem at your disposal, with cool libraries such as:

+ `Dialyzer` - Gradual typing for higher performance code and fewer bugs.
You can optionally add a [typespec](https://elixir-lang.org/getting-started/typespecs-and-behaviours.html)
to any function definition to specify the data types of its arguments and return value:
```elixir
  @spec round(number) :: integer
  def round(number), do: # implementation...
```

*Ruby connection: Ruby 3 may add optional type annotations, similar to those allowed
by Dialyzer.*
+ [Observer](http://www.akitaonrails.com/2015/11/22/observing-processes-in-elixir-the-little-elixir-otp-guidebook) -
Inspect your process ecocsytem in a variety of ways. Examine a visual graph, or
a process list. See application performance metrics similar to those provided by
NewRelic (for free, but not as robust). Manipulate live processes from a GUI.

![Elixir's Observer GUI](https://cdn-images-1.medium.com/max/2000/1*bD0Wg9J6YEzoOGREqacj5g.png)

+ [OTP](https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html) -
A group of modules and  design patterns for building large-scale concurrent systems
in Erlang/Elixir. How do I provide lightning fast responses, ensure data isn't
lost when hardware fails, and keep my system easy to reason about? Look to OTP.

+ [ETS](https://elixirschool.com/en/lessons/specifics/ets/) - Erlang/Elixir Term Storage,
Lightning fast in-memory key-value store. Safe for concurrent access by multiple processes.

### It's All in the Family
Author and developer Sasa Juric told a story about how he wrote an Erland system
without the need for external dependencies he commonly had used while working in
other ecosystems. Eg:
+ Redis/Memcached -> ETS
+ Application monitoring/metrics -> Observer
+ Queue system (DelayedJob, Resque, RabbitMQ [which is actually written in Erlang])
-> Concurrently running Elixir process on a separate node
+ Database -> Mnesia (if you really want to get hardcore).

He didn't say that this was necessarily a game changer, but did note that it is an
indication of just how deep the Erlang toolset is, and he did feel that having everything
come from your main programming language's ecosystem reduced friction in using these tools.

## Community
+ Beginner friendly, welcoming
+ Growing: packages on [Hex](https://hex.pm/) (their equivalent to RubyGems)
+ Documentation as a first-class citizen

### Resources
+ The Getting Started and OTP [guides](https://elixir-lang.org/getting-started/introduction.html) on [elixir-lang.org](https://elixir-lang.org/)
+ The Phoenix guides on [phoenixframework.org](http://phoenixframework.org/)
+ [Programming Elixir](https://pragprog.com/book/elixir/programming-elixir) by Dave Thomas
+ *[Elixir in Action](https://www.manning.com/books/elixir-in-action) by Sasa Juric*

### Nerves
OMG hardware. Run Elixir on Raspberry Pi and other fun stuff with [Nerves](http://nerves-project.org/).
Elixir makes it a breeze to deal with binary data via pattern matching so connected
devices play nicely with it.

## ¡¿Bonusly?!
OK everyone's on-board with rewriting our entire Rails app in Elixir now, right? :)
But for serious: as our application grows there may be legitimate opportunities to
leverage Elixir (or some other language) where the situation is appropriate.

For example, perhaps we want to begin development on a new feature that is relatively
independent of our existing Rails application. We could spin up a simple web server
and have it communicate with our Rails app behind the scenes.

Maybe this is a performance-critical piece of the application that needs to be able
to process data or serve many users concurrently.

TLDR; If I were starting a new web application project today, I would hands down choose
the Elixir ecosystem to write it in. Fun, fast, concurrent, wheeeeee!
