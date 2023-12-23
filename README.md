This is a **Windows-only** reproducer for https://github.com/pytest-dev/pytest/issues/9765.
There is an [ongoing PR](https://github.com/pytest-dev/pytest/pull/11708) by
`@fcharras` that would fix it.

To reproduce:
```bash
git clone https://github.com/lesteve/pytest-issue-9765-reproducer
cd pytest-issue-9765-reproducer

python setup.py develop
pytest
```

<details>

<summary>Traceback</summary>

```
Traceback (most recent call last):
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Scripts\pytest-script.py", line 9, in <module>
    sys.exit(console_main())
             ^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 192, in console_main
    code = main()
           ^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 150, in main
    config = _prepareconfig(args, plugins)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 331, in _prepareconfig
    config = pluginmanager.hook.pytest_cmdline_parse(
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_hooks.py", line 493, in __call__
    return self._hookexec(self.name, self._hookimpls, kwargs, firstresult)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_manager.py", line 115, in _hookexec
    return self._inner_hookexec(hook_name, methods, kwargs, firstresult)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_callers.py", line 130, in _multicall
    teardown[0].send(outcome)
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\helpconfig.py", line 104, in pytest_cmdline_parse
    config: Config = outcome.get_result()
                     ^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_result.py", line 114, in get_result
    raise exc.with_traceback(exc.__traceback__)
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_callers.py", line 77, in _multicall
    res = hook_impl.function(*args)
          ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 1078, in pytest_cmdline_parse
    self.parse(args)
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 1428, in parse
    self._preparse(args, addopts=addopts)
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 1330, in _preparse
    self.hook.pytest_load_initial_conftests(
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_hooks.py", line 493, in __call__
    return self._hookexec(self.name, self._hookimpls, kwargs, firstresult)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_manager.py", line 115, in _hookexec
    return self._inner_hookexec(hook_name, methods, kwargs, firstresult)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_callers.py", line 152, in _multicall
    return outcome.get_result()
           ^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_result.py", line 114, in get_result
    raise exc.with_traceback(exc.__traceback__)
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\pluggy\_callers.py", line 77, in _multicall
    res = hook_impl.function(*args)
          ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 1156, in pytest_load_initial_conftests
    self.pluginmanager._set_initial_conftests(
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 563, in _set_initial_conftests
    self._try_load_conftest(anchor, importmode, rootpath)
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 585, in _try_load_conftest
    self._getconftestmodules(x, importmode, rootpath)
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 609, in _getconftestmodules
    mod = self._importconftest(conftestpath, importmode, rootpath)
          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\rjxQE\AppData\Local\miniforge3\envs\pytest\Lib\site-packages\_pytest\config\__init__.py", line 657, in _importconftest
    assert mod not in mods
AssertionError
```

</details>

Our investigation with `@fcharras` shows that in our case we need the
combination of three things for the issue to appear:
- a `conftest.py` inside the package (`project/conftest.py`)
- a plugin inside the package, early loaded with `pytest -p`
- install the package in editable mode either with:
  ```
  python setup.py develop
  ```
  or with the following `pip` command (which I think does something similar as
  the previous one):
  ```
  pip install -e . --no-use-pep517 --no-build-isolation
  ```
