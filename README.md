# tce-make
The tce-make script makes tinycore extensions from tcm files.

### Also included
tcm files of extensions maintained by yiyus

## Usage
The tce-make script reads one or more tcm files and generates .tar.gz.bfe
files containing all the files needed to submit a tinycore extension.

The tcm file must contain files as a .tcz.info file, with the following
particularities:

- Size is ignored, it is calculated from the tcz file.
- A comment"Compiled from Core X.x", where X is the version of the
  current system, is added to Comments
- The current date is added before the Current comment

Additionally, some additional fields are supported in tcm files:

- Dependencies: dependencies to load the extension
- Git: download and decompress master.zip
- Build-deps: dependencies to build the extension
- Make: commands to run before building extension

The commands in Make must fill the directory EXTENSION to build
EXTENSION.tcz and the corresponding EXTENSION.tar.gz.bfe file.

More than one extension can be generated from the same tcm file (useful
for example to create -doc packages).

For more information, see the included .tcm files.
