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

The following commands can be used to download and install ROMS. You must register in [here](https://www.myroms.org/index.php?page=RomsCode) to download the model (**original standalone version, not support with coupling!**).

```
if you already registered and got user name and password
> svn checkout -r 809 --username [ROMS USER NAME] https://www.myroms.org/svn/src/trunk roms-r809

if not, you could download the source code from GitHub
> wget  



```




## Day3: Model Coupling with Regional Earth System Model (RegESM) modeling system

The open source RegESM modeling system is developed by ITU. The source is maintained by GitHub to allow version controlling and issue traking. The source code can be reachable from [here](https://github.com/uturuncoglu/RegESM). 