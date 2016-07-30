# Earth System Modeling Summer School 2016

**Location:** 5-7 September 2016, Istanbul Technical University, Istanbul, Turkey

The summer school web page can be found in [here](http://hbm.itu.edu.tr/egitimler/ysm2016/) (in Turkish)

## Login to the HPC System (Anadolu@UHeM)

To login your UHeM account that is created for the summer school, please type following commands (you need to setup VPN connection to reach the server from outside of the campus):

```
> ssh lnode.uhem.itu.edu.tr -l [workshop user name]
> [type your password belong to your workshop user name]
```

The provided accounts will be available until **10 Sept. 2016** and they are limited to submit job with **max. 32 processor per user**.  

## Installation of Required Libraries (not required, optional)
This section includes detailed information to prepare required software environment that will be used in the hands-on section of the summer school. Please note that:

* The commands shown in the rest of the document is tested on **Anadolu@UHeM** cluster and might slightly change in other HPC system or server. 

* The required third-party libraries and tools to install and run the models (i.e. RegCM, ROMS and RegESM) are already installed to the **/RS/progs/workshop** folder.

Before starting to install or use the third-party libraries, you must load following modules (Intel Fortan/C/C++ compiler and MPI) on **Anadolu@UHeM**.

```
module load intel/composer_xe_2013_sp1.1.106
source /RS/progs/intel/impi/4.0.3.008/bin64/mpivars.sh
```

It is also possible to add these commands to ~/.bashrc (for bash shell users which is default in most of the systems) file for automatization of loading necessary environment variables.  

The **install.sh** script can be used for the installation of required tools (i.e. [zlib](http://www.zlib.net), [HDF5](https://www.hdfgroup.org/HDF5/), [netCDF C/C++ and Fortran](http://www.unidata.ucar.edu/software/netcdf/), [xerces-c](http://xerces.apache.org/#xerces-c) and [ESMF](https://www.earthsystemcog.org/projects/esmf/) libraries). The script can be executed using following command (the command line argument given after bash script is the installation directory such as **/opt/progs**).

```
./install.sh `pwd`
or
./install.sh /opt/progs
```







`command`[^emphasize].
[^emphasize]: The command line argument given after bash script is the installation directory such as **/opt/progs**.