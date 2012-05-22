## WHAT

Lexicon is a simple collection of `dict` subclasses providing extra power:

* `AliasDict`, a dictionary supporting both simple and complex key aliasing:
    * Alias a single key to another key, so that e.g. `mydict['bar']` points to
    `mydict['foo']`, for both reads and writes.
    * Alias a single key to a list of other keys, for writing only, e.g. with
    `active_groups = AliasDict({'ops': True, 'biz': True, 'dev': True,
    'product': True})` one can make an alias `'tech'` mapping to `('ops',
    'dev')` and then e.g. `active_groups['tech'] = False`.
* `AttributeDict`, supporting attribute read & write access, e.g.
  `mydict = AttributeDict({'foo': 'bar'})` exhibits `mydict.foo` and
  `mydict.foo = 'new value'`.
* `Lexicon`, a subclass of both of the above which exhibits both sets of
  behavior.

## HOW

* `pip install lexicon`
* `from lexicon import Lexicon` (or one of the superclasses)
* Use as needed.

You can install the [development
version](https://github.com/bitprophet/lexicon/tarball/master#egg=lexicon-dev)
via `pip install lexicon==dev`.

## API

### `AliasDict`

In all examples, `'myalias'` is the alias and `'realkey'` is the "real",
unaliased key.

* `alias(from_='myalias', to='realkey')`: Alias `myalias` to `realkey` so
  `d['myalias']` behaves exactly like `d['realkey']` for both reads and writes.
  See below for details on how an alias affects other dict operations.
* `alias(from_='myalias', to=('realkey', 'otherrealkey'))`: Alias `myalias` to
  both `realkey` and `otherrealkey`. As you might expect, this only works well
  for writes, as there is never any guarantee that all targets of the alias
  will contain the same value.
* `unalias(from_='myalias')`: Removes the `myalias` alias; any subsequent
  reads/writes to `myalias` will behave as normal for a regular `dict`.
* `'myalias' in d` (aka `__contains__`): Returns True when given an alias, so
  if `myalias` is an alias to some other key, dictionary membership tests will
  behave as if `myalias` is set.
* `del d['myalias']` (aka `__del__`): This effectively becomes `del
  d['realkey']` -- to remove the alias itself, use `unalias()`.
* `del d['realkey']`: Deletes the real key/value pair (i.e. it calls
  `dict.__del__`) but doesn't touch any aliases pointing to `realkey`.
    * As a result, "dangling" aliases pointing to nonexistent keys will raise
    `KeyError` on access, but will continue working if the target key is
    repopulated later.

### `AttributeDict`

* `d.key = 'value'` (aka `__setattr__`): Maps directly to `d['key'] = 'value'`.
* `d.key` (aka `__getattr__`): Maps directly to `d['key']`.
* `del d.key` (aka `__delattr__`): Maps directly to `del d['key']`.

### `Lexicon`

Lexicon subclasses from `AttributeDict` first, then `AliasDict`, with the end
result that attribute access will honor aliases. E.g.:

    d = Lexicon()
    d.alias('myalias', to='realkey')
    d.myalias = 'foo'
    print d.realkey # prints 'foo'