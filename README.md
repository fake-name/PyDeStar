PyDeStar
=========


Introduction
============

Do you have python scripts that use `from xxx import *` from half a dozen packages?
Is their code nearly impossible to follow as a result?

PyDeStar walks the AST of your python script, and rewrites your code with fully qualified
names.


How:
---------

This handles arbitrary cases with star imports.

All star imports are replaced with flat imports:

```
from os.path import *

val = join("yes", "path")
```

Is converted to

```
import os.path

val = os.path.join("yes", "path")
```


The star-imported libraries are scanned for exported symbols.

All functions and variables are examined. Any names which are assigned to are masked off.
Function calls and constants used in the `rhs` of an assignment are checked to see if they
exist in the star-imported libraries, and if so, the relevant use is updated to use the
full `os.path.xxx` or similar name.

Note that this is *not* a perfect tool, but if you're dealing with horrible code that
star-imports a lot of different packages, it can fix probably >95% of the star-imported stuff.

It won't work for dynamic attribute lookup (`getattr()`), or other high-dynamism approaches,
but I don't think there's a way of fixing those sort of things short of runtime
inspection of all the function calls.

It also can run into issues where one library star-imports another, because symbols can
then be present in multiple star-imported packages. It chooses the package with the
shortest name, which is basically an arbitrary decision.

This uses some string-munging hacks, mostly rather then rewriting the AST, and mirroring
the rewritten AST back to source via `astor`, it instead uses the AST to determine precise
strings which are then used to search+replace in the text source on a per-line basis.
This was primarily chosen because it was far easier to implement, though it means that it is
probably possible to come up with a malicious input source file that would result in a
harmful output. However, this tool assumes it'll be used as part of a programmer-driven
refactoring with a human in the loop, which would render such things harmless.

Basically, it's a refactoring tool, more then a daily-use fixer. OTOH, for
one-iff refactoring large codebases/scripts that have bad practices, it is hugely useful.

---------------


This was based originally on the `autoflake` tool (https://github.com/myint/autoflake), but
after it languished as a open PR for more then two years (https://github.com/myint/autoflake/pull/31),
and I periodically revisited it when dealing with random scripts, I've decided to package it up as it's
own thing.



