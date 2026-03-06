hw.s-ol.nu KiCad library
========================

## 3D Model Paths
To get 3D Models working, configure the `SOL_LIB_DIR` Environment Variable
in the "Configure Paths" dialog to point at this repository.

If you are distributing your project as a Git repository,
it is recommended to add this repo as a submodule (e.g. in `lib/s-ol`) and configure a relative path as follows:

    $ mkdir lib
    $ git submodule add https://git.s-ol.nu/hw/kicad-lib.git lib/s-ol

    SOL_LIB_DIR: ${KIPRJMOD}/lib/s-ol
