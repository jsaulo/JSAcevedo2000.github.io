I used this tutorial created by Pierre DERIAN with some changes to reflect the relevent revison of XIOS2 and the metoffice surge nemo code (different to main trunk) Derian's git hub is here https://github.com/pderian/NEMOGCM-Docker

Follow the steps below as there is some differences to Pierre's method::

	cd $HOME
	git clone https://github.com/NOC-MSM/Belize_workshop.git

	cd Belize_workshop/BUILD_TOOLS


Checkout NEMO code and XIOS code into directories at same level as NEMO-Docker::

	svn co http://forge.ipsl.jussieu.fr/nemo/svn/trunk/NEMOGCM@8395 NEMOGCM
	svn co -r1242 http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS/trunk xios-2.0_r1242
	ln -s xios-2.0_r1242/ XIOS2

Copy the arch files that came from Pierre's github into the NEMO tree::

	cp arch_NEMOGCM/arch* NEMOGCM/ARCH; cp arch_XIOS/arch* XIOS2/arch



The docker image can now be built::

	cd Docker
	docker build -t nemo/compiler .

You may need to add the user to the docker group::

	$ sudo usermod -aG docker $USER

You will need to log out and then back in.

The image can now be run, the path to SRC needs to be absolute, so switch
``/Users/jeff`` to the output of "echo $HOME"::

	docker run -v /Users/jeff/Belize_workshop:/Belize_workshop -t -i nemo/compiler /bin/bash

This should result in the container running and the terminal will move to the container e.g. (root@"container id"#) XIOS can now be compiled::

	cd /Belize_workshop/BUILD_TOOLS/XIOS2
	./make_xios --dev --netcdf_lib netcdf4_seq --arch DEBIAN

This should successfully build and result in an executable in the XIOS
 directories, ``xios_server.exe``, which will need to go into the EXPeriment
 directory at run time. *(It took 30 mins on my Mac)*.

Some environmental variables need to be defined (this can't be
  done until inside the container)::

	export CONFIG=BLZ
	export WDIR=/Belize_workshop
	export CDIR=$WDIR/BUILD_TOOLS/NEMOGCM/CONFIG
	export EXP=$WDIR/RUN_NEMO/EXP_demo

NEMO can now be built. Create an empty configuration::

	cd $CDIR
	./makenemo -n $CONFIG -m DEBIAN clean

Answer yes to the first question (OPA_SRC), no to all the others.

Copy MY_SRC fortran modifications into directory::

	mv $WDIR/BUILD_TOOLS/MY_SRC $CDIR/$CONFIG/.

Copy compiler flag options into directory::

	cp $WDIR/BUILD_TOOLS/cpp_BLZ.fcm $CDIR/$CONFIG/.

**ACTUALLY I COMMENT OUT THE key_diaharm_fast AND key_FES14_tides FLAGS...**

Then build NEMO again::

	./makenemo -n $CONFIG -m DEBIAN

This should build a nemo.exe file. At this point you can abandon BUILD_TOOLS except for two executables that need to go into an EXPeriment directory::

	ln -s $WDIR/BUILD_TOOLS/xios-2.0_r1242/bin/xios_server.exe $EXP/xios_server.exe
	ln -s $CDIR/$CONFIG/BLD/bin/nemo.exe $EXP/opa