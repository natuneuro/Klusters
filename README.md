# Klusters
Klusters is a powerful and easy-to-use application designed to help neurophysiologists sort action potentials recorded from multiple neurons on groups of electrodes (e.g., tetrodes or multisite silicon probes).

Klusters is developped by Lynn Hazan in G. Buzsáki's lab (CMBN, Rutgers Newark, USA), distributed under the GNU Public License v2.



Software complie on Ubuntu 20.04

## Documentation
For more information about this software please read:

- [Klusters handbook](http://klusters.sourceforge.net/UserManual/index.html)

## Compile guide

  1. Install [LibNeurosuite](https://github.com/natuneuro/neurosuite) and [LibKlustersShared](https://github.com/natuneuro/LibKlustersShared) before compiling Klusters 
  
  2. Go to `/Klusters` directory and type:
  ```
  mkdir build
  cd build
  cmake ..
  make
  sudo make install
  ```
