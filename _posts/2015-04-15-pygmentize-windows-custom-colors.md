---
layout: post
title: Pygmentize on Windows with custom colorscheme
tags: [pygmentize, windows, cmd, colorscheme, TerminalFormatter, JsonLexer]
---

While working on project demo I was searching for a way to demonstrate rest api via console with predefined batch files.

Rest service responses are json, and it will be nice to highlight them right inside console.

There is nice utility [pygmentize](http://pygments.org/) which allows you do exactly that.

Notice: For colors to work you must install [ansicon](http://adoxa.altervista.org/ansicon/)

The only thing I do not like about it is default colors:

![Pygmentize default colorscheme](/images/pygmentize/before.png)

So here is a way to customize them:

```python
import sys
from pygments import highlight
from pygments.lexers import JsonLexer
from pygments.formatters import TerminalFormatter
from pygments.token import Keyword, Name, Comment, String, Error, Number, Operator, Generic, Token, Whitespace


# https://github.com/nex3/pygments/blob/master/pygments/formatters/terminal.py
cs = {
    Token:              ('darkgray', 'darkgray'),

    Whitespace:         ('', ''),
    Comment:            ('', ''),
    Comment.Preproc:    ('', ''),
    Keyword:            ('white', 'white'),
    Keyword.Type:       ('', ''),
    Operator.Word:      ('', ''),
    Name.Builtin:       ('', ''),
    Name.Function:      ('', ''),
    Name.Namespace:     ('', ''),
    Name.Class:         ('', ''),
    Name.Exception:     ('', ''),
    Name.Decorator:     ('', ''),
    Name.Variable:      ('', ''),
    Name.Constant:      ('', ''),
    Name.Attribute:     ('', ''),
    Name.Tag:           ('lightgray', 'lightgray'),
    String:             ('yellow', 'yellow'),
    Number:             ('fuchsia', 'fuchsia'),

    Generic.Deleted:    ('', ''),
    Generic.Inserted:   ('', ''),
    Generic.Heading:    ('', ''),
    Generic.Subheading: ('', ''),
    Generic.Error:      ('', ''),

    Error:              ('', ''),
}

#data = '{"foo": "bar", "ing": 2, "bool": false}'
data = ''.join(sys.stdin.readlines())

print highlight(data, JsonLexer(), TerminalFormatter(colorscheme=cs))
```


and here is how it looks like:

![Pygmentize default colorscheme](/images/pygmentize/after.png)

now values are bright, keys are grayed and tokens are almost not visible, I bed you can do it event more readable ;)

In my case usage was something like this:

```sh
ps.cmd -method get -uri /_cat/indices |
```
