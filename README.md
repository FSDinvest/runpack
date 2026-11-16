# runpack

PyArmor **production** runtime (runtime id `015668`, Linux x86_64) for the obfuscated Odoo modules of the FSDI ecosystem.

It is generated with the registered **PyArmor Pro** license of FSDINVEST (product `FSDiPyArm`).
For development and testing against the trial runtime see `runpack_trial`.

## What it is

Odoo modules obfuscated with PyArmor import their runtime as a top-level package:

```python
from pyarmor_runtime_015668 import __pyarmor__
```

This repository provides that package (`pyarmor_runtime_015668`) so it can be installed once with `pip`,
instead of copying the runtime into every module. It is the same idea as Exo's `runpack`.

- Runtime id: `015668` (PyArmor Pro license of FSDINVEST).
- Platform: `linux.x86_64` (what Odoo.sh runs). It does not run on Windows or macOS.

The `.so` in `pyarmor_runtime_015668/` is specific to one CPython version: `pyarmor.cli.core` ships a
different binary per Python minor version, so a single tag can only serve one. Pin the tag that matches
the Python version of the Odoo.sh stage you are building for.

| Tag     | Python (Odoo.sh stage) | Odoo version |
|---------|-------------------------|--------------|
| `python_3.12` | 3.12              | Odoo 18      |

## Use it in a project

Add the line matching your stage's Python version to the `requirements.txt` at the root of the project
(the repository Odoo.sh builds):

```
runpack @ git+https://github.com/FSDinvest/runpack.git@python_3.12   # Python 3.12
```

Pin a tag so the runtime only changes when you decide. The build must be able to read this repository
(public, or a token in the URL for a private one). If the Odoo.sh stage's Python version does not match
the tag, the module fails to import with an `undefined symbol` error from `pyarmor_runtime.so`.

Odoo.sh installs `requirements.txt` on every build. Nothing else is needed: the modules find
`pyarmor_runtime_015668` on the Python path.

## Build modules against this runtime

Build with the **same Python version** as the runtime you are targeting (`pyarmor.cli.core` resolves its
own binary from the interpreter that runs it, and pulls in `pyarmor.cli.core.linux` for the cross-platform
`.so` on first use):

```bash
# use the Python interpreter matching the target tag above, e.g. a 3.12 install for python_3.12
python -m pyarmor.cli gen -r -O dist/<module> --platform linux.x86_64 \
    --use-runtime /path/to/runpack <module_source>
```

`--use-runtime` points at a **folder that contains the `.pyarmor.ikey`** of the tag you are targeting
(only the key is needed, not the `pyarmor_runtime_015668` folder).

The `.pyarmor.ikey` is **not** in this repository. It is the build key that pairs with the runtime of each
tag: it is only used when obfuscating, never at run time, and it is never installed on Odoo.sh. Keep one
copy per tag in the internal source repositories (one folder per tag, e.g. `python_3.12/`) and never in
a repository delivered to a customer. Without the key of a tag, new modules cannot be built against that
tag. Regenerating the runtime creates a new runtime and a new key, which means a new tag and rebuilding
all the modules.

Keep `__manifest__.py`, XML and CSV files in clear text; only the `.py` files are obfuscated.
Do not put a `pyarmor_runtime_015668` folder inside the modules.

## Do not commit

The PyArmor registration file (`pyarmor-regfile-*.zip`), the registration code, any license file
and any `.pyarmor.ikey` / `pyarmor.rkey` key files.
