# Klusters
Software complie on Ubuntu 20.04

## Documentation
For more information about this software please read:

- [Klusters handbook](http://klusters.sourceforge.net/UserManual/index.html)

## Compile guide

  1. Install [libneurosuite](https://github.com/natuneuro/neurosuite) and [LibKlustersShared](https://github.com/natuneuro/LibKlustersShared) before compilling Neuroscope 
  
  2. Go to `/Neuroscope` directory and type:
  ```
  mkdir build
  cd build
  cmake ..
  make
  sudo make install
  ```
