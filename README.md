# vkExamples
Some Vulkan examples


### Dependencies

* [CMake][cmake]: for generating compilation targets.

#### 1) Configure

Assume the source directory is `$SOURCE_DIR` and
the build directory is `$BUILD_DIR`:

For building on Linux (assuming using the Ninja generator):

```bash
cd $BUILD_DIR

cmake -GNinja -DCMAKE_BUILD_TYPE={Debug|Release|RelWithDebInfo} \
      -DCMAKE_INSTALL_PREFIX=`pwd`/install $SOURCE_DIR
```

For building on Windows:

```bash
cd $BUILD_DIR
cmake $SOURCE_DIR -G "Visual Studio 14 2015 Win64" -DCMAKE_INSTALL_PREFIX=`pwd`/install
# The CMAKE_INSTALL_PREFIX part is for testing (explained later).
```