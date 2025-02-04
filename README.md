# AtlEventProcess

### Lossless compression of raw data for the ATLAS experiment at CERN

Contains a simplified & modernized version of [ATLCopyBSEvent.cxx](https://gitlab.cern.ch/atlas/athena/-/blob/main/Event/ByteStreamCnvSvc/test/AtlCopyBSEvent.cxx).

This repository is for the Google Summer of Code (GSOC) project [Lossless compression of raw data for the ATLAS experiment at CERN](https://hepsoftwarefoundation.org/gsoc/2024/proposal_ATLASrawcompression.html). 

The goal of this project is to study the performance and effectiveness of various compression algorithms, specifically on ATLAS RAW data. The ATLAS experiment produces extremely large amounts of data, and it is only expected to increase with future planned upgrades within the LHC. Prior studies into compression of the data has shown that due to the highly redundant nature of the generated data, lossless data compression algorithms are extremely effective in reducing the binary size of ATLAS data. Here, we would like to find an algorithm that has a good balance of compression time, and compressed binary size.

### Process

This project can be split into two major parts. The first was studying which compression library should be used, mainly by looking at how effective different libraries are at compressing ATLAS RAW data. The second was modernizing the aforementioned [ATLCopyBSEvent.cxx](https://gitlab.cern.ch/atlas/athena/-/blob/main/Event/ByteStreamCnvSvc/test/AtlCopyBSEvent.cxx), as well as integrating the chosen compression library for the files being processed.

To assist with the former, I made use of [lzbench](https://github.com/inikep/lzbench), a tool created to benchmark a variety of different LZ77/LZSS/LZMA compression libraries, with the command ``` lzbench -ebrotli/zstd/lzlib/xz/zlib/lz4 decompressed.data``` where decompressed.data is an example ATLAS raw data file (in this case, the data displayed below is gathered from /cvmfs/atlas-nightlies.cern.ch/repo/data/data-art/Tier0ChainTests/TCT_Run3/data22_13p6TeV.00431493.physics_Main.daq.RAW._lb0525._SFO-16._0001.data, although I also tested with varying files to very similar results). All of the libraries and forms of compression use very similar methods to compress and decompress data, although each library will have varying quirks and capabilities that may be better or worse for our purposes here. 

Below is a table of data for compressors that were thought to be relevant to look at. One thing that is important to note is that much of the ATLAS experiment data has previously used zlib with compression level 1 for compression, so it will be treated as something of a baseline and the remaining libraries will be compared to it. Below are tables of libraries that were thought to be particularly notable, for each, we take note of the library used and what compression level, the ratio ((compressed size / original size) * 100), as well as compression and decompression speed.

| Library | Level | Ratio | Compression | Decompression |
|---------|-------|-------|-------------|---------------|
|zlib | 1 | 62.23 | 67.7 MB/s |  234 MB/s |
|zlib | 2 | 61.84 | 60.2 MB/s |  223 MB/s |
|zlib | 3 | 61.33 | 49.6 MB/s |  222 MB/s |
|zlib | 4 | 60.99 | 47.0 MB/s  | 240 MB/s |
|zlib | 5 | 60.39 | 32.9 MB/s |  219 MB/s |
|zlib | 6 | 60.02 | 22.5 MB/s  | 226 MB/s |
|zlib | 7 | 59.88 | 18.4 MB/s  | 218 MB/s |
|zlib | 8 | 59.76 | 12.0 MB/s  | 218 MB/s |
|zlib | 9 | 59.71 | 6.96 MB/s |  219 MB/s |

| Library | Level | Ratio | Compression | Decompression |
|---------|-------|-------|-------------|---------------|
|zstd | 1 | 62.61 | 457 MB/s  | 670 MB/s |
|zstd | 2 | 61.68 | 365 MB/s | 896 MB/s |
|zstd | 3 | 60.68 | 265 MB/s  | 903 MB/s |
|zstd | 4 | 60.12 | 156 MB/s  | 474 MB/s |
|zstd | 5 | 59.48 | 106 MB/s |  494 MB/s |
|zstd | 6 | 58.99 | 83.6 MB/s |  511 MB/s |
|zstd | 7 | 58.45 | 74.4 MB/s | 1037 MB/s |
|zstd | 8 | 58.35 | 65.6 MB/s |  525 MB/s |
|zstd | 9 | 57.94 | 54.3 MB/s |  976 MB/s |
|zstd | 10 | 57.60 | 41.3 MB/s |  424 MB/s |
|zstd | 11 | 57.52 | 27.7 MB/s |  805 MB/s |
|zstd | 12 | 57.48 | 23.8 MB/s |  487 MB/s |
|zstd | 13 | 57.54 | 11.6 MB/s |  493 MB/s |
|zstd | 14 | 57.49 | 10.6 MB/s  | 946 MB/s |
|zstd | 15 | 57.39 | 8.65 MB/s | 1093 MB/s |
|zstd | 16 | 55.79 | 7.32 MB/s |  907 MB/s |
|zstd | 17 | 55.56 | 4.62 MB/s | 909 MB/s |
|zstd | 18 | 55.10 | 3.84 MB/s  | 469 MB/s |
|zstd | 19 | 54.83 | 2.91 MB/s  | 490 MB/s |

| Library | Level | Ratio | Compression | Decompression |
|---------|-------|-------|-------------|---------------|
|brotli | 0 | 64.80 | 354 MB/s  | 221 MB/s |
|brotli | 1 | 62.79 | 233 MB/s  | 225 MB/s |
|brotli | 2 | 59.91 | 163 MB/s  | 259 MB/s |
|brotli | 3 | 59.66 | 119 MB/s  | 252 MB/s |
|brotli | 4 | 57.74 | 57.1 MB/s |  268 MB/s| 
|brotli | 5 | 56.65 | 31.4 MB/s |  252 MB/s|
|brotli | 6 | 56.31 | 21.9 MB/s  | 218 MB/s|
|brotli| 7 | 56.09 | 13.2 MB/s  | 234 MB/s|
|brotli | 8 | 55.95 | 8.81 MB/s  | 245 MB/s|
|brotli  | 9 | 55.78 | 6.57 MB/s  | 241 MB/s|
|brotli | 10 | 49.90 | 1.00 MB/s |  161 MB/s|
|brotli | 11 | 50.31 | 0.55 MB/s  | 147 MB/s|

| Library | Level | Ratio | Compression | Decompression |
|---------|-------|-------|-------------|---------------|
|lzlib | 0 | 54.82 | 22.3 MB/s | 32.5 MB/s |
|lzlib | 1 | 53.49 | 10.0 MB/s  | 34.5 MB/s |
|lzlib | 2 | 52.75 | 7.64 MB/s | 33.4 MB/s |
|lzlib | 3 | 51.74 | 5.50 MB/s | 33.4 MB/s |
|lzlib | 4 | 51.28 | 4.30 MB/s | 35.1 MB/s |
|lzlib | 5 | 50.84 | 3.37 MB/s | 35.2 MB/s |
|lzlib | 6 | 50.30 | 2.30 MB/s | 33.7 MB/s |

| Library | Level | Ratio | Compression | Decompression |
|---------|-------|-------|-------------|---------------|
|xz | 0 | 54.67 | 17.8 MB/s | 38.5 MB/s |
|xz | 1 | 53.89 | 15.8 MB/s | 40.5 MB/s |
|xz | 2 | 53.56 | 9.04 MB/s | 39.9 MB/s |
|xz | 3 | 53.44 | 5.78 MB/s | 42.8 MB/s |
|xz | 4 | 50.94 | 3.82 MB/s | 41.3 MB/s |
|xz | 5 | 50.31 | 2.77 MB/s | 42.1 MB/s |
|xz | 6 | 50.27 | 2.87 MB/s | 40.2 MB/s |

While the above is largely a dump of data collected, we will do some analysis to come to a conclusion about which compression library we think will be most effective for our purposes. To start, one notable upside is that almost all of the libraries, by virtue of being different forms of dictionary compression at their core, have decompression speeds that are fairly agnostic of the compression level and ratio. This, along with the fact that we care more about compression ratio for overall space-saving effectiveness and compression speed for realtime time-saving effectiveness, decompression speed will play a very small, if any, role in the decisions made after this. We can also see clearly that as we increase the compression levels for every library, they plateau very quickly. In general, the lowest and highest compression levels within a single library may only see a few percent difference in compression level, while being over an order of magnitude slower.

Below we are plotting compression speed and compression ratio generated by the above data plotting in this [colab](https://colab.research.google.com/drive/12uCIDyMJEIvKDe3O2V4KeugHHjqZYKlV?usp=sharing):

![ratio vs speed](https://github.com/user-attachments/assets/68119bd4-daa7-406e-9787-fc1ab7d3110a)

The ideal library would have high compression speed and low compression ratio. We can see that brotli covers the widest range, giving us the option to have the slowest compression speed in exchange for the lowest compression ratio, but also the second-fastest speed with the highest ratio. It is also immediately noticeable that zlib consistently gives compression ratios that are higher than desirable for the speed being compressed. Both xz and lzlib perform very similarly, where they do not offer the same range of options that brotli and zstd do, and they only operate in the slower and lower compression ratio end of the spectrum, but they are an improvement over brotli and zstd within that range. 

In the end, to compare each library with the most similar compression speed to zlib at compression level 1 we see:
| Library | Level | Ratio | Compression | Decompression |
|---------|-------|-------|-------------|---------------|
|zlib | 1 | 62.23 | 67.7 MB/s |  234 MB/s |
|zstd | 8 | 58.35 | 65.6 MB/s |  525 MB/s |
|brotli | 4 | 57.74 | 57.1 MB/s |  268 MB/s| 

Whereas when looking at the most similar compression ratios  we see:
| Library | Level | Ratio | Compression | Decompression |
|---------|-------|-------|-------------|---------------|
|zlib | 1 | 62.23 | 67.7 MB/s |  234 MB/s |
|zstd | 1 | 62.61 | 457 MB/s  | 670 MB/s |
|brotli | 1 | 62.79 | 233 MB/s  | 225 MB/s |

*lzlib and xz are not exactly comparable, their compression is much slower but with much lower ratios across their entire operable range.

Here we see that if we are looking to get a similar compression ratio, zlib performs the slowest by far at close to a fourth of the speed brotli, and zstd reaches almost two times brotli's speed. If we are looking to get similar compression times, while we do not have as close of a match as compression ratio, both zstd and brotli result in a reduction of a few percent of the original filesize.  

Given the data collected, we chose to use {} as our compression algorithm. The next step was to go through ATLCopyBSEvent.cpp and modernize it, then integrate the library of our choosing. 

### Build Instructions:
Building this project requires some CERN specific libraries such as eformat, ers, DataWriter, and DataReader. 
The most recommended way to build is to go through a virtual machine such as [Athena on Lima](https://atlassoftwaredocs.web.cern.ch/athena/lima/).

Once the virtual machine is installed, run the following commands to setup the correct GCC version (13.1.0) and allow CMake to find the required headers.

```
export ATLAS_LOCAL_ROOT_BASE=/cvmfs/atlas.cern.ch/repo/ATLASLocalRootBase
source ${ATLAS_LOCAL_ROOT_BASE}/user/atlasLocalSetup.sh
asetup Athena,main,latest
export CXXFLAGS=-isystem\ /cvmfs/atlas.cern.ch/repo/sw/tdaq/tdaq-common/tdaq-common-11-02-01/installed/include/
```

Then, to build the project:
```
mkdir build
cd build
cmake ..

# To run the program normally
./AtlEventProcess <parameters>

# To run the tests
make tests
```

The projects input parameters are as follows:
```
[-c --compress <compression level>] [-d --deflate] -e [--event] <eventNumbers> [-r, --run <runnumber>] [-l, --listevents] [-t --checkevents] -o, --out outputfile inputfiles....
Where eventNumbers is a comma-separated list of events
```

| Argument | Purpose|
|----------|------- |
|-c or --compress| Compress the output file using zstd with the specified compression level|
|-d or --deflate| Write a file that was previously compressed using ZLIB|
|-e or --event | Output only particular event numbers (comma separated, e.g. -e 1,2,3 ...)|
|-r or --run | Output only particular run numbers (comma separated, e.g. -r 4,5,6 ...)|
|-l or --listevents | List the events that are being processed|
|-t or --checkevents | Check the events that are being processed |
|-o or --out | Define the output file name|
| | name of inputfiles separated by spaces|
