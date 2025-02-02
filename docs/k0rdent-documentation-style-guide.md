# k0rdent documentation contributors' guide

k0rdent documentation is growing rapidly, and community members are very welcome to contribute by filing pull requests against documents in this repository. Thank you for your interest in k0rdent!

This page gives technical guidance for contributing to k0rdent documentation. It is _not_ a complete style guide. In most cases, Project k0rdent aspires to follow style recommendations adopted by the Kubernetes Project's SIG Docs group. Please see the [Kubernetes Documentation Style Guide](https://kubernetes.io/docs/contribute/style/style-guide/) for details.

## How k0rdent docs are written

k0rdent documentation is written mostly in [CommonMark](https://commonmark.org/) as adopted by GitHub, with a few CommonMark-compatible markdown enhancements used by mkdocs and extensions (see below) to enable more-readable rendering of code, glossary items, images, and admonitions (e.g., NOTE:, TIP:, WARNING:, etc.). This guide details our markdown style, along with those extensions. See below.

## How k0rdent docs are built

k0rdent documentation is built from markdown (.md) documents using the [mkdocs](https://www.mkdocs.org/) static site generator with the [material](https://squidfunk.github.io/mkdocs-material/) theme. We use several mkdocs extensions and plugins for rendering glossary items, admonitions (e.g., Note, Tip, Warning, etc.) and other special kinds of content.

Below you'll find instructions for setting up mkdocs locally to build the k0rdent documentation from a local clone of the docs repository: handy for reviewing the rendered look and feel of changes and additions before filing a pull request.

## Install mkdocs to build k0rdent docs locally

mkdocs can be run on almost any contemporary Linux environment, desktop or server, inside a container, or on MacOS, Windows, or WSL. It requires recent (in k0rdent's case, 3.9+) versions of Python and pip.

    > NOTE:
    > k0rdent's mkdocs environment requires Python 3.9+ and a corresponding version of pip. 

Assuming you have a machine (desktop, laptop, VM) for editing with appropriate versions of Python and pip:

1. Begin by cloning the k0rdent / docs repo locally using the git CLI or any git helper application.

    > NOTE:
    > the k0rdent / docs repo has a .gitignore already created in its root that will ignore the local /venv folder
    > and its contents you will create in step 3, below.

2. Switch into the cloned project directory:
   
   ```shell
   cd docs
   ```

3. Create a venv into which to install mkdocs and dependencies, saved in the /docs repo in a file called `requirements.txt`.

   ```shell
   python3 -m venv venv
   source venv/bin/activate
   ```

   Your terminal should now show `(venv)` in the prompt. This will create a local /venv folder in your copy of the repo, which the
   already-present .gitignore will ignore when you make commits and file pull requests. 

4. Install mkdocs and dependencies in the venv, using `requirements.txt`.

    > NOTE:
    > The file `requirements.txt` is in the /docs folder in k0rdent / docs
   
   ```shell
   pip install -r /docs/requirements.txt 
   ```

Once you've done this, you can execute mkdocs to dynamically build and serve docs locally on localhost (`http://127.0.0.0:8000`):

```shell
python3 mkdocs serve
```
