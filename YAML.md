YAML
====

http://yaml.org


Python
------

https://pypi.python.org/pypi/pyyaml

```
$ pip install pyyaml
```

Features
--------

```yaml
key_one: &anchor
  sub_key_one: value one
  sub_key_two: value two
key_two:
  sub_key_three: value three
  <<: *anchor
```
