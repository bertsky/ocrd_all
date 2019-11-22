# OCR-D/ocrd_all

This is a project which gets the source of all OCR-D modules.
It also includes a Makefile for their installation into a virtual environment (venv) or Docker container.
(A venv is a local user directory with shell scripts to load/unload itself
in the current shell environment via PATH and PYTHONHOME.)

## Preconditions

Make sure that there is enough free disk space. 3 GiB or more is recommended for
the required submodules, build data, temporary data, installed virtual environment
and pip cache.

Install GNU make and git, and wget if you want to download Tesseract models.

    # on Debian / Ubuntu:
    sudo apt install make git wget

Install the packages for Python3 development and for Python3 virtual environments
for your operating system / distribution.

    # on Debian / Ubuntu:
    sudo apt install python3-dev python3-venv

Some modules use the Tesseract library. If your distribution provides Tesseract 4.1
or newer, install the development package:

    # on Debian / Ubuntu:
    sudo apt install libtesseract-dev

Ubuntu packages for Tesseract 5.0.0 (alpha) are available at the PPA
https://launchpad.net/~alex-p/+archive/ubuntu/tesseract-ocr-devel.

Otherwise or for the latest Tesseract code it can also be built locally.

Other modules will have additional system dependencies.

System dependencies for all modules on Ubuntu 18.04 (or similar) can also be installed _automatically_ by running:

    # on Debian / Ubuntu:
    sudo apt install make git
    sudo make deps-ubuntu

Moreover, the (shell) environment must have a Unicode-based localization. (Otherwise Python code based on `click` will not work, i.e. most OCR-D CLIs.) This is true for most installations today, and can be verified by:

    locale | fgrep .UTF-8

This should show several `LC_*` variables. Otherwise, either select another localization globally...

    sudo dpkg-reconfigure locales

... or use the Unicode-based POSIX locale temporarily:

    export LC_ALL=C.UTF-8
    export LANG=C.UTF-8


## Usage

Run `make` with optional parameters for _variables_ and _targets_ like so:

    make [PYTHON=python3] [VIRTUAL_ENV=./venv] [target(s)]

### Targets

<dl>
  <dt>ocrd</dt>
  <dd>(default goal) Install OCR-D/core and its CLI `ocrd` into the venv.</dd>
  <dt>all</dt>
  <dd>Install executables from all modules into the venv. (Depends on _modules_ and _ocrd_.)</dd>
  <dt>modules</dt>
  <dd>Download/update all modules, but do not install anything.</dd>
  <dt>deps-ubuntu</dt>
  <dd>Install system packages for all modules</dd>
  <dt>fix-pip</dt>
  <dd>Fix incompatible/inconsistent pip requirements between all modules</dd>
  <dt>docker</dt>
  <dd>(Re-)build a docker image for all modules/executables.</dd>
  <dt>clean</dt>
  <dd>Remove the venv.</dd>
  <dt>show</dt>
  <dd>Print the venv directory, the module directories, and the executable names.</dd>
  <dt>help</dt>
  <dd>Print available targets and variables.</dd>
</dl>

Further targets:
<dl>
  <dt>[any module name]</dt>
  <dd>Download/update that module, but do not install anything.</dd>
  <dt>[any executable name]</dt>
  <dd>Install that CLI into the venv. (Depends on that module and on _ocrd_.)</dd>
</dl>

### Variables

<dl>
  <dt>PYTHON</dt>
  <dd>name of the Python binary to use (at least python3 required)</dd>
  <dt>VIRTUAL_ENV</dt>
  <dd>Directory prefix to use for local installation. (This is set automatically when activating a virtual environment on the shell. The build system will re-use existing venvs.)</dd>
  <dt>PIP_OPTIONS</dt>
  <dd>Extra options to pass to `pip install` (e.g. -q or -v)</dd>
</dl>

### Examples

The following examples assume a working development installation of Tesseract.
To build the latest Tesseract locally, run this command first:

    # Get code, build and install Tesseract with the default English model.
    make tesseract

Optionally install additional Tesseract models.

    # Download models from tessdata_fast into the venv's tessdata directory.
    make frk.traineddata
    make script/Latin.traineddata
    make script/Fraktur.traineddata

Running `make ocrd` or just `make` downloads/updates and installs the `core` module,
including the `ocrd` CLI in a virtual Python 3 environment under `./venv`.

Running `make ocrd-tesserocr-recognize` downloads/updates the `ocrd_tesserocr` module
and installs its CLIs, including `ocrd-tesserocr-recognize` in the venv.

Running `make modules` downloads/updates all modules.

Running `make all` additionally installs the executables from all modules.

### Results

To use the built executables, simply activate the virtual environment:

    . ${VIRTUAL_ENV:-venv}/bin/activate
    ocrd --help
    ocrd-...

For the Docker image, run it with your data path mounted as a user:

    docker run -it -u $(id -u):$(id -g) $PWD:/data ocrd/all
    ocrd --help
    ocrd-...


## Issues

### No published/recent version on PyPI

The following Python modules need an installation from code for different reasons:

- clstm (needs modified code for Python3)
- cor-asv-ann (not available in PyPI)
- cor-asv-fst (not available in PyPI)
- dinglehopper (not available in PyPI)
- ocrd_cis (not available in PyPI)
- tesserocr (too old in PyPI)

### Conflicting requirements

Merging all packages into one venv does not always work.
Modules may require mutually exclusive sets of dependent packages.

_pip does not stop or resolve conflicts – it merely warns._

- `Pillow`:
   * `==5.4.1` (required by ocrd_typegroups_classifier)
   * `>=6.2.0` (required by all others)
- Tensorflow:
   * `tensorflow-gpu==1.14.0` (required by ocrd_calamari and OCR-D-LAYoutERkennung)
   * `tensorflow` (required by cor-asv-ann and ocrd_keraslm)
   
   Both can be installed in parallel in different versions, but may depend on a mutually exclusive set of `tensorboard` and `tensorflow_estimator`.
   
   Moreover, in the future, some modules (but not others) may depend on `tensorflow>=2.0`, which again is incompatible.
- OpenCV:
   * `opencv-python-headlesss` (required by core and others, avoids pulling in X11 libraries)
   * `opencv-python` (required by OCR-D-LAYoutERkennung)
   * custom build on ARM...

- ...

### System requirements

Not all modules advertise their system package requirements via `make deps-ubuntu`.

- clstm: depends on `scons libprotobuf-dev protobuf-compiler libpng-dev libeigen3-dev swig`
