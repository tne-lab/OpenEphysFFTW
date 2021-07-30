# OpenEphysFFTW

This is a small library to wrap FFTW functionality useful for Open Ephys.

It is currently used by the [Phase Calculator](https://github.com/tne-lab/phase-calculator) and [Real-Time Coherence/Spectrogram plugin](https://github.com/tne-lab/Coherence-Spectrogram-Viewer).

## Dependency

* On Linux or Mac, you need to install FFTW 3 (if not already present) into the standard library/header locations. See [here](http://www.fftw.org/download.html) for instructions on building from source. Many distributions may also provide FFTW 3 available in their package repositories. For instance, Ubuntu/Debian/Linux Mint provide the package `libfftw3-dev` (available on Ubuntu 14.04 "trusty" and later). The package on Arch Linux appears to be called simply `fftw`.

* On Windows, the precompiled FFTW library is included in the `libs` folder - no need to install it manually.

## Installation

This library should be built outside of the main GUI file tree using CMake. In order to do so, it must be in a sibling directory to plugin-GUI\* and the main GUI must have already been compiled.

See `OpenEphysFFTW/CMAKE_README.txt` and/or the wiki page [here](https://open-ephys.atlassian.net/wiki/spaces/OEW/pages/1259110401/Plugin+CMake+Builds) for build instructions.

\* If you have the GUI built somewhere else, you can specify its location by setting the environment variable `GUI_BASE_DIR` or defining it when calling cmake with the option `-DGUI_BASE_DIR=<location>`.
