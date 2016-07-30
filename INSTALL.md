# Regional Earth System Modeling Summer School 2016

**Location:** 5-7 September 2016, Istanbul Technical University, Istanbul, Turkey

The offical page of summer school web page can be found in [here](http://hbm.itu.edu.tr/egitimler/ysm2016/) (in Turkish)

## Login Information for HPC System (Anadolu@UHeM)

To login your [UHeM](http://www.uhem.itu.edu.tr) account that is created for the summer school, please type following commands (**you need to setup VPN connection to reach the server from outside of the campus!**):

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
> module load intel/composer_xe_2013_sp1.1.106
> source /RS/progs/intel/impi/4.0.3.008/bin64/mpivars.sh
```

It is also possible to add these commands to **~/.bashrc** (for bash shell users which is default in most of the systems) file for automatization of loading necessary environment variables.  

The **install.sh** script can be used for the installation of required tools (i.e. [zlib](http://www.zlib.net), [HDF5](https://www.hdfgroup.org/HDF5/), [netCDF C/C++ and Fortran](http://www.unidata.ucar.edu/software/netcdf/), [xerces-c](http://xerces.apache.org/#xerces-c) and [ESMF](https://www.earthsystemcog.org/projects/esmf/) libraries). The script can be executed using following command (the command line argument given after shell script is the installation directory i.e. **/opt/progs**).

```
> ./install.sh `pwd`
or in case of explicitly defined installation directory 
> ./install.sh /opt/progs
```

Then source the **env_progs** file to use installed libraries and tools using following command (optinonally you might add the command to **~/.bashrc** file),

```
> source /RS/progs/workshop/env_progs
```

## Day1: Installation and Usage of RegCM

The [ICTP](http://www.ictp.it/research/esp.aspx)'s RegCM (**Reg**ional **C**Limate **M**odel) development is managed by SVN repository that can be found in [here](https://gforge.ictp.it/gf/project/regcm/scmsvn/?action=AccessInfo). To retrieve and install latest tagged version of the source code, please issue following commands,

```
> cd /RS/users/[workshop user name]/workshop
> mkdir -p day1/src
> cd day1/src
> wget https://gforge.ictp.it/gf/download/frsrelease/252/1580/RegCM-4.5.0.tar.gz
> tar -zxvf RegCM-4.5.0.tar.gz
> cd RegCM-4.5.0
> ./configure --prefix=`pwd` CC=${CC} FC=${FC} MPIFC=mpiifort
> make
> make install
```

### Use Case

The atmospheric model use case that will be used in the hands-on sessions includes,

* A parent domain that covers entire Mediterranean Basin (MED50) with 50 km horizontal resolution.
* A nested domain that covers entire Black Sea and Turkey with 10 km horizontal resolution. In the cupling session (day 3), we will couple Black Sea ocean model with inner-most model domain (TR10).
* The parent domain will be forced by ERA-Interim dataset that can be retrieved from [here](http://www.ecmwf.int/en/research/climate-reanalysis/era-interim).
* Jan. 2010 is selected for the test simulations

![Domain map with high-resolution nest](https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/images/Fig_01_domain.png)

### Creating Initial and Boundary Conditions (ICBC)





