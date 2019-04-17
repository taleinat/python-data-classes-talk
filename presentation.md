# Data Classes in Python

<div style="text-align:center">
Scaling with Python @ Fundbox
</div>

<div style="text-align:center">
April 17<sup>th</sup> 2019
</div>

<div style="text-align:center">
Tal Einat
</div>

Notes:
- A few words about myself
- TODO

---

# Why data classes?

---

## Toy example: Text search hits

<div class="centered">
(Think regular expression matches.)
</div>

```C
struct Match {
    int start;
    int end;
    char* text;
}
```

---

## Native data types: tuple

```python
match = (a, b, c)
```

Usage example:

```python
match_length = match[1] - match[0]
assert text[match[0]: match[1]] == match[2]
```

VVV

Slightly better:

```python
start, end, text = match
assert text[start, end] == text
```

VVV

Oops!

```python
match_start, match_end, match_text = match
assert text[match_start, match_end] == match_text
```

---

## Tuples: Pros and Cons

<div class="col-container">

<div class="col">

<p><strong>Pros:</strong></p>
<ul>
<li>Native data type</li>
<li>Simple</li>
<li>Fast</li>
<li>Low memory overhead</li>
<li>Immutable</li>
<li>Tuple unpacking</li>
</ul>

</div>

<div class="col">

<p><strong>Cons:</strong></p>
<ul>
<li>Opaque</li>
<li>Unpacking → Intermediate variables</li>
<li>Non-extensible</li>
<li>Immutable</li>
</ul>

</div>

</div>

Notes:

- opaque: neither the class or attrs are named
- extensible: need to change, or at least check, all use sites
    - and there's no good way to find them
- immutable: not always wanted

---

## Native data types: dict

```python
match = {'start': a, 'end': b, 'text': c}

match = {
    'start': a,
    'end': b,
    'text': c,
}

match = dict(
    start=a,
    end=b,
    text=c,
)
```

VVV

Usage examples:

```python
match_length = match['end'] - match['start']
```
```python
assert text[match['start']:match['end']] == match['text']
```

---

## Dicts: Pros and Cons

<div class="col-container">

<div class="col">

<p><strong>Pros:</strong></p>
<ul>
<li>Native data type</li>
<li>Simple</li>
<li>Fast</li>
<li>Low memory overhead</li>
<li>Mutable</li>
</ul>

</div>

<div class="col">

<p><strong>Cons:</strong></p>
<ul>
<li>Unnamed</li>
<li>No dict unpacking</li>
<li>Can add arbitrary keys, including non-strings</li>
<li>Keys can be missing
<pre><code>text[match.get('start', 0):
     match.get('end', len(text))]</code></pre>
<li>Mutable</li>
</ul>

</div>

</div>

Notes:

- "I'll just throw this in this dict to fetch it elsewhere..."
- Missing keys → defensive programming
- Mutable: No frozen dict!

---

## namedtuples

```python
from collections import namedtuple

Match = namedtuple('Match', 'start end text')
```

```python
match = Match(a, b, c)
# or
match = Match(start=a, end=b, text=c)
```

Usage example:

```python
match_length = match.end - match.start
assert text[match.start: match.end] == match.text
```

VVV

It's actually still a tuple:

```python
match_length = match[1] - match[0]
assert text[match[0]: match[1]] == match[2]
```

```python
start, end, text = match
assert text[start, end] == text
```

Notes:

- I made the same mistake again!

---

## namedtuples: Pros and Cons

<div class="col-container">

<div class="col">

<p><strong>Pros:</strong></p>
<ul>
<li>In the stdlib</li>
<li>Named</li>
<li><code>isinstance()</code></li>
<li>Fast, low memory</li>
<li>Immutable + <code>._replace()</code></li>
<li>Tuple unpacking</li>
<li>repr, hash, comparisons</li>
<li><code>._asdict()</code><li>
</ul>

</div>

<div class="col">

<p><strong>Cons:</strong></p>
<ul>
<li>Customize by inheritance</li>
<li>Attribute and index access</li>
<li>Immutable</li>
<li>Deep, dark magic!</li>
<li>Surprising</li>
</ul>

</div>

</div>

Notes:

- Inheritance breaks the default repr
- Class definition generated and `eval()`-ed at runtime
- IDEs still struggle
- uses `__new__()`

VVV

```python
class Match(namedtuple('_Match', 'start end text')):
    @property
    def length(self):
        return self.end - self.start
```

VVV

## namedtuple surprises

```python
>>> (0, 5, 'abcde') == Match(0, 5, 'abcde')
True
```
```python
>>> len(Match(0, 5, 'abcde'))
3
```
```python
>>> match, = Match(0, 5, 'abcde'))
>>> match
0
```

---

## Plain classes

```python
class Match:
    def __init__(self, start, end, text):
        self.start = start
        self.end = end
        self.text = text
```

```python
match_length = match.end - match.start
```

```python
assert text[match.start:match.end] == match.text
```

Notes:

- IMO this is better: clearer

---

## Plain classes: Pros and Cons

<div class="col-container">

<div class="col">

<p><strong>Pros:</strong></p>
<ul>
<li>Straightforward</li>
<li>Named</li>
<li><code>isinstance()</code></li>
<li>IDE support</li>
<li>Mutable (or not!)</li>
<li>Flexible</li>
</ul>

</div>

<div class="col">

<p><strong>Cons:</strong></p>
<ul>
<li>Flexible, but DIY</li>
<li>Extremely verbose (boilerplate!!)</li>
<li>Unpythonic?</li>
<li>Can add arbitrary keys</li>
<li>Keys can be missing</li>
</ul>

</div>

</div>

Notes:

- Uncommon in most codebases

VVV

## Flexibility by DIY

```python
def __repr__(self):
    return 'Match(start={}, end={}, text={})'.format(
        self.start, self.end, self.text,
    )
```

```python
def __repr__(self):
    return 'Match(start={start}, end={end}, text={text})'.format(
        **self
    )
```

Notes:

- eq, hash, str, copying, serialization
- Uncommon in most codebases

---

## Raymond Hettinger @PyCon

<div style="text-align:center">
![There must be a better way](https://media.giphy.com/media/3scoQYue48MeZL2bBi/giphy.gif "Dataclasses, featuring Raymond Hettinger in a GIF.")<!-- .element height="70%" width="70%" -->
</div>

---

## attrs: Classes Without Boilerplate

```python
import attr

@attr.s
class Match:
    start = attr.ib()
    end = attr.ib()
    text = attr.ib()
```

```python
match_length = match.end - match.start
```

```python
assert text[match.start:match.end] == match.text
```

Notes:

- "cute" short names
- similar to namedtuple

---

## attrs: Pros and Cons

<div class="col-container">

<div class="col">

<p><strong>Pros:</strong></p>
<ul>
<li>Looks like a class def</li>
<li>Concise, declarative</li>
<li>Named, <code>isinstance()</code></li>
<li>Mutable (or not!)</li>
<li>Flexible <strong>without DIY</strong></li>
<li>Less magic than namedtuple</li>
<li>Lots of extra features...</li>
</ul>

</div>

<div class="col">

<p><strong>Cons:</strong></p>
<ul>
<li>3<sup>rd</sup> party</li>
<li>Slightly magic</li>
<li>"Cute" syntax (has alternative)</li>
<li>Learning curve for advanced features</li>
<li>"Use whichever variant fits your taste better."</li>
</ul>

</div>

</div>

Notes:

- Popular, well-maintained
- Example features:
    - "frozen"
    - hash caching

---

## dataclasses: "attrs in the stdlib"

```python
from dataclasses import dataclass

@dataclass
class Match:
    start: int
    end: int
    text: str
```

```python
match_length = match.end - match.start
```

```python
assert text[match.start:match.end] == match.text
```

Notes:

- Inspired by attrs and heavily based on it
- Must use type annotations on all attributes
    - Use `typing.Any` when needed
- Relies on maintaining attribute definition order
    - Python 3.6+
- attrs also supports using type annotations

---

## dataclasses: Pros and Cons

<div class="col-container">

<div class="col">

<p><strong>Pros:</strong></p>
<ul>
<li>Looks like a class def</li>
<li>stdlib → IDE support!</li>
<li>Concise, declarative</li>
<li>Named, <code>isinstance()</code></li>
<li>Mutable (or not!)</li>
<li>Flexible <strong>without DIY</strong></li>
<li>Even less magic than attrs</li>
<li>Lots of extra features...</li>
</ul>

</div>

<div class="col">

<p><strong>Cons:</strong></p>
<ul>
<li>Python 3.7+ (backport for 3.6)</li>
<li>New</li>
<li>Learning curve for advanced features</li>
<li>Requires type annotations</li>
</ul>

</div>

</div>

Notes:

- Popular, well-maintained
- Example features:
    - "frozen"
    - hash caching

---

## Real example: fuzzysearch

```python
class LevenshteinSearchParams(object):
    def __init__(self,
                 max_substitutions=None,
                 max_insertions=None,
                 max_deletions=None,
                 max_l_dist=None):
        self.check_params_valid(max_substitutions, max_insertions,
                                max_deletions, max_l_dist)

        self.max_substitutions = max_substitutions
        self.max_insertions = max_insertions
        self.max_deletions = max_deletions
        self.max_l_dist = self._get_max_l_dist(
            max_substitutions, max_insertions,
            max_deletions, max_l_dist,
        )
```

VVV

```python
@property
def unpacked(self):
    return (
        self.max_substitutions,
        self.max_insertions,
        self.max_deletions,
        self.max_l_dist,
    )
```

VVV

```python
@classmethod
def check_params_valid(cls,
                       max_substitutions, max_insertions,
                       max_deletions, max_l_dist):
    if not all(x is None or (isinstance(x, int) and x >= 0)
               for x in 
               [max_substitutions, max_insertions, max_deletions, max_l_dist]):
        raise TypeError(
            "All limits must be positive integers or None.")

    if max_l_dist is None:
        n_limits = (
            (1 if max_substitutions is not None else 0) +
            (1 if max_insertions is not None else 0) +
            (1 if max_deletions is not None else 0)
        )
        if n_limits < 3:
            if n_limits == 0:
                raise ValueError('No limitations given!')
            elif max_substitutions is None:
                raise ValueError('# substitutions must be limited!')
            elif max_insertions is None:
                raise ValueError('# insertions must be limited!')
            elif max_deletions is None:
                raise ValueError('# deletions must be limited!')
```

VVV

## Simplified

```python
class Params:
    def __init__(self,
                 one=None, two=None,
                 three=None, four=None):
        self._validate(one, two, three, four)

        self.one = one
        self.two = two
        self.three = three
        self.four = four

        self._post_init()

    @classmethod
    def _validate(cls, one, two, three, four):
        # validations logic

    def _post_init(self):
        # additional initialization logic
```

Notes:

- Actually, we've left just the boilerplate!
- This is the boring, uninteresting stuff!
- Mutable! Because I hadn't thought about it.
- No repr
- No proper comparison, hashing

---

## Simplified, with attrs

```python
from attrs import attrs, attrib

@attrs
class Params:
    one = attrib(default=None)
    two = attrib(default=None)
    three = attrib(default=None)
    four = attrib(default=None)

    def __attrs_post_init__(self):
        # validation logic
        # additional initialization logic
```

VVV

## Simplified, with dataclasses

```python
from dataclasses import dataclass

@dataclass
class Params:
    one: int = None
    two: int = None
    three: int = None
    four: int = None

    def __post_init__(self):
        # validation logic
        # additional initialization logic
```

---

# Uncertainty

- This is a top enemy of software systems
- Tends to grow quickly
- Many things we do aim to address this:
    - Testing
    - Error handling
    - Defensive programming
        - `data.get('key', None)`
        - `try: ... except Exception:`
- Certainity → Organizational Scalability

Notes:

- Don't use bare `except:`!

---

## Immutability

- A basic concept in functional languages
- Growing adoption in Javascript circles
- Useful examples: configuration
- Immutability promotes certainty
    - An object won't suddenly change
    - Changes done explicitly by making copies

VVV

### Immutability in our example

```python
@dataclass(frozen=True)
class Params:
    # ...
    
    def __post_init__(self):
        # validation logic
        self.four = min(self.four, self.one + self.two + self.three)
```

Notes:

- Identical with attrs

VVV

### replacing attributes

```python
p = Params(1, 2, 3, 4)
```

```python
attrs.evolve(p, four=42)
```

```python
dataclasses.replace(p, four=42)
# similar to namedtuple._replace()
```

---

## Default values

_Avoid mutable defaults!_

```python
def foo(items=[]):
```

A common pattern:

```python
def foo(items=None):
    if items is None:
        items = []
```

```python
def foo(items=None):
    items = items or []
```

---

## Default values and factories

```python
@dataclass
class Foo:
    a: int = 0
    b: List[int] = field(default_factory=list)
    c: Any = None
```

Notes:

- Similar in attrs
- `Any` here just FYI

---

## Slots

- An obscure feature of Python classes
- Hard-codes the list of attribute names
- Allows memory optimization
- Turns typos in assignment into errors
    - rather than assigning an arbitrary attribute

Plain Python object example:

```python
class Param:
    __slots__ = ['one', 'two', 'three', 'four']
    def __init__(self, one, two, three, four):
        # ...
```

Notes:

- This is just an example of a corner you might run into.
- Slots is notorious for making trouble. Just use `frozen` when possible.
- If you need mutable data objects, slots can be useful.

VVV

## Slots (cont.)

- attrs: `@attrs(slots=True)`
- dataclasses:
    - doesn't have special support for slots
    - Worse, with dataclasses, using slots severly limits the usable features.
        - No default values
        - Generally can't use `field()`
    - There are workarounds for this...

---

# Extensibility

- tuples and namedtuples are hard to extend
- dicts are easy to extend
    - ... too easy! Can be extended at any place in the code.
- Plain classes make extending much safer
    - But, updating all of the boilerplate is very fragile!
    - Nobody writes unit tests for every data class...
- attrs / dataclasses give the best of both worlds
    - Both allow sub-classing.
    - Mixing with `@property` is tricky, though.

Notes:

- I've seen dict-hacking lead to horrible messes many times.
- I personally don't recommend sub-classing data classes.
- "consenting adults" approach

---

## Final Remarks

- I highly recommend using dataclasses if you can, attrs otherwise.
- "Explicit is better than implicit"
- Easy → Actually used

---

## Example recommended use cases

- Configuration
- Functions with common or similar signatures
    - E.g. call chains
- Components and module boundaries
    - Part of the contract / interface

---

# Questions?
