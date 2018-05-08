## Fragment源码解析
### 1、简介
Fragment是Android3.0后引入的一个新的API，他出现的初衷是为了适应大屏幕的平板电脑， 当然现在他仍然是平板APP UI设计的宠儿，而且我们普通手机开发也会加入这个Fragment，我们可以把他看成一个小型的Activity，又称Activity片段。

### 2、用法
#### 2.1 Fragment添加
```
FragmentManager fragmentManager=getSupportFragmentManager();
//开启一个事物
FragmentTransaction fragmentTransaction=fragmentManager.beginTransaction();
fragmentTransaction.add(R.id.xxx,new Fragment());
//提交事物
fragmentTransaction.commit();
```

#### 2.2 Fragment替换
```
FragmentManager fragmentManager=getSupportFragmentManager();
//开启一个事物
FragmentTransaction fragmentTransaction=fragmentManager.beginTransaction();
fragmentTransaction.replace(R.id.xxx,new Fragment());
//提交事物
fragmentTransaction.commit();
```

#### 3、源码解析
#### 3.1、add()方法
```
@Override
public FragmentTransaction add(Fragment fragment, String tag) {
    doAddOp(0, fragment, tag, OP_ADD);
    return this;
}

private void doAddOp(int containerViewId, Fragment fragment, String tag, int opcmd) {
    final Class fragmentClass = fragment.getClass();
    final int modifiers = fragmentClass.getModifiers();
    if (fragmentClass.isAnonymousClass() || !Modifier.isPublic(modifiers)
            || (fragmentClass.isMemberClass() && !Modifier.isStatic(modifiers))) {
        throw new IllegalStateException("Fragment " + fragmentClass.getCanonicalName()
                + " must be a public static class to be  properly recreated from"
                + " instance state.");
    }

    fragment.mFragmentManager = mManager;

    if (tag != null) {
        if (fragment.mTag != null && !tag.equals(fragment.mTag)) {
            throw new IllegalStateException("Can't change tag of fragment "
                    + fragment + ": was " + fragment.mTag
                    + " now " + tag);
        }
        fragment.mTag = tag;
    }

    if (containerViewId != 0) {
        if (containerViewId == View.NO_ID) {
            throw new IllegalArgumentException("Can't add fragment "
                    + fragment + " with tag " + tag + " to container view with no id");
        }
        if (fragment.mFragmentId != 0 && fragment.mFragmentId != containerViewId) {
            throw new IllegalStateException("Can't change container ID of fragment "
                    + fragment + ": was " + fragment.mFragmentId
                    + " now " + containerViewId);
        }
        fragment.mContainerId = fragment.mFragmentId = containerViewId;
    }

    addOp(new Op(opcmd, fragment));
}

void addOp(Op op) {
    mOps.add(op);
    op.enterAnim = mEnterAnim;
    op.exitAnim = mExitAnim;
    op.popEnterAnim = mPopEnterAnim;
    op.popExitAnim = mPopExitAnim;
}
```
>add方法中又调用了doAddOp方法，这个方法主要就是获取你传进来的一些参数，然后再调用addOp方法设置一些参数；整体来说add方法主要就是设置一些参数，所以我们需要调用commit方法来改变，那么进到commit方法里面看一下

#### 3.2、commit方法
```
@Override
public int commit() {
    return commitInternal(false);
}

...省略一些方法...

void moveToState(Fragment f, int newState, int transit, int transitionStyle, boolean keepActive) {
	这个方法会回调fragment的生命周期，最后会把fragment（调用onCreateView方法是会生成一个View）添加到container（也就是Activity）中
}
```

#### 3.3、replace方法
```
@Override
public FragmentTransaction replace(int containerViewId, Fragment fragment, String tag) {
    if (containerViewId == 0) {
        throw new IllegalArgumentException("Must use non-zero containerViewId");
    }

    doAddOp(containerViewId, fragment, tag, OP_REPLACE);
    return this;
}
```

我们可以看到，和add方法一样都是调用doAddOp方法，只不过传的最后一个参数是OP_REPLACE