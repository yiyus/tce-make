# tce-make.sh
The tce-make.sh script makes tinycore extensions from tcm files.

## Usage:

Read one or more tcm files and generate .tar.gz.bfe files containing
all the files needed to submit a tinycore extension.

The tcm file must contain fields as a .tcz.info file, with the following
particularities:

- Size is ignored, it is calculated from the tcz file.
- A comment "Compiled from Core X.x", where X is the version of the
  current system, is added to Comments
- The current date is added before the Current comment

Additionally, some extra fields are supported in tcm files:

- Dependencies: dependencies to load the extension
- Git: download and decompress master.zip
- Build-deps: dependencies to build the extensions
- Make: commands to run before building extensions

Lines starting with # are ignored.

The commands in Make must fill the directory EXTENSION to build
EXTENSION.tcz and the corresponding EXTENSION.tar.gz.bfe file.

One extension is generated for each title in the tcm file (useful
for example to create -doc packages). Once a new title is defined,
the contents of all the info fields is inherited from the previous
one, but they can be redefined.

### See also
Sample .tcm files are included in this repository.
