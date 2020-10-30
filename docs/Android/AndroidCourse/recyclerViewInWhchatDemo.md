> git地址: [WechatDemo](https://github.com/ktKongTong/WechatDemo)
> #### **功能说明**

在微信的基本界面下放入消息列表，并一些特定的效果
长按消息item的置顶，取消置顶，标为已读/未读，以及删除功能的实现。

> #### **分析**

**Android现有版本的wechat消息界面主要有以下几个要素**
- 消息列表
- 顶部工具栏
- 下拉小程序界面

暂时不考虑小程序界面和顶部工具栏。

**消息列表的item中主要有以下几个元素**

- 头像
- 消息名称
- 最后一条消息内容
- 最后消息时间
- 消息红点提醒
- 消息免打扰的提示

如图所示

![消息item(已置顶)](./pic/recyclerViewInWhchatDemo/消息item(已置顶).jpg ':size=40%')
![消息item(未置顶)](./pic/recyclerViewInWhchatDemo/消息item(未置顶).jpg ':size=40%')

每个item点按会进入新Activity显示对应的消息，长按会根据item的属性显示不同的菜单项。

**具体的菜单项如下(未考虑公众号，服务号等item)**
- 删除
- 置顶聊天
- 取消置顶聊天
- 标为未读
- 标为已读

如图所示

![菜单置顶](./pic/recyclerViewInWhchatDemo/菜单置顶.jpg ':size=80x100')
![菜单未置顶](./pic/recyclerViewInWhchatDemo/菜单未置顶.jpg ':size=80x100')

使用recyclerview实现消息列表，使用PopupMenu实现长按菜单

> #### **功能实现**

##### 1. 消息对象News及单项布局

消息列表使用recyclerview实现，可以认为里面的item对应一个对象。

因此创建News类，包含如下属性
- 消息接收者(包含头像，名称)
- 是否置顶
- 是否已读
- 是否免打扰
- 最后消息时间
- 最后消息内容

```java
public class News {
    // 消息id
    private String newsId;
    // 发送者
    private Conn sender;
    // 接收者
    private Conn receiver;
    //是否未读
    private boolean isNews;
    //是否置顶
    private boolean isTop;
    //是否免打扰
    private boolean isDisturb;
    // 最后通信时间
    private Date newsTime;
    //setter or getter
}
```

为单个item创建布局，在layout文件夹中新建home_item.xml布局文件
头像，消息提示小红点，免打扰提示使用ShapeableImageView，通信时间，名称，通信内容使用TextView
采用约束布局，效果如下图

![home_item](./pic/recyclerViewInWhchatDemo/home_item.jpg ':size=40%')

在HomeFragment的布局中使用RecyclerView，item项使用home_item布局
```xml
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/home_recycler_view"
        android:layout_width="match_parent"
        app:layout_constraintTop_toTopOf="parent"
        android:layout_height="wrap_content"/>
```
##### 2. 适配器HomeRecyclerViewAdapter创建

随后创建适配器HomeRecyclerViewAdapter，继承RecyclerView的Adapter

创建构造方法，方便传入数据。


重写onCreateViewHolder，onBindViewHolder，getItemCount方法，创建静态内部类ViewHolder。

静态内部类ViewHolder用于创建视图，布局管理器调用适配器的onCreateViewHolder()方法，随后onBindViewHolder()将视图持有者绑定到相应数据。

静态内部类ViewHolder如下，创建布局，找到布局中的那些控件。
```java
static class ViewHolder extends RecyclerView.ViewHolder{
    View newsItem;
    TextView newsName;
    TextView newsTime;
    TextView newsContent;
    ImageView newsAvatar;
    ImageView newsDot;
    ImageView newsNoDisturb;
    public ViewHolder(@NonNull View itemView) {
        super(itemView);
        newsItem = itemView.findViewById(R.id.home_item);
        newsName = itemView.findViewById(R.id.news_name);
        newsTime = itemView.findViewById(R.id.news_time);
        newsContent = itemView.findViewById(R.id.news_content);
        newsAvatar = itemView.findViewById(R.id.news_avatar);
        newsDot = itemView.findViewById(R.id.news_dot);
        newsNoDisturb = itemView.findViewById(R.id.news_no_disturb);
    }
}
```

为适配器创建构造方法，传入数据
```java
public class HomeRecyclerViewAdapter extends RecyclerView.Adapter<HomeRecyclerViewAdapter.ViewHolder>{
    private List<News> newsList = new ArrayList<>();
    private Context context;
    //构造方法传入数据
    public HomeRecyclerViewAdapter(List<News> newsList,Context context){
        this.newsList=newsList;
        this.context = context;
    }
    //other....
}
```
##### 3. onCreateViewHolder()为item创建布局

重写的onCreateViewHolder()方法如下，这个方法在为每一个item创建布局时执行，可用于实现不同项的使用不同布局。

```java
public class HomeRecyclerViewAdapter extends RecyclerView.Adapter<HomeRecyclerViewAdapter.ViewHolder>{
    //other......
    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.home_item,parent,false);
        ViewHolder viewHolder = new ViewHolder(view);
        return viewHolder;
    }
    //other......
}
```
##### 4.onBindViewHolder()绑定数据与视图

重写的onBindViewHolder()方法如下，这个方法将创建的item视图与数据绑定,并且可以为item设置点击，长按等事件。

在这个方法中，根据传入的position获取list中的News对象，然后将对象的属性绑定到视图上，并根据相关的属性设置消息红点，消息免打扰的可见性

这一步是数据绑定
```java
public class HomeRecyclerViewAdapter extends RecyclerView.Adapter<HomeRecyclerViewAdapter.ViewHolder>{
    //other......
    @Override
    public void onBindViewHolder(@NonNull final ViewHolder holder, final int position) {
        //绑定数据
        News news = this.newsList.get(position);
        Glide.with(context)
                .load(news.getReceiver().getAvatar())
                .into(holder.newsAvatar);
        holder.newsTime.setText(news.getNewsTime().toString());
        holder.newsName.setText(news.getReceiver().getName());
        holder.newsContent.setText(news.getNewsContent());
        // 是否置顶，更改背景颜色
        if(news.isTop()){
            holder.newsItem.setBackgroundColor(Color.GRAY);
        }else{
            holder.newsItem.setBackgroundColor(Color.WHITE);
        }
        // 是否已读，小红点
        if(news.isNews()){
            holder.newsDot.setVisibility(View.VISIBLE);
        }else{
            holder.newsDot.setVisibility(View.INVISIBLE);
        }
        // 是否免打扰,免打扰图标
        if(news.isDisturb()){
            holder.newsNoDisturb.setVisibility(View.VISIBLE);
        }else{
            holder.newsNoDisturb.setVisibility(View.INVISIBLE);
        }
        //监听事件
        // ......
    }
    //other......
}
```

到此处，已经可以根据对象的属性，正确显示item的内容。

##### 5.onBindViewHolder()事件处理接口

同样在onBindViewHolder()中，添加item的点击事件监听，如下，但是具体的事件处理并没有放在此处，而是通过接口放在了HomeFragment中实现
```java
public class HomeRecyclerViewAdapter extends RecyclerView.Adapter<HomeRecyclerViewAdapter.ViewHolder>{
    private OnItemClickListener onItemClickListener;
    //    点击/长按监听接口
    public interface OnItemClickListener{
        void onItemLongClick(View view, int pos);
        void onItemClick(View view, int pos);
    }
    public void setOnItemClickListener(OnItemClickListener onItemClickListener) {
        this.onItemClickListener = onItemClickListener;
    }
    //other......
    @Override
    public void onBindViewHolder(@NonNull final ViewHolder holder, final int position) {
        //绑定数据
        //other......
        // 监听单击事件
        holder.itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                onItemClickListener.onItemClick(holder.itemView,position);
            }
        });
        // 监听长按事件
        holder.itemView.setOnLongClickListener(new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View view) {
                onItemClickListener.onItemLongClick(holder.itemView,position);
                return false;
            }
        });
    }
}
```
##### 6.getItemCount()重写

还有一项重写getItemCount()，如下
```java
public class HomeRecyclerViewAdapter extends RecyclerView.Adapter<HomeRecyclerViewAdapter.ViewHolder>{
    //other......
    @Override
    public int getItemCount() {
        return this.newsList.size();
    }
    //other......
}
```

至此，适配器的基本工作已经完成。

下一步就是在fragment的onCreateView方法中，创建布局管理器，绑定适配器，放入数据，并将上一步的未实现的点击事件处理实现。

##### 7.fragment中创建recyclerView

手动创建对象List<News>并添加数据
```java
//上述的一系列操作均在该方法中实现
 void initRecyclerView(){
    final List<News> newsList=new ArrayList<>();
    //获取联系人数据
    Conn conn1 = new Conn("name1eeeeeeeeeeeeee","https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=3072531546,2743940875&fm=26&gp=0.jpg");
    Conn conn2 = new Conn("name2eeeeeeeeeeeeee","https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=3072531546,2743940875&fm=26&gp=0.jpg");
    Conn conn3 = new Conn("name3eeeeeeeeeeeeee","https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=3072531546,2743940875&fm=26&gp=0.jpg");
    Conn conn4 = new Conn("name4eeeeeeeeeeeeee","https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=3072531546,2743940875&fm=26&gp=0.jpg");
    News news1 =new News(conn1,conn1, new Date(1604048567),true);
    News news2 =new News(conn2,conn2,new Date(1604048568),false);
    News news3 =new News(conn3,conn3,new Date(1604048569),true);
    News news4 =new News(conn4,conn4,new Date(1604048590),false);
    newsList.add(news1);
    newsList.add(news2);
    newsList.add(news3);
    newsList.add(news4);
    //other......
 }

 获取RecyclerView控件
```java
void initRecyclerView(){
    //获取数据
    //other......
    //获取recyclerView，创建布局管理器，为recyclerView设置布局管理器，创建上一步中实现的适配器，为recyclerView设置适配器
    recyclerView=view.findViewById(R.id.home_recycler_view);
    LinearLayoutManager layoutManager = new LinearLayoutManager(getActivity());
    recyclerView.setLayoutManager(layoutManager);
    final HomeRecyclerViewAdapter homeRecyclerViewAdapter = new HomeRecyclerViewAdapter(newsList,getActivity());
    recyclerView.setAdapter(homeRecyclerViewAdapter);
    //other......
    //点击事件处理
}
```
至此只剩下Adapter中留下的点击事件处理未实现
##### 8.recyclerView中item的单击事件处理

单击事件使用Intent转入聊天详情页面。
```java
void initRecyclerView(){
    //other.....
    //点击事件处理
    homeRecyclerViewAdapter.setOnItemClickListener(new HomeRecyclerViewAdapter.OnItemClickListener() {
        @Override
        public void onItemLongClick( View view, final int pos) {
            //长按事件处理
        }
        @Override
        public void onItemClick(View view, int pos) {
            Intent intent=new Intent(view.getContext(), SendMessageActivity.class);
            view.getContext().startActivity(intent);
        }
    }
}
```
##### 9.recyclerView中item的长按事件处理

核心问题是长按事件的处理，长按菜单项使用PopupMenu实现

长按事件有
- 删除
- 置顶
- 取消置顶
- 标为未读
- 标为已读

在资源目录menu下创建菜单menu3，内容如下

![menuContent](./pic/recyclerViewInWhchatDemo/menuContent.jpg ':size=40%')

创建PopupMenu，绑定布局，获取长按的对象，根据对象的属性显示菜单项。
```java
@Override
public void onItemLongClick( View view, final int pos) {
    PopupMenu popupMenu = new PopupMenu(view.getContext(),view);
    popupMenu.getMenuInflater().inflate(R.menu.menu3,popupMenu.getMenu());
    News news = newsList.get(pos);
    //是否置顶
    if(news.isTop()){
        popupMenu.getMenu().getItem(1).setVisible(false);
        popupMenu.getMenu().getItem(2).setVisible(true);
    }else {
        popupMenu.getMenu().getItem(1).setVisible(true);
        popupMenu.getMenu().getItem(2).setVisible(false);
    }
    //是否是新消息(标为已读，未读)
    if(news.isNews()){
        popupMenu.getMenu().getItem(3).setVisible(false);
        popupMenu.getMenu().getItem(4).setVisible(true);
    }else {
        popupMenu.getMenu().getItem(3).setVisible(true);
        popupMenu.getMenu().getItem(4).setVisible(false);
    }
    //other......
    //点击某个菜单项的事件处理
}
```
##### 10.popupMenu中菜单项单击事件处理

对于删除菜单项，首先从list中删除item对象，随后更新视图

对于标为未读/标为已读，从list中取对象，更新对象属性，新对象替换原list中对象。更新视图

对于置顶菜单项，从list中取出对象，并从list中删除，更新对象属性，插入list的首部。更新视图

对于取消置顶项，要考虑这样一个问题。消息列表的排列是按时间有序排列的。因此取出置顶项之后，要从第一个未置顶项(News静态成员，topCount)开始比对，放到时间顺序正确的位置，插入列表，随后更新视图。

视图更新使用notifyItemRanged()
```java
//PopupMenu点击事件
@Override
public void onItemLongClick( View view, final int pos) {
    //菜单显示
    //other......
    //PopupMenu点击事件
    popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
        @Override
        public boolean onMenuItemClick(MenuItem item) {
            switch(item.getItemId()){
                case R.id.delete:
                    newsList.remove(pos);
                    homeRecyclerViewAdapter.notifyItemRangeChanged(pos,newsList.size()-pos);
                    Toast.makeText(view.getContext(),"WeChat:delete",Toast.LENGTH_SHORT).show();
                    break;
                case R.id.to_top:
                    News topNews = newsList.get(pos);
                    News.setTopCount(News.getTopCount()+1);
                    topNews.setTop(true);
                    newsList.remove(pos);
                    newsList.add(0,topNews);
                    homeRecyclerViewAdapter.notifyItemRangeChanged(0,pos+1);
                    Toast.makeText(view.getContext(),"WeChat:toTop",Toast.LENGTH_SHORT).show();
                    break;
                case R.id.cancel_to_top:
                    News cancelTopNews = newsList.get(pos);
                    cancelTopNews.setTop(false);
                    newsList.remove(pos);
                    News.setTopCount(News.getTopCount()-1);
                    int i = News.getTopCount();
                    while (i>=0){
                        // 剩余需要比对的数量
                        if(newsList.size()-i<=0){
                            newsList.add(cancelTopNews);
                            break;
                        }
                        if(newsList.get(i).getNewsTime().getTime()>cancelTopNews.getNewsTime().getTime()){
                            newsList.add(i,cancelTopNews);
                            break;
                        }
                        i++;
                    }
                    homeRecyclerViewAdapter.notifyItemRangeChanged(pos,i-pos+1);
                    Toast.makeText(view.getContext(),"WeChat:cancelToTop",Toast.LENGTH_SHORT).show();
                    break;
                case R.id.sign_read:
                    News signReadNews = newsList.get(pos);
                    signReadNews.setNews(false);
                    newsList.set(pos,signReadNews);
                    homeRecyclerViewAdapter.notifyItemRangeChanged(pos,1);
                    Toast.makeText(view.getContext(),"WeChat:signRead",Toast.LENGTH_SHORT).show();
                    break;
                case R.id.sign_unread:
                    News signUnreadNews = newsList.get(pos);
                    signUnreadNews.setNews(true);
                    newsList.set(pos,signUnreadNews);
                    homeRecyclerViewAdapter.notifyItemRangeChanged(pos,1);
                    Toast.makeText(view.getContext(),"WeChat:signUnread",Toast.LENGTH_SHORT).show();
                    break;
            }
            return false;
        }
    });
    //菜单显示
    popupMenu.show();
```

至此功能全部实现。

> #### **运行界面展示**

![show](./pic/recyclerViewInWhchatDemo/result.gif ':size=20%')
