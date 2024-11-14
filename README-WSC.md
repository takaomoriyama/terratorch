

## Install additional RPMs for building modules
Update C/C++ compilers from gcc-8.5.0-18.el8.ppc64le to gcc-8.5.0-22.el8.ppc64le because default C compiler stops with internal error while compiling numpy module. Also Cargo (Rust package manager) is required to build konia-rs.
Since we don't have root privilege on WSC, we need to do some trick to install additional RPMs.

Prepare yumdownloader command which is installed only on login node.
```
# On  compute node
$ mkdir ~/bin
$ scp c699cloud01:/usr/bin/yumdownloader ~/bin
$ chmod +x ~/bin/yumdownloader
```

## Install additional RPMs
```
# On  compute node
$ cc --version
cc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-18)
$ mkdir ~/rpmfiles
$ yumdownloader --destdir ~/rpmfiles --resolve gcc-8.5.0 cargo
$ for i in ~/rpmfiles/*.rpm; do rpm -ivh --nodeps --dbpath=$HOME/rpmdb --relocate /=$HOME/.local --badreloc $i; done
$ cc --version
cc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-22)
```

## Create conda enviroment from terratorch-minimal

```
# On compute node
$ source /gpfs/wscgpfs02/ce107/miniforge3/etc/profile.d/conda.sh
$ conda create -n my-terratorch-minimal --clone terratorch-minimal
$ conda activate my-terratorch-minimal
```

## Clone repositories
```
$ cd WORKDIR
$ git clone git@github.com:takaomoriyama/terratorch.git
$ git clone git://github.com/NASA-IMPACT/Prithvi-WxC.git
# Edit Prithvi-WxC/pyproject.toml, and change line
# "torch >= 2.2",
# to 
# "torch >= 2.1.2",
$ cd terratorch
# Install pre-reqs
$ pip install -r requirements/required-wsc.txt -r requirements/dev.txt

# We can't do this because dependency condition from Prithvi-WxC.git can not be satisified.
#$ pip install -e .[wxc]  

$ pip install -e .
$ pip install ../Prithvi-WxC
$ pip install git+https://github.com/IBM/granite-wxc.git
$ terratorh
INFO:albumentations.check_version:A new version of Albumentations is available: 1.4.18 (you have 1.4.10). Upgrade using: pip install --upgrade albumentations
Segmentation fault (core dumped)
```

## Getting stack trace with gdb

```
$ which terratorch
/gpfs/wscgpfs02/moriyama/.conda/envs/my-terratorch-minimal/bin/terratorch
$ gdb `which python`
(gdb) run /gpfs/wscgpfs02/moriyama/.conda/envs/my-terratorch-minimal/bin/terratorch
Starting program: /scratch/moriyama/.conda/envs/my-terratorch-minimal/bin/python /gpfs/wscgpfs02/moriyama/.conda/envs/my-terratorch-minimal/bin/terratorch
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/glibc-hwcaps/power9/libthread_db-1.0.so".                                            
[New Thread 0x7ffeaafff170 (LWP 2492741)]
[Detaching after vfork from child process 2492742]
[Detaching after vfork from child process 2492745]
INFO:albumentations.check_version:A new version of Albumentations is available: 1.4.18 (you have 1.4.10). Upgrade using: pip install --upgrade albumentations
[New Thread 0x7ffe95ecf170 (LWP 2492754)]
[Thread 0x7ffe95ecf170 (LWP 2492754) exited]

Thread 1 "python" received signal SIGSEGV, Segmentation fault.
0x126fdcd095a8b990 in ?? ()
Missing separate debuginfos, use: yum debuginfo-install glibc-2.28-211.el8.ppc64le
(gdb) where
#0  0x126fdcd095a8b990 in ?? ()
#1  0x00007fff95a8ba80 in c10::ThreadLocalDebugInfo::get(c10::DebugInfoKind) ()
   from /scratch/moriyama/.conda/envs/my-terratorch-minimal/lib/python3.10/site-packages/torch/lib/libc10.so
#2  0x00007fff95a2414c in c10::memoryProfilingEnabled() ()
   from /scratch/moriyama/.conda/envs/my-terratorch-minimal/lib/python3.10/site-packages/torch/lib/libc10.so
#3  0x00007fff95a24f1c in c10::ProfiledCPUMemoryReporter::New(void*, unsigned long) ()
   from /scratch/moriyama/.conda/envs/my-terratorch-minimal/lib/python3.10/site-packages/torch/lib/libc10.so
#4  0x00007fff95a268fc in c10::DefaultCPUAllocator::allocate(unsigned long) const ()
   from /scratch/moriyama/.conda/envs/my-terratorch-minimal/lib/python3.10/site-packages/torch/lib/libc10.so
#5  0x00007fffbda585f0 in c10::StorageImpl::StorageImpl(c10::StorageImpl::use_byte_size_t, c10::SymInt const&, c10::Allocator*, bool) () from /scratch/moriyama/.conda/envs/my-terratorch-minimal/lib/python3.10/site-packages/torch/lib/libtorch_cpu.so
#6  0x00007fffbda5cc30 in at::TensorBase at::detail::_empty_generic<long>(c10::ArrayRef<long>, c10::Allocator*, c10::DispatchKeySet, c10::ScalarType, c10::optional<c10::MemoryFormat>) ()
   from /scratch/moriyama/.conda/envs/my-terratorch-minimal/lib/python3.10/site-packages/torch/lib/libtorch_cpu.so
...
#172 0x00000001001c9118 in _PyRun_AnyFileObject ()
#173 0x00000001000690c4 in Py_RunMain ()
#174 0x0000000100069998 in pymain_main ()
#175 0x0000000100069b84 in Py_BytesMain ()
#176 0x000000010005b1e8 in main ()
