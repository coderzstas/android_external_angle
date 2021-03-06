# ANGLE Restricted Traces

The files in this directory are traces of real applications.  We host them
internally because they may contain third party IP which we don't want
to share publicly.

## Setup

In order to compile and run with these, you must be granted access by Google,
then authenticate with the cloud with your @google account:
```
gcloud auth login
```
```
download_from_google_storage --config
<enter 0 for the project ID>
```
Add the following to ANGLE's .gclient file:
```
    "custom_vars": {
      "checkout_angle_internal":"True"
    },
```
Then use gclient to pull down binary files from a cloud storage bucket.
```
gclient runhooks
```
To build the tests, add the following GN argument:
```
build_angle_trace_perf_tests = true
```
Build the angle_perftests:
```
autoninja -C out/Release angle_perftests
```
Run them like so:
```
out/Release/angle_perftests --gtest_filter=TracePerfTest*
```

## Updating traces

The Manhattan and TRex traces were captured on Windows with an Intel driver (due to better compressed texture support).

Update START and END for each range.

### Windows

```
cd gfxbench\out\install\vs2017-x64\lib
python ..\..\..\..\scripts\refresh_angle_libs.py --verbose
set TFW_PACKAGE_DIR=../../../build/vs2017-x64/testfw/tfw-dev
set ANGLE_DEFAULT_PLATFORM=vulkan

# TRex
set START=200
set END=210
set LABEL=trex
mkdir ..\..\..\..\..\angle\src\tests\perf_tests\restricted_traces\%LABEL%_%START%
set ANGLE_CAPTURE_LABEL=%LABEL%_%START%
set ANGLE_CAPTURE_FRAME_START=%START%
set ANGLE_CAPTURE_FRAME_END=%END%
set ANGLE_CAPTURE_OUT_DIR=..\..\..\..\..\angle\src\tests\perf_tests\restricted_traces\%LABEL%_%START%
..\bin\testfw_app.exe -b ../../../build/vs2017-x64/testfw/tfw-dev --gfx egl -w 1920 -h 1080 -t gl_trex --ei -frame_step_time=40

# Manhattan
set START=10
set END=20
set LABEL=manhattan
mkdir ..\..\..\..\..\angle\src\tests\perf_tests\restricted_traces\%LABEL%_%START%
set ANGLE_CAPTURE_LABEL=%LABEL%_%START%
set ANGLE_CAPTURE_FRAME_START=%START%
set ANGLE_CAPTURE_FRAME_END=%END%
set ANGLE_CAPTURE_OUT_DIR=..\..\..\..\..\angle\src\tests\perf_tests\restricted_traces\%LABEL%_%START%
..\bin\testfw_app.exe -b ../../../build/vs2017-x64/testfw/tfw-dev --gfx egl -w 1920 -h 1080 -t gl_manhattan --ei -frame_step_time=40
```

### Linux

```
cd gfxbench/out/install/linux/lib
python ../../../../scripts/refresh_angle_libs.py --verbose
export PLATFORM=linux
export TFW_PACKAGE_DIR=../../../build/linux/testfw_Release/tfw-dev
export ANGLE_DEFAULT_PLATFORM=vulkan
export LD_LIBRARY_PATH=.

# TRex
export START=200
export END=210
export LABEL=trex
mkdir -p ../../../../../angle/src/tests/perf_tests/restricted_traces/${LABEL}_${START}
export ANGLE_CAPTURE_LABEL=${LABEL}_${START}
export ANGLE_CAPTURE_FRAME_START=$START
export ANGLE_CAPTURE_FRAME_END=$END
export ANGLE_CAPTURE_OUT_DIR=../../../../../angle/src/tests/perf_tests/restricted_traces/${LABEL}_${START}
../bin/testfw_app -b $TFW_PACKAGE_DIR --gfx egl -w 1920 -h 1080 -t gl_trex --ei -frame_step_time=40

# Manhattan
export START=10
export END=20
export LABEL=manhattan
mkdir -p ../../../../../angle/src/tests/perf_tests/restricted_traces/${LABEL}_${START}
export ANGLE_CAPTURE_LABEL=${LABEL}_${START}
export ANGLE_CAPTURE_FRAME_START=$START
export ANGLE_CAPTURE_FRAME_END=$END
export ANGLE_CAPTURE_OUT_DIR=../../../../../angle/src/tests/perf_tests/restricted_traces/${LABEL}_${START}
../bin/testfw_app -b $TFW_PACKAGE_DIR --gfx egl -w 1920 -h 1080 -t gl_manhattan --ei -frame_step_time=40
```

## Upload to the cloud

```
cd ~/chromium/src/third_party/angle/src/tests/perf_tests/restricted_traces
upload_to_google_storage.py --bucket chrome-angle-capture-binaries --archive trex_200
upload_to_google_storage.py --bucket chrome-angle-capture-binaries --archive manhattan_10
```

After uploading, add the sha1 files created by the upload and submit them with your changes.

## Adding new tests

After you capture and upload a set of frames from an application using the steps above you'll need to:

 * Add the new traces to [`restricted_traces/restricted_traces.json`](restricted_traces.json).
 * Re-run code generation with [`scripts/run_code_generation.py`][run_code_generation].
 * Update your CL to include the new generated code. Your trace tests should now be turned on.

[run_code_generation]: ../../../../scripts/run_code_generation.py
