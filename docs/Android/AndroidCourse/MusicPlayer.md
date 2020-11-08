> git地址: [MusicPlayer](https://github.com/ktKongTong/MusicPlayer)

> #### **功能说明**

实现基本的音乐播放器功能，从Android的/sdcard/Music路径下读取音乐文件，显示为列表<br>
在底部显示当前播放歌曲。<br>
实现上一首，下一首，播放，暂停以及对应的状态显示。<br>
如图所示

![UI](./pic/MusicPlayer/UI.jpg ':size=20%')

> #### **分析**

**歌曲列表显示**<br>
使用recyclerView<br>
列表的item包含音乐封面，音乐名称等信息，如下<br>
此处仅写专辑封面以及歌曲名字

![item](./pic/MusicPlayer/item.jpg ':size=40%')

**底部播放器控件**<br>
用于展示当前播放曲目信息，以及相应的功能按钮<br>
![bottomPlayer](./pic/MusicPlayer/bottomPlayer.jpg ':size=40%')

**音乐播放**<br>
使用服务以及mediaPlayer<br>
音乐播放服务与activity的界面交互使用广播<br>

> #### **功能实现**
##### 1. 音乐对象Music及单项布局
创建Music类，包含如下属性
- 音乐文件名称
- 音乐封面
- 作者
- 文件路径
- 时长
- 所属专辑
- 文件种类(MP3,FLAC等)
<br>在具体实现中，只使用了文件名称，封面，文件路径信息。

```java
public class Music implements Parcelable {
    private String name;
    private byte[] coverImage;
    private Type type;
    private String album;
    private String artist;
    private String Duration;
    private String filePath;
//setter、getter、construct、interface实现
}
```
Music用于将文件映射为实体类，有一个构造方法，传入文件路径 <br>
用MediaMetadataRetriever读取文件的信息，并转为相应的属性<br>

```java
public class Music implements Parcelable {
    // other...
    public Music(String filepath,String filename) {
        this.setFilePath(filepath+filename);
        if(filename.endsWith(".mp3")){
            this.setType(Type.MP3);
        }else if(filename.endsWith(".flac")){
            this.setType(Type.FLAC);
        }
        if(this.getType() == Type.MP3){
            this.setName(filename.substring(0,filename.length()-4));
        }else if(this.getType() == Type.FLAC){
            this.setName(filename.substring(0,filename.length()-5));
        }
        MediaMetadataRetriever metadataRetriever = new MediaMetadataRetriever();
        metadataRetriever.setDataSource(filepath+filename);
        byte[] embeddedPicture = metadataRetriever.getEmbeddedPicture();
        if(embeddedPicture != null){
            this.setCoverImage(BitmapFactory.decodeByteArray(embeddedPicture, 0, embeddedPicture.length));
        }else {
            this.setCoverImage((Bitmap) null);
        }
        this.setAlbum(metadataRetriever.extractMetadata(android.media.MediaMetadataRetriever.METADATA_KEY_ALBUM));
        this.setArtist(metadataRetriever.extractMetadata(android.media.MediaMetadataRetriever.METADATA_KEY_ARTIST));
        this.setDuration(metadataRetriever.extractMetadata(android.media.MediaMetadataRetriever.METADATA_KEY_DURATION));
    }
    //getter、setter
}
```
根据完整的的文件名称设置Type，Type是枚举类<br>
因为后续步骤中需要将Music作为数据使用广播传递,实现了Parcelable的接口<br>
```java
public class Music implements Parcelable {
    //other......
    protected Music(Parcel in) {
        name = in.readString();
        type = in.readParcelable(Type.class.getClassLoader());
        album = in.readString();
        artist = in.readString();
        Duration = in.readString();
        filePath = in.readString();
    }

    public static final Creator<Music> CREATOR = new Creator<Music>() {
        @Override
        public Music createFromParcel(Parcel in) {
            return new Music(in);
        }

        @Override
        public Music[] newArray(int size) {
            return new Music[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeString(name);
        parcel.writeParcelable(type, i);
        parcel.writeString(album);
        parcel.writeString(artist);
        parcel.writeString(Duration);
        parcel.writeString(filePath);
    }
    //setter、getter
}
```
内部枚举类也需要实现Parcelable<br>
不能序列化coverImage，这是Bitmap的byte数组，如果序列化会使数据过大传递时报TransactionTooLargeException异常，[官方文档](https://developer.android.google.cn/guide/components/activities/parcelables-and-bundles?hl=zh_cn#sdbp)中建议传递的数据不超过50kb<br>
目前所想到的就是传递文件路径，接受数据之后再次获取Bitmap<br>

![Parcelable](./pic/MusicPlayer/Parcelable.jpg ':size=40%')

为单个item创建布局，在layout文件夹中新建music_item.xml布局文件<br>
包含一个ImageView，TextView显示封面和歌曲名称<br>
在HomeFragment的布局中使用RecyclerView，item项使用music_item布局<br>
效果如下<br>
![item_layout](./pic/MusicPlayer/item_layout.jpg ':size=40%')

##### 2. MusicAdapter的创建
创建MusicAdapter，继承recyclerView的adapter<br>
创建构造方法，方便传入数据。<br>
重写onCreateViewHolder，onBindViewHolder，getItemCount方法，创建静态内部类ViewHolder。<br>
此处略过，创建方法以及步骤见 [recyclerView在wechat中的使用](recyclerViewInWhchatDemo.md)<br>

##### 3. MainActivity中创建recyclerView
在活动创建时，扫描/sdcard/Music下的mp3或flac文件，转为Music对象，加入musicList中，即为适配器显示的数据。<br>
固定步骤，创建布局管理器，recyclerView，新建适配器，为recyclerView设置适配器，实现点击事件<br>
```java
 void initRecyclerView(){
    //扫描文件
    File files = new File(Environment.getExternalStorageDirectory().getPath()+"/Music/");
    for (String str:files.list()){
        if(str.endsWith(".mp3")||str.endsWith(".flac")){
            Music music = new Music(Environment.getExternalStorageDirectory().getPath()+"/Music/",str);
            musicList.add(music);
        }
    }
    //绑定布局
    LinearLayoutManager layoutManager = new LinearLayoutManager(MainActivity.this);
    recyclerView.setLayoutManager(layoutManager);
    MusicAdapter musicAdapter = new MusicAdapter(musicList,MainActivity.this);
    recyclerView.setAdapter(musicAdapter);
    musicAdapter.setOnItemClickListener(this);
}

// recyclerView长按事件
    @Override
    public void onItemLongClick(View view, int pos) {
        Toast.makeText(MainActivity.this,String.valueOf(pos)+":ItemLongClick",Toast.LENGTH_SHORT).show();
    }
// recyclerView单击事件
    @Override
    public void onItemClick(View view, int pos) {
        Toast.makeText(MainActivity.this,String.valueOf(pos)+":ItemClick",Toast.LENGTH_SHORT).show();
        //活动跳转(播放详情页)
        //Intent intent = new Intent(view.getContext(), PlayActivity.class);
        //view.getContext().startActivity(intent);
        Intent broadIntent = new Intent("org.xr.action.CTL_ACTION");
        //发送广播，播放点击的新曲目
        broadIntent.putExtra("control",3);
        broadIntent.putExtra("index",pos);
        sendBroadcast(broadIntent);
    }
```

##### 4. 底部播放控件的布局

底部播放控件包含当前正播放的曲目信息以及上一首、下一首、播放/暂停的按钮<br>
封面使用ImageView，歌曲名称，上一首，下一首,播放暂停按钮均使用TextView(使用iconfont)<br>
具体布局如下
``` xml
<androidx.constraintlayout.widget.ConstraintLayout
    android:id="@+id/bottom_player"
    app:layout_constraintBottom_toBottomOf="parent"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    <!--封面-->
    <com.google.android.material.imageview.ShapeableImageView
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        android:id="@+id/music_cover"
        android:layout_width="50dp"
        android:layout_height="50dp"
        />
    <!--歌曲名称    -->
    <TextView
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toEndOf="@+id/music_cover"
        android:id="@+id/music_name"
        android:layout_marginTop="10dp"
        android:layout_marginStart="10dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:contentDescription="name"/>
    <!--上一首-->
    <TextView
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/music_play"
        android:id="@+id/music_pre"
        android:layout_width="50dp"
        android:textSize="30sp"
        android:gravity="center"
        android:layout_height="50dp"
        android:text="@string/icons"/>
    <!--开始/暂停-->
    <TextView
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/music_next"
        android:id="@+id/music_play"
        android:layout_width="50dp"
        android:textSize="30sp"
        android:gravity="center"
        android:layout_height="50dp"
        android:text="@string/icons" />
    <!--下一首-->
    <TextView
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        android:id="@+id/music_next"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:textSize="30sp"
        android:gravity="center"
        android:text="@string/icons" />
</androidx.constraintlayout.widget.ConstraintLayout>
```
效果如下<br>
![bottomPlayer](./pic/MusicPlayer/bottomPlayer.jpg ':size=40%')

##### 5.音乐服务于MainActivity的交互方式

![交互方式](./pic/MusicPlayer/交互.jpg ':size=100%')

##### 6. 音乐服务
实现具体的音乐播放事务，由MainActivity唤起，并传递过来播放列表的数据。<br>
创建MusicService，继承Service<br>
重写onBind，onCreate，onStartCommand方法<br>
onBind默认即可
```java
public class MusicService extends Service {
    //other
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
    //other
}
```
在OnstartCommand中，接受服务启动时MainActivity传过来的播放列表
```java
public class MusicService extends Service {
    //other
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Bundle args= intent.getBundleExtra("bundle");
        index = intent.getIntExtra("index",0);
        musicList = args.getParcelableArrayList("musicList");
        return super.onStartCommand(intent, flags, startId);
    }
    //other
}
```

在onCreate()中注册广播，与MainActivity交互，控制音乐的播放以及界面变化
```java
public class MusicService extends Service {
    @Override
    public void onCreate(){
        serviceReceiver = new MyReceiver();
        IntentFilter filter = new IntentFilter();
        filter.addAction(MainActivity.CTL_ACTION);
        registerReceiver(serviceReceiver, filter);
        //other
    }
}
```

广播接收器如下，接收control，index，playStatus，根据信号控制播放<br>
在最后根据播放状态，向MainActivity发送广播改变界面<br>
```java
public class MyReceiver extends BroadcastReceiver{
    @Override
    public void onReceive(final Context context, Intent intent){
        int control = intent.getIntExtra("control", -1);
        index = intent.getIntExtra("index",index);
        status = intent.getIntExtra("playStatus",status);
        switch (control){
            // 播放或暂停
            case 1:
                // 原来处于没有播放状态
                if (status == 0x11){
                    // 播放音乐
                    play(musicList.get(index));
                    status = 0x12;
                }
                // 原来处于播放状态
                else if (status == 0x12){
                    // 暂停
                    mPlayer.pause();
                    status = 0x13;
                }
                // 原来处于暂停状态
                else if (status == 0x13){
                    // 播放
                    mPlayer.start();
                    // 改变状态
                    status = 0x12;
                }
                break;
            // 停止声音
            case 2:
                // 如果原来正在播放或暂停
                if (status == 0x12 || status == 0x13){
                    // 停止播放
                    mPlayer.stop();
                    status = 0x11;
                }
            //上一首，下一首
            case 3:
                play(musicList.get(index));
                status = 0x12;
        }
        // 广播通知Activity更改图标、文本框
        Intent sendIntent = new Intent(MainActivity.UPDATE_ACTION);
        sendIntent.putExtra("playStatus", status);
        sendIntent.putExtra("index", index);
        // 发送广播，将被Activity组件中的BroadcastReceiver接收到
        sendBroadcast(sendIntent);
    }
}
```
在onCreate中实现MediaPlayer的播放完成事件，播放下一首
```java
public class MusicService extends Service {
    @Override
    public void onCreate(){
        //pther
        mPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener(){
            @Override
            public void onCompletion(MediaPlayer mp){
                index++;
                if (index+1 > musicList.size()){
                    index = 0;
                }
                currentSong = musicList.get(index);
                //发送广播通知Activity更改当前播放歌曲信息
                Intent sendIntent = new Intent(MainActivity.UPDATE_ACTION);
                sendIntent.putExtra("index", index);
                // 发送广播，将被Activity组件中的BroadcastReceiver接收到
                sendBroadcast(sendIntent);
                // 播放音乐
                play(musicList.get(index));
            }
        });
    }
}
```

##### 7. MainActivity中的广播
在活动创建时注册广播，启动音乐服务
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    PlayBroadcastReceiver playBroadcastReceiver = new PlayBroadcastReceiver();
    // 创建IntentFilter
    IntentFilter filter = new IntentFilter();
    // 指定BroadcastReceiver监听的Action
    filter.addAction(UPDATE_ACTION);
    // 注册BroadcastReceiver
    registerReceiver(playBroadcastReceiver, filter);
    Intent intent = new Intent(this, MusicService.class);
    Bundle args = new Bundle();
    args.putParcelableArrayList("musicList", musicList);
    intent.putExtra("bundle",args);
    intent.putExtra("index",index);
    // 启动后台Service
    startService(intent);
}
```
广播接收器如下，根据播放状态改变界面。
```java
//广播
class PlayBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent){
        // 获取Intent中的update消息，update代表播放状态
        int status = intent.getIntExtra("playStatus", -1);
        // 获取Intent中的current消息，current代表当前正在播放的歌曲
        index = intent.getIntExtra("index",index);
        currentSong = musicList.get(index);
        if (currentSong!=null){
            name.setText(currentSong.getName());
            coverImage.setImageBitmap(currentSong.getCoverImage());
        }
        switch (status){
            //未播放状态
            case 0x11:
                //若为播放状态，改为停止
                if(playButton.getText() == IconFont.PLAY.getValue()){
                    playButton.setText(IconFont.STOP.getValue());
                }
                status = 0x11;
                break;
            // 控制系统进入播放状态
            case 0x12:
                // 播放状态下设置使用暂停图标
                playButton.setText(IconFont.PLAY.getValue());
                // 设置当前状态
                status = 0x12;
                break;
            // 控制系统进入暂停状态
            case 0x13:
                // 暂停状态下设置使用播放图标
                playButton.setText(IconFont.STOP.getValue());
                // 设置当前状态
                status = 0x13;
                break;
        }
    }
}
```

MainActivity中在点击recyclerView 中的item时，通知音乐服务播放新曲目<br>
点击上一首，下一首，播放暂停按钮时，改变播放状态<br>
底部播放栏点击事件如下
```java
//    bottomPlayer点击事件
@Override
public void onClick(View view) {
    Intent intent = new Intent("org.xr.action.CTL_ACTION");
    switch (view.getId()){
        case R.id.music_pre:
            if(index>0){
                index--;
            }else{
                index = musicList.size()-1;
            }
            intent.putExtra("index", index);
            intent.putExtra("control", 3);
            break;
        case R.id.music_next:
            if(index+1<musicList.size()){
                index++;
            }else{
                index=0;
            }
            intent.putExtra("index", index);
            intent.putExtra("control", 3);
            break;
        case R.id.music_play:
            intent.putExtra("control", 1);
            intent.putExtra("index", index);
            break;
        case R.id.bottom_player:
            Intent intentBottom = new Intent(view.getContext(), PlayActivity.class);
            view.getContext().startActivity(intentBottom);
            break;
    }
    sendBroadcast(intent);
}
```
recyclerView的点击事件如下
```java
// recyclerView单击事件
@Override
public void onItemClick(View view, int pos) {
    Toast.makeText(MainActivity.this,String.valueOf(pos)+":ItemClick",Toast.LENGTH_SHORT).show();
//        Intent intent = new Intent(view.getContext(), PlayActivity.class);
//        view.getContext().startActivity(intent);
    Intent broadIntent = new Intent("org.xr.action.CTL_ACTION");
//      播放新曲目
    broadIntent.putExtra("control",3);
    broadIntent.putExtra("index",pos);
    sendBroadcast(broadIntent);
}
```
> #### **运行界面展示**

[result](./pic/MusicPlayer/result.mp4 ':include controls height=400px')
