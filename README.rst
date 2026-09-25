
.. _pycart-ex-arrow:

Demo 2: Closer Analysis of Simple Arrow Shape
=============================================

The second example is similar to the first pyCart demo except that four fins
have been added and more details of the input files are explained. To get
started, clone the following repo and enter the folder and run a couple of
commands:

    .. code-block:: console

        $ git clone https://github.com/nasa-ddalle/pycart02-arrow.git
        $ cd pycart02-arrow
        $ ./copy-files.py
        $ cd work/

This will copy all of the files into a newly created ``work/`` folder. Follow
the instructions below by entering that ``work/`` folder; the purpose is that
you can easily delete the ``work/`` folder and restart the tutorial at any
time.

The geometry used for this shape is a capped cylinder with four fins and 9216
faces and seven components.  The surface triangulation, ``arrow.tri``, is
shown below.

    .. figure:: arrow01.png
        :width: 4in
        
        Simple bullet shape triangulation with four fins
        
This example is set up with a larger run matrix in order to demonstrate more of
the features of pyCart.

    .. code-block:: none
    
        $ pycart -c
        Using pycart JSON file: pyCart.json
        Case Config/Run Directory    Status  Iterations Que CPU Time
        ---- ----------------------- ------- ---------- --- --------
        0    poweroff/m1.25a0.0r0.0  ---     /          .
        1    poweroff/m1.25a1.0r0.0  ---     /          .
        2    poweroff/m1.25a1.0r15.0 ---     /          .
        3    poweroff/m1.25a1.0r30.0 ---     /          .
        4    poweroff/m1.25a1.0r45.0 ---     /          .
        5    poweroff/m1.5a0.0r0.0   ---     /          .
        6    poweroff/m1.5a1.0r0.0   ---     /          .
        7    poweroff/m1.5a1.0r15.0  ---     /          .
        8    poweroff/m1.5a1.0r30.0  ---     /          .
        9    poweroff/m1.5a1.0r45.0  ---     /          .
        10   poweroff/m1.75a0.0r0.0  ---     /          .
        11   poweroff/m1.75a1.0r0.0  ---     /          .
        12   poweroff/m1.75a1.0r15.0 ---     /          .
        13   poweroff/m1.75a1.0r30.0 ---     /          .
        14   poweroff/m1.75a1.0r45.0 ---     /          .
        15   poweroff/m2.0a0.0r0.0   ---     /          .
        16   poweroff/m2.0a1.0r0.0   ---     /          .
        17   poweroff/m2.0a1.0r15.0  ---     /          .
        18   poweroff/m2.0a1.0r30.0  ---     /          .
        19   poweroff/m2.0a1.0r45.0  ---     /          .
        20   poweroff/m2.5a0.0r0.0   ---     /          .
        21   poweroff/m2.5a1.0r0.0   ---     /          .
        22   poweroff/m2.5a1.0r15.0  ---     /          .
        23   poweroff/m2.5a1.0r30.0  ---     /          .
        24   poweroff/m2.5a1.0r45.0  ---     /          .

        ---=25,
        
Input Files
-----------
Let's look at the files in this folder.

    * ``pyCart.json``: Master settings for running pyCart, input to ``pycart``
    * ``arrow.tri``: Surface triangulation (ASCII)
    * ``arrow.xml``: Names for components of the surface
    * ``matrix.csv``: Run matrix
    
JSON Settings
^^^^^^^^^^^^^
The ``pyCart.json`` file contains the master settings divided into several
sections, which we will discuss in more detail.  The overall contents of the
file look something like the following, with the ``...`` replaced by more
content.

    .. code-block:: javascript
    
        {
            // Iteration control and command-line inputs
            "RunControl": {
                // Run sequence
                "PhaseSequence": [0],
                "PhaseIters": [200],
                ...
            },
            
            ...
            
            // RunMatrix (i.e. run matrix) description
            "RunMatrix": {
                "Keys": ["Mach", "alpha_t", "phi", "config"],
                "File": "matrix.csv",
                "GroupMesh": false
            }
        }

The first section (actually, the order does not matter, but it's the first
section in the file provided) is the ``"RunControl"`` section, which has
settings for the overall run procedure (such as number of iterations, whether
or not to submit the job to a queue, etc.) and command-line inputs to the
various Cart3D programs.

    .. code-block:: javascript
    
        "RunControl": {
            // Run sequence
            "PhaseSequence": [0],
            "PhaseIters": [200],
            // Verbosity
            "Verbose": true,
            // System configuration
            "nProc": 8,
            // Options for ``flowCart``
            "flowCart": {
                "it_fc": 200,
                "mpi_fc": 0,
                "cfl": 1.1,
                "mg_fc": 3,
                "y_is_spanwise": true
            },
            // Defines the flow domain automatically
            "autoInputs": {"r": 8},
            // Volume mesh options
            "cubes": {
                "maxR": 10,
                "pre": "preSpec.c3d.cntl",
                "cubes_a": 10,
                "cubes_b": 2,
                "reorder": true
            }
        },
        
The ``"flowCart"`` section contains command-line inputs for running
``flowCart``, which is the main flow solver of Cart3D, or ``mpix_flowCart``,
which is the MPI version of the same. Many of the variable names, such as
*it_fc*, are copied from Cart3D's template ``aero.csh`` scripts or
command-line inputs to Cart3D's ``flowCart``. The three main options (which are
required for any pyCart project) are *PhaseSequence*, *PhaseIters*, and
*it_fc*.

    +-----------------+-------------------------------------------------------+
    | Variable        | Description                                           |
    +=================+=======================================================+
    | *it_fc*         | Number of iterations for each call to ``flowCart``,   |
    |                 | short for ``iterations_flowCart``; command-line input |
    |                 | is ``flowCart -N $it_fc``                             |
    +-----------------+-------------------------------------------------------+
    | *PhaseSequence* | Input sequence, tells pyCart to run phase 0; in more  |
    |                 | complex projects, this will be a list like ``[0,1,3]``|
    +-----------------+-------------------------------------------------------+
    | *PhaseIters*    | Min iterations for each phase; this tells pyCart to   |
    |                 | continue calling ``flowCart`` until 200 iterations    |
    |                 | have been run.  If this was ``400``, pyCart would     |
    |                 | automatically run ``flowCart`` twice using the first  |
    |                 | run's results as inputs to the second                 |
    +-----------------+-------------------------------------------------------+
    
The *Verbose* option is relatively self-explanatory in that more information is
printed to either the terminal or the PBS output file.  In particular, each
command that is issued to the top-level terminal also prints the name of the
directory in which it is run and the names of the files storing its ``STDOUT``
and ``STDERR`` output.
    
For a simple case, these parameters seem unnecessarily confusing. Why not just
tell ``flowCart`` how many iterations to run and be done with it? For one
thing, *PhaseIters* specifies a required number of iterations whereas *it_fc* just
suggests to ``flowCart`` or ``mpix_flowCart`` how many iterations to run. If
``flowCart`` exits early due to some kind of failure, this convention means
that pyCart will clearly alert us.

Secondly, some applications require more sophisticated approach. A common
example is a hypersonic case that needs to be run in first-order mode for a few
iterations first. It might have something like ``"PhaseIters": [0, 400]`` and
``"PhaseSequence": [0, 1]``. This tells pyCart to run input set ``0`` until it
has run at least ``0`` iterations and then phase ``1`` until it has run at
least ``400`` iterations.

The remaining inputs are quite a bit simpler. For example *nProc* sets the
total number of cores or threads to use. The next section allows pyCart to use
the Cart3D binary ``autoInputs`` to create the flow domain and basic volume
mesh parameters with the command ``autoInputs -r 8``, which sets the farfield
boundary at roughly 8 times the size of your surface triangulation.

Running ``autoInputs`` creates files ``input.c3d`` and ``preSpec.c3d.cntl``,
which are given as inputs to the volume generator ``cubes``.

    .. code-block:: javascript
    
        "Mesh": {
            // Surface triangulation
            "TriFile": "arrow.tri"
        },
        
The *Mesh* section controls inputs to the Cart3D commands that produce the
volume mesh. The *TriFile* setting is relatively obvious and points to the name
of the surface triangulation.

    .. code-block:: javascript
    
        "Config": {
            // Defer to a file for most things.
            "File": "arrow.xml",
            // Declare forces and moments
            "Force": ["cap", "body", "fins", "bullet_no_base", "bullet_total"],
            "RefPoint": {"bullet_no_base": [0.0, 0.0, 0.0]},
            // Reference quantities
            "RefArea": 3.14159,
            "RefLength": 1.0
        },
        
The *Config* section gives instructions about which components to track, what
moment reference points to use, and similar definitions. The XML file allows
Cart3D and pyCart to refer to define groups of components and refer to
components by name instead of memorizing their numbers. The *Force* option
specifies a list of components on which ``flowCart`` should track the force at
each iteration. This creates files ``cap.dat``, ``body.dat``, ``fins.dat``,
etc. Then *RefPoint* specifies the list of components for which to also track
the moments, and the moment reference point to use for each such component. In
this case, the moments will be reported alongside the forces in
``bullet_no_base.dat``.

The *RefArea* and *RefLength* parameters are used here to specify global
reference values, but it is possible to use different reference lengths or
areas for different components in the same run.

    .. code-block:: javascript
    
        "RunMatrix": {
            "Keys": ["Mach", "alpha_t", "phi", "config"],
            "File": "matrix.csv",
            "GroupMesh": false
        }

The final section (actually, the order is irrelevant, but it's the last section
in this file) describes the run matrix, i.e. trajectory. The *Keys* parameter
lists the names of variables that will change in the run matrix, i.e. the
independent variables. In this case, we are using Mach number, total angle of
attack, velocity roll angle, and configuration name. There is a set of
predefined trajectory keys, and all four of these examples are in that set,
but later examples will show how to define customized trajectory keys in this
section.

The *File* parameter points to a file in which the cases to run are listed, and
*GroupMesh* specifies whether or not each case can use the same mesh.  Setting
it to ``true`` means that ``cubes`` is only run once for the matrix (more
accurately, once for each group, but this example has only one group) and
linked into each case folder; with ``false``, as here, each case builds and
keeps its own mesh.  The *config* key (here always ``poweroff``) gives a name
for the folder in which to put all the cases of that configuration, which
explains why a typical case is named ``poweroff/m1.5a1.0r0.0``, for example.

There are two more sections in the ``pyCart.json``, which describe various
products.

Triangulation File: ``arrow.tri``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The surface geometry is defined in an ASCII file in a straightforward Cart3D
format.  A summary of the contents is shown below.

    .. code-block:: none
    
        4610  9216
        +4.81527351e-03 +9.80171422e-02 +0.00000000e+00
        +1.92147203e-02 +1.95090326e-01 +0.00000000e+00
        ...
        +6.37716534e+00 +1.58689240e-08 +1.06574801e+00
        385 386 16
        386 387 17
        ...
        2565 4257 2530
        1
        1
        ...
        11

The first line is a summary of the contents of the file.  It states that there
are ``4610`` nodes, i.e. three-dimensional points in space, and ``9216``
triangles.  What follows is 4610 lines with three floating point numbers per
line.  Next is 9216 lines in which each line defines one triangle.  For example,
the first triangle connects node ``385`` to node ``386`` to node ``16``.  After
9216 such lines, there are 9216 more lines with a single integer on each line
that defines the component ID of each triangle.  Thus triangle 1 is part of
component 1, triangle 2 is part of component 1, and the last triangle is part of
component 11.

Component Names: ``arrow.xml``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Cart3D uses an optional XML file that associates names with each component.  It
uses a standard XML format with component IDs (the numbers at the end of the
``.tri`` file discussed above) with a ``Face Label`` value inside a
``<Data>`` tag.  It also allows for the definition of a "container" component
that is the combination of several other components.  This makes it possible to 
track ``fin1`` separately while also tracking all the ``fins`` as a group.  The
contents of the file are shown below.

    .. code-block:: xml
    
        <?xml version="1.0" encoding="ISO-8859-1"?>

        <Configuration Name="bullet sample" Source="bullet.tri">
        
         <!-- Containers -->
          <Component Name="bullet_no_base" Type="container" Parent="bullet_total">
          </Component>
          <Component Name="fins" Type="container" Parent="bullet_no_base">
          </Component>
         
          <Component Name="bullet_total"   Type="container">
          </Component>
         <!-- Containers -->
        
         <!-- body -->
          <Component Name="cap" Type="tri">
           <Data> Face Label=1 </Data>
          </Component>
         
          <Component Name="body" Type="tri">
           <Data> Face Label=2 </Data>
          </Component>
         
          <Component Name="base" Parent="bullet_total" Type="tri">
           <Data> Face Label=3 </Data>
          </Component>
         <!-- body -->
         
         <!-- fins -->
          <Component Name="fin1" Parent="fins" Type="tri">
           <Data> Face Label=11 </Data>
          </Component>
          
          <Component Name="fin2" Parent="fins" Type="tri">
           <Data> Face Label=12 </Data>
          </Component>
          
          <Component Name="fin3" Parent="fins" Type="tri">
           <Data> Face Label=13 </Data>
          </Component>
          
          <Component Name="fin4" Parent="fins" Type="tri">
           <Data> Face Label=14 </Data>
          </Component>
         <!-- fins -->
        
        </Configuration>

Run Matrix File: ``matrix.csv``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The conditions at which Cart3D are read from this file, which is a simple list
of conditions.

    .. code-block:: none
    
        # Mach, alpha, phi, config
        1.25,   0.00,   0.0  , poweroff
        1.25,   1.00,   0.0  , poweroff
        1.25,   1.00,   15.0 , poweroff
        ...
        2.50,   1.00,   45.0 , poweroff

The comment line at the top is not read by pyCart but is placed there for
readability.  Further, the commas are not required; pyCart and other CAPE
modules read trajectory files in a pretty general way.

Run Directives
--------------
Let's run one case, but not the first case.  We can do this by using the
``pycart -I`` command to pick out a specific index or a range of indices.

    .. code-block:: none
    
        $ pycart -I 12
        Using pycart JSON file: pyCart.json
        Case Config/Run Directory    Status  Iterations  Que CPU Time
        ---- ----------------------- ------- ----------- --- --------
        12   poweroff/m1.75a1.0r15.0 ---     /           .
          Case name: 'poweroff/m1.75a1.0r15.0' (index 12)
          Preparing surface triangulation...
          Reading tri file(s):
            arrow.tri
         > autoInputs -r 8 -t Components.i.tri -maxR 10 -nDiv 4
             (PWD = 'poweroff/m1.75a1.0r15.0/')
             (STDOUT = 'autoInputs.out')
             (STDERR = 'autoInputs.out')
        Using template for 'input.cntl' file
             Starting case 'poweroff/m1.75a1.0r15.0'
         > cubes -pre preSpec.c3d.cntl -maxR 10 -reorder -a 10 -b 2
             (PWD = 'poweroff/m1.75a1.0r15.0/')
             (STDOUT = 'cubes.out')
             (STDERR = 'cubes.out')
         > mgPrep -n 3
             (PWD = 'poweroff/m1.75a1.0r15.0/')
             (STDOUT = 'mgPrep.out')
             (STDERR = 'mgPrep.out')
         > flowCart -his -clic -N 200 -y_is_spanwise -limiter 2 -T -cfl 1.1 -mg 3 -no_fmg -binaryIO -tm 0
             (PWD = 'poweroff/m1.75a1.0r15.0/')
             (STDOUT = 'flowCart.out')
             (STDERR = 'flowCart.err')

        Submitted or ran 1 job(s).

        ---=1, 

Because ``"Verbose": true`` is set in ``pyCart.json``, every command issued is
echoed along with its working directory and the files capturing its ``STDOUT``
and ``STDERR``.

We can check the status of all the cases at Mach 1.75 using the following.  Like
the previous example, the CPU time is below 0.1 hours.

    .. code-block:: none
    
        $ pycart -I 11:15 -c
        Using pycart JSON file: pyCart.json
        Case Config/Run Directory    Status  Iterations Que CPU Time
        ---- ----------------------- ------- ---------- --- --------
        11   poweroff/m1.75a1.0r0.0  ---     /          .
        12   poweroff/m1.75a1.0r15.0 DONE    200/200    .       0.07
        13   poweroff/m1.75a1.0r30.0 ---     /          .
        14   poweroff/m1.75a1.0r45.0 ---     /          .

        ---=3, DONE=1, 

Note that the ``-I`` option displays and selects the global case indices, and
the range ``11:15`` is exclusive of the upper bound, so this covers the four
angle-of-attack cases at Mach 1.75.

We can use a more direct method to select cases with a certain Mach number using
a constraint.  Let's run the remaining Mach 1.75 cases using that capability.

    .. code-block:: none
    
        $ pycart --cons "Mach==1.75, alpha_t==1.0"
        Using pycart JSON file: pyCart.json
        Case Config/Run Directory    Status  Iterations  Que CPU Time
        ---- ----------------------- ------- ----------- --- --------
        11   poweroff/m1.75a1.0r0.0  ---     /           .
          Case name: 'poweroff/m1.75a1.0r0.0' (index 11)
          Preparing surface triangulation...
          Reading tri file(s):
            arrow.tri
         > autoInputs -r 8 -t Components.i.tri -maxR 10 -nDiv 4
             (PWD = 'poweroff/m1.75a1.0r0.0/')
             (STDOUT = 'autoInputs.out')
             (STDERR = 'autoInputs.out')
        Using template for 'input.cntl' file
             Starting case 'poweroff/m1.75a1.0r0.0'
         > cubes -pre preSpec.c3d.cntl -maxR 10 -reorder -a 10 -b 2
             (PWD = 'poweroff/m1.75a1.0r0.0/')
             (STDOUT = 'cubes.out')
             (STDERR = 'cubes.out')
         > mgPrep -n 3
             (PWD = 'poweroff/m1.75a1.0r0.0/')
             (STDOUT = 'mgPrep.out')
             (STDERR = 'mgPrep.out')
         > flowCart -his -clic -N 200 -y_is_spanwise -limiter 2 -T -cfl 1.1 -mg 3 -no_fmg -binaryIO -tm 0
             (PWD = 'poweroff/m1.75a1.0r0.0/')
             (STDOUT = 'flowCart.out')
             (STDERR = 'flowCart.err')
        12   poweroff/m1.75a1.0r15.0 DONE    200/200     .        0.1
        13   poweroff/m1.75a1.0r30.0 ---     /           .
          Case name: 'poweroff/m1.75a1.0r30.0' (index 13)
          Preparing surface triangulation...
         > autoInputs -r 8 -t Components.i.tri -maxR 10 -nDiv 4
             (PWD = 'poweroff/m1.75a1.0r30.0/')
             (STDOUT = 'autoInputs.out')
             (STDERR = 'autoInputs.out')
        Using template for 'input.cntl' file
             Starting case 'poweroff/m1.75a1.0r30.0'
         > cubes -pre preSpec.c3d.cntl -maxR 10 -reorder -a 10 -b 2
             (PWD = 'poweroff/m1.75a1.0r30.0/')
             (STDOUT = 'cubes.out')
             (STDERR = 'cubes.out')
         > mgPrep -n 3
             (PWD = 'poweroff/m1.75a1.0r30.0/')
             (STDOUT = 'mgPrep.out')
             (STDERR = 'mgPrep.out')
         > flowCart -his -clic -N 200 -y_is_spanwise -limiter 2 -T -cfl 1.1 -mg 3 -no_fmg -binaryIO -tm 0
             (PWD = 'poweroff/m1.75a1.0r30.0/')
             (STDOUT = 'flowCart.out')
             (STDERR = 'flowCart.err')
        14   poweroff/m1.75a1.0r45.0 ---     /           .
          Case name: 'poweroff/m1.75a1.0r45.0' (index 14)
          Preparing surface triangulation...
         > autoInputs -r 8 -t Components.i.tri -maxR 10 -nDiv 4
             (PWD = 'poweroff/m1.75a1.0r45.0/')
             (STDOUT = 'autoInputs.out')
             (STDERR = 'autoInputs.out')
        Using template for 'input.cntl' file
             Starting case 'poweroff/m1.75a1.0r45.0'
         > cubes -pre preSpec.c3d.cntl -maxR 10 -reorder -a 10 -b 2
             (PWD = 'poweroff/m1.75a1.0r45.0/')
             (STDOUT = 'cubes.out')
             (STDERR = 'cubes.out')
         > mgPrep -n 3
             (PWD = 'poweroff/m1.75a1.0r45.0/')
             (STDOUT = 'mgPrep.out')
             (STDERR = 'mgPrep.out')
         > flowCart -his -clic -N 200 -y_is_spanwise -limiter 2 -T -cfl 1.1 -mg 3 -no_fmg -binaryIO -tm 0
             (PWD = 'poweroff/m1.75a1.0r45.0/')
             (STDOUT = 'flowCart.out')
             (STDERR = 'flowCart.err')

        Submitted or ran 3 job(s).

        ---=3, DONE=1,
        
It is also possible to select these cases using ``pycart --filter m1.75a1``,
``pycart --glob "*m1.75a1*"``, or ``pycart --re "m1\.75a1"``. The last of these
checks for a regular expression, which allows more complex filters to be
applied.
        
Run Folders and Output Files
----------------------------
Let's take a look at the files that pyCart created.  Because ``GroupMesh`` is
``false``, the ``poweroff/`` folder only contains the case folders:

    .. code-block:: none
    
        $ cd poweroff/
        $ ls
        m1.75a1.0r0.0  m1.75a1.0r15.0  m1.75a1.0r30.0  m1.75a1.0r45.0

All of the mesh files live in each case folder.

    .. code-block:: none
    
        $ cd m1.75a1.0r0.0/
        $ ls
        autoInputs.out        forces.dat          pycart_start.dat
        body.dat              functional.dat      pycart_time.dat
        bullet_no_base.dat    history.dat         run.00.200
        bullet_total.dat      input.00.cntl       run_cart3d.pbs
        cape                  input.c3d           Mesh.c3d.Info
        cap.dat               input.cntl          Mesh.mg.c3d
        case.json             loadsCC.dat         Mesh.R.c3d
        check.00200           mgPrep.out          moments.dat
        checkDT.00200         preSpec.c3d.cntl
        Components.i.00200.plt cutPlanes.00200.plt
        Components.i.tri      entire.dat
        Components.i.triq     fins.dat
        conditions.json       flowCart.err
        Config.xml            cubes.out

The ``.out`` files save STDIO printouts from the mesh-generation commands.
The ``Mesh.mg.c3d`` is the actual mesh file, including multigrid levels
(i.e., coarsened grids). Our surface triangulation, ``arrow.tri`` is copied
to ``Components.i.tri`` in this folder; and the configuration file
``arrow.xml`` is copied to ``Config.xml``. The single mesh without
multigrid levels is ``Mesh.R.c3d``, and the remaining files are created by
``autoInputs``.  If ``GroupMesh`` were ``true``, all of these files would
appear in the ``poweroff/`` folder instead, created once and linked into
each case folder.

The contents of ``input.c3d`` set the minimum and maximum *x*, *y*, and *z*
coordinates for the domain on which Cart3D is solved, and is a pretty unique
file.  In this case, it is created automatically by ``autoInputs`` based on the
physical size of the ``Components.i.tri`` surface.  The other auto-created
file, ``preSpec.c3d.cntl`` defines regions in which the volume mesh should
have increased resolution.  Calling ``cubes`` also generates regions of
increased resolution based on distance from the surface, but this file can be
used to request more detail.  In addition to some header lines, the contents
look something like the following.

    .. code-block:: none
    
        # BBox: level   Xmin   Xmax      Ymin   Ymax      Zmin    Zmax
        #       (int)  (float) (float) (float) (float)  (float) (float)
        
        
        $__Prespecified_Adaptation_Regions:     # <-Section head (req'd)
        BBox: 6   -0.800   8.800   -4.800   4.800   -4.800   4.800   #  Config BBox
        BBox: 7   -0.299   1.299   -0.800   0.800   -0.799   0.799   #  Comp #0
        BBox: 7    1.700   7.300   -0.800   0.800   -0.800   0.800   #  Comp #1
        BBox: 7    7.201   8.799   -0.800   0.800   -0.799   0.799   #  Comp #2
        BBox: 7    6.479   7.653   -0.401   0.401    1.099   1.900   #  Comp #10
        BBox: 7    6.479   7.653   -1.900  -1.099   -0.401   0.401   #  Comp #11
        BBox: 7    6.479   7.653   -0.401   0.400   -1.900  -1.099   #  Comp #12
        BBox: 7    6.479   7.653    1.099   1.900   -0.400   0.401   #  Comp #13

The third row of *BBox* commands define a region with *x*-coordinates between
1.7 and 7.3, *y*-coordinates between -0.8 and +0.8, and *z*-coordinates between
-0.8 and +0.8.  Within this region, ``cubes`` must make a mesh that has been
refined at least 7 times.  In other words, the mesh size must be at least 128
times smaller than the original mesh.

The folder ``cape/`` stores pyCart's own log files for this case, and
``input.cntl`` is a link to ``input.00.cntl``; everything else is a real
file created or copied by pyCart or ``flowCart``.

Most of the files ending with ``.dat`` are iterative history files. Some of
these are standard results of running ``flowCart``, and others are specifically
requested. The most special of these is ``history.dat``, which contains the
residual history. In pyCart, this file is used to determine how many iterations
have been run. With the exception of some comment lines, each line reports one
iteration number and the residual at that iteration.

The files ``forces.dat`` and ``moments.dat`` report the forces and
moments on the ``entire`` component, i.e. the entire triangulation. These files
are always produced, report results before any axis changes, and are ignored by
pyCart. Five other files, ``body.dat``, ``bullet_no_base.dat``,
``bullet_total.dat``, ``cap.dat``, and ``fins.dat``, are specifically requested.
Cart3D produces them because the ``input.cntl`` file contains lines ``Force
body``, ``Force cap``, etc. in the ``$__Force_Moment_Processing:`` section.
Although we did not request ``entire`` in our pyCart setup, it got produced
here because the template ``input.cntl`` file contains the line ``Force
entire``. These ``.dat`` files are used by pyCart to read the iterative history
of forces and moments on parts of the vehicle.

The volume and surface results files are ``check.00200``,
``Components.i.00200.plt``, ``Components.i.triq``, and ``cutPlanes.00200.plt``.
The ``check.00200`` file is a binary file used and created by Cart3D, and the
``plt`` files are Tecplot files. These Tecplot files are created by Cart3D, and
pyCart changes the file names by inserting the iteration numbers to which they
correspond. Finally, the ``Components.i.triq`` file is very similar to the
surface triangulation except with extra info describing the state solution at
each vertex. Note that the ``Components.i.00200.plt`` and ``Components.i.triq``
files do not contain identical information because the Tecplot file references
the Cartesian volume mesh projected onto the surface while the ``triq`` file
only has solution data at the triangulation vertices.

Also in this folder are the files ``run_cart3d.pbs``, which is a script used to
run ``flowCart``.

    .. code-block:: bash
    
        #!/bin/bash
        #PBS -S /bin/bash
        #PBS -N pyCart-m1.75a1r0poweroff
        #PBS -r n
        #PBS -j oe
        #PBS -l select=1:ncpus=1
        #PBS -l walltime=8:00:00
        #PBS -q normal
        
        # Go to the working directory.
        cd /path/to/pycart02-arrow/work/poweroff/m1.75a1.0r0.0
        
        # Set umask.
        umask 0027
        
        # Additional shell commands
        
        # Call the main executable
        python3 -m cape.pycart run

The script includes some PBS settings (which are not used in this example), a
command to change to the correct folder using an absolute path, and a command
to determine the correct Cart3D command; the last is normally
``flowCart`` or ``mpix_flowCart``, invoked by re-entering ``pycart`` (which
reads the case settings and reconstructs the command line).  The file
``case.json`` contains all of the ``pyCart.json`` settings from the
``"flowCart"`` section, because they are needed to determine the command-line
inputs.

That covers the essential files for this example.  The very import
``input.cntl`` file (which in this case is just a link to
``input.00.cntl``) is worthy of far more discussion, and there are several
other files that have varying degrees of utility, but that will have to come at
a different time and place.

