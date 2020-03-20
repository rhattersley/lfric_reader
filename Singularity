Bootstrap: docker
From: ubuntu:18.04

%labels
    Author wolfgang.hayek@niwa.co.nz

%help
    This container provides a "headless" version of ParaView (only ParaView Server
    and ParaView Python, does not include a GUI and does not require an X context
    at runtime) with a reader plugin for output from the LFRic code. It also provides
    Iris (scitools.org.uk/iris) and essential Python packages.

%environment

    # Required for auto-loading the netCDFLFRicReader plugin
    export LD_LIBRARY_PATH=/usr/local/lib:/usr/local/lib/netCDFLFRicReader
    export PV_PLUGIN_PATH=/usr/local/lib/netCDFLFRicReader

%post -c /bin/bash

    apt-get update && apt-get upgrade -y

    # Set timezone to UTC non-interactively
    ln -fs /usr/share/zoneinfo/UTC /etc/localtime
    apt-get install -y tzdata
    dpkg-reconfigure --frontend noninteractive tzdata

    apt-get install -y xz-utils pkg-config python3-dev python3-numpy python3-matplotlib \
                       git wget make cmake libmpich-dev libosmesa6-dev libtbb-dev \
                       libnetcdf-dev libhdf5-dev \
                       unzip libgeos-dev libproj-dev libudunits2-dev python3-pip python3-setuptools \
                       cython3 python3-cartopy python3-netcdf4 python3-scipy python3-toolz \
                       python3-pandas

    # Pyke is required by Iris but not available through package managers
    wget https://sourceforge.net/projects/pyke/files/pyke/1.1.1/pyke3-1.1.1.zip
    echo "a7d12d66d4c2ec12576a8187d3001384 pyke3-1.1.1.zip" | md5sum --check
    unzip pyke3-1.1.1.zip
    cd pyke-1.1.1
    python3 setup.py install
    cd ..
    rm -r pyke-1.1.1 pyke3-1.1.1.zip

    # These are not available/recent enough on RPMs
    # Install without dependencies to avoid netCDF/HDF5 library clash with VTK/ParaView
    pip3 install --no-deps cf-units cftime nc-time-axis dask pyugrid stratify scitools-iris

    # Select ParaView major.minor version and patch to form "major.minor.patch"
    pv_version=5.8
    pv_patch=0
    pv_sha256=219e4107abf40317ce054408e9c3b22fb935d464238c1c00c0161f1c8697a3f9

    # Download and verity ParaView sources
    pv_url="https://www.paraview.org/paraview-downloads/download.php"
    pv_url_params="submit=Download&version=v${pv_version}&type=source&os=Sources&downloadFile="
    pv_filename=ParaView-v${pv_version}.${pv_patch}.tar.xz
    wget "${pv_url}?${pv_url_params}${pv_filename}" -O ${pv_filename}
    echo "${pv_sha256} ${pv_filename}" | sha256sum --check

    # Build ParaView without GUI and X ("headless") - this will provide ParaView Server
    # and ParaView Python. Use same netCDF/HDF5 libraries as Python-netCDF4 to avoid
    # runtime issues.
    tar -xf ${pv_filename}
    mkdir -p ParaView-v${pv_version}.${pv_patch}/build
    cd ParaView-v${pv_version}.${pv_patch}/build
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DPARAVIEW_BUILD_EDITION=CANONICAL \
    -DPARAVIEW_BUILD_SHARED_LIBS=ON \
    -DPARAVIEW_INSTALL_DEVELOPMENT_FILES=ON \
    -DPARAVIEW_USE_QT=OFF \
    -DPARAVIEW_USE_PYTHON=ON \
    -DPARAVIEW_USE_MPI=ON \
    -DPARAVIEW_USE_VTKM=OFF \
    -DPARAVIEW_ENABLE_RAYTRACING=OFF \
    -DPARAVIEW_ENABLE_VISITBRIDGE=OFF \
    -DPARAVIEW_PLUGIN_ENABLE_NetCDFTimeAnnotationPlugin=ON \
    -DPARAVIEW_PLUGIN_ENABLE_StreamLinesRepresentation=ON \
    -DPARAVIEW_PLUGIN_ENABLE_StreamingParticles=ON \
    -DPARAVIEW_PLUGIN_ENABLE_SurfaceLIC=ON \
    -DVTK_USE_X=OFF \
    -DVTK_OPENGL_HAS_OSMESA=ON \
    -DVTK_SMP_IMPLEMENTATION_TYPE=tbb \
    -DVTK_MODULE_ENABLE_VTK_GeovisCore=YES \
    -DVTK_MODULE_ENABLE_VTK_IONetCDF=YES \
    -DVTK_MODULE_USE_EXTERNAL_VTK_netcdf=ON \
    -DVTK_MODULE_USE_EXTERNAL_VTK_hdf5=ON
    make -j 4
    make install
    cd ../..
    rm -r ParaView-v${pv_version}.${pv_patch} ${pv_filename}

    # Build netCDFLFRicReader plugin
    git clone https://github.com/tinyendian/lfric_reader.git
    mkdir -p lfric_reader/src/cxx/build
    cd lfric_reader/src/cxx/build
    cmake .. -DBUILD_TESTING=OFF -DCMAKE_INSTALL_PREFIX=/usr/local
    make -j 4
    make install
    cd ../../../..
    rm -rf lfric_reader
