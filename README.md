pytestify
=========

A tool to automatically change unittest to pytest. Similar to
[unittest2pytest](https://github.com/pytest-dev/unittest2pytest),
but with a few more features and written using AST and tokenize, rather
than lib2to3.

Big thanks to [pyupgrade](https://github.com/asottile/pyupgrade/), which
this project has learned from.

## Installation

`pip install pytestify`

## Usage

`pytestify path/to/file.py`

or

`pytestify path/to/folder/`

Please read over all changes that pytestify makes. It's a new
package, so there are bound to be issues.

## Implemented features

### Test class names

Remove TestCase parent class, and make sure tests start with `Test`

```python
class TestThing(unittest.TestCase):  # class TestThing:
class TestThing(TestCase, ClassB):   # class TestThing(ClassB):
class ThingTest(unittest.TestCase):  # class TestThing:
class Thing(unittest.TestCase):      # class TestThing:
```

### Setup / teardowns

```python
def setUp(self):          # def setup_method(self):
def tearDown(self):       # def teardown_method(self):
def setUpClass(self):     # def setup_class(self):
def tearDownClass(self):  # def teardown_class(self):
```

### Asserts

Rewrite unittest assert methods using the `assert` keyword.

```python
# asserting one thing
self.assertTrue(a)       # assert a
self.assertFalse (a)     # assert not a
self.assertIsNone(a)     # assert a is None
self.assertIsNotNone(a)  # assert a is not None

# asserting two things
self.assertEqual(a, b)      # assert a == b
self.assertNotEqual(a, b)   # assert a != b
self.assertIs(a, b)         # assert a is b
self.assertIsNot(a, b)      # assert a is not b
self.assertIn(a, b)         # assert a in b
self.assertNotIn(a, b)      # assert a not in b
self.assertListEqual(a, b)  # assert a == b
self.assertDictEqual(a, b)  # assert a == b
self.assertSetEqual(a, b)   # assert a == b
self.assertItemsEqual(a, b) # assert sorted(a) == sorted(b)
self.assertGreater(a, b)    # assert a > b
self.assertLess(a, b)       # assert a < b
self.assertGreaterEqual(a, b)  # assert a >= b
self.assertLessEqual(a, b)  # assert a <= b
self.assertRegex(a, b)      # assert a.search(b)
self.assertNotRegex(a, b)   # assert not a.search(b)

self.assertAlmostEqual(a, b)
#   assert a == pytest.approx(b)
self.assertAlmostEqual(a, b, places=2)
#   assert a == pytest.approx(b, abs=0.01)

# error messages
self.assertTrue(a, msg='oh no!')  # assert a, 'oh no!'
```

Note: `assertCountEqual` is NOT supported here. It is tricky to implement, and pytest has voiced
[hesitations](https://github.com/pytest-dev/pytest/issues/5548) about including it in
the near future. You can still use [unittest's implementation](https://stackoverflow.com/a/45946306).

### Multi-line asserts

Since `assert (a == b, 'err')`  is equivalent to asserting a tuple, and thus is always `True`.

```python
self.assertEqual(    # assert \
    a,               #     a == \
    b,               #     b
)

self.assertEqual(    # assert \
    a,               #     a == \
    b,               #     b,
    msg='oh no!'     #     'oh no!'
)
```

### Exceptions

```python
self.assertRaises(OSError)             # pytest.raises(OSError)
self.assertWarns(OSError)              # pytest.warns(OSError)
with self.assertRaises(OSError) as e:  # with pytest.raises(OSError) as e
with self.assertWarns(OSError) as e:   # with pytest.warns(OSError) as e
```

### Skipping / Expecting failure

```python
# decorated
@unittest.skip('some reason')    # @pytest.mark.skip('some reason')
@unittest.skipIf(some_bool)      # @pytest.mark.skipif(some_bool)
@unittest.skipUnless(some_bool)  # @pytest.mark.skipif(not some_bool)
@unittest.expectedFailure        # @pytest.mark.xfail

# not decorated
unittest.skip('some reason')     # pytest.skip('some reason')
unittest.skipTest('some reason') # pytest.skip('some reason')
unittest.fail('some reason')     # pytest.fail('some reason')
```
