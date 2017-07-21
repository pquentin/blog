Title: Indenting JSON in Python
Date: 2017-07-17T09:00:00+04:00
Category: Logiciel

<img alt="Main JSON railroad diagram - http://www.json.org/fatfree.html" src="{filename}/images/json_value.gif" style="float: right; max-width:50%; max-height: 300px; height:auto; padding: 0 1em 1em 0"/>

At work, I recently wanted to visualize how we annotated documents
that we store as JSON in Elasticsearch. We annotate substrings with
different codes, and I wanted to show those codes using colors in a
terminal. This basically boils down to outputting JSON with [ANSI
escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code). The
easier part was inserting color codes inside the strings. But once
that was done, I could not simply print the resulting JSON, because
the escape codes would be escaped, not interpreted in my terminal!

In other words, I had to write a JSON encoder that would look exactly
like a JSON encoder, except that it would not escape anything. (Note
that this requires trusting the input.)

The interesting part was indentation: I wanted the result to be
indented with two spaces at each level, as if I had used
`json.dumps(indent=2)`. Printing `[1, [2, 3], 4]` should give this:

```
[
  1,
  [
    2,
    3
  ],
  4
]
```

When I looked at the wanted output above, I was tempted to print line
by line, prepending each value with the correct output. It did not
really go well, and after a few special cases I gave up and checked
how [CPython did
it](https://github.com/python/cpython/blob/master/Lib/json/encoder.py).

Their algorithm, which I ended up adopting, is more natural for a
recursive solution. The key observation is that you only have two
types of JSON objects that affect spacing: *object* (dict) and *array*
(list and tuple). If you handle them correctly, the other ones can
simply be added to the output without any spaces.

So, for lists, you need to output:

 * the opening `[` plus the correct indentation until the first item,
 * all items separated by commas and indentation,
 * the indentation between the last item and `]`.

You don't need to worry about printing the items themselves: just call
your function recursively! Here is how it looks like with ints and
lists:

```
def output_unescaped_json(value, *, indent=0):
    out = ''
    next_indent = indent + 2

    if isinstance(value, int):
        out += str(value)
    elif isinstance(value, list):
        # opening [ and indentation until first item
        out += '[\n' + ' ' * next_indent
        # each item separated by commas and indentation
        out_items = [
            output_unescaped_json(item, indent=next_indent)
            for item in value
        ]
        sep = ',\n' + ' ' * next_indent
        out += sep.join(out_items)
        # indentation between the last item and ]
        out += '\n' + ' ' * indent + ']'
    else:
        assert False, type(value)

    return out
```

As with textbook recursive algorithms, it is elegant because you're
only solving one problem at a time, but it can be tricky to get to the
solution, convince you that it works, and debug it. If you want to
make sure you fully understand what is going on, try working out the
dictionary case (`{'key': 'value'}`).

You may be wondering: why is this a recursive algorithm? This must
mean it's [not
pythonic](http://neopythonic.blogspot.fr/2009/04/tail-recursion-elimination.html),
right? While it would be possible to rewrite this as an iterative
algorithm, it's still going to traverse the whole tree, and the
iterative algorithm will only be harder to read for marginal gains,
and this is the kind of optimizations that CPython avoids doing. And
if your JSON is so deep that you exceed the recursion limit, you have
other problems to solve first. :)

However, there is an obvious optimization that I left out: using
`yield` and `yield from` and build the resulting string with a single
`join` call. But that requires adding another function for the `join`
call. I decided to keep things simple here.

<!-- vim: spelllang=en
-->
