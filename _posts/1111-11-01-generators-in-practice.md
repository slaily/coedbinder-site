---
title: "Generators - In Practice"
categories:
  - notebooks
sidebar:
  - title: "Title"
    image: http://placehold.it/350x250
    image_alt: "image"
    text: "Some text here."
  - title: "Another Title"
    text: "More text here."
---

### Table of contents
---
- **Cases**
- **Pipeline (Unix)**
- **Building a real-world _Shell_ command into script**
- **Benchmarking**
- **Summary**
---

### Cases
---

Generators, can be particularly effective and powerful when applied to certain kinds of programming problems in systems.

- **Files**
    - Searching
        - Keywords in text.
        - Word frequency.
        - Syntax errors.
        - Others
    - Filtering
        - Based on specific column in .csv file.
        - Based on combinations.
        - Others
    - Ordering
        - Commands for execution from a configuration file.
        - By unique columns.
        - Others
    - Sorting
        - Memory log files by Memory Consumption status.
        - Others
    - Moving
        - Reorganizing files in a directory or multiple directories.
        - Others
    - Parsing
        - JSON
        - HTML
        - XML
        - Others
    - Templating engines
    - Others
- **Threads**
    - Microthreads.
    - Others
- **Networking**
    - Building an interactive two-player network game - 'Rock paper scissors'.
    - Simultaneously manage client connections.
    - Sending objects through sockets.
    - Others

---
**NOTE**

> The cases that you can apply Generators are not limited to the list above. The aim is to give you more meaningful cases to get an idea and unlock your creativity or get recognized in your problem.
---

### Pipeline (Unix)
---

A pipeline is a series of processes where each process consumes the output of the prior process and produces output for the next. Similar to using a pipe in the UNIX shell. You must have used pipes in the shell before.

Linux:

```bash
ps aux | grep python
```

Windows equivalent:

```powershell
tasklist | find "python"
```

![Pipeline diagram of UNIX ps and aux command](/assets/images/generators-in-practice/pipeline_diagram_unix_ps_aux_command.png)

- The first process is _ps aux_.
- The second process is _grep python_, it consumes the output of the prior process.
- The result.

### Building a real-world Shell command into script
---

One real-world example of an application that we might take is Linux and Unix-like operating systems shell command _wc_. The _wc_ command allows us to count the number of lines, words, characters of each given file.


```python
# Let's see the command in action.
!wc data/molecules/*.pdb
```

---
Let's create a Python script named _wc_command.py_ that does the same thing by using Generators.

Tasks:
- Split the logic into a small processes to form a pipeline.
- Implement

![Pipeline diagram of UNIX ps and aux command](/assets/images/generators-in-practice/pipeline_diagram_unix_wc_command.png)


```python
# wc_command.py
from pathlib import Path


def open_files(files_paths):
    """Opens the files for reading by lazy evaluating them.

    :yields: class:`io.TextIOWrapper`
    """
    for file_path in files_paths:
        with open(file_path, 'r') as file:
            yield file


def counter_generator(texts):
    """Counts the number of lines, words, characters
    for each given text.

    :yields: `tuple`
    """
    for text in texts:
        lines_count = len(text.splitlines())
        words_count = len(text.split())
        characters_count = len(text)

        yield (
            lines_count,
            words_count,
            characters_count
        )


# Yielding all matching files.
files_paths = Path('data/molecules').rglob('*.pdb')
# Open the files.
files = open_files(files_paths)
# Read the files.
files_texts = (file.read() for file in files)

for tl, tw, tc in counter_generator(files_texts):
    # Prints for each text:
    # 1. Number of lines.
    # 2. Number of words.
    # 3. Number of characters.
    print(tl, tw, tc)

```

---
Summarizing, we have a directory called _molecules/_ that contains six files describing some simple organic molecules. The files are with _.pdb_ extension indicates that these files are in Protein Data Format, a simple text format that specifies the type and position of each atom in the molecule. Our program does counting the number of lines, words, and characters for each of the files.

### Benchmarking
---

In this section are going to benchmark the above written code versus the same logic but implemented with use of _list.append(...)_ method.

You are going to see:

- Time execution of the _wc_command.py_ script.
- Time execution of the _wc_command_append.py_ script that use _list.append(...)_ method.
- Memory usage of the _wc_command.py_ script with large collection of test files.
- Memory usage of the _wc_command_append.py_ script that use _list.append(...)_ method with large collection of test files.
- Performance overview



```python
# wc_command.py
"""
The script executing several times wrapped
in docstring source code and measures its
execution time.
"""
code = """
from pathlib import Path


def open_files(files_paths):
    for file_path in files_paths:
        with open(file_path, 'r') as file:
            yield file


def counter_generator(texts):
    for text in texts:
        lines_count = len(text.splitlines())
        words_count = len(text.split())
        characters_count = len(text)

        yield (
            lines_count,
            words_count,
            characters_count
        )


files_paths = Path('data/molecules').rglob('*.pdb')
files = open_files(files_paths)
files_texts = (file.read() for file in files)

for tl, tw, tc in counter_generator(files_texts):
    print(tl, tw, tc)

"""


import timeit
print('{0:.2f}'.format(timeit.timeit(code, number=3)))

```


```python
# wc_command_append.py
"""
The script executing several times wrapped
in docstring source code and measures its
execution time.
"""
code = """
from pathlib import Path


def open_files(files_paths):
    files = []

    for file_path in files_paths:
        files.append(open(file_path, 'r'))

    return files


def read_files(files):
    texts = []

    for file in files:
        texts.append(file.read())
        file.close()

    return texts


def counter(texts):
    results = []

    for text in texts:
        lines_count = len(text.splitlines())
        words_count = len(text.split())
        characters_count = len(text)
        results.append((
            lines_count,
            words_count,
            characters_count
        ))

    return results


files_paths = Path('data/molecules').rglob('*.pdb')
files = open_files(files_paths)
files_texts = read_files(files)

for tl, tw, tc in counter(files_texts):
    print(tl, tw, tc)

"""


import timeit
print('{0:.2f}'.format(timeit.timeit(code, number=3)))

```

---

Memory usage: Implementation with Generators

---
![Memory usage of Generators method](/assets/gifs/generators-in-practice/wc_command_generator_memory_usage.gif "segment")

---

Memory usage: Implementation with _list.append(...)_ method

---
![Memory usage of append method](/assets/gifs/generators-in-practice/wc_command_append_memory_usage.gif "segment")

---
**CAUTION**

> Gifs are real, they are recorded on my workstation. The scripts are tested with the files from _molecules/_ directory, but with the difference that all of the files(.pdb) are around 300mb each one, total directory size >= 2GB. Sorry for the inconvenience that you can't see them in real, but I'm not able to upload them here because they are too large.
---

#### Performance overview
---

Starting with the tests for time execution of the scripts, you can't see the benefit of using Generators, even _list.append(...)_ implementation gave less execution time. In reality, Generators are considered to runs fast in comparison with _list.append(...)_ method usage, but not in every case. Moving forward to the gifs, showing memory usage, there we see graphics and numbers that differs based on the implementation techniques used. As you can see from the version with _list.append(..)_ method, memory got a lot of load and pressure in comparison with Generators. The yield keeps returning the next result until exhausted and only consumes enough memory to hold the generator and a single result at any moment in time - the implementation with _list_ consumes enough memory to hold all of the results at once. When you get to really large files or data sets that can make the difference between something that runs regardless of the size and something that crashes. So yield is slightly more expensive, per operation, but much more reliable and often faster in cases where you deal with large files or don't exhaust the results. In our case, Generators not only saves a memory load and pressure, moreover runs faster.

### Summary
---

- Generators are flexible, can be used in many cases.
- Generators are advanced technique.
- Don't use Generators for small inputs.
- Build a pipeline of Generators.
- Тhe most benefit of Generators is memory consumption.
