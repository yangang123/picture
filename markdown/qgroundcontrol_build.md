qgroundcontrol3.2.0 

# ���
qgroundcontrol3.2.0��px4��������վ, С���������£��Լ�����Դ�����1��qground����վ��

# ��װ����

## ���뻷��
ubuntu : 16.04
qt: 5.9.0
qgroundcontorl: 3.2.0

## 1. qt��װ 
```
$ cd ~/
$ wget http://download.qt.io/official_releases/qt/5.9/5.9.0/qt-opensource-linux-x64-5.9.0.run
$ sudo chmod +x qt-opensource-linux-x64-5.9.0.run
$ ./qt-opensource-linux-x64-5.9.0.run
```

ע��ѡ��"Desktop_gcc_64-bit",�����������������

<div align="center">
<p>  ��װ </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/qt_5.9.0_install.png" height="720" width="1280" > 
</div>

<div align="center">
<p>  ����û��kit�Ĵ��� </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/no_qmake_Install.PNG" height="720" width="1280" > 
</div>

ж��qt��ʽ
```
yangang@ubuntu:~/work/tools/qt5.9.0$ ls
5.9             dist  Examples             Licenses         MaintenanceTool.dat  network.xml
components.xml  Docs  InstallationLog.txt  MaintenanceTool  MaintenanceTool.ini  Tools
yangang@ubuntu:~/work/tools/qt5.9.0$ ./MaintenanceTool
```
<!-- ![ͼƬ](./qgroundcontrol/qt_5.9.0_install.png)
![ͼƬ](./qgroundcontrol/no_qmake_Install.png) -->

## 2. ����qgroundcontrolԴ��
�汾�л���ע��Ҫ����git submodules����ͬ����ģ��
```
$ git clone https://github.com/mavlink/qgroundcontrol.git
$ git checkout v3.2.0 
$ sudo git submodule  init
$ sudo git submodule  update  
```

## 3. ����qgroundcontrol����

<!-- ![ͼƬ](./qgroundcontrol/build.png) -->

<div align="center">
<p>  ����û��kit�Ĵ��� </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/build.png" height="720" width="1280" > 
</div>

1. ����
   
Project ERROR: sdl development package not found

�������
```
yangang@ubuntu:~/work/tools/game-tools$ sudo apt-cache search "libSDL2*"
libsdl1.2-dev - Simple DirectMedia Layer development files
```
2. ����2 
error: gstreamer-video-1.0 development package not found

���������

�л�qgroundcontrol��3.2.0, �Ͳ��������

3. �������

<div align="center">
<p>  ������� </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/build_end.png" height="720" width="1280" > 
</div>


4. ����
   
����������������⣬����ͨ�� https://github.com/mavlink/qgroundcontrol.git��
issue���ҵ�һЩ�������

## ����qgroundcontrol

<div align="center">
<p>  ���� </p> 
<img src="https://github.com/yangang123/picture/raw/master/qgroundcontrol/run.png" height="720" width="1280" > 
</div>

# �ܽ�
����һֱ�ڵ���qt�汾��qgroundcontrol�汾��Ӧ�����⣬�����ҵ�������2���汾��ƥ��ġ�