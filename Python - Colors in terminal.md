
Python and colored output on Linux terminal
===========================================

There is a nice package called [blessings](https://pypi.python.org/pypi/blessings).

```python
from blessings import Terminal
term = Terminal()
print('I am ' + term.bold + 'bold' + term.normal + '!')
print('I am', term.bold('bold') + '!')
print('All your {t.red}base {t.underline}are belong to us{t.normal}'.format(t=term))
```

Most useful features: `normal`, `bold`, `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`
