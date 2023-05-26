# Bitextor installation

Bitextor can be installed via Docker, Conda or built from source.

## Docker installation

Bitextor is available via Docker:

```bash
# download latest release:
docker pull bitextor/bitextor
# OR master branch nightly build:
# docker pull bitextor/bitextor:edge
docker run --name bitextor bitextor/bitextor
```

For more information about Docker installation and usage consult [our wiki](https://github.com/bitextor/bitextor/wiki/Bitextor-Docker).

## Conda installation

Same as with Docker, Bitextor can be easily installed using a Conda environment with the following commands:

```bash
BITEXTOR_CONDA_CHANNELS="-c conda-forge -c bitextor -c bioconda -c dmnapolitano -c esarrias"

conda install $BITEXTOR_CONDA_CHANNELS bitextor
```

For latest updates, nighty version is available (new versions are only released when major features/bug fixes are introduced):

```bash
conda install $BITEXTOR_CONDA_CHANNELS bitextor-nightly
```

If you want a concrete version, you can look in the [Anaconda Repository](https://anaconda.org/anaconda/repo) or use the following command:

```bash
conda search -c bitextor bitextor
```

Be aware that you might need Hunspell dictionaries for Bicleaner and Bicleaner AI due to Hunspell. Check [FastSpell](https://github.com/mbanon/fastspell) for more instruction about Hunspell dictionaries.

In order to install Miniconda or Anaconda you can follow the instructions of the [official page](https://conda.io/projects/conda/en/latest/user-guide/install/index.html), but if you want to install Miniconda (Linux x64), you should execute the following (it is an interactive installer, so you will need to follow the steps):

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

If you are experiencing troubles installing new versions of Bitextor in your environment, you can try the following commands:

```bash
# Be sure you do not have any other versions installed
conda uninstall bitextor
conda uninstall bitextor-nightly

# Remove old and cached packages which might be installing other unexpected dependencies/versions
conda clean --all
```

Currently we only support Linux x64 for Conda environment.

## Manual installation

Step-by-step Bitextor installation from source.

### Download Bitextor's submodules

```bash
# if you are cloning from scratch:
git clone --recurse-submodules https://github.com/bitextor/bitextor.git

# otherwise:
git submodule update --init --recursive
```

### Required packages

These are some external tools that need to be in the path before installing the project. If you are using an apt-like package manager you can run the following commands line to install all these dependencies:

```bash
# mandatory:
sudo apt install git time python3 python3-venv python3-pip golang-go build-essential cmake libboost-all-dev liblzma-dev time curl pigz parallel

# optional, feel free to skip dependencies for components that you don't expect to use:
## wget crawler:
sudo apt install wget
## warc2text:
sudo apt install uchardet libuchardet-dev libzip-dev
## Bicleaner and Bicleaner AI:
sudo apt install autopoint libhunspell-dev
### Hunspell dictionaries (example)
sudo apt install hunspell-es
## Biroamer:
sudo apt install libgoogle-perftools-dev libsparsehash-dev
## Heritrix, PDFExtract and boilerpipe:
sudo apt install openjdk-8-jdk
## PDFExtract:
## PDFExtract also requires protobuf installed for CLD3 (installation instructions below)
sudo apt install autoconf automake libtool ant maven poppler-utils apt-transport-https ca-certificates gnupg software-properties-common
```

If you are using a RPM based system, use these instead:

```bash
# mandatory:
sudo dnf install git time python-devel python3-pip golang-go cmake pigz parallel boost-devel xz-devel uchardet zlib-devel gcc-c++
## sentence splitter:
sudo dnf install java-latest-openjdk-devel.x86_64
## Moses Perl tokenizer
sudo dnf install perl-FindBin perl-Time-HiRes perl-Thread
## warc2text:
sudo dnf install uchardet-devel libzip-devel
## Bicleaner and Bicleaner AI
sudo dnf install hunspell hunspell-devel
### Hunspell dictionaries (example)
sudo dnf install hunspell-es
## Bicleaner:
sudo dnf install gettext-devel gcc-gfortran python3-devel openblas-devel lapack-devel
```

### C++ dependencies

Compile and install Bitextor's C++ dependencies:

```bash
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=$HOME/.local ..
# other prefix can be used, as long as 'bin' is in the PATH and 'lib' in LD_LIBRARY_PATH
make -j install
```

Optionally, it is possible to skip the compilation of the dependencies that are not expected to be used:

```bash
cmake -DSKIP_MGIZA=ON -DCMAKE_INSTALL_PREFIX=$HOME/.local .. # MGIZA is used for dictionary generation
# other dependencies that can optionally be skipped:
# WARC2TEXT, PREVERTICAL2TEXT, DOCALIGN, BLEUALIGN, HUNALIGN, BIROAMER, KENLM
```

If you are installing Bitextor within a conda environment, you might like to execute the following commands instead of the provided above:

```bash
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX="$CONDA_PREFIX" ..
CPATH="$CONDA_PREFIX/include" LD_LIBRARY_PATH="$CONDA_PREFIX/lib" make -j install
```

### Golang packages

Additionally, Bitextor uses [giashard](https://github.com/paracrawl/giashard) for WARC files preprocessing.

```bash
# build and place the necessary tools in $HOME/go/bin

# mandatory
tools="giashard"
# all tools (optional)
#tools="giashard giashardid giamerge giastat"

for tool in $(echo "$tools"); do
  go install github.com/paracrawl/giashard/cmd/${tool}@latest
done
```

### Pip dependencies

Furthermore, most of the scripts in Bitextor are written in Python 3. The minimum requirement is Python>=3.7.

Some additional Python libraries are required. They can be installed automatically with `pip`. We recommend using a virtual environment to manage Bitextor installation.

```bash
# create virtual environment & activate
python3 -m venv /path/to/virtual/environment
source /path/to/virtual/environment/bin/activate

# install dependencies in virtual enviroment
pip3 install --upgrade pip
# bitextor (mandatory dependencies):
pip3 install .
# additional dependencies:
pip3 install git+https://github.com/MSeal/cython_hunspell@2.0.3
pip3 install ./third_party/bifixer
pip3 install ./third_party/bicleaner
pip3 install ./third_party/bicleaner-ai[transliterate]
pip3 install ./third_party/kenlm --config-settings="--build-option=--max_order=7" # Bicleaner and Bicleaner AI dependency
pip3 install ./third_party/biroamer && \
python3 -c "from flair.models import SequenceTagger; SequenceTagger.load('flair/ner-english-fast')"
## neural
pip3 install ./third_party/neural-document-aligner
pip3 install ./third_party/vecalign
```

If you don't want to install all Python requirements in `requirements.txt` because you don't expect to run some of Bitextor modules, you can comment those `*.txt` in `requirements.txt` and rerun Bitextor installation. Also, there are different optional packages that you can choose depending on your needs instead of install all of them:

```bash
# Install all the optional dependencies
pip3 install .[all]
# Install warc2preprocess
pip3 install .[w2p]
# Install dictionary pipeline dependencies instead of MT
pip3 install .[dictionary]
```

### [Optional] Heritrix

[Heritrix](https://github.com/internetarchive/heritrix3) is Internet Archive's web crawler. To use it in Bitextor, first download Heritrix from [here](https://github.com/internetarchive/heritrix3/wiki#downloads) and unzip the release.

```bash
# download
wget https://repo1.maven.org/maven2/org/archive/heritrix/heritrix/3.4.0-20210923/heritrix-3.4.0-20210923-dist.zip
unzip heritrix-3.4.0-20210923-dist.zip
```

To use heritrix, Java has to be installed and `JAVA_HOME` environment variable must point to Java installation. `HERITRIX_HOME` environment variable must be set to the path where heritrix was unzipped. Make sure that `heritrix` binary is executable.

```bash
# configure
export JAVA_HOME=/path/to/jdk-install-dir
export HERITRIX_HOME=/path/to/heritrix-3.4.0-20210923-dist
chmod u+x $HERITRIX_HOME/bin/heritrix
```

Before running Bitextor with heritrix, Heritrix Web UI should be launched, specifying the username and the password. The URL will be `https://localhost:8443`, unless specified otherwise.

```bash
# run
$HERITRIX_HOME/bin/heritrix -a admin:admin
```

Heritrix Web UI settings (URL and username:password), along with the installation directory should be passed to Bitextor via `heritrixUser`, `heritrixUrl` and `heritrixPath` configuration parameters.

```yaml
heritrixUser: "admin:admin"
heritrixUrl: "https://localhost:8443"
heritrixPath: "/path/to/heritrix-3.4.0-20210923-dist"
```

If you experience problems with these steps or want additional information please refer to [this guide](https://heritrix.readthedocs.io/en/latest/getting-started.html).

In Docker it is located at `/home/docker/heritrix-3.4.0-20210923-dist` and is not running by default, i.e. it should be launched manually before executing Bitextor crawling with Heritrix.

### [Optional] Protobuf

CLD3 (Compact Language Detector v3), is a language identification model that can be used optionally during preprocessing. It is also a requirement for PDFExtract. CLD3 needs `protobuf` to work, the instructions for installation are the following:

```bash
# Install protobuf from official repository: https://github.com/protocolbuffers/protobuf/blob/master/src/README.md
# Maybe you need to uninstall any other protobuf installation in your system (from apt or snap) to avoid compilation issues
sudo apt-get install autoconf automake libtool curl make g++ unzip
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.18.1/protobuf-all-3.18.1.tar.gz
tar -zxvf protobuf-all-3.18.1.tar.gz
cd protobuf-3.18.1
./configure
make
make check
sudo make install
sudo ldconfig
```

### Some known installation issues

Depending on the version of *libboost* that you are using given a certain OS version or distribution package from your package manager, you may experience some problems when compiling some of the sub-modules included in Bitextor. If this is the case you can install it manually by running the following commands:

```bash
sudo apt-get remove libboost-all-dev
sudo apt-get autoremove
wget https://boostorg.jfrog.io/artifactory/main/release/1.77.0/source/boost_1_77_0.tar.gz
tar xvf boost_1_77_0.tar.gz
cd boost_1_77_0/
./bootstrap.sh
./b2 -j4 --layout=system install || echo FAILURE
cd ..
rm -rf boost_1_77_0*
```
