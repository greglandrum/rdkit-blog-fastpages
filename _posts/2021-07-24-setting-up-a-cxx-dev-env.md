---
toc: true
layout: post
description: It's actually pretty easy
categories: [tutorial, technical]
title: Using the RDKit in a C++ program
---

*Note:* the instructions in this blog post currently only work on linux systems.
There's a configuration problem with the way we use cmake on the Mac and Windows
that needs to be cleared up. I will update the post after that's done.

Last week I (re)discoverered that it's pretty easy to use the RDKit in other C++
projects. This is obviously somthing that's possible, but I thought of it as
being something of a pain. It turns out that it's not, as I hope to show you in
this post.

I started by setting up a fresh conda environment and grabbing an RDKit build
from conda-forge, this is a bit heavyweight since you end up with a bunch of
python packages as well as the RDKit itself (I'm going to look into making this
more minimal), but it's much easier than doing your own build.

The first thing is to set up a conda environment:
```
conda create -n rdkit_dev
conda activate rdkit_dev
conda install -c conda-forge mamba
mamba install -c conda-forge cmake rdkit eigen
```
Note: I start by installing [mamba](https://github.com/mamba-org/mamba) here
because it makes doing conda installs much, much faster.

Here's a simple demo program which reads in a set of molecules from an input
file and generates tautomer hashes for them. It uses the `boost::timer` library
in order to separately time how long it takes to read the molecules and generate
the hashes. I called this file `tautomer_hash.cpp`:
```
#include <GraphMol/FileParsers/MolSupplier.h>
#include <GraphMol/MolHash/MolHash.h>
#include <GraphMol/RDKitBase.h>
#include <RDGeneral/RDLog.h>
#include <algorithm>
#include <boost/timer/timer.hpp>
#include <iostream>
#include <vector>

using namespace RDKit;

void readmols(std::string pathName, unsigned int maxToDo,
              std::vector<RWMOL_SPTR> &mols) {
  boost::timer::auto_cpu_timer t;
  // using a supplier without sanitizing the molecules...
  RDKit::SmilesMolSupplier suppl(pathName, " \t", 1, 0, true, false);
  unsigned int nDone = 0;
  while (!suppl.atEnd() && (maxToDo <= 0 || nDone < maxToDo)) {
    RDKit::ROMol *m = suppl.next();
    if (!m) {
      continue;
    }
    m->updatePropertyCache();
    // the tautomer hash code uses conjugation info
    MolOps::setConjugation(*m);
    nDone += 1;
    mols.push_back(RWMOL_SPTR((RWMol *)m));
  }
  std::cerr << "read: " << nDone << " mols." << std::endl;
}

void generatehashes(const std::vector<RWMOL_SPTR> &mols) {
  boost::timer::auto_cpu_timer t;
  for (auto &mol : mols) {
    auto hash =
        MolHash::MolHash(mol.get(), MolHash::HashFunction::HetAtomTautomer);
  }
}
int main(int argc, char *argv[]) {
  RDLog::InitLogs();
  std::vector<RWMOL_SPTR> mols;
  BOOST_LOG(rdInfoLog) << "read mols" << std::endl;

  readmols(argv[1], 10000, mols);
  BOOST_LOG(rdInfoLog) << "generate hashes" << std::endl;
  generatehashes(mols);

  BOOST_LOG(rdInfoLog) << "done " << std::endl;
}
```
This is a pretty crappy program since it doesn't do much error checking, but the
purpose here is to demonstrate how to get the environment setup, not to teach
how to write nice C++ programs. :-)

The way to make the build easy is to use cmake to set everything up, so I need a
`CMakeLists.txt` file that defines my executable and its RDKit dependencies:
```
cmake_minimum_required(VERSION 3.18)

project(simple_cxx_example)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)


find_package(RDKit REQUIRED)
find_package(Boost COMPONENTS timer system REQUIRED)
add_executable(tautomer_hash tautomer_hash.cpp)
target_link_libraries(tautomer_hash RDKit::SmilesParse RDKit::MolHash
   Boost::timer)
```
This tells cmake to find the RDKit and boost installs (which "just works" since
cmake, boost, and the RDKit were all installed from conda), defines the
executable I want to create, and then lists the RDKit and boost libraries I use.
And that is pretty much that.

Now I create a build dir, run `cmake` to setup the build, and run `make` to actually
build my program:
```
(rdkit_dev) glandrum@Badger:~/RDKit_blog/src/simple_cxx_example$ mkdir build
(rdkit_dev) glandrum@Badger:~/RDKit_blog/src/simple_cxx_example$ cd build
(rdkit_dev) glandrum@Badger:~/RDKit_blog/src/simple_cxx_example/build$ cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Failed
-- Looking for pthread_create in pthreads
-- Looking for pthread_create in pthreads - not found
-- Looking for pthread_create in pthread
-- Looking for pthread_create in pthread - found
-- Found Threads: TRUE  
-- Found Boost: /home/glandrum/miniconda3/envs/rdkit_dev/lib/cmake/Boost-1.74.0/BoostConfig.cmake (found suitable version "1.74.0", minimum required is "1.74.0")  
-- Found Boost: /home/glandrum/miniconda3/envs/rdkit_dev/lib/cmake/Boost-1.74.0/BoostConfig.cmake (found version "1.74.0") found components: timer system 
-- Configuring done
-- Generating done
-- Build files have been written to: /home/glandrum/RDKit_blog/src/simple_cxx_example/build
(rdkit_dev) glandrum@Badger:~/RDKit_blog/src/simple_cxx_example/build$ make tautomer_hash
[ 50%] Building CXX object CMakeFiles/tautomer_hash.dir/tautomer_hash.cpp.o
[100%] Linking CXX executable tautomer_hash
[100%] Built target tautomer_hash
```
And now I can run the program:
```
(rdkit_dev) glandrum@Badger:~/RDKit_blog/src/simple_cxx_example/build$ ./tautomer_hash /scratch/RDKit_git/Code/Profiling/GraphMol/chembl23_very_active.txt
[07:51:33] read mols
read: 10000 mols.
 0.819242s wall, 0.740000s user + 0.070000s system = 0.810000s CPU (98.9%)
[07:51:33] generate hashes
 0.872662s wall, 0.870000s user + 0.010000s system = 0.880000s CPU (100.8%)
[07:51:34] done 
(rdkit_dev) glandrum@Badger:~/RDKit_blog/src/simple_cxx_example/build$ 
```

If you don't feel like copy/pasting, the source files for this post are [available from github](https://github.com/greglandrum/rdkit_blog/tree/master/src/simple_cxx_example). 

This all works so nicely because of the time and effort Riccardo Vianello invested a few years ago to improve the RDKit's cmake integration.

Next step: add this to the documentation! 
