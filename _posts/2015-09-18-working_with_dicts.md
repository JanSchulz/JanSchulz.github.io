---
layout: post
title: "Two functions for working with JSON/dicts"
comments: True
---

I recently had to explore a JSON API and came up with the following two functions to make working with the returned JSON/dict easier:

```python
def get_from_dict(data, name):
    names = name.split(".")
    for n in names:
        try: 
            i = int(n)
            data = data[i]
        except:
            data = data.get(n, "None")
        if data is None:
            raise Exception("Key not found: %s (%s)" % (n, name))
    return data

def find_in_dict(data, text):
    _stack = []
    def _find(data, stack):
        if isinstance(data, list):
            for i, value in enumerate(data):
                stack.append(str(i))
                if _find(value, stack):
                    return True
                else:
                    stack.pop()
        elif isinstance(data, dict):
            for key, value in data.items():
                stack.append(key)
                if _find(value, stack):
                    return True
                else:
                    stack.pop()
        elif data == text or text in data:
            return True
        return False 
    if _find(data, _stack):
        return ".".join(_stack)
    else:
        raise Exception("Not found in data: %s" % (text,))
```

Example:

```python
>>> data = {"result":[{"name":"Jan Schulz"}, {"name":"Another name", "bday":"1.1.2000"}]}
>>> print(find_in_dict(data, "Schulz"), get_from_dict(data, find_in_dict(data, "Schulz")))
('result.0.name', 'Jan Schulz')
```

Someone might find this useful (and at least I can find it again :-) )