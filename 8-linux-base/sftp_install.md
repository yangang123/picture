�ص�:
1��sftp �� ftp ���ż���һ�����﷨�͹��ܡ�
2��SFTP Ϊ SSH��һ���֣���һ�ִ��䵵���� Blogger �ŷ����İ�ȫ��ʽ��
3��SFTP����û�е������ػ����̣�������ʹ��sshd�ػ����̣��˿ں�Ĭ����22���������Ӧ�����Ӳ���
4��SFTP��ȫ�Էǳ���
5��SSH����Ѿ�����SFTP��ȫ�ļ�������ϵͳ

��װ����:

1. �鿴�Ƿ�װssh
ssh -version

2. û�з���ssh�Ͱ�װssh ������ʵ���ļ�������ǲ�Ҫ��װclient��
��װssh-client���sudo apt-get install openssh-client
��װssh-server���sudo apt-get install openssh-server   

3. ����ssh����
sudo /etc/init.d/ssh restart

4. �鿴�Ƿ������ɹ�
zht@zht-Ubuntu:~$ ps -e|grep ssh
 2151 ?        00:00:00 ssh-agent
 5313 ?        00:00:00 sshd

5. �����Ƿ�ɹ�

The authenticity of host '192.168.1.107 (192.168.1.107)' can't be established.
ECDSA key fingerprint is SHA256:f83tSawxo1U2Guyu6p5xvSmyfBCSUbJJsNXIbZKjnWQ.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.107' (ECDSA) to the list of known hosts.
yangang@192.168.1.107's password: 

������������롿

6. �ɹ���¼�Ľ���

Welcome to Ubuntu 15.10 (GNU/Linux 4.2.0-16-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

224 packages can be updated.
0 updates are security updates.


7.��flashFTP������
  1) �½�1������
  2) ѡ��Э�飬SFTP, ������ַ192.168.1.107, �û���yangang
     ����725807
