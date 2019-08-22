@[TOC]

# 资源
> androidBluetoothChar_CodeFramework.md
> 标题: 闫刚 androidBluetoothChar代码架构

# 简介
Android的官网查资料看到这个蓝牙聊天室的例子, 对于想使用bluetooth通信的android程序可以看下我的讲解,主要讲解蓝牙是如何发送和接收数据

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/3-android/resource/Android-BluetoothChat.jpg" height="720" width="1280" > 
</div>

# 界面布局layout 
layout/activity_device_list.xml  : 显示蓝牙设备列表
layout/activity_main.xml         : 界面主入口
layout/device_name.xml           : 蓝牙设备名列表
layout/fragment_bluetooth_chat.xml : 包括发送按钮, 内容输入接口的界面
layout/message.xml                : 消息格式
 
menu/main.xml                     : 主界面中有"HIDE LOG/SHOW LOG"菜单界面
menu/blutooth_chat.xml            : 蓝牙扫描，蓝牙设备选择，发送蓝牙菜单界面

# code 

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/3-android/resource/Android-BluetoothChat-studio.png" height="720" width="1280" > 
</div>

```java
BluetoothChatFragment.java  蓝牙聊天的页面层
BluetoothChatService.java   蓝牙聊天收发数据服务
Constants.java     
DeviceListActivity.java     显示蓝牙设备列表页面层
MainActivity.java           主界面

SampleActivityBase.java

Log.java
LogFragment.java  
LogNode.java  
LogView.java  
LogWrapper.java  
MessageOnlyLogFilter.java
```
## 界面交互逻辑
```java
public class MainActivity extends SampleActivityBase
  ->protected void onCreate(Bundle savedInstanceState)
    ->BluetoothChatFragment fragment = new BluetoothChatFragment();
    -> transaction.replace(R.id.sample_content_fragment, fragment);                  //启动蓝牙聊天窗口 
       -> public void onCreate(Bundle savedInstanceState) 
          -> mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
          ->onCreateView() 
            ->onViewCreated()
                ->mConversationView = (ListView) view.findViewById(R.id.in);
                ->mOutEditText = (EditText) view.findViewById(R.id.edit_text_out);
                ->mSendButton = (Button) view.findViewById(R.id.button_send);
       -> public void onStart() {
           ->   setupChat();
             -> mConversationArrayAdapter = new ArrayAdapter<String>(getActivity(), R.layout.message);
            ->  mConversationView.setAdapter(mConversationArrayAdapter);
            -> mSendButton.setOnClickListener(new View.OnClickListener() {
                ->  public void onClick(View v) {
                     ->    sendMessage(message);
            -> mChatService = new BluetoothChatService(getActivity(), mHandler);     //创建服务         
            -> public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {   //创建菜单
        inflater.inflate(R.menu.bluetooth_chat, menu);

            -> public boolean onOptionsItemSelected(MenuItem item) {
                case R.id.secure_connect_scan: {
                    Intent serverIntent = new Intent(getActivity(), DeviceListActivity.class); //创建1个意图
                    startActivityForResult(serverIntent, REQUEST_CONNECT_DEVICE_SECURE);       //在启动activity的传递请求
                case R.id.insecure_connect_scan: {
                case R.id.discoverable: {    
    }
```

##  MainActivity的菜单选择
```java
实现1个菜单接口，指定布局文件是menu文件夹的main.xml
public boolean onCreateOptionsMenu(Menu menu)
    getMenuInflater().inflate(R.menu.main, menu);

实现菜单切换的用户功能
public boolean onOptionsItemSelected(MenuItem item)    
  switch(item.getItemId()) {


## DeviceListActivity功能说明
```java
public class DeviceListActivity extends Activity
   -> protected void onCreate(Bundle savedInstanceState)
      -> setContentView(R.layout.activity_device_list);
      -> newDevicesListView.setOnItemClickListener(mDeviceClickListener);

private AdapterView.OnItemClickListener mDeviceClickListener
            = new AdapterView.OnItemClickListener() 
      ->setResult(Activity.RESULT_OK, intent);  //设置activity的返回值
      ->finish();                               //退出当前activity
```


## 蓝牙发送数据

定义输入输出流
```java
private final InputStream mmInStream;
private final OutputStream mmOutStream;

tmpIn = socket.getInputStream();
tmpOut = socket.getOutputStream();

mSendButton.setOnClickListener(new View.OnClickListener() {
  public void onClick(View v) {
          -> sendMessage(message);
          -> mChatService.write(send);
            -> mmOutStream.write(buffer);
```

## 蓝牙接收数据

显示接收的数据
mConversationArrayAdapter.add(mConnectedDeviceName + ":  " + readMessage);
显示发送的数据
mConversationArrayAdapter.add("Me:  " + writeMessage);

```java
  private class ConnectedThread extends Thread {
  public void run() {
  bytes = mmInStream.read(buffer);
       mHandler.obtainMessage(Constants.MESSAGE_READ, bytes, -1, buffer).sendToTarget();                     
        -> private final Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            case Constants.MESSAGE_WRITE:
              mConversationArrayAdapter.add("Me:  " + writeMessage);
            case Constants.MESSAGE_READ:
              mConversationArrayAdapter.add(mConnectedDeviceName + ":  " + readMessage);
```