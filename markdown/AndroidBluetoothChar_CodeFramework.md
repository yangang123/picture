@[TOC]

# ���
Android�Ĺ��������Ͽ���������������ҵ�����, ������ʹ��bluetoothͨ�ŵ�android������Կ����ҵĽ���,��Ҫ������������η��ͺͽ�������

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/picture/raw/master/AndroidBluetooth/Android-BluetoothChat.jpg" height="720" width="1280" > 
</div>

# ���沼��layout 
layout/activity_device_list.xml  : ��ʾ�����豸�б�
layout/activity_main.xml         : ���������
layout/device_name.xml           : �����豸���б�
layout/fragment_bluetooth_chat.xml : �������Ͱ�ť, ��������ӿڵĽ���
layout/message.xml                : ��Ϣ��ʽ
 
menu/main.xml                     : ����������"HIDE LOG/SHOW LOG"�˵�����
menu/blutooth_chat.xml            : ����ɨ�裬�����豸ѡ�񣬷��������˵�����

# code 

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/picture/raw/master/AndroidBluetooth/Android-BluetoothChat-studio.png" height="720" width="1280" > 
</div>

```java
BluetoothChatFragment.java  ���������ҳ���
BluetoothChatService.java   ���������շ����ݷ���
Constants.java     
DeviceListActivity.java     ��ʾ�����豸�б�ҳ���
MainActivity.java           ������

SampleActivityBase.java

Log.java
LogFragment.java  
LogNode.java  
LogView.java  
LogWrapper.java  
MessageOnlyLogFilter.java
```
## ���潻���߼�
```java
public class MainActivity extends SampleActivityBase
  ->protected void onCreate(Bundle savedInstanceState)
    ->BluetoothChatFragment fragment = new BluetoothChatFragment();
    -> transaction.replace(R.id.sample_content_fragment, fragment);                  //�����������촰�� 
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
            -> mChatService = new BluetoothChatService(getActivity(), mHandler);     //��������         
            -> public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {   //�����˵�
        inflater.inflate(R.menu.bluetooth_chat, menu);

            -> public boolean onOptionsItemSelected(MenuItem item) {
                case R.id.secure_connect_scan: {
                    Intent serverIntent = new Intent(getActivity(), DeviceListActivity.class); //����1����ͼ
                    startActivityForResult(serverIntent, REQUEST_CONNECT_DEVICE_SECURE);       //������activity�Ĵ�������
                case R.id.insecure_connect_scan: {
                case R.id.discoverable: {    
    }
```

##  MainActivity�Ĳ˵�ѡ��
```java
ʵ��1���˵��ӿڣ�ָ�������ļ���menu�ļ��е�main.xml
public boolean onCreateOptionsMenu(Menu menu)
    getMenuInflater().inflate(R.menu.main, menu);

ʵ�ֲ˵��л����û�����
public boolean onOptionsItemSelected(MenuItem item)    
  switch(item.getItemId()) {


## DeviceListActivity����˵��
```java
public class DeviceListActivity extends Activity
   -> protected void onCreate(Bundle savedInstanceState)
      -> setContentView(R.layout.activity_device_list);
      -> newDevicesListView.setOnItemClickListener(mDeviceClickListener);

private AdapterView.OnItemClickListener mDeviceClickListener
            = new AdapterView.OnItemClickListener() 
      ->setResult(Activity.RESULT_OK, intent);  //����activity�ķ���ֵ
      ->finish();                               //�˳���ǰactivity
```


## ������������

�������������
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

## ������������

��ʾ���յ�����
mConversationArrayAdapter.add(mConnectedDeviceName + ":  " + readMessage);
��ʾ���͵�����
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