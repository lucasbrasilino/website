---
title: "Installing Cocotb with Pyenv on Ubuntu 16.04"
date: 2020-09-12 14:50:00 -0500
categories: [HDL, Verification, Simulation]
tags: [python, cocotb, verilog]
---


Python is  everywhere. Many core components  of a modern distribution  depend on
it. In  many times  this fact  makes difficult  to upgrade  Python to  a version
different from the  distribution default.  There are some  alternatives, such as
installing  newer  versions from  source  code,  installing  it in  a  different
directory,   or   using   third-party    repository   or   PPAs   then   running
`update-alternatives`. However, the latter approach often leads  to problems, 
since we are changing the **global** Python version  (i.e. the one used as default for
the entire system). The former usually works, but you'll need to deal with
setting up `virtualenv`s and a bunch of environment variables.

If for any reason you use an older distribution, chances are that you also want to use a
software like `jupiter`, `numpy` or `TensorFlow` that might ask for an updated
Python version. You are in a dilemma: *"if I update, I break the system, if not,
I can't use cool software out there."*

Enter   [**Pyenv**](https://github.com/pyenv/pyenv){:target="_blank"}.  It   was
developed to resolve  that problem. You can install as  many Python versions you
want,   and   even    use   a   particular   version    *per   directory*   (AKA
_application-specific environment_).  How cool is  that?? I've adopted pyenv for
about 6 months and I'm very happy with it.

I still use Ubuntu 16.04 LTS due to
[proprietary EDAs](https://www.xilinx.com/products/design-tools/vivado.html){:target="_blank"}
and
[related softwares](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html){:target="_blank"} 
I  use in  my work.  So before  adopting Pyenv  I was  somewhat stuck  to Python
versions  2.7.12  and  3.5.2.   To  run  other,  I  used  to  use  the
`virtualenv` + environment variables approach. **Not fun at all!!**. I was often running
into problems.  Pyenv presents an elegant solution for these headaches.

We here go through the procedure to install Pyenv, Python 3.6.8 and Cocotb (at
this time of writing, stable version 1.4.0).
This isn't a complete Pyenv reference. For such, I suggest you to visit
[this page](https://realpython.com/intro-to-pyenv/){:target="_blank"} to learn more.


# Installing pyenv

This section describes how to install Pyenv for a non-root user.
To complete this task you'll need `curl` and `git` installed available in your system.
Installation is as simple as:

```bash
$ curl https://pyenv.run | bash
```

A `~/.pyenv` directory will be created with a clone of its github repository.
Then add the configuration printed out by the installer to your
`~/.bashrc`. Something like:

```
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

Finally, you may re-execute your shell:

```bash
$ exec $SHELL
```

Then do a test by running `pyenv`.

```bash
$ pyenv -v
pyenv 1.2.21
```

# Before installing any Python

A few deb packages are recommended to be available in your system previous to
installing Python for cocotb:

```bash
$ sudo apt install build-essential libz-dev libssl-dev libsqlite3-dev libbz2-dev libreadline6-dev libffi-dev
```

# Installing a Python for cocotb

Run the following command to list which Python versions you can install using Pyenv:

```bash
$ pyenv install --list | less
```

For cocotb, I have installed version `3.8.6`.
**But here is the catch**. cocotb
needs a Python built with `--enable-shared` option, which Pyenv doesn't use by
default. So, to correctly build and install Python under Pyenv, execute:

```bash
$ PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install -v 3.8.6
```

You can check if version 3.8.6 was installed with:

```bash
$ pyenv versions
* system (set by /home/lucas/.pyenv/version)
  3.8.6
```

# Setting up an application-specific environment

Here comes a pretty cool functionality provided by Pyenv. You can setup an
**application-specific Python environment**. Each environment is contained within a
directory. This means you can have as many Python versions you
want. When you `cd` to a directory, Pyenv "automagically" sets the environment
for that directory. This setup is accomplished by the `pyenv local <VERSION>` command.

Naturally we will setup an environment for cocotb. Let's say that it will be in the
`~/cocotb` directory. Starting from user's home directory, you'll do:

```bash
$ mkdir cocotb
$ cd cocotb
$ python -V
Python 2.7.12
$ pyenv local 3.8.6
$ python -V
Python 3.8.6
```

As you can see, Python 3.8.6 is enabled for the `~/cocotb` directory.

# Installing auxiliary Python packages

After Pyenv finishes the previous step, `pip` and
 `setuptools` are already available (current directory is `~/cocotb`):

```bash
$ pip list
Package    Version
---------- -------
pip        20.2.1
setuptools 49.2.1
```

If a pip version warning shows up, you can update it with:

```bash
$ python -m pip install --upgrade pip
```

However, you will need to install [Python `wheel`](https://pythonwheels.com/) as well:

```bash
$ pip install wheel
$ pip list
Package    Version
---------- -------
pip        20.2.4
setuptools 49.2.1
wheel      0.35.1
```

# Installing cocotb

You may install cocotb in two ways. Using `pip` (easier) or using source
code. Let's go through both ways.

## Using pip

Remember we still need to be in `~/cocotb`. Just use pip to install cocotb.
The latest stable version will be installed:

```bash
$ pip install cocotb
$ pip list
Package    Version
---------- -------
cocotb     1.4.0
pip        20.2.4
setuptools 49.2.1
wheel      0.35.1
```

Another cool thing is that `cocotb-config` will already be in your `$PATH`:

```bash
$ cocotb-config --version
1.4.0
```

**Very convenient, huh??**

Now you can download cocotb [examples](https://github.com/cocotb/cocotb/tree/stable/1.4/examples) and run.

## From source code

Installing cocotb from source code is also very easy. We'll use the repository
in this example, however you can also use tarballs available at cocotb's github repo.

Since we'll be installing version 1.4.X, we will checkout `stable/1.4` branch.
Again, we are starting from `~/cocotb`:

```
$ git clone https://github.com/cocotb/cocotb.git cocotb-repo
$ cd cocotb-repo
$ git checkout stable/1.4
$ python setup.py install
```

**Note that using the repository, you may be installing a version, say 1.4.1,
  which might not be available for installing using `pip`.**
  

I hope Pyenv will be useful to you on running various Python systems as it was
for me. I encourage you to try cocotb + Pyenv approach. Using both together has been very
advantageous for sure!
