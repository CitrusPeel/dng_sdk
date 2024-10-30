# dng_sdk

This project adds Linux support to Adobe's dng_sdk, built with cmake.

## How to Build

```bash
cmake -B build dng_sdk/projects/linux
cmake --build build -j $(nproc)
ls -lh build/libdng.a
```

## Reference

* [dng_sdk](https://helpx.adobe.com/camera-raw/digital-negative.html) from Adobe
* [emmcb/adobe-dng-sdk](https://github.com/emmcb/adobe-dng-sdk/blob/master/CMakeLists.txt)
