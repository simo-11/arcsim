# prepare Ubuntu 20.04.3 LTS

add .gitignore
snikula@LNOR071265:/github/arcsim$ cp Makefile.linux Makefile

## remove files that do not need version control

### apple-files
snikula@LNOR071265:/github/arcsim$ find . -name ._\* -type f | wc
    631     631   23892
snikula@LNOR071265:/github/arcsim$ find . -name ._\* -type f -exec rm {} +
snikula@LNOR071265:/github/arcsim$ rm .DS_Store

## dependencies

$ sudo apt install scons make g++ libjsoncpp-dev libalglib-dev \
 liblapack-dev libatlas-base-dev gfortran libz-dev libboost-date-time ctags

### fixes / updates

#### jsoncpp 
replaced by getting libjsoncpp
included jsonccp is version 0.5.0 from 2014 or even older
in github https://github.com/open-source-parsers/jsoncpp latest is 1.9.5
github/arcsim/dependencies/jsoncpp$ echo $(cat version )
0.5.0

 File "/mnt/c/github/arcsim/dependencies/jsoncpp/SConstruct", line 31
    print "Using platform '%s'" %platform
          ^
SyntaxError: Missing parentheses in call to 'print'. Did you mean print("Using platform '%s'" %platform)?
  File "/mnt/c/github/arcsim/dependencies/jsoncpp/SConstruct", line 35
    print "LD_LIBRARY_PATH =", LD_LIBRARY_PATH
SyntaxError: Missing parentheses in call to 'print'. Did you mean print("LD_LIBRARY_PATH =", LD_LIBRARY_PATH)?
Python 2 -> 3

scons: Reading SConscript files ...
ModuleNotFoundError: No module named 'commands':
  File "/mnt/c/github/arcsim/dependencies/jsoncpp/SConstruct", line 28:
    import commands
make: *** [Makefile:12: lib/libjson.a] Error 2

```
	#import commands
	#version = commands.getoutput('%s -dumpversion' %CXX)
	#platform = 'linux-gcc-%s' %version
	platform='linux-gcc-9'
```

NameError: name 'apply' is not defined:
  File "/mnt/c/github/arcsim/dependencies/jsoncpp/SConstruct", line 227:
    srcdist_cmd = env['SRCDIST_ADD']( source = """
  File "/mnt/c/github/arcsim/dependencies/jsoncpp/SConstruct", line 143:
    apply( self.env.SrcDist, (self.env['SRCDIST_TARGET'],) + args, kw )
make: *** [Makefile:12: lib/libjson.a] Error 2

apply is no longer available in python3

reverted to using ready package

#### taucs
/usr/bin/ld: cannot find -llapack
/usr/bin/ld: cannot find -lf77blas
/usr/bin/ld: cannot find -lcblas
/usr/bin/ld: cannot find -latlas
/usr/bin/ld: cannot find -lgfortran

The following additional packages will be installed:
  libblas-dev libblas3 libgfortran5 liblapack3
Suggested packages:
  liblapack-doc
The following NEW packages will be installed:
  libblas-dev libblas3 libgfortran5 liblapack-dev liblapack3

snikula@LNOR071265:/github/arcsim/dependencies$ make
..
cc   \
-obin/linux/iter \
obj/linux/iter.o \
-Llib/linux/ -ltaucs \
-llapack -L/usr/lib64/atlas -L/usr/lib64/atlas-sse3 -lf77blas -lcblas -latlas -lgfortran -lm
make[1]: Leaving directory '/mnt/c/github/arcsim/dependencies/taucs'
mkdir -p include/ lib/
cp taucs/build/*/*.h taucs/src/*.h include/
cp: will not overwrite just-created 'include/taucs_config_build.h' with 'taucs/build/linux/taucs_config_build.h'
make: *** [Makefile:20: lib/libtaucs.a] Error 1

snikula@LNOR071265:/github/arcsim/dependencies$         cp taucs/lib/*/libtaucs.a lib/

## main build

src/sparse.hpp:118:21:   required from here
/usr/include/c++/9/ostream:691:5: error: no type named ‘type’ in ‘struct std::enable_if<false, std::basic_ostream<char>&>’