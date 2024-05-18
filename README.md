# aahh
pov: getting the minecraft classloader using jni
```cpp
    jobject getMinecraftClassLoader(JNIEnv* jniEnv) {
        jobject targetClassLoader = NULL;
        typedef jobjectArray(JNICALL* JVM_GetAllThreads)(JNIEnv* env, jclass dummy);
        JVM_GetAllThreads getAllThreads = (JVM_GetAllThreads)GetProcAddress(GetModuleHandleA("jvm.dll"), "JVM_GetAllThreads");
        jobjectArray threadsArray = getAllThreads(jniEnv, NULL);
        int threadsCount = jniEnv->GetArrayLength(threadsArray);
        for (int i = 0; i < threadsCount; i++) {
            jobject thread = jniEnv->GetObjectArrayElement(threadsArray, i);
            jclass thread_class = jniEnv->FindClass("java/lang/Thread");
            jmethodID getName = jniEnv->GetMethodID(thread_class, "getName", "()Ljava/lang/String;");
            jstring threadName = (jstring)jniEnv -> CallObjectMethod(thread, getName);
            if (jstring2string(jniEnv, threadName) == "Client thread") {
                jfieldID ctxClsLoader = jniEnv->GetFieldID(thread_class, "contextClassLoader", "Ljava/lang/ClassLoader;");
                jobject classLoader = jniEnv->GetObjectField(thread, ctxClsLoader);
                targetClassLoader = classLoader;
            }
        }
        return targetClassLoader;
    }
