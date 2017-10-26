% Savio SL7/installation training
% November 1, 2017


# Introduction

We'll spend about 30 minutes going through this material as a demonstration. I encourage you to login to your account and try out the various examples yourself as we go through them. The remainder of the scheduled time will be open time for you to work on your installations with each other and with help from us.

Some of this material is based on the extensive Savio documention we have prepared and continue to prepare, available at [http://research-it.berkeley.edu/services/high-performance-computing/user-guide](http://research-it.berkeley.edu/services/high-performance-computing/user-guide). In particular we have some detailed installation instructions under the "Installing your own" tab at [http://research-it.berkeley.edu/services/high-performance-computing/accessing-and-installing-software](http://research-it.berkeley.edu/services/high-performance-computing/accessing-and-installing-software).

The materials for this tutorial are available using git at [https://github.com/ucberkeley/savio-training-sl7-2017](https://github.com/ucberkeley/savio-training-sl7-2017) or simply as a [zip file](https://github.com/ucberkeley/savio-training-sl7-2017/archive/master.zip).

These *install.html* and *install_slides.html* files were created from *install.md* by running 'make' within the repository directory (see *Makefile* for details on how that creates the html files).

Please see this [zip file](https://github.com/ucberkeley/savio-training-intro-2017/archive/master.zip) for materials from our introductory training on September 19, including accessing Savio, data transfer, and basic job submission.


# Outline

This training session will cover the following topics:

 - Differences between SL6 and SL7 on Savio
 - Software installation
     - Installing third-party software
     - Installing Python and R packages that rely on third-party software
         - Python example
         - R example
 - Hands-on software installation help

# What's new with SL7 on Savio?

 - All Python and R packages will be available as soon as you load the main `python` or `r` modules
 - Updated Python (Python 3.6) and R (R 3.4.x) will be available.
 - An extensive set of machine learning/deep learning and related packages will be available: tensorflow, h2o, mxnet, theano, superlearner

You can ssh to `sl7.brc.berkeley.edu` and run `module avail` to see the list. Do note that we'll update the versions in many cases before final release. 

# Third-party software installation - overview

In general, third-party software will provide installation instructions on a webpage, Github README, or install file inside the package source code.

The key for installing on Savio is making sure everything gets installed in your own home, project, or scratch directory and making sure you have the packages on which the software depends on also installed by you or loaded from the Savio modules. 

A common installation approach is the GNU build system (Autotools), which involves three steps: configure, make, and make install.

  - *configure*: this queries your system to find out what tools (e.g., compilers and other packages) you have available to use in building and using the software
  - *make*: this compiles the source code in the software package
  - *make install*: this moves the compiled code (library files and executables) and header files and the like to their permanent home

# Installation logistics

We recommend you set up a directory structure that mimics how we install software on Savio. We'll assume you are installing in a group directory, but you could equally well do this in your home directory or even on scratch (albeit with the danger the installation might be purged).

If you haven't already set up the directory structure for your group, here are the steps:

```
cd /global/home/groups/my_group
mkdir sl7
mkdir sl7/sources
mkdir sl7/modules
mkdir sl7/scripts
mkdir sl7/modfiles
chmod -R g+rwX sl7
```

However, you can manage your directories as you like; the above is just a suggestion.

# Third-party software installation - examples

Here are a couple examples of installing a piece of software in your home directory.

### yaml package example

```
# install yaml, an optional dependency for Python yaml package
PKG=yaml
V=0.1.7

DIR=/global/home/groups/my_group/sl7
SRCDIR=${DIR}/sources/${PKG}/${V}
INSTALLDIR=${DIR}/modules/${PKG}/${V}

mkdir -p ${INSTALLDIR}
mkdir -p ${SRCDIR}

cd ${SRCDIR}
wget http://pyyaml.org/download/libyaml/${PKG}-${V}.tar.gz
tar -xvzf ${PKG}-${V}.tar.gz
cd ${PKG}-${V}
# --prefix is key to install in directory you have access to
./configure  --prefix=$INSTALLDIR | tee ../configure-${V}.log
make | tee ../make-${V}.log
make install | tee ../install-${V}.log
```

To save the information on how you installed the software, we recommend you save the code above. In this case you would save it in `sl7/scripts/yaml/install.sh`.

# Third-party software installation - examples

Here are a couple examples of installing a piece of software in your home directory.

### geos package example

```
PKG=geos
V=3.6.2

DIR=/global/home/groups/my_group/sl7
SRCDIR=${DIR}/sources/${PKG}/${V}
INSTALLDIR=${DIR}/modules/${PKG}/${V}

mkdir -p ${INSTALLDIR}
mkdir -p ${SRCDIR}

cd ${SRCDIR}
wget http://download.osgeo.org/${PKG}/${PKG}-${V}.tar.bz2
tar -xvjf ${PKG}-${V}.tar.bz2
cd ${PKG}-${V}
# --prefix is key to install in directory you have access to
./configure  --prefix=$INSTALLDIR | tee ../configure-${V}.log
make | tee ../make-${V}.log
make install | tee ../install-${V}.log
```


For Cmake, the following may work (this is not a worked example, just some template code):
```
PKG=foo
INSTALLDIR=~/software/${PKG}
cmake -DCMAKE_INSTALL_PREFIX=${INSTALLDIR} . | tee ../cmake.log
```

# Third-party software installation - final steps

To use the software you just installed as a dependency of other software you want to install (e.g., Python or R packages), you often need to make Linux aware of the location of the following in your newly-installed software:

  - library files (for other software to link against)
  - executables (for other software to call), and 
  - header files (for other software to compile against).

For library files, you may need this:

```
# needed in the geos example to install the rgeos R package
export LD_LIBRARY_PATH=${INSTALLDIR}/lib:${LD_LIBRARY_PATH}
```

This is because Linux only looks in certain directories for the location of .so library files. A clue that you may need this is seeing an error message such as `libfoo.so` not found.

For executables (binaries), you may need this: 

```
# needed in the geos example to install the rgeos R package
export PATH=${INSTALLDIR}/bin:${PATH}
```

This is because Linux only looks in certain directories for executables.

For header files, you generally need to do something specific for the subsequent software you are installing. If you see comments about *.h* files not being found, you need to make sure the compiler can find the *include* directory that contains those files.

# Installing Python and R packages 

Here's a Python example:

```
module load python/2.7.8
module load pip
PYPKG=pyyaml
V=0.1.7
PKGDIR=/global/home/groups/my_group/sl7/yaml/${V}

# This fails:
pip install --user ${PYPKG}
ls .local/lib/python2.7/site-packages
# needs to find header files

# This fails too:
pip install --user --ignore-installed --global-option=build_ext  \
    --global-option="-I/${PKGDIR}/include" ${PYPKG}
# no -lyaml (needs to find library) files 
# in this case setting LD_LIBRARY_PATH does not work for some reason

# This succeeds:
pip install --user --ignore-installed --global-option=build_ext \
    --global-option="-I/${PKGDIR}/include" \
    --global-option="-L/${PKGDIR}/lib" ${PYPKG}
```

[[[Krishna: can we say anything about why the explicit setting of -L is needed and simply setting LD_LIBRARY_PATH does not work?]]]

# Installing Python and R packages 

Here's an R example:

```
# in this case, setting LD_LIBRARY_PATH works
# (and we also need to set PATH)
V=3.6.2
PKGDIR=/global/home/groups/my_group/sl7/geos/${V}
export LD_LIBRARY_PATH=${PKGDIR}/lib:${LD_LIBRARY_PATH}
export PATH=${PKGDIR}/bin:${PATH}
module load r
Rscript -e "install.packages('rgeos', repos = 'http://cran.cnr.berkeley.edu', \
lib = Sys.getenv('R_LIBS_USER'))"
```

You may sometimes need to use the `configure.args` or `configure.vars` arguments to `install.packages()` to provide information on the location of `include` and `lib` directories of dependencies. The Savio help email can provide support for complicated installations.

# Installation for an entire group

Optionally if you'd like your group members to have write access to the directories for the software you installed, you would do this:

```
PKG=rgeos
cd /global/home/groups/my_group/sl7
chmod -R g+rwX {sources,modfiles,modules,scripts}/${PKG}
```


# Setting up a module for your software


You may also want to set up a module for the software you just installed. This allows you and your group members to easily set your environment so that the software is accessible for you. To do this you need to do the following.

First we'll need a directory in which to store our module files:

```
V=3.6.2
MYMODULEPATH=/global/home/groups/my_group/sl7/modfiles
mkdir -p ${MYMODULEPATH}/geos
export MODULEPATH=${MODULEPATH}:${MYMODULEPATH}  # good to put this in your .bashrc
```

Now we create a module file for the version (or one each for multiple versions) of the software we have installed. E.g., for our geos installation we would edit  `${MYMODULEPATH}/geos/3.6.2` based on looking at examples of other module files. An example module file for our geos example is *example-modulefile*.

```
cp example-modfile ${MYMODULEPATH}/geos/3.6.2
```

Or see some of the Savio system-level modules in `/global/software/sl-7.x86_64/modfiles`. 

```
cat /global/software/sl-7.x86_64/modfiles/apps/ml/tensorflow/1.0.0-py36
```


# How to get additional help

 - For technical issues and questions about using Savio: 
    - brc-hpc-help@berkeley.edu
 - For questions about computing resources in general, including cloud computing: 
    - brc@berkeley.edu
 - For questions about data management (including HIPAA-protected data): 
    - researchdata@berkeley.edu


