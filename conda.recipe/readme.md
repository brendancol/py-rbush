## Release Procedure

- Ensure all tests pass.

- Update version number in `conda.recipe/meta.yaml`, `rbush/__init__.py`,
  and `setup.py`. Commit.

- Tag commit and push to github

```bash
git tag -a x.x.x -m 'Version x.x.x'
git push upstream master --tags
```

- Build conda packages

The exact procedure is platform/setup specific, so I'll define a few variables
here, to fill in with your specifics:

```bash
# Location of your conda install. For me it's `~/anaconda/`
CONDA_DIR="${CONDA_PREFIX%%anaconda/*}anaconda"

# Platform code. For me it's `osx-64`
PLATFORM=osx-64

# Version number of rbush being released (e.g. 0.2.0)
#VERSION=0.2.0
VERSION=$(python -c 'import rbush;print(rbush.__version__.split("+")[0])')
```

This assumes `conda`, `conda-build`, and `anaconda-client` are installed (if
not, install `conda`, then use `conda` to install the others). From inside the
toplevel directory:

```bash
cd conda.recipe/
conda build --python 2.7 .
conda build --python 3.5 .
conda build --python 3.6 .
```

Next, `cd` into the folder where the builds end up.

```bash
cd $CONDA_DIR/conda-bld/$PLATFORM
```

Use `conda convert` to convert over the missing platforms (skipping the one for
the platform you're currently on):

```bash
conda convert --platform osx-64 rbush-$VERSION*.tar.bz2 -o ../
conda convert --platform linux-64 rbush-$VERSION*.tar.bz2 -o ../
conda convert --platform win-64 rbush-$VERSION*.tar.bz2 -o ../
```

Use `anaconda upload`. This requires you to be setup on `anaconda.org`,
and have the proper credentials to push.

```bash
anaconda login
anaconda upload $CONDA_DIR/conda-bld/*/rbush-$VERSION*.tar.bz2 -u parietal.io
```

- Write the release notes:

 1. Run `git log` to get a listing of all the changes
 2. Remove any covered in the previous release
 3. Summarize the rest to focus on user-visible changes and major new features
 4. Paste the notes into github, under *n* `releases`, then `Tags`, then `Edit release notes`.
