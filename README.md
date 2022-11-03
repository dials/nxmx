# Read NXmx-flavour NeXus HDF5 data in Python

This package provides a neat and tidy Python interface for reading data from [HDF5 files](https://www.hdfgroup.org/solutions/hdf5/) that are structured according to the [NXmx application definition](https://manual.nexusformat.org/classes/applications/NXmx.html) of the [NeXus standard](https://www.nexusformat.org/).

## Installation
`python-nxmx` is available as `nxmx` on PyPI, so you just need Pip.
```Bash
pip install nxmx
```

## Getting started

If you have an HDF5 file in NXmx format, inspecting it with `h5ls` will look something like this:
```Bash
$ h5ls -r my-nxmx-file.h5 
/                        Group
/entry                   Group
/entry/data              Group
/entry/definition        Dataset {SCALAR}
/entry/end_time          Dataset {SCALAR}
/entry/end_time_estimated Dataset {SCALAR}
/entry/instrument        Group
/entry/instrument/beam   Group
/entry/instrument/beam/incident_beam_size Dataset {2}
/entry/instrument/beam/incident_wavelength Dataset {SCALAR}
/entry/instrument/beam/total_flux Dataset {SCALAR}
/entry/instrument/detector Group

... etc. ...
```
With `nxmx`, you can access the NXmx data in Python like this:
```Python
import h5py
import nxmx

with h5py.File("my-nxmx-file.h5") as f:
    nxmx_data = nxmx.NXmx(f)
```

## A slightly more detailed example
```Python
import h5py
import nxmx

with h5py.File("my-nxmx-file.h5") as f:
    nxmx_data = nxmx.NXmx(f)
    # Explore the NXmx data structure.
    entry, *_ = nxmx_data.entries
    print(entry.definition)  # Prints "NXmx".
    instrument, *_ = entry.instruments
    detector, *_ = instrument.detectors
    
    # Get the h5py object underlying an instance of a NX class.
    entry_group = entry._handle  # entry_group == f["entry"]

    # Find instances of a given NX class in a h5py.Group.
    beams = nxmx.find_class(instrument._handle, "NXbeam")
    # The equivalent for more than one NX class.
    beams, detectors = nxmx.find_classes(instrument._handle, "NXbeam", "NXdetector")

    # Query attributes of an object in the normal h5py way.
    # Suppose out detector has a transformation called "det_z".
    transformations, *_ = nxmx.find_class(detector._handle, "NXtransformations")
    attrs = transformations["det_z"].attrs  # Get the attributes of the "det_z" dataset.
```
