### Setup in Windows
1. Download `Caffe-Windows` from [Github](https://github.com/happynear/caffe-windows)
```bash
git clone https://github.com/happynear/caffe-windows.git
```

2. In directory `./BuildVS2013/` open the `.sln` file. For project `caffelib`, configure the `Additional Include Directories` and `Additional Library Directories` as following (directly copy, note to specify the directories of `opencv` and `boost`, and currently only supports [Boost 1.56](http://nchc.dl.sourceforge.net/project/boost/boost-binaries/1.56.0/boost_1_56_0-msvc-12.0-64.exe))

- Additional Include Directories (Debug and Release)
	```cpp
	../../src;../../include;../../3rdparty/include/openblas;../../3rdparty/include/lmdb;../../3rdparty/include/hdf5;../../3rdparty/include;C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0\include;D:\Libs\opencv300\build\include;D:\Libs\opencv300\build\include\opencv;D:\Libs\opencv300\build\include\opencv2;D:\Libs\boost_1_56_0
	```

- Additional Library Directories (Debug and Release)
	```cpp
	../../3rdparty/lib;D:\Libs\boost_1_56_0\lib64-msvc-12.0;C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0\lib\x64;D:\Libs\opencv300\build\x64\vc12\lib 
	```

- Library Input (Debug)
	```cpp
	opencv_ts300d.lib;opencv_world300d.lib;gflagsd.lib;libglog.lib;libopenblas.dll.a;libprotobufd.lib;libprotoc.lib;leveldbd.lib;lmdbd.lib;libhdf5_D.lib;libhdf5_hl_D.lib;Shlwapi.lib;cudart.lib;cuda.lib;nppi.lib;cufft.lib;cublas.lib;curand.lib;cudnn.lib;%(AdditionalDependencies)
	```
	
- Library Input (Release)
	```cpp
	opencv_ts300.lib;opencv_world300.lib;gflags.lib;libglog.lib;libopenblas.dll.a;libprotobuf.lib;libprotoc.lib;leveldb.lib;lmdb.lib;libhdf5.lib;libhdf5_hl.lib;Shlwapi.lib;cudart.lib;cuda.lib;nppi.lib;cufft.lib;cublas.lib;curand.lib;cudnn.lib;%(AdditionalDependencies)
	```

	For project `caffe`, set as above and also remember to add `../../bin` as the include directory and `caffelib(d).lib` as the input.

3. Code Modifications
	- Comment `blob.hpp (Line 140)` and remove `USE_CUDNN` in Preprocessor to make it 3d convolution supported. (if only using 2D convolution, reverse the operations)
	```cpp
	//CHECK_LE(num_axes(), 4)
    //    << "Cannot use legacy accessors on Blobs with > 4 axes."; 
	```
	- Add `return NULL` below `LOG(FATAL) << "Unknown database backend";` at `db.cpp (Line 20)`. 
	- Comment `DCHECK_LT(label_value, num_labels);` at `accuracy_layer.cpp (Line 71)` to allow label ID larger than number of labels.
	- Modify `FLAGS_log_dir = ".\\log\\";` to `FLAGS_log_dir = string(".\\log\\");` at `common.cpp (Line 63)`.


4. To compile MatCaffe in Windows, the directories and library input should be like this
	- Additional Include Directories (Release only is sufficient)
	```cpp
	../../3rdparty/include;../../src;../../include;D:\Program Files\MATLAB\R2014b\extern\include;D:\Libs\boost_1_56_0;%(AdditionalIncludeDirectories)
	```
	
	- Additional Library Directories (Release only is sufficient)
	```cpp
	../../3rdparty/lib;../../bin;D:\Program Files\MATLAB\R2014b\extern\lib\win64\microsoft;D:\Libs\opencv300\build\x64\vc12\lib;D:\Libs\boost_1_56_0\lib64-msvc-12.0;%(AdditionalLibraryDirectories)
	```

	- Library Input (Release only is sufficient)
	```cpp
	caffelib.lib;opencv_ts300.lib;opencv_world300.lib;gflags.lib;libglog.lib;libopenblas.dll.a;libprotobuf.lib;libprotoc.lib;leveldb.lib;lmdb.lib;libhdf5.lib;libhdf5_hl.lib;Shlwapi.lib;cudart.lib;cuda.lib;nppi.lib;cufft.lib;cublas.lib;curand.lib;cudnn.lib;libmx.lib;libmex.lib;libmat.lib;%(AdditionalDependencies)
	```

---------------------
Forked from https://www.github.com/BVLC/caffe master branch in 2015/11/09 . Next update time may be around 2016/01/01 .

I have made a list of some frequently asked questions in [FAQ.md](FAQ.md). If you get confused during configuring, please firstly look up for your question in the [FAQ.md](FAQ.md). This FAQ list is still under construction, I will keep adding questions into it.

Setup step:
======

1. Download third-party libraries from [BaiduYun Disk](http://pan.baidu.com/s/1sjIKsc1) or [OneDrive](http://1drv.ms/1HC3Se4) and extract the files to `caffe-windows_root/3rdparty/`. **Please don't forget to add the `./3rdparty/bin` folder to your environment variable `PATH`.**

2. Run `./src/caffe/proto/extract_proto.bat` to create `caffe.pb.h`, `caffe.pb.cc` and `caffe_pb2.py`.

3. Double click ./buildVS2013/MainBuilder.sln to open the solution. If you do not have a Nvidia GPU, please open ./build_cpu_only/MainBuilder.sln.

4. Change the compile mode to Release and X64.

5. Modify the cuda device compute capability defined in the settings (`caffelib properties` -> `CUDA C/C++` -> `Device` -> `Code Generation`) to your GPU's compute capability (such as compute_30,sm_30; etc). You can look up for your GPU's compute capability in https://en.wikipedia.org/wiki/CUDA . Some general GPUs' compute capabilities are listed below.

 - If your GPU's compute capability is below or equal to 2.1, please remove the `USE_CUDNN` macro in the proprocessor definition of all projects.
 - If you are using cpu only solution, just ignore this step.

6. Compile.

| GPU                                         | Compute Capability    |
| ------------------------------------------- |:---------------------:|
| GTX660, 680, 760, 770                       | compute_30,sm_30      |
| GTX780, Titan Z, Titan Black, K20, K40      | compute_35,sm_35      |
| GTX960, 980, Titan X                        | compute_52,sm_52      |


中文安装说明：http://blog.csdn.net/happynear/article/details/45372231

Matlab Wrapper
======
Just replace the Matlab include and library path defined in the settings and compile.
**Don't forget to add `./matlab` to your Matlab path.**

Python Wrapper
======
Similar with Matlab, replace the python include and library path and compile.

Most of the libraries listed in `./python/requirements.txt` can be installed by `pip install`. However, some of them cannot be installed so easily.

For protobuf, you may download the codes from https://github.com/google/protobuf. Copy `caffe-windows-root/src/caffe/proto/protoc.exe` to `protobuf-root/src`. Then run `python setup.py install` in `protobuf-root/python`.

For leveldb, I have created a repository https://github.com/happynear/py-leveldb-windows . Please follow the instructions in `README.md` to install it.

MNIST example
======
Please download the mnist leveldb database from http://pan.baidu.com/s/1mgl9ndu and extract it to `./examples/mnist`. Then double click `./run_mnist.bat` to run the MNIST demo.

Update log
======
2015/11/09 CuDNN v3 works well now.

2015/09/14 Multi-GPU is supported now. 

WARNING: When you are using multiple gpus to train a model, please do not directly close the command window. Instead, please use `Ctrl+C` to avoid the gpu driver from crash.

You can also press `Ctrl+Break` to save a model snapshot whenever you want during training.

Acknowlegement
======
We greatly thank [Yangqing Jia](https://github.com/Yangqing) and [BVLC group](https://www.github.com/BVLC/caffe) for developing Caffe,

[@niuzhiheng](https://github.com/niuzhiheng) for his contribution on the first generation of caffe-windows,

[@ChenglongChen](https://github.com/ChenglongChen/batch_normalization) for his implementation of Batch Normalization,

[@jackculpepper](https://github.com/jackculpepper/caffe) for his implementation of locally-connected layer,

and all people who have contributed to the caffe user group.
