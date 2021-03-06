---
layout: news
title: "Golo 0-preview9 Devoxx Edition"
---
![](http://farm4.staticflickr.com/3788/10740284683_882e78f760_z_d.jpg)

Golo 0-preview is there just in time for [the awesome Devoxx 2013 conference!](http://devoxx.be)

If you are attending Devoxx next week, do not miss our
[Golo Tools in Action session](http://devoxx.be/dv13-julien-ponge.html?presId=3475)
**Tuesday 18:05 - 18:35 Room 9**.

### What’s new?

#### Highlight of this release: class adapters

Sometimes you play with a third-party API, and this third-party API wants you to pass some object
that is an instance of some interface or class.

We now provide you with an API to generate adapter classes at runtime.

The [Spark micro web framework](http://www.sparkjava.com/) is a great example, as it expects HTTP
request handlers to subclass `spark.Route` at some point.

Here is how this can be done in Golo:

{% highlight golo %}
module sparky

import spark
import spark.Spark

function main = |args| {

  # Configuration
  let adaptation = map[
    ["extends", "spark.Route"],
    ["implements", map[
      ["handle", |this, request, response| {
        return args: get(0)
      }]
    ]]
  ]

  # Adapter "factory"
  let fabric = AdapterFabric()
  let routeMaker = fabric: maker(adaptation)

  # Make Spark happy
  get(routeMaker: newInstance("/"))
}
{% endhighlight %}

The API allows for more fancy things, including overriding methods and doing some form of
proxying.

The following snippet is an example where we proxy a list to replicate all `add` method calls into
another list called `carbonCopy`:

{% highlight golo %}
let carbonCopy = list[]
let conf = map[
  ["extends", "java.util.ArrayList"],
  ["overrides", map[
    ["*", |super, name, args| {
      if name == "add" {
        if args: length() == 2 {
          carbonCopy: add(args: get(1))
        } else {
          carbonCopy: add(args: get(1), args: get(2))
        }
      }
      return super: spread(args)
    }
  ]]
]]
let list = AdapterFabric(): maker(conf): newInstance()
list: add("bar")
list: add(0, "foo")
list: add("baz")

# Guess what is inside carbonCopy...
println(carbonCopy)
{% endhighlight %}

#### Classpath flag

The `golo` and `run` subcommands of the `golo` command-line tool now support a `classpath` flag that
adds both jars and folders to the classpath.

The “nice” thing is that it does not use a separator based syntax, hence you can do:

    golo golo --classpath lib/*.jar /some/path /tmp/all/*.jar --files main.golo

#### Classloader improvements

Thanks to the feedback from Julien Viet, we improved the way Golo deals with classloaders. More
specifically, it works fine in container-style environments such as Java application servers.

#### Minor

Our Golo Developer Advocate Philippe contributed some new augmentations: `count` and `exists` on
lists, maps, tuples, vectors and sets.

Reading passwords from the standard console input (`readPassword`) can now be made in a secure fashion.

…and of course we have small fixes there and there.

### Until next time…

- [Try Golo on Google AppEngine](http://golo-console.appspot.com/)
- [Download a preview release of Golo](/download/)
- [Read the Golo programming language guide](/documentation/next/)
- [Fork the project on GitHub](https://github.com/golo-lang/golo-lang)
- [Get in touch on our mailing-list](http://groups.google.com/group/golo-lang)
