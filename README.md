# wechatDatabaseExtractFromSamsung
三星手机提取微信聊天数据
三星手机提取微信聊天数据的方法，无需root。
注意，暴力破解密码需要英伟达显卡，一小时内破解，无显卡可能要两天。

# 1. 安装USB驱动，通过S换机助手，备份微信软件至电脑。注意，选择不加密。
[三星USB驱动下载](https://developer.samsung.com/android-usb-driver) 

[S换机助手下载链接](https://www.samsung.com/cn/apps/smart-switch/)

# 2. 使用7-Zip解压提取数据库文件
找到备份路径下的 ``SM-G9730_20230113231240\APKFILE\com.tencent.mm.data``文件，右键=>7-Zip=>打开压缩包，找到``com.tencent.mm\r\MicroMsg\f5202199666edd0e02d05dfe03dc66d1\EnMicroMsg.db``文件，单独复制出来。此处无法获得微信UID，毕竟备份的有所缺失，因此采用暴力破解。

# 3. 下载所需文件
[SQLCipher-Password-Cracker](https://github.com/whiteblackitty/SQLCipher-Password-Cracker-OpenCL) 下载这个仓库，解压 `SQLCipher-Password-Cracker-OpenCL-master.zip`

[OpenSSL](http://slproweb.com/products/Win32OpenSSL.html) 下载Win64 OpenSSL v3.0.7 EXE `Win64OpenSSL-3_0_7.exe`

[sqlite-amalgamation](https://www.sqlite.org/download.html) 下载Source Code - `sqlite-amalgamation-3400100.zip`

[pysqlcipher3](https://github.com/rigglemania/pysqlcipher3) 下载这个仓库，解压 `pysqlcipher3-master.zip`

[sqlcipher](https://github.com/sqlcipher/sqlcipher) 下载这个仓库，解压 `sqlcipher-master`

# 4 安装pysqlcipher3

以下步骤中部分需要生成或修改的文件放在仓库中，可自取，不保证可行。

1 首先需要安装python 3.7，并且安装CUDA、Cudnn。装过TensorFlow的Anaconda环境都可以。在虚拟环境内运行`python -m pip install pyopencl`

2 然后安装OpenSSL，注意需要将OpenSSL的dll安装到系统目录：`C:\Program Files\OpenSSL-Win64`。

3 不要！将`sqlite-amalgamation-3400100.zip`内的四个文件，复制到`pysqlcipher3-master\amalgamation`文件夹。

应该：在sqlcipher-master目录下，打开`Developer Command Prompt of Visual Studio`，运行`nmake /f Makefile.msc sqlite3.c`，把`sqlite3.c`和`sqlite3.h`复制到`pysqlcipher3-master\amalgamation`文件夹。

4 修改 `pysqlcipher3\src\python3`目录下`connection.h, statement.h, util.h`三个文件中`#include "sqlcipher\sqlite3.h"`为 `#include "sqlite3.h"`。

5 修改 `pysqlcipher3`目录下`setup.py`中 "openssl = os.path..."为`openssl = r"C:\Program Files\OpenSSL-Win64"`，修改`openssl_conf = os.environ.get('OPENSSL_CONF')`为`openssl_conf = r"C:\Program Files\OpenSSL-Win64\bin\cnf\openssl.cnf"`，修改`openssl_lib_path = os.path.join(openssl, "lib")`为`openssl_lib_path = os.path.join(openssl, "lib\VC")`，修改`libeay32.lib`为 `libcrypto64MD.lib`,  并加入`ext.extra_link_args.append("libssl64MD.lib")`。

6 复制`C:\Program Files\OpenSSL-Win64`目录下`libcrypto-1_1-x64.dll 、libssl-1_1-x64.dll`文件到`SQLCipher-Password-Cracker-OpenCL-master`

7 `pysqlcipher3` 目录下虚拟环境内运行 ``python setup.py build_amalgamation`` ``python setup.py install``

# 5 运行破解
`SQLCipher-Password-Cracker-OpenCL-master` 目录下`Run.py`文件修改`Encrypted_DB_PATH="EnMicroMsg.db"`，把`EnMicroMsg.db`复制过来。

`SQLCipher-Password-Cracker-OpenCL-master` 目录下虚拟环境内运行`python Run.py 0`

跑了两次默认参数，提示未能成功破解密码。参考其他链接，可以适当调整判定逻辑和参数：

```python
in pbkdf2-sha1_aes-256-cbc.cl
if(((uint)(data[5] ^ iv[5])==0x40) && ((uint)(data[6] ^ iv[6])==0x20) && ((uint)(data[7] ^ iv[7])==0x20) && ((uint)(data[56] ^ iv[56])==0x00))
```
```python
in Run.py
OUTER_PASS_LENGTH = 4
c.execute("PRAGMA cipher_compatibility=3;") #new
c.execute("PRAGMA cipher_hmac_algorithm=HMAC_SHA1;") #new
```
最后在RTX 1660Ti上Brute Try completed after a total time of 47.4 min.

![在这里插入图片描述](https://img-blog.csdnimg.cn/53c2f89754214bb8923f20527fdfc1ff.jpeg#pic_center)
# 7 数据库结构
使用sqlcipher.exe读取数据库，主要的表是群聊名单`chatRoom`，聊天记录`message`，联系人名单`rcontact`，联系人标签`ContactLabel`，还有一些wallet、biz、img、voice之类的。
# Last 参考链接
https://zhuanlan.zhihu.com/p/123942610
https://zhuanlan.zhihu.com/p/164917107
https://www.jianshu.com/p/90224ab9cdf2
https://blog.csdn.net/muzhicihe/article/details/109902849
https://blog.csdn.net/wem603947175/article/details/103584228
