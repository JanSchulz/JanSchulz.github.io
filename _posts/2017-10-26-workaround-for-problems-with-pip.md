---
layout: post
title: "Workarounds for TypeError: unsupported operand type(s) for -=: 'Retry' and 'int'"
comments: True
---

Using `pip` on a Debian/Ubuntu system can result in this error: `TypeError:
unsupported operand type(s) for -=: 'Retry' and 'int'`. E.g. see this error
report in Debian: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=870701

The problem seems to be that `requests`/`pip` vendors some libraries and
Debian/Ubuntu un-vendors them again. This results in the same code imported
multiple times under different names. Funnily the above exception results from
such a mismatch if a network request has an error/ times out. I tried to
follow the [explanaition why this results in the above
error](https://github.com/shazow/urllib3/issues/567), but gave up. :-)

Anyway: we use `python3.6 -m venv ...` to create virtualenvs and afterwards
`pip` to install requirements into the virtualenv. Sometimes the error happened
and I couldn't finish the installations even after multiple tries. As the
problem was that `pip` uses an un-vendored version of the `requests` library, the
easiest way to fix it was to switch `pip` back to use a vendored `requests`
library:

```bash
# this assumes the virtualenv is created in '.venv'
# This will copy the unbundled versions of the libs from /usr/share/python-wheels
# into .venv/share/python-wheels
(md .venv && cd .venv && $(SYSTEM_PYTHON36) -m venv --copies .)
# install a clean copy of requests...
# will also install chardet/idna/urllib3/certifi and so on into .venv
# this might stil fail with the above error...
.venv/bin/pip install requests wheel
# remove the unbundled versions of the libs
(cd .venv/share/python-wheels/ && rm requests-*.whl chardet-*.whl urllib3-*.whl)
```

Afterwards `pip install --requirement=requirements.txt` now succeeded.

If you also need a system pip, follow this instructions:
https://stackoverflow.com/a/37531821/1380673)
