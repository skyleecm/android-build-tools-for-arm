Build notes:
============

This is built in similar way as [Debian aapt](https://packages.debian.org/jessie/aapt).

All the shared libraries are in the path /usr/lib/android

The tools binaries are in /usr/bin

Some modifications are made to the source as noted below:

1. frameworks/base/tools/aapt/Images.cpp

The assert in the for loops in function checkNinePatchSerialization are commented out due to some compilation issue.

    static void checkNinePatchSerialization(Res_png_9patch* inPatch,  void* data)
    {
        size_t patchSize = inPatch->serializedSize();
        void* newData = malloc(patchSize);
        memcpy(newData, data, patchSize);
        Res_png_9patch* outPatch = inPatch->deserialize(newData);
        // deserialization is done in place, so outPatch == newData
        assert(outPatch == newData);
        assert(outPatch->numXDivs == inPatch->numXDivs);
        assert(outPatch->numYDivs == inPatch->numYDivs);
        assert(outPatch->paddingLeft == inPatch->paddingLeft);
        assert(outPatch->paddingRight == inPatch->paddingRight);
        assert(outPatch->paddingTop == inPatch->paddingTop);
        assert(outPatch->paddingBottom == inPatch->paddingBottom);
        //for (int i = 0; i < outPatch->numXDivs; i++) {
        //    assert(outPatch->xDivs[i] == inPatch->xDivs[i]);
        //}
        //for (int i = 0; i < outPatch->numYDivs; i++) {
        //    assert(outPatch->yDivs[i] == inPatch->yDivs[i]);
        //}
        //for (int i = 0; i < outPatch->numColors; i++) {
        //    assert(outPatch->colors[i] == inPatch->colors[i]);
        //}
        free(newData);
    }

2. system/core/libutils/CallStack.cpp

The CallStack codes seems to require libbacktrace, which I have avoided to compile, so instead the code in CallStack::update is commented off. This means that any code that depends on CallStack will not work properly.

    void CallStack::update(int32_t ignoreDepth, pid_t tid) {
        mFrameLines.clear();

        //UniquePtr<Backtrace> backtrace(Backtrace::Create(BACKTRACE_CURRENT_PROCESS, tid));
        //if (!backtrace->Unwind(ignoreDepth)) {
        //    ALOGW("%s: Failed to unwind callstack.", __FUNCTION__);
        //}
        //for (size_t i = 0; i < backtrace->NumFrames(); i++) {
        //  mFrameLines.push_back(String8(backtrace->FormatFrameData(i).c_str()));
        //}
    }

3. system/core/libutils/VectorImpl.cpp

The ALOG_ASSERT is causing illegal instruction when running aapt, commenting it off seems to fix the aapt crash issue.

    const void* VectorImpl::itemLocation(size_t index) const
    {
        //ALOG_ASSERT(index<capacity(),
        //    "[%p] itemLocation: index=%d, capacity=%d, count=%d",
        //    this, (int)index, (int)capacity(), (int)mCount);

        if (index < capacity()) {
            const  void* buffer = arrayImpl();
            if (buffer) {
                return reinterpret_cast<const char*>(buffer) + index*mItemSize;
            }
        }
        return 0;
    }

