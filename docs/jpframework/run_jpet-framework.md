# Prerequisites
## OS & Environment
Sandbox defined and tested on:
```
PRETTY_NAME="Ubuntu 22.04.2 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.2 LTS (Jammy Jellyfish)"

```
Environment is defined using `conda`. The defined env accumulates requirements from different sub-projects (repositories). Mainly we depends on ROOT framework.  
> Install `miniconda` on your system, [installation docs](https://docs.anaconda.com/miniconda/miniconda-install/)  
It's really worth to think about switching deafult solver to `mamba`, see [conda-libmamba-solver](https://www.anaconda.com/blog/a-faster-conda-for-a-growing-community)

### Define `jpc` environment using `jpc_env.yml` file:  
```
conda env create -n jpc -f jpc_env.yml
```
and activate;
```
conda activate jpc
```

## J-PET-Framework and friends

The executable part of the framework is contained in the so-called XYZAnalysis.x, e.g., LargeBarrelAnalysis.x, whose code is located in [j-pet-framework-examples](https://github.com/JPETTomography/j-pet-framework-examples). The [j-pet-framework](https://github.com/JPETTomography/j-pet-framework) forms the core. In addition, we need the HLD data unpacker which is defined in [j-pet-unpacker](https://github.com/JPETTomography/j-pet-unpacker).

> Detailed hands-on slides: [J-PET Software Workshop 2023](http://sphinx.if.uj.edu.pl/workshop2023/JPET-Framework-Workshop-2023.pdf)

### Get source-code (ssh)
```
git clone git@github.com:JPETTomography/Unpacker2.git
git clone git@github.com:JPETTomography/j-pet-framework.git
git clone --recursive git@github.com:JPETTomography/j-pet-framework-examples
```
> For the `jpc-sandbox` we build everything from `master` branch  

> Note: The calibration files has been downloaded to `j-pet-framework-examples/CalibrationFiles/`. The `RUN` here refers to a specific geometry, detector version, and data collection campaign.

### Build and Install The `Uncpacker2`
This is the [HLD data](https://repository.gsi.de/record/52192/files/PHN-SIS18-ACC-41.pdf) tool
```
mkdir j-pet-uncpacker2; cd Unpacker2; mkdir build
cmake -DCMAKE_INSTALL_PREFIX=../../j-pet-uncpacker2/ ../
make -j 4; make install
```

### Build and Install The `j-pet-framework`
```
mkdir framework; mkdir j-pet-framework/build; cd j-pet-framework/build;
cmake -DCMAKE_INSTALL_PREFIX=../../framework ../
make -j 4; make install
```

### Build and Install The `j-pet-framework-examples`
```
mkdir examples; cd examples
cmake -DCMAKE_PREFIX_PATH=../framework ../j-pet-framework-examples/
make -j 4
```

## How to run `Monitoring` data processing

During the experiment data taking campaign the HDL data refers to raw data, which serves as input to the J-PET Framework. Monitoring service selects every n-th HDL file, performs reconstruction and finally generates control histograms.

### Download and unpack sandbox data
```
wget http://sphinx.if.uj.edu.pl/workshop2023/dabc_17334031817.hld.xz
xz -d dabc_17334031817.hld.xz
```

#### Fill the calibration info
To run, we fill in the `userParams.json` file with references to the aforementioned calibration files. As mentioned before, all calibration data are being downloaded to `j-pet-framework-examples/CalibrationFiles/` as defined in `j-pet-framework-examples/CMakeLists.txt`.

Once we are going to run example of `LargeBarrelAnalysis` we copy corresponding calibration data to current directory:
```
cd examples/LargeBarrelAnalysis/
cp ../../j-pet-framework-examples/CalibrationFiles/5_RUN/* .
```

Now can see in the example filled [userParams.json](userParams.json) the corresponding `*.root` files with calibration data.
> ! Copy this example [userParams.json](userParams.json) to the `examples/` directory.

#### Run the analysis
```
./LargeBarrelAnalysis.x -t hld -f dabc_17334031817.hld -i 5 -l detectorSetupRun5.json -u userParams.json -r 0 100000 -b
```

#### Analysis outcome
You can see that processing output is being dumped to the `*.root` files with the names corresponding to the  `*.hld` processed file, e.g.:
```
dabc_17334031817.cat.evt.root
dabc_17334031817.hits.root
dabc_17334031817.hld.root
dabc_17334031817.phys.sig.root
dabc_17334031817.presel.evt.root
dabc_17334031817.raw.sig.root
dabc_17334031817.tslot.calib.root
dabc_17334031817.unk.evt.root
```

#### Preview the monitoring histograms
> You can utilize the VS Code plugin [ROOT File Viewer](https://marketplace.visualstudio.com/items?itemName=albertopdrf.root-file-viewer) to preview `*.root` files directly.

Note that from the perspective of the experiment monitoring the data processing (control histograms list) is defined for a specific data collection campaign (a.k.a. certain experiment), e.g., [hospital2024/data_monitoring/web/description.json](https://github.com/kkacprzak/j-pet-online-monitoring/blob/hospital2024/data_monitoring/web/description.json). For the previous, local monitoring web app the paths to the monitoring data processing output is hardcoded, see [monitoring.py](https://github.com/kkacprzak/j-pet-online-monitoring/blob/hospital2024/data_monitoring/monitoring.py).

