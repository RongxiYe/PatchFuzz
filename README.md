# PatchFuzz
The code for our paper: "PatchFuzz: An Efficient Way to Incorporate Patching with Hybrid Fuzzing".

If you want to build it for other architectures, see step 4 in the part "Build PatchFuzz" below. 



## Build PatchFuzz

First, make sure that you have installed docker. We do not test our code ouside docker since it rely on QSYM.



### Build QSYM

1. clone the repository of QSYM

```
git clone https://github.com/sslab-gatech/qsym.git
```

2. build qsym docker as guided in https://github.com/sslab-gatech/qsym/tree/master

```
echo 0|sudo tee /proc/sys/kernel/yama/ptrace_scope
cd qsym/
docker build -t qsym ./
```

If you cannot install pip correctly during setup.sh(maybe due to network issue), you can comment out the line "RUN pip install ." with "#" and do it yourself after you are inside the docker container.

3. run the container

```
docker run --cap-add=SYS_PTRACE -it qsym /bin/bash
```

4. finish what you comment in step 2 if you do

```
cd /workdir/qsym
pip install .
```



### Build PatchFuzz

1. clone our repository inside the docker

```
git clone https://github.com/RongxiYe/PatchFuzz.git
```

2. or, if the network does not work well, clone the repository outside the docker and copy it into the container（need root）

```
//clone ouside docker
docker cp PatchFuzz/ <your_container_id>:/patchfuzz
```

3. build patchfuzz-afl

```
cd /patchfuzz
make clean
make
```

If it builds successfully, you will see afl-fuzz in the directory.

4. build qemu-mode

```
sudo apt-get install libtool-bin automake bison libglib2.0-dev
cd qemu_mode
tar -xf qemu-2.10.0.tar 
./build_qemu_support.sh <ARCHES>
```



### Build Ghidra

1. download ghidra release(we use 10.0.4) in https://github.com/NationalSecurityAgency/ghidra/releases/tag/Ghidra_10.4_build  and unzip it inside docker.
2. set env "GHIDRA_INSTALL_DIR" as where you put ghidra build.

```
export GHIDRA_INSTALL_DIR=<xx/ghidra_10.4_PUBLIC/>
```

3. install java 17(maybe need root, and the time may be a bit long)

```
apt-get install software-properties-common
add-apt-repository ppa:openjdk-r/ppa
apt update
apt install openjdk-17-jdk
```

4. update python3 to 3.8 or later

```
add-apt-repository ppa:deadsnakes/ppa
update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.5 1
update-alternatives --install /usr/bin/python3 python3 <path to your newly added python> 2

#change python:
update-alternatives --config python3

#add lsb_release
ln -s /usr/share/pyshared/lsb_release.py <path to lib of your newly added python>/lsb_release.py
```

5. install pyhidra

```
python3 -m pip install pyhidra
```

6. check if ghidra is successfully installed:

```
cd /patchfuzz
python3 ghidra/ghidra_analyze.py  <target binary>
```

7. !!rebase problem: see "rebase_offset" in ghidra_analyze.py. The default base is 0x400000. If you need to test binaries on other architectures, make sure to set the correct base address in "rebase_offset".



## Run PatchFuzz

PatchFuzz has two parts: AFL and QSYM. You need to first start afl-fuzz in one shell window and run QSYM in another shell window. You can use tmux to run in the background.

### Run PatchFuzz-AFL

see run.sh and its inside comments.

example:

```
cd /patchfuzz
./run.sh $PWD <path to seed dir> <your afl workdir> <path to tested binary>
```

NOTE: Sometimes you need to modify run.sh. For the content after $AFL_CMDLINE in run.sh, please follow the rules of AFL. Use @@ for file input.

For example, if the tested binary requires input like:

```
base64 -d <a file>
```

The run.sh script should be like:

```
$AFL_ROOT/afl-fuzz -m none -M afl-master -i $INPUT -o $OUTPUT -Q -- $AFL_CMDLINE -d @@
```

NOTE2: You may need to initiate the running environment for AFL using root:

```
echo core >/proc/sys/kernel/core_pattern
cd /sys/devices/system/cpu
echo performance | tee cpu*/cpufreq/scaling_governor
```







### Run QSYM

see run_symbolic.sh and its inside comments.

You need to start this scripts AFTER you see AFL panel in the other window!!!!

example:

```
cd /patchfuzz
./run_symbolic.sh <path to tested binary> <your afl workdir>
```

NOTE: If you modify run.sh for the arguments of the tested binary, you need to modify /patchfuzz/symbolic/symbolic.py. Search "cmd" and add the arguments there. For example, for base64, it would be:

```
cmd = [binary , '-d', '@@']
```



### Directories

real_crash: true positive

candidate_crash: for manual analysis

patches: all patch information files

crashes: all crashes containing four components，which are the corresponding seed，and three patch information files

hangs: same as crashes dir, only for hangs



## Analysis scripts

add soon....
