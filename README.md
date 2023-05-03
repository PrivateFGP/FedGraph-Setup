# FedGraph-Setup

We here introduce how to set up the dependencies and run FedGraph.

## Environment Requirement

The environment in which we have evaluated FedGraph is:
- Ubuntu 20.04
- g++/gcc 9.4.0
- GNU Make 4.2.1
- cmake 3.25.1
- Python 3.9.13

Refer to [this link](https://askubuntu.com/questions/355565/how-do-i-install-the-latest-version-of-cmake-from-the-command-line) for upgrading your cmake version from kitware.

## 1 Set Up Dependencies

### 1.0 Work Directory

```bash
mkdir Artifact && cd Artifact
git clone https://github.com/PrivateFGP/FedGraph-Setup.git
```

### 1.1 Install MPC dependencies.

Install EMP.

```bash
cd Artifact
git clone https://github.com/PrivateFGP/MPC.git
cd MPC
python install.py --deps --tool --ot --sh2pc
```

Install SCI-SilentOT.

```bash
sudo apt install libgmp-dev libssl-dev
cd Artifact/MPC/
git clone https://github.com/PrivateFGP/SCI-SilentOT.git
cd ./SCI-SilentOT/SCI
bash scripts/build-deps.sh
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=./install -DCMAKE_FIND_DEBUG_MODE=ON .. -DCMAKE_BUILD_TYPE=Debug -DSCI_BUILD_NETWORKS=OFF -DSCI_BUILD_TESTS=ON -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl -DCMAKE_PREFIX_PATH=$(pwd)/../deps/build -DUSE_APPROX_RESHARE=ON
cmake --build . --target install --parallel
```

### 1.2 Install HE dependencies

Install ophelib (Paillier).

```bash
cd Artifact
sudo apt install build-essential m4 libtool-bin libgmp-dev libntl-dev
mkdir HE && cd HE
git clone https://github.com/PrivateFGP/ophelib.git
cd ophelib
mkdir build && cd build
cmake ..
make -j
sudo make install
```

### 1.3 Set Up CMake Dependencies

```bash
cd Artifact/MPC
sudo cp -r cmake/* /usr/local/lib/cmake/
```

### 1.4 Set Up PSI

```bash
cd Artifact
git clone https://github.com/PrivateFGP/Artifact-MultipartyPSI.git
cd Artifact-MultipartyPSI
cd thirdparty/
bash all_linux.get
```

## 2 Set Up FedGraph

Download and compile the code.

```bash
cd Artifact/Artifact-MultipartyPSI

git clone https://github.com/PrivateFGP/FedGraph.git
git clone https://github.com/PrivateFGP/SSHE-Worker.git

mkdir build && cd build
cmake .. -DTHREADING=ON
make -j8
```

Prepare Test Data.

```bash
cd Artifact/Artifact-MultipartyPSI
mkdir data
cp ./../FedGraph-Setup/data/* ./data/
cd FedGraph/tools/
python prepare_fed_data.py ./../../data/large_test.dat
```

## 3 Run Evaluation for FedGraph

```bash
cd Artifact/Artifact-MultipartyPSI/FedGraph/tools/
python run_cluster_participant.py
python extract_duration.py
python extract_comm.py
```
