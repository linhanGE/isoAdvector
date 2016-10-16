Welcome to the IsoAdvector project

# What is IsoAdvector?

IsoAdvector is a geometric Volume-of-Fluid method for advection of a sharp 
interface between two incompressible fluids. It works on both structured and 
unstructured meshes with no requirements on cell shapes. IsoAdvector is meant as 
a replacement for the interface compression with the MULES limiter implemented 
in the interFoam family of solvers.

The ideas behind and performance of the IsoAdvector scheme is documented in this
article:

https://arxiv.org/abs/1601.05392

A number of videos can be found in this youtube channel:

https://www.youtube.com/channel/UCt6Idpv4C8TTgz1iUX0prAA


# Requirement:

A version of the applications, src and run (tutorials) folders exists for each
supported OpenFOAM version as can be seen in the main directory of the project.
Currently the following versions are supported: OpenFOAM-4.0, 2.2.0 and
foam-extend-3.2. The could should run with other versions with minor 
modifications to the source code by the user. I will do my best to correct bugs
in all versions of the code, but it is unrealistic that I will have time to 
implement all changes and new developments to older versions of the code. The 
code is being maintained and developed in the newest OpenFOAM version, so as
a rule of thumb the best performance should be expected with this version, and 
slight difference in behaviour should be expected in older version of the code.

# Installation:

0.  Source a supported OpenFOAM environment: 

    source OpenFOAM-x.x.x/etc/bashrc

1.  In a linux terminal download the package with git by typing:

        git clone https://bitbucket.org/dhifoam/isoadvector.git

2.  Execute the Allwmake script by typing (will finish in a ~1 min):

        cd isoadvector
        ./Allwmake

    Applications will be compiled to your FOAM_USER_APPBIN and libraries will be
    compiled to your FOAM_USER_LIBBIN.
    
3.  Test installation with a simple test case by typing (finishes in secs):
    
	    cd OpenFOAM-x.x.x/run/isoAdvector/discInUniFlow
		./makeAndRunTestCase
	
    Open Paraview and color the mesh by alpha.water or alpha1 field.

# Code structure:

`src/` 

* Contains the three classes implement the IsoAdvector advection scheme:
    - `isoCutFace`
    - `isoCutCell` 
    - `isoAdvection` 
  These are compiled into a library named `libIsoAdvection`. 
* For comparison we also include the CICSAM, HRIC and mHRIC algebraic VOF 
  schemes in `finiteVolume` directory. These are compiled into a library called
  libVOFInterpolationSchemes.

`applications/` 

- `applications/solvers/isoAdvector` 
    - Solves the volume fraction advection equation in either steady flow or 
      periodic predefined flow with the option of changing the flow direction at
      a specified time.
- `applications/solvers/mulesFoam`
    - This is like isoAdvector, but using MULES instead of isoAdvector. Based on
      interFoam.
- `applications/solvers/passiveAdvectionFoam` 
    - This is essentially scalarTransportFoam but using alpha1 instead of T and 
      without diffusion term. Used to run test cases with predefined velocity 
      field using the CICSAM and HRIC schemes.
- `applications/solvers/interFlow` 
    - This is essentially the original interFoam solver with MULES replaced by 
      isoAdvector for interface advection. No changes to PIMPLE loop.
- `applications/utilities/preProcessing/isoSurf` 
    - Sets the initial volume fraction field for either a sphere, a cylinder or 
      a plane. See isoSurfDict for usage.
- `applications/utilities/postProcessing/uniFlowErrors`
    - For cases with spheres and discs in steady uniform flow calculates errors 
      relative to exact VOF solution.

`run/`

- `isoAdvector/` 
    - Contains test cases using isoAdvector to move a volume of fluid in a 
      predescribed velocity field.
- `interFlow/` 
    - Contains test cases using interFlow coupling IsoAdvector with the PIMPLE 
      algorithm for the pressure-velocity coupling.
	
#Classes
	
## IsoCutFace

- Calculates an isosurface from a function f and a function value f0.
- The function is defined in all vertex points of an fvMesh.
- The routine travels through all cells and looks at each cell face edge.
- If the f is above f0 at one vertex and below f0 at the other vertex of an 
  edge, the edge is cut at a position determined by linear interpolation.
- The routine calculates the polygonal "isoFace" inside the cell formed by 
  cutting its edges in this way.
- It also calculates the volume and cell centre of the "submerged" subcell 
  defined as the part of the original cell where f >= f0.

So far the routine assumes that a cut face is cut at two edges. For n-gons with 
n > 3 there will in general be special situations where this is not the case. 
The function should then triangulate such faces interpolating the function to 
the face centre and do the face cutting on the triangular faces.

The routine was deliberately build to travel through all cells instead of all 
faces or edges to obtain its results. The downside of this strategy is that the 
number of times an edge is visited is equal to twice the number of faces to 
which it belongs. The advantage is that parallelisation is trivial. It should be 
noted that the operation performed on each edge is extremely simple.

## IsoCutCell

## IsoAdvection 

- Calculates the total volume of water, dVf, crossing each face in the mesh 
  during the time interval from time t to time t + dt.

# Ongoing work 

Add feature and improvement ideas/requests here. Some of them will be taken over 
by the development team and converted to issues. If you see an open feature 
request in this list that you want to take on, open an issue and leave a comment 
in the issue tracker. 

# Contributors:

* Johan Roenby <jro@dhigroup.com> (Inventor and main developer)
* Hrvoje Jasak <hrvoje.jasak@fsb.hr> (Consistent treatment of boundary faces 
  including processor boundaries, code clean up)
* Vuko Vukcevic <vuko.vukcevic@fsb.hr> (Code review, profiling, porting to 
  foam-extend, bug fixing)
* Tomislav Maric <tomislav@sourceflux.de> (Source file rearrangement)
