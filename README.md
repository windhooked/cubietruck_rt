Build on current
copy patch-5.4.40-rt24.patch to armbian/build/userpatches/kernel/sunxi-current
Start build process, exclude AUFS from the build, under misc filesystems

Build on legacy
https://autostatic.com/2013/09/17/exit-beaglebone-black-hello-cubieboard2/

https://groups.google.com/forum/#!topic/cubieboard/kQp6Au_hUDE

1. Install:

# install GLX Gears, mesa GL and GLU libraries 
apt-get -y install mesa-utils

# install development tools
apt-get -y install build-essential automake pkg-config libtool ca-certificates git cmake subversion
                                                                                
# install required libraries                                                 
apt-get install libx11-dev libxext-dev xutils-dev libdrm-dev x11proto-xf86dri-dev libxfixes-dev
                                   
#  get source code                                                                                        
git clone https://github.com/robclark/libdri2                                   
git clone https://github.com/linux-sunxi/libump                                 
git clone https://github.com/linux-sunxi/sunxi-mali                             
git clone https://github.com/ssvb/xf86-video-fbturbo                            
git clone https://github.com/ptitSeb/glshim

# install mali driver
cd sunxi-mali                                                                   
git submodule init                                                              
git submodule update                                                            
git pull                                                                        
wget http://pastebin.com/raw.php?i=hHKVQfrh -O ./include/GLES2/gl2.h            
wget http://pastebin.com/raw.php?i=ShQXc6jy -O ./include/GLES2/gl2ext.h   
make config ABI=armhf VERSION=r3p0                                              
mkdir /usr/lib/mali                                                             
echo "/usr/lib/mali" > /etc/ld.so.conf.d/1-mali.conf                            
make -C include install                                                         
make -C lib/mali prefix=/usr libdir='$(prefix)/lib/mali/' install           
cd ..

2. Build

# Step 1: build and install helper libraries                                    
                                                                                
cd libdri2                                                                      
autoreconf -i                                                                   
./configure --prefix=/usr                                                       
make                                                                            
make install                                                                    
cd ..                                                                           
                                                                                
cd libump                                                                       
autoreconf -i                                                                   
./configure --prefix=/usr                                                       
make                                                                            
make install                                                                    
cd ..                 

# Step 2: build video driver                              
                                                                                
cd xf86-video-fbturbo                                                           
autoreconf -i                                                                   
./configure --prefix=/usr                                                       
make                                                                            
make install                                                                    
cd ..     

# Step 3: build GL wrapper                                               
                                                                                
cd glshim                                                                       
cmake .                                                                         
make                                                                            
cp lib/libGL.so.1 /usr/lib/ # replace the software GL library with the wrapper
cd ..    

3. Configure your system

- configure your kernel to allocate memory for the GPU

- make sure mali and mali_drm kernel modules are loaded

- give your user permissions to access /dev/ump and /dev/mali

- configure Xorg to use fbturbo driver

 

4. Test:

# run a basic test
glxgears   

# install and run a GL benchmark 
apt-get -y install globs  
/usr/lib/globs/benchmarks/GL_pointz/gl_pointz                            

# try to run a real game
apt-get -y install billard-gl
billard-gl  

This all worked out for me rather nicely. The only issue I have encountered is a segfault that many GL programs get when they shut down. I'm currently debugging this issue, but it would be helpful to know others experience it as well, and perhaps get some advice from people experienced in GLX or SDL.

 

Edit: I know glxgears is not a real benchmark, but let me give you some numbers to make it clear what I'm talking about. Results are from Orange Pi PC clocked at 1296000 Hz (and are CPU-bound):

user@bananapi:~$ glxgears
LIBGL: Initialising glshim
libGL: built on Jun 12 2016 06:12:01
LIBGL: Current folder is:/home/user
libGL:loaded: libGLESv1_CM.so
libGL:loaded: libEGL.so
2074 frames in 5.0 seconds = 414.688 FPS
2071 frames in 5.0 seconds = 414.085 FPS
2070 frames in 5.0 seconds = 413.915 FPS
