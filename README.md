# bhns-tips
Helpful or perhaps obvious tips with bhns simulations
==== Running a BBH simulation====
1) Load your SpEC modules:
 source ~/SpEC/MakefileRules/this_machine.env
2) mkdir NAMEOFSIMULATION then cd into it
3) Type: PrepareID -t bbh2 -reduce-ecc
4) Modify 'Params.input' as needed
5) type: ZeroEccParamsFromPN -h (#This is to determine the 'Omega0' and 'adot0')
5) Type: ./StartJob.sh 
6) Type: ShowQueue (#to see simulation run time etc.)
7) If your simulation dies, you can 'less SpEC.stderr'to see the standard error</code>

====What do these options in params mean??====
From my understanding:
q=mass ratio
chiA= spin from black hole 1
chiB= spin from black hole 2
D0= initial seperation
Omega0= orbital frequency
adot0= fractional expansion (gives holes radial motion)</code>

==== obtaining horizons.h5 files for trajectories ====
1) cd into your EV directory and mkdir NAME
2) cd into your new directory and ls all .h5 files
   ex: ls ../Lev1_*/Run/ApparentHorizons/Horizons.h5
3) JoinH5 -o Horizons.h5 ../Lev1_*/Run/ApparentHorizons/Horizons.h5
4) Open Jupyter

==== Jupyter script to plot trajectories====
import h5py
%matplotlib inline
import numpy as np
from matplotlib import pyplot as plt
import math

file=h5py.File("q2_3d_95011.h5") //replace "q2_3d_95011.h5" with the name of your .h5 file

file.keys() //You should get the output of "[u'AhA.dir', u'AhB.dir', u'AhC.dir']"

file["AhA.dir"].keys()
file["AhB.dir"].keys()

coordA=file["AhA.dir"]["CoordCenterInertial.dat"]
coordB=file["AhB.dir"]["CoordCenterInertial.dat"]

plt.plot(coordA[:,1],coordA[:,2])
plt.plot(coordB[:,1],coordB[:,2])
plt.show() //A plot of their orbits should appear!

==== extracting GWs ====

1) In your Ev directory, mkdir NAME (mine was wave info)

2) cd into wave info

3) we want to execute the following command to make sure the command doesn't have any errors... this will make more sense in a bit 

a) ls ../{Lev3_*,Lev3_Ringdown/Lev3_*}/Run/GW2/rh_FiniteRadii_CodeUnits.h5
 JoinH5 -o rh.h5 ../{Lev3_*,Lev3_Ringdown/Lev3_*}/Run/GW2/rh_FiniteRadii_CodeUnits.h5 

5) Now if we open  our jupyter notebook we can take a look at the wave!
//load in your libraries
import h5py
%matplotlib inline
import numpy as np
from matplotlib import pyplot as plt
import math

//this will assign your .h5 file to the name "file"
file=h5py.File("NAMEOFYOURH5FILE.h5")

//taking a look of what's in your file. you should get a list of .dir
file.keys()

//taking a look of what's inside a selected .dir
file["R0540.dir"].keys()

//reassigning the name of your .h5 file. I decided to call it "gw"
gw = h5py.File("NAMEOFYOURH5.h5")

lm22=gw["R0540.dir"]["Y_l2_m2.dat"]
plt.plot(lm22[:,0],lm22[:,1])
plt.show()

==== ParaView BBH Simulations====
//HorizonsDump= data for black hole exterior 
//Horizons.h5= data for trajectory and spin

     mkdir vis       #visulation, movie, etc
     cd vis

Next you want to join all the horizon dump data from the segments. To do this run the following(Remember to change your file path):

     JoinH5 -o NAME.h5 ../Ev/Lev0_*/Run/ApparentHorizons/HorizonsDump.h5 
     #You can either change the "0" in "Lev0_*" based on how many levels you have, or replace it with "Lev*"
          Note: If need be, repeat the above steps for ringdown.

Convert the .h5 to pvd by running the following:

     ConvertH5SurfaceToVtk NAME.h5 
     

To get the trajectories and spin data

Join the Horizons.h5 files for all the segments using the following command (remember to change the file path)

     JoinH5 -o NAME.h5 ../Lev0_*/Run/ApparentHorizons/Horizons.h5

Then,

     ExtractFromH5 Horizons.h5   
     cd extracted-Horizons

     cd extracted-Horizons
     ln -s AhA.dir/chiInertial.dat ./SpinA.dat
     ln -s AhB.dir/chiInertial.dat ./SpinB.dat
     ln -s AhC.dir/chiInertial.dat ./SpinC.dat

     ln -s AhA.dir/CoordCenterInertial.dat ./TrajA.dat
     ln -s AhB.dir/CoordCenterInertial.dat ./TrajB.dat
     ln -s AhC.dir/CoordCenterInertial.dat ./TrajC.dat
     ~/SpEC/Support/Visualization/ConvertTrajectoryToVtk -v -n TrajA,SpinA TrajA.dat SpinA.dat
     ~/SpEC/Support/Visualization/ConvertTrajectoryToVtk -v -n TrajB,SpinB TrajB.dat SpinB.dat
     ~/SpEC/Support/Visualization/ConvertTrajectoryToVtk -v -n TrajC,SpinC TrajC.dat SpinC.dat
    
