Title: Generate your own indented JSON
Date: 2017-07-17T09:00:00+04:00
Category: Logiciel

<img alt="Main JSON railroad diagram - http://www.json.org/fatfree.html" src="{filename}/images/json_value.gif" style="float: right; max-width:50%; max-height: 300px; height:auto; padding: 0 1em 1em 0"/>

At work, I recently wanted to visualize how we annotated documents
that we store as JSON in Elasticsearch. We annotate substrings with
different codes, and I wanted to show those codes using colors. This
basically boils down to outputting JSON with [ANSI escape
codes](https://en.wikipedia.org/wiki/ANSI_escape_code). The easier
part was inserting color codes inside the strings. But once that was
done, I could not simply print the resulting JSON, because the escape
codes would be escaped, not interpreted in my terminal!

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

It turns out that the natural way to do it is to indent *after* each
value, not before. In other words, instead of saying "ok I'm in a
list, let's indent by two spaces before printing the value", you'll
say "ok, I'm in a list, I'll print the list value and print the two
spaces before the next value".

Let's say we restrict ourselves to nested lists of integers (as
in the above example), this would look like:

```
def output_unescaped_json(value, *, indent=0):
    out = ''
    next_indent = indent + 2

    if isinstance(value, int):
        out += value
    elif isinstance(value, list):
        out += '[\n' + ' ' * next_indent
        sep = ',\n' + ' ' * next_indent
        for item in value:
            out += output_unescaped_json(item, indent=next_indent)
            out += sep
        out = out[:-len(sep)]
        out += '\n' + ' ' * indent + ']'
    else:
        assert False, type(value)

    return out
```

In the list case, we first simply print `[`, without having to worry
about indentation, because we know we're already at the correct place.
Then we add a newline, plus the needed indentation. Next, we output:

 * each item with a recursive call,
 * the comma,
 * the newline,
 * and the indentation needed before the next item.

Since all this is not needed for the closing `]`, we remove it after
the loop, which is simpler that figuring when not to print it. And we
finally print the closing `]`.

I found this to be simpler than trying to do this the other way
around. I actually *tried* to do it the other way around, and when I
got stuck, I looked at [how CPython does
it](https://github.com/python/cpython/blob/master/Lib/json/encoder.py),
and used the same method.

Sorry if this is unclear. If you're really interested, try working out
the dictionary case (`{'key': 'value'}`). This is the best way to
ensure you fully understand what's going on.

<!-- vim: spelllang=en
-->
