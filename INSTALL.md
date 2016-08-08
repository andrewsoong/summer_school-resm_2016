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

* The commands shown in the rest of the document is tested on **Anadolu@UHeM** cluster using **Intel C/C++/Fortran Compiler (14.0.1)** and **MPI (4.0 Update 3)** and might slightly change in other HPC system or server. 

* The required third-party libraries and tools to install and run the models (i.e. RegCM, ROMS and RegESM) are already installed to the **/RS/progs/workshop** folder.

Before starting to install or use the third-party libraries, you must load following modules (Intel Fortan/C/C++ compiler and MPI) on **Anadolu@UHeM**.

```
> module load intel/composer_xe_2013_sp1.1.106
> source /RS/progs/intel/impi/4.0.3.008/bin64/mpivars.sh
```

It is also possible to add these commands to **~/.bashrc** (for bash shell users which is default in most of the systems) file for automatization of loading necessary environment variables.  

The **install.sh** script can be used for the installation of required tools (i.e. [zlib](http://www.zlib.net), [HDF5](https://www.hdfgroup.org/HDF5/), [netCDF C/C++ and Fortran](http://www.unidata.ucar.edu/software/netcdf/), [xerces-c](http://xerces.apache.org/#xerces-c) and [ESMF](https://www.earthsystemcog.org/projects/esmf/) libraries). The script can be executed using following command (the command line argument given after shell script is the installation directory i.e. **/opt/progs**).

```
> wget https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/install.sh
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
> cd /RS/users/$USER/workshop
> mkdir -p day1/src
> cd day1/src
> wget https://gforge.ictp.it/gf/download/frsrelease/252/1580/RegCM-4.5.0.tar.gz
> tar -zxvf RegCM-4.5.0.tar.gz
> cd RegCM-4.5.0
> ./configure --prefix=`pwd` CC=${CC} FC=${FC} MPIFC=mpiifort
> make install
> cd ../..
> ln -s src/RegCM-4.5.0/bin .
```

The installation of the model might take time depending on the used system and the selected compiler. Due to the limited time of the hands-on section, the RegCM installation under **/RS/progs/workshop/hands-on/day1/src/RegCM-4.5.0** directory can be used by linking binary directory,

```
> cd /RS/users/$USER/workshop
> mkdir day1
> ln -s /RS/progs/workshop/hands-on/day1/src/RegCM-4.5.0/bin .
```

### Use Case

The atmospheric model use case that will be used in the hands-on session includes,

* A parent domain that covers entire Mediterranean Basin (MED50) with 50 km horizontal resolution.
* A nested domain that covers entire Black Sea and Turkey with 10 km horizontal resolution. In the model coupling session (day 3), we will couple Black Sea ocean model (ROMS) with inner-most atmosphere model domain (TR10).
* The parent domain will be forced by [ERA-Interim](http://www.ecmwf.int/en/research/climate-reanalysis/era-interim) dataset and nested domain ICBC will be created using output of parent model domain (MED50). This procedure is called as "one-way nesting".
* The test simulation is planed as one month long (Jan. 2010).

![Domain map with high-resolution nest](https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/images/Fig_01_domain.png)

### Running Parent Domain (MED50)

Before running the model, we need to create ICBC files,

* The whole set of suppoted initial and boundary conditions for RegCM model can be found in [here](http://clima-dods.ictp.it/regcm4/). 
* In this session, the ERA-Interim ([EIN75](http://clima-dods.ictp.it/regcm4/EIN75)) dataset will be used along with weekly SST ([OI-SST](http://clima-dods.ictp.it/regcm4/SST)) dataset (used as prescribed SST data in standalone model simulation)
* For year 2010, the data already placed under **/RS/progs/workshop/data/RCMDATA**. 

To get RegCM configuration file for MED50 simulation,

```
> cd /RS/users/$USER/workshop/day1
> wget https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/day1/regcm.in_MED50km
```

Then, edit **regcm.in_MED50km** file and modify **dirter**, **dirglob**, **dirout** paths according to your user directories and run following commands to create input files for RegCM simulation, 

``` 
> mkdir input output
> bin/terrain regcm.in_MED50km
> bin/sst regcm.in_MED50km 
> bin/icbc regcm.in_MED50km
```

This will create a bunch of files under *input/* diectory. The generation of ICBC files could take time. So, the folder given in following command contains the required input files and can be linked to the user directory.

```
> mkdir output
> ln -s /RS/progs/workshop/data/DAY1/input .
```

After having input files, now we are ready to run the model. **Anadolu@UHeM** cluster uses LSF as a job schedular. To submit the actual RegCM simulation to the cluster, we need to create job submission script. So, following script (you can call it as **regcm.lsf**) can be used for this purpose,

```
#!/bin/bash
#BSUB -P workshop
#BSUB -J regcm 
#BSUB -q workshop 
#BSUB -m anadolu_quad 
#BSUB -o %J.out
#BSUB -e %J.err
#BSUB -a intelmpi4
#BSUB -n 16 

mpirun.lsf bin/regcmMPI regcm.in_MED50km >& regcmout.txt 
```

In this case, **-n** option basically controls the number of processor that will be used for the simulation. **-P** is the name of the project that will be used for the accounting purpose.

After creating job submission script (i.e. **regcm.lsf**), it could be submmited to the system using following command,

```
> bsub < regcm.lsf
```

The simulation can be checked by looking **regcmout.txt** file using *cat* or *tail* basic Linux commands. The output files will be placed under *output/* directory. The following LSF commands might help to monitor or kill the jobs.

```
> bjobs - shows the running jobs
> bjobs -u all - shows all jobs in the system (including other users)
> bkill [JOB ID] - kill the job
```

### Running Nested Domain (TR10)

After running parent domain (MED50) simulation and having output files, the initial and boundary conditions of nested model can be created. The first step is to create the input file directory (MED50 output files and SST data) that will be used to create input file for TR10 simulation.

```
> cd /RS/users/$USER/workshop/day1
> mkdir data
> cd data
> wget https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/day1/link.sh

edit idir parameter and change the output directory before running following commands! 

> ./link.sh
> ln -s /RS/progs/workshop/data/RCMDATA/SST .
> wget https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/day1/regcm.in_TR10km
```

Then, edit **regcm.in_TR10km** file and modify **dirter**, **dirglob**, **dirout** paths according to your user directories and run following commands to create input files for RegCM simulation,

```
> bin/terrain regcm.in_TR10km 
> bin/sst regcm.in_TR10km
> bin/icbc regcm.in_TR10km
```

Edit job submission script (**regcm.lsf**) used to run parent model domain and place **regcm.in_MED50km** with **regcm.in_TR10km**. So, the new run will use configuration of nested model domain (TR10). You may also increase the number of processor from 16 to 32. Then, the LSF script can be submitted as usual.

### Visualization and Analysis of Model Results

The output of the model is in NetCDF format and can be read by common data analysis and visualization tools such as [NCL](http://www.ncl.ucar.edu), [Ferret](http://ferret.pmel.noaa.gov/Ferret/), [Grads](http://cola.gmu.edu/grads/), [R](https://www.r-project.org) or Python programming language with [Matplotlib](http://matplotlib.org) toolbox. 

The results of simulations (MED50 and TR10) can be found in **/RS/progs/workshop/data/DAY1/output**.


## Day2: Installation and Usage of ROMS

The installation and usage of ROMS oceean modeling system include set of steps;

1. Preparing model using system/architecture specific definitions (i.e. compiler, MPI and third-party libraries)
2. Modifiying source code of the model for specific application (optional!)
3. Creating build script and model configuration file (*.h) to install model
4. Creating ocean model grid, initial and boundary conditions (not covered in this school) and atmospheric forcing (partly covered) in NetCDF format.
5. Creating model parameter file (*.in) to run the model 

The simplified version of the Black Sea model will be used during the summer school,

![Bathymetry map](https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/images/Fig_02_bathymetry.png)

The following commands can be used to download ROMS modeling system (**the original version of the model does not support coupling through the RegESM driver! See section for day three for more information**) but first, you must register in [here](https://www.myroms.org/index.php?page=RomsCode) to get access to download the model using [SVN](https://subversion.apache.org) command.

```
> mkdir /RS/users/$USER/workshop/day2
> cd /RS/users/$USER/workshop/day2
> mkdir src
> cd src

if you already registered and have user name and password
> svn checkout -r 809 --username [ROMS USER NAME] https://www.myroms.org/svn/src/trunk roms-r809

if not, you could download the source code from GitHub
> wget https://github.com/uturuncoglu/summer_school-resm_2016/blob/master/day2/src/roms-r809.tar.gz
> tar -zxvf roms-r809.tar.gz 
```

In case of using original version of the ROMS ocean model (retrieved from subversion), you need to do minor modification in the source code (**roms-r809/ROMS/Nonlinear/bulk_flux.F**) to allow correct flux calculation (via Bulk Flux module) from *RegCM* model output. In this case, new configuration parameter  (CPP preprocessor flag, **SPECIFIC_HUMIDITY**) can be added as follows,

```
...
...
!
!-----------------------------------------------------------------------
!  Compute net longwave radiation (W/m2), LRad.
!-----------------------------------------------------------------------

# if defined LONGWAVE
!
!  Use Berliand (1952) formula to calculate net longwave radiation.
!  The equation for saturation vapor pressure is from Gill (Atmosphere-
!  Ocean Dynamics, pp 606). Here the coefficient in the cloud term
!  is assumed constant, but it is a function of latitude varying from
!  1.0 at poles to 0.5 at the Equator).
!
#  ifdef SPECIFIC_HUMIDITY
!  specific humidity in units of kg/kg
          vap_p=PairM*RH/(1.0_r8+0.378_r8*RH)             !WPB
#  else
      IF(RH.lt.2.0_r8) THEN                               !WPB
          cff=(0.7859_r8+0.03477_r8*TairC(i))/                          &
     &        (1.0_r8+0.00412_r8*TairC(i))
          e_sat=10.0_r8**cff   ! saturation vapor pressure (hPa or mbar)
          vap_p=e_sat*RH       ! water vapor pressure (hPa or mbar)
      ELSE                                                !WPB
          vap_p=0.001_r8*PairM*RH/(1.0_r8+0.000378_r8*RH) !WPB
      ENDIF                                               !WPB
#  endif
...
...
!
!  Compute specific humidity, Q (kg/kg).
!
#ifdef SPECIFIC_HUMIDITY
! Incoming humidity is specific humidity in (kg/kg)
          Q(i)=RH
#else
          IF (RH.lt.2.0_r8) THEN                       !RH fraction
            cff=cff*RH                                 !Vapor pres (mb)
            Q(i)=0.62197_r8*(cff/(PairM-0.378_r8*cff)) !Spec hum (kg/kg)
          ELSE          !RH input was actually specific humidity in g/kg
            Q(i)=RH/1000.0_r8                          !Spec Hum (kg/kg)
          END IF
#endif
...
...
```

**Also, not that this modification is already placed in the ROMS ocean model retrieved from GitHub using wget command and you don't need to do again!!!**

To install the model, user also need to edit an arcitecture specific file (found under **roms-r809/Compilers/**) according to used compiler, MPI library and NetCDF installation path. In our case, we will edit **Linux-ifort.mk** becasue we are using Intel Compiler and MPI. 

The default definition of NetCDF library in **Linux-ifort.mk**

```
...
ifdef USE_NETCDF4
        NC_CONFIG ?= nc-config
    NETCDF_INCDIR ?= $(shell $(NC_CONFIG) --prefix)/include
             LIBS := $(shell $(NC_CONFIG) --flibs)
else
    NETCDF_INCDIR ?= /usr/local/include
    NETCDF_LIBDIR ?= /usr/local/lib
             LIBS := -L$(NETCDF_LIBDIR) -lnetcdf
endif
...
ifdef USE_MPI
         CPPFLAGS += -DMPI
 ifdef USE_MPIF90
               FC := mpif90
 else
             LIBS += -lfmpi-pgi -lmpi-pgi
 endif
endif
...
```

must be modified as follows for **Adadolu@UHeM** cluster (be aware about empty spaces at the end of the directory definitions!),

```
...
ifdef USE_NETCDF4
        NC_CONFIG ?= nc-config
    NETCDF_INCDIR ?= $(shell $(NC_CONFIG) --prefix)/include
             LIBS := $(shell $(NC_CONFIG) --flibs)
else
    NETCDF_INCDIR ?= /RS/progs/workshop/netcdf-4.4.0/include
    NETCDF_LIBDIR ?= /RS/progs/workshop/netcdf-4.4.0/lib
             LIBS := -L$(NETCDF_LIBDIR) -lnetcdf -lnetcdff
endif
...
ifdef USE_MPI
         CPPFLAGS += -DMPI
 ifdef USE_MPIF90
               FC := mpiifort
 else
             LIBS += -lfmpi-pgi -lmpi-pgi
 endif
endif
...
```

Then, copy build script and edit *ROMS_APPLICATION*, *MY_ROOT_DIR*, *MY_ROMS_SRC* and *which_MPI* definitions as it indicated,

```
> cd /RS/users/$USER/workshop/day2/src
> cp roms-r809/ROMS/Bin/build.sh .
> vi build.sh

...
...
setenv ROMS_APPLICATION      BSEA
setenv MY_ROOT_DIR           /RS/projects/workshop/$USER/day2/src
setenv MY_ROMS_SRC           ${MY_ROOT_DIR}/roms-r809
...
...
setenv which_MPI             intelmpi
...
...
```

Then, the installation script (**build.sh**) can be run along with the model configuration file (**bsea.h**),  

```
> wget https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/day2/bsea.h
> ./build.sh
```

This will create an executable (**oceanM**) in the same directory. If you plaining to run the model in debug or serial mode then the name of the executable changes as **oceanG** and **oceanS**.

In general, the ROMS ocean model requires a set of input files such as grid definition, initial and boundary conditions and atmospheric focing to run. In this tutorial, we will use previously created files for grid and initial conditions but atmospheric forcing will be generated by using output of RegCM simulation that is performed in last day (day1). To create atmospheric forcing for the standalone ocean model simulation and to retrive required files, the following commands can be used,

```
> cd /RS/users/$USER/workshop/day2
> mkdir input
> cd input
> wget https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/day2/input.tar.gz
> tar -zxvf input.tar.gz

To create atmospheric forcing, you must edit roms_forc_bulk_ex.ncl and point out surface output of one month RegCM simulation. Then, you can run NCL script to create forcing file.

> ncl roms_forc_bulk_ex.ncl
```

After issuing previous commands, the input directory must have a file called as BSEA_forcing_20100102-20100131.nc. Also note that [NCL](http://www.ncl.ucar.edu) visualization and data analysis tool is already installed under **/RS/progs/workshop** and will be available to all the users after sourcing **env_progs** file.

The last missing file to run the model is the model parameter file (**bsea.in**). This file includes set of information about the application such as grid sizes (*Lm*, *Mm* and *N*), number of tiles in I and J direction (domain decomposition parameters, *NtileI* and *NtileJ*), model time steps and simulation lenght (*NTIMES*, *DT*, *NDTFAST*), output and restart file interval (*NRREC*, *LcycleRST*, *NRST*, *LDEFOUT*, *NHIS* and *NAVG*), diffusion and viscosity parameters (*TNU2*, *TNU4*, *VISC2* and *VISC4*), [Jerlow Water Type](https://www.myroms.org/projects/src/ticket/609) (*WTYPE*), vertical grid parameters (must be consistent with grid file, *Vtransform*, *Vstretching*, *THETA_S*, *THETA_B* and *TCLINE*), definition related with grid, initial condition and forcing files (*GRDNAME*, *ININAME* and *FRCNAME*).

To run the model, use following steps and

```
> cd /RS/users/$USER/workshop/day2
> ln -s src/oceanM .
> mkdir output
> wget https://github.com/uturuncoglu/summer_school-resm_2016/raw/master/day2/bsea.in
> cp src/roms-r809/ROMS/External/varinfo.dat .
```

edit **varinfo.dat**, which is used to define unit conversions (i.e. *scale_factor* and *add_offset*). The original definitions are

```
'Pair'                                             ! Input
  'surface air pressure'
  'millibar'                                       ! [millibar]
  'Pair, scalar, series'
  'pair_time'
  'idPair'
  'r2dvar'
  1.0d0
...
'Qair'                                             ! Input
  'surface air relative humidity'                  !  relative    or  specific
  'percentage'                                     ! [percentage  or  g/kg    ]
  'Qair, scalar, series'
  'qair_time'
  'idQair'
  'r2dvar'                                         ! relative        specific
  0.01d0  
```

and the modified definitions are

```
'Pair'                                             ! Input
  'surface air pressure'
  'millibar'                                       ! [millibar]
  'Pair, scalar, series'
  'pair_time'
  'idPair'
  'r2dvar'
  0.01d0
...
'Qair'                                             ! Input
  'surface air relative humidity'                  !  relative    or  specific
  'percentage'                                     ! [percentage  or  g/kg    ]
  'Qair, scalar, series'
  'qair_time'
  'idQair'
  'r2dvar'                                         ! relative        specific
  1.0d0  
```
edit **bsea.in** and modify parameters if it is necessary. Then, the the model can be submitted to LSF job scedular using following job submission script,

```
#!/bin/bash
#BSUB -P workshop
#BSUB -J roms 
#BSUB -q workshop 
#BSUB -m anadolu_dual 
#BSUB -o %J.out
#BSUB -e %J.err
#BSUB -a intelmpi4
#BSUB -n 32

mpirun.lsf ./oceanM bsea.in >& romsout.txt
```

and the job can be submitted using **bsub < roms.lsf** command.

## Day3: Model Coupling with Regional Earth System Model (RegESM) modeling system

The open source RegESM modeling system is developed by ITU. The source is maintained by GitHub to allow version controlling and issue traking. The source code can be reachable from [here](https://github.com/uturuncoglu/RegESM). 