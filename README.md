# What is bytehook?

bytehook is a python module which allows you to modify existing pure python functions while python is already running. It inserts a hook into any line inside the bytecode of the function and allows you to define what the hook will run.

bytehook also allows you to enable / disable certain hookpoints or remove them completely.

# Why use bytehook?

I've developed bytehook as a way to modify a running server without putting pre-existing code to debug it in certain points. While you can use it on your own, I'm using it with [pyrasite](https://github.com/lmacken/pyrasite) in order to inject the hooking code into an already running server.

# How to use

```python
from __future__ import print_function
import bytehook


def test():
    for i in range(4):
        print(i)


if __name__ == "__main__":
    print('1st try')
    test()

    def odd_or_even(locals_, globals_):
        if locals_['i'] % 2:
            print('odd')
        else:
            print('even')

    hookid = bytehook.hook(test, lineno=2, insert_func=odd_or_even, with_state=True)
    print('2nd try')
    test()
    bytehook.disable_hookpoint(hookid)
    print('3rd try')
    test()
```

Output
```
1st try
0
1
2
3
2nd try
even
0
odd
1
even
2
odd
3
3rd try
0
1
2
3
```

# Limitations

* bytehook can only modify pure python functions i.e. functions which are not written in C or using C-API.
* bytehook modification affects only the next entry into the function, so if you have a loop function which never exits, if you inject a hook into it while it's running, it will not load the changes.
* bytehook passes state to your hook function if you want, by calling locals() and globals(), if the arguments are passed by value like an integer, you could not modify it back in the original function upon returning.
* bytehook currently supports CPython 2.7 for now.
* bytehook is currently experimental, and has not been tested with all pure function types.

# Performance impact

The performance impact by default is negligible, if you need to inspect / modify the local / global variables of the hooked function, bytehook will run global()/local() each time it enters the hook function, and pass those in, that might affect performance.
