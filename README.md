pyndri
======

pyndri is a Python interface to the Indri search engine (http://www.lemurproject.org/indri/).

Requirements
------------

During development, we use Python 3.5. Some of the [examples](examples) require numpy.

**NOTE**: Python2 support has ended; if you want to continue using it, please check out the  `python2` tag.

Installation
------------

You first need to install [Indri](https://www.lemurproject.org/indri.php) before you can install pyndri.

### Linux

While there are many different Linux distributions, the following seems to work for most. Below we assume a Debian-based system (tested on Ubuntu and Mint), however, [your mileage may vary](https://en.wiktionary.org/wiki/your_mileage_may_vary).

First, install dependencies:

    sudo apt install g++ zlib1g-dev python3.5-dev python3-pip
    sudo pip3 install setuptools

To install Indri:

    wget <url-to-indri-5.11.tar.gz>
    tar xzvf indri-5.11.tar.gz
    cd indri-5.11
    ./configure CXX="g++ -D_GLIBCXX_USE_CXX11_ABI=0"
    make
    sudo make install

Finally, install pyndri:

    sudo pip3 install pyndri

### macOS

The instructions for macOS as similar to those for Linux. You most likely want to install a modern version of Python, GCC and the development package of zlib using [Homebrew](https://brew.sh/) or [MacPorts](https://www.macports.org/).

The following instructions would work on macOS Sierra:

First, install gcc 5 or later, e.g.:

    brew install gcc5

To install Indri:

    download Indri from here: https://sourceforge.net/projects/lemur/files/lemur/indri-5.11/indri-5.11.tar.gz/download
    tar xzvf indri-5.11.tar.gz
    cd indri-5.11
    make CC="gcc-5 -D_GLIBCXX_USE_CXX11_ABI=0" CXX="g++-5 -D_GLIBCXX_USE_CXX11_ABI=0"
    sudo make install CC="gcc-5 -D_GLIBCXX_USE_CXX11_ABI=0" CXX="g++-5 -D_GLIBCXX_USE_CXX11_ABI=0"

To install pyndri (tested with an Anaconda python 3.5 environment):

    git clone https://github.com/cvangysel/pyndri.git
    cd pyndri
    CC=gcc-5 CXX=g++-5 python setup.py build
    CC=gcc-5 CXX=g++-5 python setup.py install


### Windows + [Cygwin](https://www.cygwin.com/) (experimental)

To get Indri running under Windows with Cygwin, a small adjustment needs to be made to the above instructions. First, you probably want to install the [apt-cyg](https://github.com/transcode-open/apt-cyg) package manager and install a modern version of Python, GCC and the zlib development package.

To install Indri:

    wget <url-to-indri-5.11.tar.gz>
    tar xzvf indri-5.11.tar.gz
    cd indri-5.11
    ./configure CXX="g++ -D_GNU_SOURCE=1 -D_GLIBCXX_USE_CXX11_ABI=0"
    make
    sudo make install

Finally, install pyndri:

    sudo pip install pyndri

Examples
--------

How to iterate over all documents in a repository:

    import pyndri

    index = pyndri.Index('/path/to/indri/index')

    for document_id in range(index.document_base(), index.maximum_document()):
        print(index.document(document_id))

The above will output pairs of external document identifiers and document terms:

    ('eUK237655', (877, 2171, 191, 2171))
    ('eUK956325', (880, 2171, 345, 2171))
    ('eUK458961', (3566, 1, 2, 3199, 504, 1726, 1, 3595, 1860, 1171, 1527))
    ('eUK390317', (3228, 2397, 2, 945, 1, 3097, 3, 145, 3769, 2102, 1556, 970, 3959))
    ('eUK794201', (770, 247, 1686, 3712, 1, 1085, 3, 830, 1445))

How to launch a Indri query to an index and get the identifiers and scores of retrieved documents:

    import pyndri

    index = pyndri.Index('/path/to/indri/index')

    # Queries the index with 'hello world' and returns the first 1000 results.
    results = index.query('hello world', results_requested=1000)

    for int_document_id, score in results:
        ext_document_id, _ = index.document(int_document_id)
        print(ext_document_id, score)

The above will print document identifiers and language modeling scores:

    eUK306804 -8.77414652243
    eUK700967 -8.8712247934
    eUK437700 -8.88184436222
    eUK107263 -8.89119022464
    ...

The token to term identifier mapping can be extracted as follows:

    import pyndri

    index = pyndri.Index('/path/to/indri/index')
    token2id, id2token, id2df = index.get_dictionary()

    id2tf = index.get_term_frequencies()

Citation
--------

If you use pyndri to produce results for your scientific publication, please refer to our [ECIR 2017](https://arxiv.org/abs/1701.00749) paper.

	@inproceedings{VanGysel2017pyndri,
	  title={Pyndri: a Python Interface to the Indri Search Engine},
	  author={Van Gysel, Christophe and Kanoulas, Evangelos and de Rijke, Maarten},
	  booktitle={ECIR},
	  volume={2017},
	  year={2017},
	  organization={Springer}
	}

Frequently Asked Questions
--------------------------

### Importing `pyndri` in Python causes the error `Undefined symbol std::__cxx11::basic_string ...`

You are using GCC 5 (or above) and this version of the compiler includes new implementations of common types (`std::string`, etc.). You have to recompile Indri first by setting the `_GLIBCXX_USE_CXX11_ABI` macro to 0.

	make clean
	./configure CXX="g++ -D_GLIBCXX_USE_CXX11_ABI=0"
	make
	sudo make install

Afterwards, recompile pyndri from a clean install.

### Importing `pyndri` in Python causes the error `undefined symbol: std::throw_out_of_range_fmt ...`

Your Python version was compiled with a different standard library than you used to compile Indri and pyndri with. For example, the Anaconda distribution comes with pre-compiled binaries and its own standard library.

We do not provide support for this, as this is a problem with your Python installation and goes beyond the scope of this project. However, we've identified three possible paths:

   * Re-compile Python yourself from source.
   * Compile Indri and pyndri with the standard library of your Python distribution. This might be difficult, as the headers are often not included in the distribution.
   * Use the Python executables part of your Linux distribution. Be sure to install the development headers (e.g., `python3.5-dev` using `apt-get` on Ubuntu).
   
### Compiling on macOS gives: `g++-5: error: unrecognized command line option '-stdlib=libstdc++'`

This might happen when you first try to compile with clang. An easy (and dirty) solution to this is to delete the Indri folder and re-extract the folder from the tar ball. Then proceed with the rest of the installation instructions normally. 

### Compiling pyndri on macOS gives: `fatal error: antlr/NoViableAltException.hpp`

This is a matter of properly setting the environment variables. For example, you might want to set the environment variables as follows (after installing Indri):

    git clone https://github.com/cvangysel/pyndri.git
    cd pyndri
    export CPATH="/usr/local/include/:${CPATH}"
    export LD_INCLUDE_PATH=/usr/local/lib/
    export LD_LIBRARY_PATH=/usr/local/lib/
    CC=gcc-5 CXX=g++-5 python setup.py build
    CC=gcc-5 CXX=g++-5 python setup.py install

License
-------

Pyndri is licensed under the [MIT license](LICENSE). Please note that [Indri](http://www.lemurproject.org/indri.php) is licensed separately. If you modify Pyndri in any way, please link back to this repository.
