<!--
Keep this document short & concise,
linking to external resources instead of including content in-line.
See 'release/text/readme.html' for the end user read-me.
-->

Frontier Installation Instructions
==================================

Below are instructions of how to install things on Frontier in the ``$HOME`` directory (as of 6/14/2023).
My username is used in these instructions, so you would have to change it to yours.

These are instructions have been modified from: ``https://wiki.blender.org/wiki/Building_Blender/Linux/OpenSUSE``

First load the relevant modules:
```bash
module swap PrgEnv-cray PrgEnv-gnu
module load gcc/11.2.0 cmake git subversion mesa amd-mixed
module unload darshan-runtime
module -t list

craype-x86-trento
libfabric/1.15.2.0
craype-network-ofi
perftools-base/22.12.0
xpmem/2.5.2-2.4_3.45__gd0f7936.shasta
cray-pmi/6.1.8
craype/2.7.19
cray-dsmml/0.2.2
cray-libsci/22.12.1.1
PrgEnv-gnu/8.3.3
gcc/11.2.0
hsi/default
DefApps/default
cray-mpich/8.1.23
cmake/3.23.2
git/2.36.1
subversion/1.14.1
mesa/21.3.1
amd-mixed/5.3.0
```

Fork the blender repository online (``https://github.com/blender/blender``), clone your fork on Frontier in your ``$HOME`` directory:

```bash
mkdir ~/blender-git
cd ~/blender-git
git clone https://github.com/michael-sandoval/blender.git
cd ~/blender-git/blender
git remote add upstream https://projects.blender.org/blender/blender.git
git fetch upstream
git branch --set-upstream-to=upstream/main
git pull
```

Setup the relevant ``lib`` directory:

```bash
mkdir ~/blender-git/lib
cd ~/blender-git/lib
svn checkout https://svn.blender.org/svnroot/bf-blender/trunk/lib/linux_x86_64_glibc_228
cd ~/blender-git/lib/linux_x86_64_glibc_228/epoxy/include/epoxy/
mkdir EGL && cd EGL
wget https://raw.githubusercontent.com/KhronosGroup/EGL-Registry/main/api/EGL/eglplatform.h
```

Checkout the 3.5 release of Blender and update things:

```bash
cd ~/blender-git/blender
git checkout -b 3.5_frontier remotes/upstream/blender-v3.5-release
make update # currently works for SVN revision 63404
```

Add HIP support to the headless template by adding the following line to ``~/blender-git/blender/build_files/cmake/config/blender_headless.cmake``:

```
set(WITH_CYCLES_HIP_BINARIES ON  CACHE BOOL "" FORCE)
```

Finally, make Blender in headless mode:

```bash
make headless
```

To run the Blender Benchmark script on Frontier:

```bash
cd ~/blender-git/
wget https://mirrors.ocf.berkeley.edu/blender/release/BlenderBenchmark2.0/script/blender-benchmark-script-3.1.0.tar.gz
tar -xvf blender-benchmark-script-3.1.0.tar.gz
cd ~/blender-git/blender-benchmark-script-3.1.0/
```

Update ``render.py`` in the benchmark script directory to enable using all devices on a single node (by default only would use 1 GPU).
This is done by changing line 52 from ``False`` to ``True``:

```python
for device in devices:                                                      
    device.cycles_device.use = True #False
```

If you try running the benchmarking script now, you'll run into a ``hipcc`` memory error (similar to ``https://github.com/E3SM-Project/E3SM/issues/4984``).
To fix things you'll need to pre-compile things before running -- these flags fix the error: ``-mllvm -amdgpu-early-inline-all=true -mllvm -amdgpu-function-calls=false``.
This brings the overall compile command to this:

```bash
hipcc -Wno-parentheses-equality -Wno-unused-value --hipcc-func-supp -O3 -ffast-math --amdgpu-target=gfx90a -mllvm -amdgpu-early-inline-all=true -mllvm -amdgpu-function-calls=false -I ~/blender-git/build_linux_headless/bin/3.5/scripts/addons/cycles/source --genco ~/blender-git/build_linux_headless/bin/3.5/scripts/addons/cycles/source/kernel/device/hip/kernel.cpp -o "/ccs/home/msandov1/.cache/cycles/kernels/cycles_kernel_gfx90a_8C586800A966B420091DFE69EBF4AB65"
```

Next, download a scene to try and benchmark (e.g., ``bmw27``):

```bash
wget https://mirrors.ocf.berkeley.edu/blender/release/BlenderBenchmark2.0/scenes/bmw27.tar.bz2
tar -xvf bmw27.tar.bz2
```

Finally, you should now be ready to run the benchmarking script like so (Do NOT run this on the login node -- ALWAYS use a compute node):

```bash
salloc -A STF007 -N1 -t 00:10:00
~/blender-git/build_linux_headless/bin/blender --background --factory-startup -noaudio --debug-cycles --enable-autoexec --engine CYCLES bmw27/main.blend --python main.py -- --device-type HIP
```

> Note: Make sure your modules are still loaded from when you built Blender (see above)

The rest of this README is the default Blender provided README.

Blender
=======

Blender is the free and open source 3D creation suite.
It supports the entirety of the 3D pipeline-modeling, rigging, animation, simulation, rendering, compositing,
motion tracking and video editing.

![Blender screenshot](https://code.blender.org/wp-content/uploads/2018/12/springrg.jpg "Blender screenshot")

Project Pages
-------------

- [Main Website](http://www.blender.org)
- [Reference Manual](https://docs.blender.org/manual/en/latest/index.html)
- [User Community](https://www.blender.org/community/)

Development
-----------

- [Build Instructions](https://wiki.blender.org/wiki/Building_Blender)
- [Code Review & Bug Tracker](https://projects.blender.org)
- [Developer Forum](https://devtalk.blender.org)
- [Developer Documentation](https://wiki.blender.org)


License
-------

Blender as a whole is licensed under the GNU General Public License, Version 3.
Individual files may have a different, but compatible license.

See [blender.org/about/license](https://www.blender.org/about/license) for details.
