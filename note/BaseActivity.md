# Activity基类研究

## BaseActivity
本Activity属于package net.oschina.app.improve.base.activities;包

先来看一下本项目BaseActivity的继承结构

<div align=center>
<img src="https://github.com/ainiyiwan/OSChina-v2.9.0/blob/master/png/baseAct.jpg"/>
</div>

看来we have a lot of work to do!!!

### SwipeBackActivity
从图中看来，这个BaseActivity继承自SwipeBackActivity，你们这个SwipeBackActivity是何方神圣，其实就是[这个库](https://github.com/ikew0ng/SwipeBackLayout),一个侧滑返回的库，
这个以后我会单开一个项目进行研究，敬请期待。
### 友盟统计方案，和之前的库一样，就不用看了
### fragment相关操作封装
```java
private Fragment mFragment;

protected void addFragment(int frameLayoutId, Fragment fragment) {
    if (fragment != null) {
        FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
        if (fragment.isAdded()) {
            if (mFragment != null) {
                transaction.hide(mFragment).show(fragment);
            } else {
                transaction.show(fragment);
            }
        } else {
            if (mFragment != null) {
                transaction.hide(mFragment).add(frameLayoutId, fragment);
            } else {
                transaction.add(frameLayoutId, fragment);
            }
        }
        mFragment = fragment;
        transaction.commit();
    }
}

protected void replaceFragment(int frameLayoutId, Fragment fragment) {
    if (fragment != null) {
        FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
        transaction.replace(frameLayoutId, fragment);
        transaction.commit();
    }
}

```
### 如果Bundle传接失败 直接杀掉Activity 新鲜
```java
if (initBundle(getIntent().getExtras())) {
    setContentView(getContentView());

    initWindow();

    ButterKnife.bind(this);
    initWidget();
    initData();
} else {
    finish();
}
```
### 抽象方法封装 必须实现
```java
protected abstract int getContentView();
```
### ImageLoader封装
```java
public synchronized RequestManager getImageLoader() {
    if (mImageLoader == null)
        mImageLoader = Glide.with(this);
    return mImageLoader;
}
```
其实封装并不好，后期要替换其他的ImageLoader,重构操作并不会少，但是一般不会替换，Glide应该不会停更，毕竟大厂
### 下面研究BaseActivity相关子类的封装
### BaseBackActivity
#### 相对于父类来说 封装了Dialog的显隐
```java
private ProgressDialog mWaitDialog;

protected void showLoadingDialog(String message) {
    if (mWaitDialog == null) {
        mWaitDialog = DialogHelper.getProgressDialog(this, true);
    }
    mWaitDialog.setMessage(message);
    mWaitDialog.show();
}

protected void dismissLoadingDialog() {
    if (mWaitDialog == null) return;
    mWaitDialog.dismiss();
}
```
#### 重写了返回按钮监听
```java
@Override
public void onBackPressed() {
    super.onBackPressed();
    finish();
}
```
### BaseRecyclerViewActivity(BaseBackActivity的子类)
**这个类做的就是初始化一个通用的布局，一个RecyclerView加上一个刷新控件，我认为这样做挺好的，只要能少写一行代码，那么我认为这个封装就是成功的，所以，谨记，以后重复的工作放到父类中，父类重复的操作放到父类的父类，如果想改变什么，可以重写，这就是封装的真谛吧**
#### 父类不能确定的交给子类来实现
```java
protected abstract Type getType();

protected abstract BaseRecyclerAdapter<T> getRecyclerAdapter();
```
#### 请关注我下面这段言论，所谓的封装其实就和大多数中国的父母一样，为什么这么说呢，大多数父母，在有生之年，拼了命的给孩子买房买车，这就等于父类帮你实现了许多方法，但是吃喝拉撒这种事还是要你自己来做的，所以，就抽象出来，交给你来实现，但是呢，我们每个人只能有一个父类，所以如果你个人能力强，可以多做点，父类少封装点，如果你没什么能力，或者是富二代，那么你要做的就很少了，因为父类都帮你做了，哎，伟大的父母。
### AccountBaseActivity
#### 关于实现抽象方法感想
```java
@Override
protected int getContentView() {
    return 0;
}
```
**这个Activity不是抽象的所以必须实现父类的抽象方法，但是又不能确定布局，所以直接返回零，让子类实现，关于子类和父类这件事，实现肯定是以子类为准，父类希望孩子平平淡淡，子类希望自己飞黄腾达，理想肯定是由子类决定**
#### 封装一个LocalBroadcastManager
```java
protected LocalBroadcastManager mManager;
 protected boolean sendLocalReceiver() {
        if (mManager != null) {
            Intent intent = new Intent();
            intent.setAction(ACTION_ACCOUNT_FINISH_ALL);
            return mManager.sendBroadcast(intent);
        }

        return false;
    }

    /**
     * register localReceiver
     */
    private void registerLocalReceiver() {
        if (mManager == null)
            mManager = LocalBroadcastManager.getInstance(this);
        IntentFilter filter = new IntentFilter();
        filter.addAction(ACTION_ACCOUNT_FINISH_ALL);
        if (mReceiver == null)
            mReceiver = new BroadcastReceiver() {
                @Override
                public void onReceive(Context context, Intent intent) {
                    String action = intent.getAction();
                    if (ACTION_ACCOUNT_FINISH_ALL.equals(action)) {
                        finish();
                    }
                }
            };
        mManager.registerReceiver(mReceiver, filter);
    }
```
#### 封装其他常用方法
```java
protected void hideKeyBoard(IBinder windowToken) {
    InputMethodManager inputMethodManager = this.mInputMethodManager;
    if (inputMethodManager == null) return;
    boolean active = inputMethodManager.isActive();
    if (active) {
        inputMethodManager.hideSoftInputFromWindow(windowToken, 0);
    }
}

/**
 * request network error
 *
 * @param throwable throwable
 */
protected void requestFailureHint(Throwable throwable) {
    if (throwable != null) {
        throwable.printStackTrace();
    }
    showToastForKeyBord(R.string.request_error_hint);
}
```
### BackActivity
#### 帮助子类实现一些方法
```java
@Override
protected void initWindow() {
    super.initWindow();
    mToolBar = (Toolbar) findViewById(R.id.toolbar);
    if (mToolBar != null) {
        setSupportActionBar(mToolBar);
        ActionBar actionBar = getSupportActionBar();
        if (actionBar != null) {
            actionBar.setDisplayHomeAsUpEnabled(true);
            actionBar.setHomeButtonEnabled(false);
        }
    }
}

@Override
public boolean onSupportNavigateUp() {
    finish();
    return super.onSupportNavigateUp();
}
```
### DetailActivity(BackActivity的子类)
**大部分不懂封装思想的人，看到这个类肯定很困惑，其实我也是，因为这已经算是顶级的封装了，后期慢慢体会**
#### 封装了一个框架，就剩中间一个空布局叫个子类来填充
```java
protected abstract DetailFragment getDetailFragment();
```
### ShareActivity(BackActivity的子类)
**之前自己一直想封装一个分享，看来有门道，可以参照这个来搞**
#### 封装了一个框架，就剩中间一个空布局叫个子类来填充
```java
protected abstract ShareFragment getShareFragment();
```
## 关于Fragment的封装见