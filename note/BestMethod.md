# 漂亮的常用方法

### 1.清除app缓存 AppContext
```java
 /**
     * 清除app缓存
     */
    public void clearAppCache() {
        //DataCleanManager.cleanDatabases(this);
        // 清除数据缓存
        DataCleanManager.cleanInternalCache(this);
        // 2.2版本才有将应用缓存转移到sd卡的功能
        if (isMethodsCompat(android.os.Build.VERSION_CODES.FROYO)) {
            DataCleanManager.cleanCustomCache(MethodsCompat
                    .getExternalCacheDir(this));
        }

        /*
        Run.onUiSync(new Action() {
            @Override
            public void call() {
                // Glide 清理内存必须在主线程
                Glide.get(OSCApplication.getInstance()).clearMemory();
            }
        });

        AppOperator.runOnThread(new Runnable() {
            @Override
            public void run() {
                // Glide 清理磁盘必须在子线程
                Glide.get(OSCApplication.getInstance()).clearDiskCache();
            }
        });
        */

        // 清除编辑器保存的临时内容
        Properties props = getProperties();
        for (Object key : props.keySet()) {
            String _key = key.toString();
            if (_key.startsWith("temp"))
                removeProperty(_key);
        }
    }
```

### 2.判断当前版本是否兼容目标版本的方法 AppContext
```java
public static boolean isMethodsCompat(int VersionCode) {
        int currentVersion = android.os.Build.VERSION.SDK_INT;
        return currentVersion >= VersionCode;
    }
```