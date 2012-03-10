These are API files for extending [SciTE](http://www.scintilla.org/SciTE.html) text editor, with autocomplete and calltip definitions, for some of my favorite Python packages.

Archives contain two files inside: <u>regular</u> - which uses SciTE 3.0.4. feature of escape characters in calltip definitions (multi-line), and <u>lite</u> - which can be used with previous SciTE versions, and provides one line function description

### Usage

Locate SciTE's `python.properties` file, and point `api.*.py=` property to the path of API files. For example:

```
api.$(file.patterns.py)=$(SciteUserHome)/.local/share/scite/api/python.api;\
$(SciteUserHome)/.local/share/scite/api/numpy.api;\
$(SciteUserHome)/.local/share/scite/api/scipy.api

calltip.python.end.definition=)
calltip.python.word.characters=._$(chars.alpha)$(chars.numeric)
calltip.python.parameters.start=(
calltip.python.parameters.end=)
calltip.python.parameters.separators=,
```

### In action

![screen-shot](http://i.imgur.com/2M3iM.png "Autocompletion example")

![screen-shot](http://i.imgur.com/3bSCi.png "Rather extensive calltip example")

### Generation

Files are generated with this procedure (example for <u>NumPy</u> package):

```python
import os.path, inspect

def sig(func):
    argspec = inspect.getargspec(func)
    return func.__name__ + inspect.formatargspec(*argspec)

pkg = 'numpy'
i = __import__(pkg, fromlist=[''])

subs_dir = os.path.dirname(i.__file__)
subs = [d for d in os.listdir(subs_dir) if os.path.isfile(os.path.join(subs_dir, d + '/__init__.py'))]

for s in subs:
    p = '%s.%s' % (pkg, s)
    try:
        i = __import__(p, fromlist=[''])
        cont = dir(i)
        
        bloat = ['Notes', 'Examples', 'Raises', 'See Also', 'References', 'Methods']
        
        for c in cont:
            if c and c[0].isalpha() and c[-1].isalnum():
                doc_str = getattr(i, c).__doc__
                for b in bloat:
                    if doc_str: doc_str = doc_str.split(b)[0]
                if doc_str: doc_str = doc_str.replace('\n', '\\n').replace('    ','\\t')
                if inspect.isfunction(getattr(i, c)):
                    print '%s.%s\\n%s' % (p, sig(getattr(i, c)), doc_str)
    except: pass
```

then additionally regex-ed for issues, and compacted to following style:
  
* definitions are recognized if:
	* package is imported explicitly or in common namespace (`import numpy` or `import numpy as np`)
	* sub-packages are imported explicitly within main package or separately explicitly (`import numpy.fft` or `from numpy import fft`)
	* in either case implicit loading is not recommended although API files can easily be changed to that preference
* lines are limited to max. 2000 characters for obvious reasons and function calls formatted to roughly 80 characters (in rare cases)

### Changelog

0.1

* numpy.api reflect numpy 1.6.1
* scipy.api reflects scipy 0.10.0
