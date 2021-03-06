# Intro

This project demonstrates using [PyInstaller](https://github.com/pyinstaller/pyinstaller) to create an executable binary (`exe` or `executable`) on Windows, Mac and Linux. Here's a good [tutorial](https://realpython.com/pyinstaller-python/). In this project, the intention was to demonstrate `HOWTO` distribute a Python desktop application (executable) but with some data science packages (dependencies). In this project's case, `scikit-learn`. 

# Dependency Setup

## Quick and Dirty (QED)

Note for this demo, you need to install 2 libraries. If you do not care and just want to install the dependencies into the current Python environment, type in the following. (We will talk more later about `joblib`, yuck).

```bash
pip install scikit-learn pyinstaller joblib==0.11
```

## Conda

If you're using `conda`.

```bash
conda env create -f environment.yml
conda activate python-executable
```

To update the environment through the `yaml` file.
```bash
conda env update -f environment.yml
```

To remove the environment.

```bash
conda remove --name python-executable --all
```

# PyInstaller TL;DR

With `PyInstaller`, you may create an executable that may be distributed as one directory, `onedir`, or one single file, `onefile`. To generate the `onedir` or `onefile` distributions, type in the following. Note that these commands will create a file `my-app.spec`, which you may then modify to further customize your build.

```bash
pyinstaller cli.py --name my-app --onedir --additional-hooks-dir=.
pyinstaller cli.py --name my-app --onefile --additional-hooks-dir=.
```

The `my-app.spec` file generated by the commands above
were renamed according to the following. 
* `my-app.onedir.spec`
* `my-app.onefile.spec`

Now, you may run pyinstaller against the spec file to generate the distributions.

```bash
pyinstaller my-app.onedir.spec
pyinstaller my-app.onefile.spec
```

Notice the use of the `hook-*.py` file. This file helps
pyinstaller to find hidden submodules. Look at the [official
documentation](https://pythonhosted.org/PyInstaller/hooks.html#understanding-pyinstaller-hooks) 
or [SO](https://stackoverflow.com/questions/20602721/pyinstaller-a-module-is-not-included-into-onefile-but-works-fine-with-oned).

# Troubleshooting

## Mac

On Mac, you will have to clear out the `build` and `dist` directories if they exists.

```bash
rm -fr build/ dist/ && pyinstaller my-app.onedir.spec
rm -fr build/ dist/ && pyinstaller my-app.onefile.spec
```

Also seems like you need to use `multiprocessing.freeze_support()`, otherwise, the program is loaded multiple times. But that does not seem like the solution either. Even after the program exits, on OSX, you can still see the process running. The best solution was downgrading `joblib==0.11`.

## Linux

On Linux, it will complain about `OSError: Python library not found: libpython3.7.so.1.0, libpython3.7mu.so.1.0, libpython3.7m.so.1.0`. You need to copy over the file as follows.

```bash
sudo cp ~/anaconda3/envs/python-executable/lib/libpython3.7m.so.1.0 /usr/lib
```

## Windows

I can't believe it, but, on Windows, `PyInstaller` worked without a hiccup! +1 for Windows.

# Notes
Note the following heartaches.

* `joblib` is the main culprit behind your distributed executable file running over and over. The [Internets](https://en.wikipedia.org/wiki/Internets) cite `multiprocessing` as the culprit, but only after downgrading `joblib` did I get stable behavior.
* You cannot expect PyInstaller to work without some manual steps across Windows, Mac and Linux (as you can see from the above). 
* Notice the `hook-sklearn.py` file? PyInstaller attempts to intelligently and efficiently map out your dependencies and includes them in the final executable distribution. However, in some cases, the dependency heuristic algorithm fails and you need to help PyInstaller pull in the dependencies. The best practice I've seen is to create a `hook-*.py` file for each module (package). In this example, we're helping PyInstaller pull in transitive (or hidden) dependencies from `scikit-learn`. 
* `onedir` vs `onefile` point of view. PyInstaller recommends `onedir` for easier debugging and `onefile` to remove cognitive load on your intended users. I have noticed that `onedir` is faster to execute; I suspect that `onefile` being a big file bundling all your dependencies have a runtime overhead; email me with the technical explanation if you go that far. You have to pick your poison. ;)
* You should only run `PyInstaller` once to create the `my-app.spec` (spec file). This first step bootstraps your subsequent build commands as you will then directly modify the spec file and pass it in as a parameter.
* Notice that the `main` [guard](https://docs.python.org/3/library/__main__.html) is in `cli.py`? That's the recommended pattern. `cli.py` has the main guard so it will be your main entry point. You can then build switching/conditional logic to call other programs; in this case, the `main` method in `doit.py`.

# References

This project demonstration was kludged from the University of [The Internets](https://en.wikipedia.org/wiki/Internets). 

* [PyInstaller loads script multiple times](https://stackoverflow.com/questions/32672596/pyinstaller-loads-script-multiple-times)
* [Issue 3907](https://github.com/pyinstaller/pyinstaller/issues/3907)
* [Issue 1921](https://github.com/pyinstaller/pyinstaller/issues/1921)
* [PyInstaller exe keeps opening itself](https://www.bountysource.com/issues/70003153-pyinstaller-exe-keeps-opening-itself)
* [PyInstaller app opens multiple copies in endless loop on osx](https://stackoverflow.com/questions/54942950/pyqt5-pyinstaller-app-opens-multiple-copies-in-endless-loop-on-osx)
* [PyInstaller error, oserror python library not found](https://stackoverflow.com/questions/43067039/pyinstaller-error-oserror-python-library-not-found-libpython3-4mu-so-1-0-lib)

# Take a Look!

Check out [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin). He's my among my favorite coding gurus.

# Citation

```
@misc{vang_python_exe_2019, 
title={HOWTO distribute a Python executable to Windows, Mac, and Linux}, 
url={https://github.com/vangj/python-executable}, 
journal={GitHub},
author={Vang, Jee}, 
year={2019}, 
month={Jun}}
```

# Copyright Stuff

```
Copyright 2019 Jee Vang

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
