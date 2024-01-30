# Compile static Parthenon tests

1. autoconf ≥ 2.71
    
    ```bash
    autoconf --version
    wget https://ftp.gnu.org/gnu/autoconf/autoconf-2.71.tar.xz
    tar -xf autoconf-2.71.tar.xz
    cd autoconf-2.71/
    ./configure
    time make 
    sudo make install 
    . ~/.profile
    autoconf --version
    
    ```
    
2. Static MPICH
    
    ```bash
    #MPICH
    cd /workspace
    wget https://www.mpich.org/static/downloads/4.1.2/mpich-4.1.2.tar.gz
    tar xzvf mpich-4.1.2.tar.gz
    cd mpich-4.1.2

    # static mpicc 
    ./configure --prefix=/workspace/mpich-4.1.2-install --enable-static
    make && sudo make install 
    export MPI_HOME=/workspace/mpich-install
    export PATH=$MPI_HOME/bin:$PATH
    export LD_LIBRARY_PATH=$MPI_HOME/lib:$LD_LIBRARY_PATH
    
    ```
    
3. Install the libs.
   
   *Please be aware of that this part doesn't include all the libs. If all the libs are correctly installed in your system, you can skip this part.*
   Here, we give some examples of how to install the static libs.
    
    ```bash
    git clone https://github.com/systemd/systemd-stable.git
    cd systemd-stable
    ./configure --auto-features=disabled --default-library=static -D standalone-binaries=true -D static-libsystemd=true -D static-libudev=true -D link-udev-shared=false -D link-systemctl-shared=false -D link-networkd-shared=false -D link-timesyncd-shared=false
    make
    find ./ -name "libudev*"
    cp ./build/libudev.a /usr/local/lib/
    
    # The following are for some other libraries.
    # libxml2
    wget https://gitlab.gnome.org/GNOME/libxml2/-/archive/v2.10.1/libxml2-v2.10.1.tar.gz
    # libhwloc
    wget https://download.open-mpi.org/release/hwloc/v2.6/hwloc-2.6.0.tar.gz
    
    # compile these libs (Not include all, some libs in the system 
    # is .so(dynamic) not .a (Static))
    # the commands to install these libs are very similar
    cd libdir
    ./configure --enable-static # or --static
    make && sudo make install
    
    ```
    **Note:** If you missed some libraries, the error will be "undefine references funct_name()". Just google the func_name and install the corresponding static lib.  
    
5. Parthenon
    
    ```bash

    # download Parthenon
    sudo apt update
    sudp apt install cmake
    git clone https://github.com/parthenon-hpc-lab/parthenon.git
    cd parthenon
    git submodule update --init
    
    mkdir build
    cd build
    
    # disable optional dependenies (HDF5)
    cmake -DCMAKE_INSTALL_PREFIX=/workspace/parthenon-install \
     -DPARTHENON_DISABLE_HDF5=ON \
     -DCMAKE_PREFIX_PATH=/workspace/mpich-4.1.2-install/ \
     ../
    
    # generate the .o files for each test.
    make -j90
    
    # here, we need to be hackish to compile the static apps
    # 
    cd example/advection   # This is one of the example tests
    
    # What we do here is to get all the libraries the test would use and manually 
    # link them. If some static lib is missing, we should install it by source code. (See section 3.)
    # compile this application: advection-example
    /workspace/mpich-4.1.2-install/bin/mpicxx \
    -static -O2 -g -DNDEBUG \
    -DKOKKOS_DEPENDENCE "CMakeFiles/advection-example.dir/advection_driver.cpp.o" "CMakeFiles/advection-example.dir/advection_package.cpp.o" "CMakeFiles/advection-example.dir/main.cpp.o" "CMakeFiles/advection-example.dir/parthenon_app_inputs.cpp.o" \
    -o advection-example  \
    ../../src/libparthenon.a \
    -Bstatic -lnuma \
    /workspace/mpich-4.1.2-install/lib/libmpicxx.a \
    /workspace/mpich-4.1.2-install/lib/libmpi.a  \
    /usr/local/lib/libhwloc.a  \
    /usr/local/lib/libxml2.a \
    /usr/local/lib/libz.a  \
    /usr/lib/x86_64-linux-gnu/libefa.a \
    /usr/lib/x86_64-linux-gnu/libnl-3.a \
    /usr/lib/x86_64-linux-gnu/libnl-route-3.a \
    /usr/lib/x86_64-linux-gnu/libibverbs.a \
    /usr/lib/x86_64-linux-gnu/libuuid.a \
    /usr/lib/x86_64-linux-gnu/libnuma.a \
    /usr/lib/gcc/x86_64-linux-gnu/9/libatomic.a \
    /usr/lib/x86_64-linux-gnu/libpthread.a \
    /usr/lib/x86_64-linux-gnu/libdl.a \
    /usr/lib/x86_64-linux-gnu/librt.a \
    ../../Kokkos/containers/src/libkokkoscontainers.a \
    ../../Kokkos/core/src/libkokkoscore.a \
    -ldl \
    ../../Kokkos/simd/src/libkokkossimd.a \
    -ludev -lcap \
    /usr/lib/x86_64-linux-gnu/libltdl.a
    
    # compile calculate_pi
    cd ../calculate_pi/
    /workspace/mpich-4.1.2-install/bin/mpicxx \
    -static -O2 -g -DNDEBUG \
    -DKOKKOS_DEPENDENCE "CMakeFiles/pi-example.dir/calculate_pi.cpp.o" "CMakeFiles/pi-example.dir/pi_driver.cpp.o" \
    -o pi-example \
    ../../src/libparthenon.a \
    -Bstatic -lnuma \
    /workspace/mpich-4.1.2-install/lib/libmpicxx.a \
    /workspace/mpich-4.1.2-install/lib/libmpi.a  \
    /usr/local/lib/libhwloc.a  \
    /usr/local/lib/libxml2.a /usr/local/lib/libz.a  \
    /usr/lib/x86_64-linux-gnu/libefa.a \
    /usr/lib/x86_64-linux-gnu/libnl-3.a \
    /usr/lib/x86_64-linux-gnu/libnl-route-3.a \
    /usr/lib/x86_64-linux-gnu/libibverbs.a \
    /usr/lib/x86_64-linux-gnu/libuuid.a \
    /usr/lib/x86_64-linux-gnu/libnuma.a \
    /usr/lib/gcc/x86_64-linux-gnu/9/libatomic.a \
    /usr/lib/x86_64-linux-gnu/libpthread.a \
    /usr/lib/x86_64-linux-gnu/libdl.a \
    /usr/lib/x86_64-linux-gnu/librt.a \
    ../../Kokkos/containers/src/libkokkoscontainers.a \
    ../../Kokkos/core/src/libkokkoscore.a -ldl \
    ../../Kokkos/simd/src/libkokkossimd.a -ludev \
    -lcap /usr/lib/x86_64-linux-gnu/libltdl.a
    
    ```
    
7. Compile other apps.  
  Step 1: go to `parthenon/build/example/appdir`  
  Step 2: check file `appdir/CMakeFiles/app_name.dir/link.txt`  
  Step 3: add `-static` to the command in link.txt and delete all the libs after `-o app_name`  
  Step 4: add the following libs after `-o app_name`
    
    ```bash
    ../../src/libparthenon.a \
    -Bstatic -lnuma \
    /workspace/mpich-4.1.2-install/lib/libmpicxx.a \
    /workspace/mpich-4.1.2-install/lib/libmpi.a  \
    /usr/local/lib/libhwloc.a  \
    /usr/local/lib/libxml2.a /usr/local/lib/libz.a  \
    /usr/lib/x86_64-linux-gnu/libefa.a \
    /usr/lib/x86_64-linux-gnu/libnl-3.a \
    /usr/lib/x86_64-linux-gnu/libnl-route-3.a \
    /usr/lib/x86_64-linux-gnu/libibverbs.a \
    /usr/lib/x86_64-linux-gnu/libuuid.a \
    /usr/lib/x86_64-linux-gnu/libnuma.a \
    /usr/lib/gcc/x86_64-linux-gnu/9/libatomic.a \
    /usr/lib/x86_64-linux-gnu/libpthread.a \
    /usr/lib/x86_64-linux-gnu/libdl.a \
    /usr/lib/x86_64-linux-gnu/librt.a \
    ../../Kokkos/containers/src/libkokkoscontainers.a \
    ../../Kokkos/core/src/libkokkoscore.a -ldl \
    ../../Kokkos/simd/src/libkokkossimd.a -ludev \
    -lcap /usr/lib/x86_64-linux-gnu/libltdl.a
    ```
