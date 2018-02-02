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
通过查看reInit的引用，发现了很多好玩的类，需要好好研究，见[这里](https://github.com/ainiyiwan/OSChina-v2.9.0/blob/master/note/ToStudy.md)
#### 初始化必用的类以及三方库
```java
private void init() {
    OSCSharedPreference.init(this, "osc_update_sp");
    // 初始化异常捕获类
    AppCrashHandler.getInstance().init(this);
    // 初始化账户基础信息
    AccountHelper.init(this);
    // 初始化网络请求
    ApiHttpClient.init(this);
    //初始化百度地图
    SDKInitializer.initialize(this);
    DBManager.init(this);

    if (OSCSharedPreference.getInstance().hasShowUpdate()) {//如果已经更新过
        //如果版本大于更新过的版本，就设置弹出更新
        if (BuildConfig.VERSION_CODE > OSCSharedPreference.getInstance().getUpdateVersion()) {
            OSCSharedPreference.getInstance().putShowUpdate(true);
        }
    }
}

```
#### 应用升级
```java
if (OSCSharedPreference.getInstance().hasShowUpdate()) {//如果已经更新过
    //如果版本大于更新过的版本，就设置弹出更新
    if (BuildConfig.VERSION_CODE > OSCSharedPreference.getInstance().getUpdateVersion()) {
        OSCSharedPreference.getInstance().putShowUpdate(true);
    }
}
```
这一块以后会抽取出来，生成一个更新库

## Activity研究

### BaseActivity
奇怪的是这个项目竟然有两个BaseActivity，我们先来研究老版本，也就是package net.oschina.app.base;包下的
#### 新颖的友盟埋点方案(我之前是一个类一个类的写，吐血。。。)
```java
private final String packageName4Umeng = this.getClass().getName();
@Override
protected void onPause() {
    super.onPause();
    MobclickAgent.onPageEnd(this.packageName4Umeng);
    MobclickAgent.onResume(this);

    if (this.isFinishing()) {
        TDevice.hideSoftKeyboard(getCurrentFocus());
    }
}

@Override
protected void onResume() {
    super.onResume();
    MobclickAgent.onPageStart(this.packageName4Umeng);
    MobclickAgent.onResume(this);
}
```
#### 常用方法封装
首先封装接口
```java
public interface DialogControl {

    public abstract void hideWaitDialog();

    public abstract ProgressDialog showWaitDialog();

    public abstract ProgressDialog showWaitDialog(int resid);

    public abstract ProgressDialog showWaitDialog(String text);
}

```
baseActivity实现接口
```java
public abstract class BaseActivity extends AppCompatActivity implements
        DialogControl{}

@Override
public ProgressDialog showWaitDialog() {
    return showWaitDialog(R.string.loading);
}

@Override
public ProgressDialog showWaitDialog(int resid) {
    return showWaitDialog(getString(resid));
}
   ...
```
子类使用...

#### ButterKnife初始化
```java
 // 通过注解绑定控件
ButterKnife.bind(this);
```
## 