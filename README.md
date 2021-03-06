# HARMONI Simulator Pipeline

Written by Simon Zieleniewski & Sarah Kendrew

Last Edited: 28-04-16


## PURPOSE:

The HARMONI simulation pipeline code is designed to simulate observations using the HAMONI integral field spectrograph on the E-ELT. The pipeline takes a high spectral and spatial resolution input FITS format datacube, and processes it according to the chosen observing parameters. The code returns several output cubes:
 - a mock observed cube (with variance as first extension)
 - a background cube (with variance as first extension) (both noisy and noiseless versions returned)
 - a SNR cube

With further options to return:
 - a sky subtracted ("Reduced") cube (with variance as first extension)
 - a transmission cube
 - an object cube giving object flux only (both noisy and noiseless versions returned)

This pipeline provides a self contained and consistent way of performing detailed simulations of the performance of HARMONI. The mock observed output cubes can be analysed as if they are real observational data.


## FILES:

The pipeline currently uses 11 main python files:
```
hsim.py
TheSimulator.py
TheSimulator_GUI.py
TheSimulator_Specres.py
TheSimulator_PSFs.py
TheSimulator_Background.py
TheSimulator_Transmission.py
TheSimulator_ADR.py
TheSimulator_Loop.py
TheSimulator_Lowres.py
TheSimulator_Config.py
```

as well as several functions found in the src/modules/ sub directory.

There are also several data files:
radiance_resolution_0.15_angstroms_MICRONS.txt - From ESO Skycalc
transmission_resolution_0.15_angstroms_MICRONS.txt - From ESO Skycalc
EELT_mirror_reflectivity_mgf2agal.dat.txt - From ESO E-ELT DRM
HARMONIthruput.txt
DetectorQE.txt
HARMONI_R500_low_res_mode.txt


## DEPENDENCIES:

The following Python modules are essential to run the pipeline:
numpy - http://www.scipy.org/scipylib/download.html
scipy - http://www.scipy.org/scipylib/download.html
astropy - http://www.astropy.org/

The following are strongly recommended for optimum running:
wxPython - This is used to load the GUI. It can be downloaded from: http://www.wxpython.org/download.php
pprocess - This is used to allow parallel processing to improve performance. Download from https://pypi.python.org/pypi/pprocess


Make sure the modules can by found by your Python implementation. E.g. in a Python environment try:
```python
        >>> import numpy
        >>> import astropy
        >>> import scipy
        >>> import wx
        >>> import pprocess
```

there should be no errors!


## DIRECTORY SETUP:

The root directory contains several sub directories containing essential data files:

```
./src - main python functions stored here
        /modules - additional python functions stored here
./Output_cubes - output cubes are stored here
./Sim_data - Simulation data is stored here
	 /Background
		/ASM_background - radiance_resolution_0.15_angstroms_MICRONS.txt stored here
	/Throughput - EELT_mirror_reflectivity_mgf2agal.dat.txt, HARMONIthruput.txt, DetectorQE.txt stored here
		/ASM_throughput - transmission_resolution_0.15_angstroms_MICRONS.txt stored here
	/PSFs
	        /SCAO - SCAOdata.txt stored here
	        /LTAO - LTAOdata.txt stored here
        /R500 - HARMONI_R500_mode_data.txt stored here
```

## HOW TO USE:

The code can be run in two ways:

1. Running the GUI:
``$ python hsim.py``
This opens the graphical interface from which you can enter the various parameters and start the simulation.

2. From the command line:
``$ python hsim.py -c arg1 arg2 arg3 etc…``

Use:
``$ python hsim.py -c``
to display arguments list
	
The commands must be entered in the following order:

1. datacube: Input datacube filepath
2. DIT: Exposure time [s]
3. NDIT: No. of exposures
4. band: Photometric band [V+R, Iz+J, H+K, V, R, Iz, J, H, K, V-high, R-high, z, J-high, H-high, K-high, None]
5. x-spax: x spatial pixel (spaxel) scale [mas]
6. y-spax: y spatial pixel (spaxel) scale [mas]
7. telescope: Telescope type (E-ELT, VLT)
8. AO: AO mode (LTAO, SCAO, Gaussian)
9. seeing: Atmospheric seeing FWHM [arcsec] - Between 0.67"-1.10"
10. zenith_ang: Zenith angle [deg]
11. user_PSF: Path to user uploaded PSF file - Enter None if not required
12. PSF blur: Additional telescope jitter [mas]
13. Site_temp: Site/telescope temperature [K]
14. Spec_Nyquist: (True/False) - Set spectral sampling to Nyquist sample spectral resolution element
15. Spec_samp: Spectral sampling [A/pix] - Only used if Spec_Nyquist = False, but enter a value regardless!
16. Noise_force_seed: Force random number seed to take a set value (0=No, 1-2=yes and takes that value)
17. Remove_background: (True/False) - Subtract background spectrum
18. Return_object: (True/False) - Return object cube
19. Return_transmission: (True/False) - Return transmission cube
20. Turn ADR off: (True/False)

Use -h or --help to display help message and exit
Use -c or --cline option to use command line.
Use -o or --odir when using command line to specify output file directory. Default is /hsim/Output_cubes/
Use -p or --proc when using command line to specify number of cores for parallel processing. Default is N_PROC-1 (if N_PROC>1)

An example command line entry:
$ python hsim.py -c -p 5 -o /Path/to/output_dir/ /Path/to/Input_cubes/Input_cube.fits 900 20 H+K 20. 20. E-ELT LTAO 0.67 0. None 0. 280.5 True 1. 0 True True True


## INPUT CUBES:

The pipeline takes an input FITS file datacube containing spatial and spectral information about the astrophysical source of interest. The format should be an (x,y,lambda) datacube with each pixel containing the value of flux per sq. arcsec (e.g. J/s/m^2/A/arcsec^2).

The following FITS headers are essential:

- NAXIS1, NAXIS2, NAXIS3 - Size of the x, y, and wavelength axes respectively.
- CTYPE1, CTYPE2, CTYPE3 - Axis type. Accepted values: [x, y, WAVELENGTH] or [RA, DEC, WAVELENGTH].
- CUNIT1, CUNIT2, CUNIT3 - Units for each axis. Accepted values: [mas, mas, microns], [mas, mas, angstroms], [mas, mas, metres], [mas, mas, nm].
- CDELT1, CDELT2, CDELT3 - Sampling for each axis in units given by CUNIT1/2/3 headers.
- CRVAL3 - Value of array element given by CRPIX3 in units of CUNIT3.
- CRPIX3 - Denotes the array element for CRVAL3. Should be set to 1 so CRVAL3 denotes first wavelength value.
- FUNITS (or BUNIT) - Flux units for each element in datacube. Accepted values: [J/s/m2/um/arcsec2], [J/s/m2/A/arcsec2], [erg/s/cm2/um/arcsec2], [erg/s/cm2/A/arcsec2].
- SPECRES - Spectral resolution FWHM in units of CUNIT3.

All other headers will be ignored by the pipeline so extra information can be added.

N.B, please make sure all header keys do not have white space in them! The FITS handling modules used break down when they find header keys with spaces in. Please use underscores instead.


## OPTIONAL INPUT PSF IMAGE/CUBE:

The pipeline can take an input PSF FITS file instead of using the built-in PSFs. The format should either be a 2D image of a single PSF, or a 3D cube containing PSFs covering the full wavelength range covered by the input datacube.

The following FITS header are essential:

2D PSF FITS image:
- NAXIS1, NAXIS2 - Size of the x, y axes respectively.
- CTYPE1, CTYPE2 - Axis type. Accepted values: [x, y] or [RA, DEC].
- CUNIT1, CUNIT2 - Units for each axis. Accepted values: [mas, mas]], [arcsec, arcsec].
- CDELT1, CDELT2 - Sampling for each axis in units given by CUNIT1/2 headers.

3D PSF FITS cube:
- NAXIS1, NAXIS2, NAXIS3 - Size of the x, y, and wavelength axes respectively.
- CTYPE1, CTYPE2, CTYPE3 - Axis type. Accepted values: [x, y, WAVELENGTH] or [RA, DEC, WAVELENGTH].
- CUNIT1, CUNIT2, CUNIT3 - Units for each axis. Accepted values: [mas, mas, microns], [mas, mas, angstroms], [mas, mas, metres], [mas, mas, nm].
- CDELT1, CDELT2, CDELT3 - Sampling for each axis in units given by CUNIT1/2/3 headers.
- CRVAL3 - Value of array element given by CRPIX3 in units of CUNIT3.
- CRPIX3 - Denotes the array element for CRVAL3. Should be set to 1 so CRVAL3 denotes first wavelength value.





