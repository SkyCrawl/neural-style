#neural-style Installation

This guide will walk you through the installation of `neural-style` on OS X using [MacPorts package manager](https://www.macports.org/). It's a simplified guide designed to avoid having to use some affiliated installation scripts that may mess your environment up a little. In some aspects, this guide will also be a little more in-depth. In others, it will be less in-depth.

## Step 1: Install XCode command line tools and MacPorts

1. Installation is described [here](https://www.macports.org/install.php).
2. Download the ports tree with `sudo port selfupdate`.
3. Open terminal. If you had a previous terminal session open and `which port` doesn't return anything, you should open a new terminal tab or window (that will refresh your PATH environmental variable without doubling its content - that's what the `source ~/.profile` command does by default, unfortunately).

## Step 2: Install dependencies for Torch7

This is MacPorts-specific extract of <https://raw.githubusercontent.com/torch/ezinstall/master/install-deps>:

Commands to enter into terminal:
```bash
# Note: git should already be present in your system but it doesn't hurt to have the latest version...
# Note: I haven't quite figured out why 'xorg-server' should be needed but whatever...
sudo port install OpenBLAS cmake git readline wget imagemagick zmq graphicsmagick xorg-server protobuf-cpp
sudo port install gnuplot +pdflib -luaterm

# Note: can easily adjust to other versions of python (e.g. 'python35' and 'py35-*' packages).
sudo port install python_select python27 py27-ipython

# Troubleshoot for the previous command:
# 1) If 'Error: org.macports.install for port <XXX> returned: no destroot found at: ...', execute 'sudo port clean <XXX>' and try again.
# 2) If 'Error: org.macports.activate for port <XXX> returned: Image error: ...', execute 'sudo port -f activate <XXX>' and try again.

# and finally:
sudo port select --set python python27
sudo port select --set ipython py27-ipython
```

## Step 3: Install Torch7

First clone:
```bash
git clone https://github.com/torch/distro.git --recursive
```

Then apply a hotfix for `./torch/install.sh` script, e.g. replace `cmake .. -DCMAKE_INSTALL_PREFIX="${PREFIX}"` with `cmake .. -DCMAKE_INSTALL_PREFIX="$PREFIX"` (remove curly braces). On my Mac, Torch7 wouldn't compile otherwise, possibly because of using bash.

Compile and install Torch7:
```bash
# Note: we haven't installed Qt 4.x so 'qtlua' and 'qttorch' won't be installed but no biggie... everything works for me. Feel free to adjust PREFIX for your own needs.
export PREFIX=/usr/local
# Note: using 'sudo' is necessary only if PREFIX points into a protected folder.
[sudo] -E bash ./install.sh -s
```

We used the `s` option so that we could manually fix our environment (the script rather clutters it). Again, adjust this with your own PREFIX, if you changed it. Add the following to your `~/.profile` (alter your bash profile):
```bash
$(/usr/local/bin/luarocks path)
export LUA_CPATH='/usr/local/lib/?.dylib;'\$LUA_CPATH
```

## Step 4: Install neural-style

```bash
git clone https://github.com/jcjohnson/neural-style --recursive
sh ./neural-style/models/download_models.sh
# Note: using 'sudo' is necessary only if PREFIX from step 3 points into a protected folder.
[sudo] luarocks install loadcaffe
```

Using `sudo` is necessary only if you install into 

## (Optional) Step 5: Add GPU support to neural-style

If you have a [CUDA-capable GPU from NVIDIA](https://developer.nvidia.com/cuda-gpus) then you can
speed up `neural-style` using the CUDA technology.

Download and install the [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit), version 8.0 works for me. I hated the default installation location of `/Developer/NVidia/...` (ends with `CUDA-x.y`), so I moved it to `<new-installation-path>` as follows:
```bash
# Note: make sure not to remove other CUDA installations with the second command...
sudo mv "<default-installation-path>" "<new-installation-path>"
sudo rm -rf /Developer
```

Regardless of whether you move the installation or not, you need to adjust your environment again:
```bash
export PATH="$PATH:<installation-path>/bin"
export DYLD_LIBRARY_PATH="$DYLD_LIBRARY_PATH:<installation-path>/lib"
```

Applications should now automatically detect your CUDA installation from the CUDA compiler - if `which nvcc` doesn't return `<installation-path>/bin` even after opening a new terminal tab or window, you probably did something wrong.

The only remaining thing to do:
```bash
# Note: using 'sudo' is necessary only if PREFIX from step 3 points into a protected folder.
[sudo] luarocks install cutorch
[sudo] luarocks install cunn
```

## (Optional) Step 6: Add cuDNN backend support to neural-style

Neural-style uses the `cunn` backend by default but `cuDNN` is a library from NVIDIA that efficiently implements many of the operations (like convolutions and pooling) commonly used in deep learning. Generally, it uses less memory and performs better.

After registering as a developer with NVIDIA, you can [download cuDNN here](https://developer.nvidia.com/cudnn). For me, version 5.0 worked. Then:
```bash
tar -xzvf cudnn-7.0-osx-x64-v4.0-prod.tgz
# Note: using 'sudo' is necessary only if we moved CUDA installation into a protected folder.
[sudo] mv ./cuda/include/ "<installation-path>/include/"
[sudo] mv ./cuda/lib/ "<installation-path>/lib/"
rm -rf ./cuda
```

Next we need to install Torch bindings for cuDNN:
```bash
# Note: using 'sudo' is necessary only if PREFIX from step 3 points into a protected folder.
[sudo] luarocks install cudnn
```

## Final notes

Overall, it's a little longer process but it has its benefits:

* No messing up your environment. Personally, I hate my shell profile being split across several files (e.g. `~/.profile` and `~/.bashrc`).
* Upgrading dependencies installed with MacPorts is extremely easy.
* Targetting specific versions for dependencies installed with MacPorts is extremely easy.
* Dependencies installed with MacPorts will no longer be installed from their respective `master` branch on GitHub and that reduces the risk of errors coming from further development.

Downsides:
* The process comes with a little maintenance. Installing other MacPorts packages may activate other installations of the same dependencies (e.g. `pdflib`) which might result in some issues. Should that ever happen, however, you only need to repeat step 2.
