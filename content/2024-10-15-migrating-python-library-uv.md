Title: Migrating a Python library to uv
Date: 2024-10-15T11:20:00+04:00
Category: Logiciel

<figure style="float:right; width: 40%">
    <img alt="Elephants walking in Mto Tarangire national park - by Matt Benn on Unsplash" src="{static}/images/elephants.jpg" style="width: 100%; max-height: 300px; height: auto; padding: 0" />
</figure>

Ever since [Hynek recommended it](https://www.youtube.com/watch?v=8UuW8o4bHbw), I've been trying to use the [uv packaging tool](https://docs.astral.sh/uv/). While the uv docs are excellent (including [guides](https://docs.astral.sh/uv/guides/) and [integration guides](https://docs.astral.sh/uv/guides/integration/)), there's a missing part in my opinion: **how do you migrate from existing tooling?** There's no guide to migrate from Poetry or PDM, for example. And more importantly to me, nothing if you want to migrate a library to uv, which is the subject of this blog post.

The typical workflow of a library is different than the workflow of an application:

* Libraries are published to PyPI, not deployed to a Docker registry
* Libraries should allow a wide range of dependency versions, while applications pin them.
* Libraries typically use pip and pip-tools, not Poetry or PDM.

Thankfully, since uv is designed to be the 'one tool' for Python packaging, it supports libraries too. 

## Should you migrate at all?

First, you need to be OK with Astral reimplementing the whole Python packaging ecosystem in Rust without having a clear business model. This has been [discussed at length](https://mastodon.social/@glyph/113093806840686501), with interesting takes from both sides.

Second, there are still technical limitations:

* [uv does not work well with dynamic versions](https://github.com/astral-sh/uv/issues/7533), which is very common in open source libraries
* Since [PEP 751](https://peps.python.org/pep-0751/) is not ready yet, uv uses its own lock file, `uv.lock`. It's not generally supported by other tools:
    * Nox [does not support it](https://github.com/wntrblm/nox/issues/841) (but [has documented workarounds](https://nox.thea.codes/en/stable/cookbook.html#using-a-lockfile)).
    * dependabot [does not support uv.lock](https://github.com/dependabot/dependabot-core/issues/10478) and [alternatives such as gha-update](https://gha-update.readthedocs.io/en/latest/) do not either.

With that said, the experience of using uv itself is really good. You no longer have to mess with virtualenvs, pip-tools, pipx and so on. And the speed is amazing.

## Requirements

OK, let's suppose you do want to migrate. There are a few requirements.

**uv itself** You can install uv with your favorite package manager or see [the installation instructions](https://docs.astral.sh/uv/getting-started/installation/) for more options.

**pyproject.toml** If you still have a setup.py file in your repository, you need to [move to pyproject.toml first](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/). You can stick to setuptools or use an alternative like hatch.

**Python** You don't have to install Python separately! If needed, uv will install Python for you using [python-build-standalone](https://gregoryszorc.com/docs/python-build-standalone/main/index.html). While convenient, it comes with [a number of quirks](https://gregoryszorc.com/docs/python-build-standalone/main/quirks.html). That said, you can choose to install Python yourself, for example using the [excellent Fedora builds](https://fedoralovespython.org/) or the macOS Python.org installer + [MOPup](https://github.com/glyph/MOPUp). If uv finds a suitable Python installation in the PATH, it will use it.

**Other tools** You also don't need to install pip, virtualenv, etc. as uv reimplements those tools. And you can install tools like nox, tox or pre-commit with `uv tool install`.

## Migrating

With that out of the way, let's see how you're going to use uv in the different parts of your project.

**Shell integration** With uv, there's no need to activate your virtualenv. There's also no need for a separate step to install dependencies. The only command needed here is `uv sync`. This command will:

* Install Python if needed
* Create a virtual environment for you, ignoring any existing ones
* Install the dependencies listed in pyproject.toml and create a `uv.lock` file in the process.

When everything is installed and the cache is warm, running `uv sync` is faster than `git status`! Which means that you can run `uv sync` as part of your shell prompt and always have an up-to-date installation.

**Test automation**

* If you use nox, set `@nox.session(venv_backend='uv')` and `UV_PROJECT_ENVIRONMENT` as documented in the nox [cookbook](https://nox.thea.codes/en/stable/cookbook.html#using-a-lockfile.).
* If you only use pytest, run `uv run pytest`.
* If you use tox, use [the tox-uv](https://github.com/tox-dev/tox-uv) plugin.

To avoid running `uvx nox` (or tox) every time, run `uv tool install nox` (or tox) to install it. Now you can simply run `nox` (or tox). This also means you no longer need pipx.

**GitHub Actions** Use the [setup-uv](https://github.com/astral-sh/setup-uv) action. It also supports caching, which is useful for not hitting PyPI servers too hard, not for speed.

**Dependency pinning** While libraries don't pin dependencies in `pyproject.toml`, they still commonly use a pinned requirements file for their test dependencies. If this is your case, you can commit your `uv.lock` file to git. It is cross-platform and supports multiple Python versions! At the time of writing, [Renovate supports it, but dependabot does not](https://docs.astral.sh/uv/guides/integration/dependency-bots/).

**pre-commit** Follow https://docs.astral.sh/uv/guides/integration/pre-commit/.

**twine** Use [uv publish](https://docs.astral.sh/uv/guides/publish/). That said, it's better to use the [PyPI publish GitHub Action](https://github.com/pypa/gh-action-pypi-publish) which is more secure and abstracts the uploading tool for you.

## Examples

Looking for inspiration? Those projects use uv:

 * [stamina](https://github.com/hynek/stamina/)
 * [trustme (local uv branch)](https://github.com/pquentin/trustme/compare/nox...pquentin:trustme:uv?expand=1)
 
Do you know about other projects? Please let me know.


## Conclusion

Migrating your Python library to uv is not as hard as it looks, while bringing a number of benefits:

 * using a single packaging tool,
 * not having to worry about virtualenvs anymore,
 * enjoying categorically faster install speeds,
 * using a lock file that works across Python and OS versions.
