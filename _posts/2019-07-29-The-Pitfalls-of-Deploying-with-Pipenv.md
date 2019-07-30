---
layout: post
title: "The Pitfalls of Deploying with Pipenv"
tags: [Pipenv, Python, Deploy, Deploy]
categories: [Devops]
description: Deploying with Pipenv is easy, except when it's not. Some best practices and pitfalls to be aware of.
---

#  Pipenv (and cron) Deployment Pitfalls

Well I just had a hellish 8 hour troubleshooting session. Meaning, I need to write down what I shouldn't do next time. Admitedly, much of this time was spent fighting with cron, but Pipenv has also earned itself a blog post. 

## CD into the Pipenv environment

I'm not really sure why I thought that I could just execute my python script from anywhere using `Pipenv run python full/path/to/foo.py`. In my head, Pipenv would understand which environment it was in because the full path is given to the python script. (I'm not sure this is all that unreasonable; the use case of directly calling python in one environment using a different environment is nonsense.) Apparently Pipenv needs the working directory to be within the virtual environment. 

## Beware of the `PATH` if you are using Pipenv from cron

The `PATH `variable is quite important when it comes to Pipenv, as the environment specific packages must come before your Python distribution or system packages. Cron has a "feature" which changes the `PATH` before it executes the cron job. 

I've seen some debate around the two solutions to this problem: using absolute paths everywhere or declaring your `PATH` at the top of script ran by cron. Unfortunately, since Python relies on the `PATH` variable to find packages in the first place, you'll have to go with the second option. I'd recommend making a bootstrap script like so. 

```bash
PATH="/bin:/usr/bin:whatever/is/in/here"
cd my-Pipenv-env/
Pipenv run foo.py
```

## Use the `--deploy` flag with Pipenv

It can be very confusing when your dependencies don't install because your `Pipfile` and `Pipfile.lock` are out of sync. The `--deploy` flag will throw and error if they are not. For example `Pipenv --python 3.7 install --deploy` If you run into the error simply run `Pipenv update` back in your dev environment to get them in sync, commit your `Pipfile` and `Pipfile.lock`, and deploy again.

## Pyenv will likely yell at you about dependencies

When I first stumbled upon [Pyenv](https://github.com/pyenv/pyenv) I was ecstatic. I thought that I could simply use it in conjunction with Pipenv to deploy just about any python package, targeting any Python version, on any machine - all in an automated fashion. In many instances you can, but Pyenv doesn't actually ensure that the dependencies to build the specific Python version you're targeting are satisfied. Oftentimes, you'll run into some complaint about your `gcc` compiler. You can read more about 

## Pyenv is not meant to be installed and used within one script

The above is somewhat understandable as you wouldn't necessarily want your system installed compilers to be overwritten. What I can't understand is why Pyenv is not installable completely automatically by default. The [pyenv-installer](https://github.com/pyenv/pyenv-installer) will do much of the leg work for you, but requires a shell restart. If you use this within another shell script, you'll have some issues with stale environment variables. I had to combine `pyenv-installer` with some manual variable manipulation to install and use it within one shell script.

```bash
function installPyenv() {
    sudo rm -r ~/.pyenv
    curl https://pyenv.run | bash
    PYENV_ROOT="${HOME}/.pyenv"
    export PYENV_ROOT
    echo "export PATH=\"${PYENV_ROOT}/bin:\$PATH\"" >> ~/.bash_profile
    echo "eval \"\$(pyenv init -)\"" >> ~/.bash_profile
    echo "eval \"\$(pyenv virtualenv-init -)\"" >> ~/.bash_profile
    . ~/.bash_profile
}
```
