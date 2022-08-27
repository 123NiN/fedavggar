# fedavggar
1联邦学习训练：
运行：首先保证服务端和客户端在同一个局域网下
1）服务端:
1.安装python3.6、pip3，并运行以下依赖包
pip3 install -U MNN
pip3 install cherrypy
pip3 install ws4py
2.运行FedGAR/Fedserver/server.py
3）客户端：
4）数据和初始化模型通过命令行工具adb下载到Android设备本地
  Cd  FedGAR/data
  Adb push gar.snapshot.mnn /data/local/tmp/mnn/gar.snapshot.mnn
5)修改app/src/main/java/com/demo/MainActivity.java中的SERVER_URL为服务端的ip地址
6)连接android设备，并运行项目（必须和服务端在同一个局域网下才能正常运行）

2 垃圾分类Gui demo
确保模型文件和window_trash.py在一个路径内，run window_trash.py
3.垃圾分类 app demo
利用Android Studio 打开GarClassification文件，连接Android，编译并运行

  

