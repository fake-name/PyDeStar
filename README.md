PyDeStar
=========


Introduction
============

Do you have python scripts that use `from xxx import *` from half a dozen packages?
Is their code nearly impossible to follow as a result?

PyDeStar walks the AST of your python script, and rewrites your code with fully qualified
names.

Usage:

```
pydestar.py <file.py> <-i (to write changes)>
```

Requires the `astor` package as it's only dependency. Currently tested on >= python3.5.


What
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

How:
---------

 - The star-imported libraries are scanned for exported symbols.
 - All functions and variables are examined.   
   Any names which are assigned to are masked off.  
   Function calls and constants used in the `rhs` of an assignment are checked to see if they
exist in the star-imported libraries, and if so, the relevant use is updated to use the
full `os.path.xxx` or similar name.
 - The input source file line is permuted with the generated changes

Notes and Caveats:
---------

Note that this is *not* a perfect tool, but if you're dealing with horrible code that
star-imports a lot of different packages, it can fix probably >95% of the star-imported stuff.

It won't work for dynamic attribute lookup (`getattr()`), or other high-dynamism approaches,
but I don't think there's a way of fixing those sort of things short of runtime
inspection of all the function calls.

It also can run into issues where one library star-imports another, because symbols can
then be present in multiple star-imported packages. It chooses the package with the
shortest name, which is basically an arbitrary decision.


Basically:
```  
library_1
 (contains thing)
   +    +
   |    |
   |    +>library_2
   |     (from library_1 import thing)
   |        +
   |        |
   v        v
Target script
from library_1 import *
from library_2 import *

```

Since the tool sees "thing" in both library_1 and library_2, it may replace
references to "thing" with "library_2.thing". This will function, though it 
can be confusing.

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



