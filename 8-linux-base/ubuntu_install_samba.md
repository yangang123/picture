��ubuntu�°�װsamba������

һ. samba�İ�װ:

sudo apt-get install samba

��. ��������Ŀ¼:

mkdir /home/yangang(�û���)/share
sudo chmod 777 /home/user(�û���)/share

��. ����Samba�����ļ�:

1. �������е������ļ�

sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak

2. �޸��������ļ�

sudo gedit /etc/samba/smb.conf

��smb.conf������

[share]
      path = /home/user(�û�)/share     //���洴����samba����·��
      available = yes
      browsealbe = yes
      public = yes
      writable = yes

 
��. ����samba������
sudo /etc/init.d/samba restart

����ʹ��

���Ե�windows������ipʹ���ˣ����ļ��д����� "\\" + "Ubuntu������ip
ʾ��: 192.168.136.129