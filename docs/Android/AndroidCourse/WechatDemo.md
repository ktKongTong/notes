> git地址: [WechatDemo](https://github.com/ktKongTong/WechatDemo)

> #### **功能说明**

实现微信的门户界面

主要是四个底部导航栏切换以及滑动切换

如下图所示

![wechatDemo](./pic/WechatDemo/wechatDemo.jpg ':size=20%')

> #### **分析**

从图中可以看出,门户界面主要分为三个部分
顶部工具栏,中间内容以及底部导航栏

首先看底部导航栏,在四个页面中都是一样的,所以可以将其作为公用部件,放于MainActivity中

四个页面的主要内容各不一样,使用Fragment实现,具体内容暂不考虑,将四个fragment结合使用<code>ViewPager</code>实现

至于顶部工具栏,可以放在fragment中实现

> #### **实现**

##### 1. 底部导航栏的布局

直接使用md的BottomNavigationView控件,将这个控件放在主布局的底部

其在mainActivity的布局文件中内容如下
```xml
<com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/navigation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        android:background="?android:attr/windowBackground"
        app:labelVisibilityMode="labeled"
        app:menu="@menu/menu_bottom_nav"
        app:layout_constraintBottom_toBottomOf="parent"
        android:layout_marginBottom="0dp"/>
```

其中特有属性<code>app:menu="@menu/menu_bottom_nav" </code>
用于指定底部菜单项,在此处引用名为menu_bottom_nav的xml文件,其中含有菜单的具体内容,如下所示
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/bottom_navigation_home"
        android:icon="@drawable/ic_launcher_foreground"
        android:title="test1"/>
    <item
        android:id="@+id/bottom_navigation_conn"
        android:icon="@drawable/ic_launcher_foreground"
        android:title="test3"/>
    <item
        android:id="@+id/bottom_navigation_find"
        android:icon="@drawable/ic_launcher_foreground"
        android:title="test2"/>
    <item
        android:id="@+id/bottom_navigation_me"
        android:icon="@drawable/ic_launcher_foreground"
        android:title="test4"/>
</menu>
```
在该文件中,先有一个menu节点,其中包含了四个item项,每一项代表一个菜单项,在每一个item项中可以做更为详细的配置,此处不赘述

有了底部布局之后,需要在mainActivity中绑定

先创建对象

<code>private BottomNavigationView bottomNavigationView;</code>

在onCreate方法中绑定

<code>bottomNavigationView = (BottomNavigationView) findViewById(R.id.navigation);</code>

至此,底部导航栏的显示效果实现.下一步创建四个页面的fragment嵌入mainActivity中

##### 2. 四个Fragment的创建

首先分别为四个Fragment创建布局,因为不需要关注详细内容,只需指定使用的布局即可
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="me" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

创建新类继承Fragment类,重写onCreateView,将布局绑定到fragment上即可
```java
public class MeFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.me_fragment, container, false);
    }
}
```
至此fragment创建完成

四个fragment互不相干,单个实现很容易,核心问题是,如何将四个fragment绑到mainActivity上,并能够实现翻页的操作

##### 3. ViewPager和适配器FragmentAdapter
ViewPager是一个很合适的解决方案,将其作为四个fragment的载体绑定到mainActivity上,不过要直接连接fragment与viewpager还不合适,中间需要adapter作为桥梁

所以先创建一个新类,继承自FragmentPagerAdapter类

创建构造方法接受外部传入的fragments数组,用于viewPager显示
```java
private Fragment[] fragments;
public MyFragmentPagerAdapter(@NonNull FragmentManager fm, Fragment[] fragments) {
    super(fm);
    this.fragments = fragments;
}
```
重写<code>getItem()</code>
```java
@Override
public Fragment getItem(int position) {
    return fragments[position % fragments.length];
}
```
重写<code>getCount()</code>
```java
@Override
public int getCount() {
    return fragments.length;
}
```

这两个方法告诉viewPager显示的fragment页数,以及每一页对应哪一个fragment

在mainActivity的布局文件中创建viewPager的布局
```xml
<androidx.viewpager.widget.ViewPager
    android:id="@+id/tab_pager"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1"
    android:overScrollMode="never"
    app:layout_constraintTop_toTopOf="parent" >
</androidx.viewpager.widget.ViewPager>
```

在mainActivity中创建ViewPager对象以及四个fragment对象
```java
private ViewPager viewPager;
private Fragment[] fragments={  //作为参数传递给adapter
    new HomeFragment(),
    new ConnFragment(),
    new FindFragment(),
    new MeFragment()
};
```

在onCreate方法中,绑定viewPager布局(),新建上述adapter对象,将viewpager的adapter设置为新建adapter对象

```java
MyFragmentPagerAdapter myFragmentPagerAdapter = new MyFragmentPagerAdapter(getSupportFragmentManager(),fragments);
viewPager = (ViewPager) findViewById(R.id.tab_pager);
viewPager.setOffscreenPageLimit(1);
viewPager.setAdapter(myFragmentPagerAdapter);
```

到这里,UI上的工作就完成了,下一步是页面点击切换,滑动切换的实现

##### 4.页面点击切换,滑动切换的实现
viewPager提供了监听页面切换的方法,bottomNavigation提供了底部菜单项的点击监听
主要问题是两个联动,点击能实现页面切换,页面切换能同时使底部导航栏的选中项更改

新建页面切换的监听对象,要重写三个方法

<code>onPageScrolled</code>,<code>onPageSelected</code>,<code>onPageScrollStateChanged</code>

<code>onPageSelected</code>是当前页面发生更改时会触发的,其余两个方法暂时无需考虑

当页面切换触发方法时,传入当前页面的position(第几位),获取底部导航栏控件,调用getMenu()获取菜单,getItem(position)获取指定菜单项,setChecked(true)将其设为选中
这就实现了页面更改时,底部导航栏选中项更改

```java
ViewPager.OnPageChangeListener onPageChangeListener = new ViewPager.OnPageChangeListener() {
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

    }
    @Override
    public void onPageSelected(int position) {
        bottomNavigationView.getMenu().getItem(position).setChecked(true);
    }

    @Override
    public void onPageScrollStateChanged(int state) {

    }
};
viewPager.addOnPageChangeListener(onPageChangeListener);    //绑定到该viewPager
```

新建底部导航栏的点击监听对象,重写<code>onNavigationItemSelected</code>方法,该方法在菜单项被点击时触发,传入被点击的菜单项

getItemId()获取菜单项的id,然后根据id判断选中的第几项,调用viewPager的setCurrentItem()方法将当前页面切换到选中菜单项对应的页面

```java
//监听底部点击
BottomNavigationView.OnNavigationItemSelectedListener onNavigationItemSelectedListener = new BottomNavigationView.OnNavigationItemSelectedListener() {
    @Override
    public boolean onNavigationItemSelected(@NonNull MenuItem item) {
    switch (item.getItemId()) {
        case R.id.bottom_navigation_home:
            viewPager.setCurrentItem(0);
            break;
        case R.id.bottom_navigation_conn:
            viewPager.setCurrentItem(1);
            break;
        case R.id.bottom_navigation_find:
            viewPager.setCurrentItem(2);
            break;
        case R.id.bottom_navigation_me:
            viewPager.setCurrentItem(3);
            break;
        default:
            break;
    }
    return false;
    }
};
bottomNavigationView.setOnNavigationItemSelectedListener(onNavigationItemSelectedListener); //绑定到该bottomNavigationView
```

> #### **运行界面展示**

![show](./pic/WechatDemo/res.gif ':size=20%')