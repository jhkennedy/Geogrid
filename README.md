# Geogrid

ISCE module for mapping between pixel displacement in a radar image pair (in radar coordinates) and motion velocity at a user-defined geographic-coordinate grid (in geographic coordinates) where the Digital Elevation Model (DEM) and/or local surface slope and coarse motion velocity maps are defined


Copyright (C) 2019 California Institute of Technology.  Government Sponsorship Acknowledged.

Citation: https://github.com/leiyangleon/geogrid

## 1. Authors


Piyush Agram (JPL/Caltech; piyush.agram@jpl.nasa.gov), Yang Lei (GPS/Caltech; ylei@caltech.edu)


       
       
## 2. Features


* user can define a grid in geographic coordinates provided in the form of a DEM with arbitrary EPSG code, 
* the program will extract the portion of the grid that overlaps with the given coregistered radar image pair, 
* return the range and azimuth pixel indices in the radar image pair for each grid point
* return the range and azimuth coarse displacement given the motion velocity maps and the local surface slope maps in the direction of both geographic x- and y-coordinates (they must be provided at the same grid as the DEM)
* return the matrix of conversion coefficients that can convert the fine range and azimuth displacement between the two radar images (estimated with the ISCE module "autorift" or "ampcor"/"denseampcor") to motion velocity in geographic x- and y-coordinates
* all outputs are in the format of GeoTIFF with the same EPSG code as input

## 3. Demo

<img src="figures/optical1.png" width="50%">

***Test area and dataset: optical image over the Jakobshavn glacier where the red rectangle marks boundary of the Sentinel-1A/B image pair (20170221-20170227). Input files in this test scenario consist of the Digital Elevation Model (DEM), local surface slope maps (in both x- and y-direction) and coarse motion velocity maps (in both x- and y-direction) over the entire Greenland, where all maps share the same geographic-coordinate grid with 240-m spacing and spatial reference system with EPSG code 3413 (a.k.a WGS 84 / NSIDC Sea Ice Polar Stereographic North).***



<img src="figures/geogrid.png" width="100%">

***Output of "geogrid" module: (a) range pixel index at each grid point, (b) azimuth pixel index at each grid point, (c) range coarse displacement at each grid point, (d) azimuth coarse displacement at each grid point. Note: only the portion of the grid overlapping with the radar image has been extracted and shown.***

This is obtained by implementing the following command line:

       testGeogrid.py -m master_image_folder -s slave_image_folder -d demname -sx dhdxname -sy dhdyname -vx vxname -vy vyname

where "master_image_folder" and "slave_image_folder" are the folders storing master and slave image information (e.g. radar parameters), and "demname", "dhdxname", "dhdyname", "vxname", "vyname" are defined below in the instructions.


Using the matrix of conversion coefficients, when fine pixel displacement are estimated from radar data, they can be immediately converted to motion velocity. See the final result below by using the matrix of conversion coefficients from the "geogrid" module and the radar-estimated fine pixel displacement from the "autorift" module.


<img src="figures/autorift2.png" width="100%">

***Final motion velocity results by combining outputs from "geogrid" and "autorift" modules: (a) estimated motion velocity from Sentinel-1 data (x-direction; in m/yr), (b) coarse motion velocity from input data (x-direction; in m/yr), (c) estimated motion velocity from Sentinel-1 data (y-direction; in m/yr), (b) coarse motion velocity from input data (y-direction; in m/yr). Notes: all maps are established exactly over the same geographic-coordinate grid from input.***


## 4. Install

* First install ISCE
* Put the "geoAutorift" folder and the "Sconscript" file under the "contrib" folder that is one level down ISCE's source directory (denoted as "isce-version"; where you started installing ISCE), i.e. "isce-version/contrib/" (see the snapshot below)

<img src="figures/install_snapshot.png" width="35%">

* run "scons install" again from ISCE's source directory "isce-version" using command line


## 5. Instructions



* It is recommended to run ISCE up to the step where coregistered SLC's are done, e.g. "mergebursts" for using topsApp.

For quick use:
* Refer to the file "testGeogrid.py" for the usage of the module and modify it for your own purpose
* Input files include the master image folder (required), slave image folder (required), a DEM (required), local surface slope maps, velocity maps
* Output files include 1) the range and azimuth pixel indices, 2) the range and azimuth coarse displacement, 3) the conversion coefficients from radar range and azimuth displacement to motion velocity in geographic x-coordinate, and 4) the conversion coefficients from radar range and azimuth displacement to motion velocity in geographic y-coordinate. 

_Note: among these, 1) will always be created, while 2-4) will be generated contingent upon that local surface slope and velocity maps are provided_

For modular use:
* In Python environment, type the following to import the "geogrid" module and initialize the "geogrid" object

       import isce
       from components.contrib.geoAutorift.geogrid.Geogrid import Geogrid
       obj = Geogrid()
       obj.configure()

* The "geogrid" object has several parameters that have to be set up (listed below; can also be obtained by referring to "testGeogrid.py"): 

       ------------------radar parameters------------------
       startingRange:       starting range
       rangePixelSize:      range pixel size
       sensingStart:        starting azimuth time
       prf:                 pulse repition frequency 
       lookSide:            look side, e.g. -1 for right looking 
       repeatTime:          time period between the acquisition of the two radar images
       numberOfLines:       number of lines (in azimuth)
       numberOfSamples:     number of samples (in range)
       orbit:               ISCE orbit data structure
       ------------------input file names------------------
       demname:             (input; required) file name of the DEM
       dhdxname:            (input; not required) file name of the local surface slope in x-coodinate
       dhdyname:            (input; not required) file name of the local surface slope in y-coodinate
       vxname:              (input; not required) file name of the motion velocity in x-coodinate
       vyname:              (input; not required) file name of the motion velocity in y-coodinate
       ------------------output file names------------------
       winlocname:          (output) file name for the range and azimuth pixel indices (at each grid point)
       winoffname:          (output) file name of the range and azimuth coarse displacement (at each grid point)
       winro2vxname:        (output) file name of the conversion coefficients from radar displacement (range and azimuth) to motion velocity in x-coordinate (at each grid point)
       winro2vyname:        (output) file name of the conversion coefficients from radar displacement (range and azimuth) to motion velocity in y-coordinate (at each grid point)
       Note: "winoffname", "winro2vxname" and "winro2vyname" will be created only when "dhdxname", "dhdyname", "vxname", and "vyname" are provided

* After the above parameters are set, run the module as below to create the output files

       obj.geogrid()


