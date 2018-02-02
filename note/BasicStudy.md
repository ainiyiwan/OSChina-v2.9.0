# 基类研究

## Application研究
先来看一下本项目application的继承结构

<div align=center>
<img src="https://github.com/ainiyiwan/OSChina-v2.9.0/blob/master/png/application.jpg"/>
</div>

### BaseApplication
#### 1.Application Context 最佳写法
```java
    static Context _context;

    @Override
    public void onCreate() {
        super.onCreate();
        _context = getApplicationContext();
        //LeakCanary.install(this);
    }

    public static synchronized BaseApplication context() {
        return (BaseApplication) _context;
    }
```
#### 2.常用方法封装
这个Application是把sharePreference封装在这里，我也不知道这样到底好不好
#### 3.方法重载
```java
public static void showToast(int message) {
    showToast(message, Toast.LENGTH_LONG, 0);
}

public static void showToast(String message) {
    showToast(message, Toast.LENGTH_LONG, 0, Gravity.BOTTOM);
}
```
#### AppContext 

```java
public class AppContext extends BaseApplication {}
```
#### 1.默认分页大小
```java
public static final int PAGE_SIZE = 20;// 默认分页大小
```
个人观点：大公司当然可以这样做，因为大公司很多东西都是规范下来的，但是小公司，不同页面的分页大小都不一样，这个就不用配置
#### 2.漂亮的单例
```java
private static AppContext instance;

@Override
public void onCreate() {
    super.onCreate();
    instance = this;
    DBManager.init(this);
}

/**
 * 获得当前app运行的AppContext
 *
 * @return AppContext
 */
public static AppContext getInstance() {
    return instance;
}
```
#### 3.封装常用类
```java
public Properties getProperties() {
    return AppConfig.getAppConfig(this).get();
}
```
### OSCApplication
#### reInit引发的血案
```java
public static void reInit() {
    ((OSCApplication) OSCApplication.getInstance()).init();
}
```
通过查看reInit的引用，发现了很多好玩的类，需要好好研究，见[这里]()
#### 