---
title: java 之 JNA
date: 2018-10-15 19:00:12
index_img:
- /images/java/jna.jpg
tags: 
- jna
categories:
- Java
---

我们知道JAVA调用C和C++ 通常用是JNI，JNI虽然好用但是对于JAVA程序猿并不是很有友好，有更多的学习成本。对于java程序猿来说JNA的学习成本就低的多。在工程中只要引入一个jar包就可以非常方便的使用。对于官方的一些样例这里我也就不在重复了。主要还是从我自己这边使用遇到一些问题说起。官方入门的一些请移步[官方文档](https://github.com/java-native-access/jna)。

不过在开始之前一定一定要了解的是类型映射关系。也就是在C或者C++ 中的类型在JAVA中对应的类型是什么？直接上表：

| C基本类型 |    长度     |  Java类型  |       Windows类型       |
| :-------: | :---------: | :--------: | :---------------------: |
|   char    |   8位整型   |    byte    |       BYTE、TCHAR       |
|   short   |  16位整型   |   short    |          WORD           |
|  wchar_t  | 16/32位字符 |    char    |          TCHAR          |
|    int    |  32位整型   |    int     |          DWORD          |
|    int    |   布尔值    |  boolean   |          BOOL           |
|   long    | 32/64位整型 | NativeLong |          LONG           |
| long long |  64位整型   |    long    |         __int64         |
|   float   | 32位浮点型  |   float    |                         |
|  double   | 64位浮点型  |   double   |                         |
|   char*   |   C字符串   |   String   |         LPTCSTR         |
|   void*   |    指针     |  Pointer   | LPVOID、HANDLE、LP*XXX* |

#### 开发过程

我这边接到一个另外一个部门提供的一个dll库和头文件，需要使用JNA的方式进行开发使用。提供一个NVSSDK.dll 和其他的一些依赖的dll库和头文件。

##### 定义一个接口和编写相应的结构体映射

定义一个类编写要使用dll库中的方法和对应的结构体，具体可参考相关的头文件。简易是和头文件对应着看。

```java
package com.megvii.camera.proxy;

import com.sun.jna.Library;
import com.sun.jna.Pointer;
import com.sun.jna.Structure;
import com.sun.jna.Structure.ByValue;
import com.sun.jna.ptr.IntByReference;
import com.sun.jna.win32.StdCallLibrary.StdCallCallback;
import com.sun.jna.Callback;

public interface NVSSDK extends Library {
  public static class PicTime extends Structure {

        public int uiYear;
        public int uiMonth;
        public int uiDay;
        public int uiWeek;
        public int uiHour;
        public int uiMinute;
        public int uiSecondsr;
        public int uiMilliseconds;
    }

    public static class PicData extends Structure {
        public PicData(Pointer pointer) {
            super(pointer);
            useMemory(pointer);
            read();
        }

        public PicTime tPicTime;
        public int iDataLen;
        public Pointer pcPicData;
    }
  public static class FacePicStream extends Structure {
        public FacePicStream(Pointer pointer) {
            super(pointer);
            useMemory(pointer);
            read();
        }
        public int iStructLen;
        public int iSizeOfFull;        
        public Pointer tFullData;
        public int iFaceCount;
        public int iSizeOfFace;        
        public Pointer[] tPicData = new Pointer[32];
        public int iFaceFrameId;       
        public PicTime tNewPicTime;
    }
   public static interface NET_PICSTREAM_NOTIFY extends StdCallCallback {
        int PicDataNotify(int _ulID, int _lCommand, Pointer _tInfo, int _iLen,
                          Pointer _lpUserData);
    }
	 public static class NetPicPara extends Structure {
        public int iStructLen;                //Structure length
        public int iChannelNo;
        public NET_PICSTREAM_NOTIFY cbkPicStreamNotify;
        public Pointer pvUser;
    }
    int NetClient_StartRecvNetPicStream(int _iLogonID, NetPicPara _ptPara, int _iBufLen, IntByReference _puiRecvID);

```

上面继承Structure的是C中的结构体，继承StdCallCallback的是C中回调方法，NetClient_StartRecvNetPicStream是C中我们要调用的方法。

在另外一个类中加载动态库

```java
package com.megvii.camera.proxy;

import com.sun.jna.Native;
import com.sun.jna.ptr.IntByReference;
public class NetClient {
    private static NVSSDK nvssdk;
    private static NetClient netClient;
    private NetClient() {
    }

    public static NetClient getInstance() {
        if (netClient == null) {
            synchronized (NetClient.class) {
                if (netClient == null) {
                    netClient = new NetClient();
                    nvssdk = (NVSSDK) Native.loadLibrary("NVSSDK",
                            NVSSDK.class);
                }
            }
        }
        return netClient;
    }
   //开始接收图片流
    public int StartRecvNetPicStream(int _iLogonID, NVSSDK.NetPicPara _ptPara, int _iBufLen, IntByReference _puiRecvID) {
        return nvssdk.NetClient_StartRecvNetPicStream(_iLogonID, _ptPara, _iBufLen, _puiRecvID);
    }
}
```

##### 相关的头文件

```c
typedef struct tagFacePicStream
{
	int 			iStructLen;			//Structure length
	int				iSizeOfFull;		//The size of strcut PicData
	PicData*		ptFullData;			//The data of full screen
	int				iFaceCount;			//The current frame detects the number of face
	int				iSizeOfFace;		//The size of strcut FacePicData
	FacePicData*	ptFaceData[108];
	int				iFaceFrameId;		//The face jpeg frame no
	PicTime			tNewPicTime;		//The picture capture time, ptFullData contain time is out of date
} FacePicStream, *pFacePicStream;

typedef struct tagSnapPicData
{
	int 		iSnapType;		//Snap type
	int 		iWidth;			//Picture wide
	int 		iHeight;		//Picture high
	int			iSize;			//The size of strcut PicData
	PicData*	ptPicData;
} SnapPicData, *pSnapPicData;

typedef struct tagPicTime
{
	unsigned int uiYear;
	unsigned int uiMonth;
	unsigned int uiDay;
	unsigned int uiWeek;
	unsigned int uiHour;
	unsigned int uiMinute;
	unsigned int uiSecondsr;
	unsigned int uiMilliseconds;
} PicTime, *pPicTime;
//回调
typedef int (__stdcall *NET_PICSTREAM_NOTIFY)(unsigned int _uiRecvID, long _lCommand, void* _pvBuf, int _iBufLen, void* _pvUser);
typedef struct tagNetPicPara
{
	int 					iStructLen;				//Structure length
	int						iChannelNo;
	NET_PICSTREAM_NOTIFY	cbkPicStreamNotify;
	void*					pvUser;
	int						iPicType;				//Client request picture stream type.Structured proprietary(This field is not sent or sent 0: indicates that the server is adaptively sent and sent based on the current configuration.)
													//bit0£ºFace picture stream 
													//bit1£ºpedestrian 
													//bit2:plate number
													//bit3:motor vehicles
													//bit4:Non-motor vehicle
} NetPicPara, *pNetPicPara;
//方法
int __stdcall NetClient_StartRecvNetPicStream(int _iLogonID, NetPicPara* _ptPara, int _iBufLen, unsigned int* _puiRecvID);
```

##### 使用过程

需要留意的是FacePicStream和PicData的构造方法，这个方法是和指针转换时使用的，如果没有这么写的话Java是无法获取到C中指针的值的。以下的回调方法中回调了一个指针，将**指针转换成头文件中对应的结构体**。

```java 
private void SetPicQueue(final C4SCamera camera) {
        NVSSDK.NetPicPara picPara = new NVSSDK.NetPicPara();
        picPara.iChannelNo = 0;
        picPara.iStructLen = picPara.size();
        picPara.cbkPicStreamNotify = new NVSSDK.NET_PICSTREAM_NOTIFY() {
            public int PicDataNotify(int _ulID, int _lCommand, Pointer _tInfo, int _iLen, Pointer _lpUserData) {
                if (_tInfo == null || NVSSDK.NET_PICSTREAM_CMD_FACE != _lCommand) {
                    return 0;
                }
                if (!camera.capture.get()) {
                    return 0;
                }
                NVSSDK.FacePicStream picStream = new NVSSDK.FacePicStream(_tInfo);
                if (picStream.tFullData != null) {
                    NVSSDK.PicData fullData = new NVSSDK.PicData(picStream.tFullData);
                    if (fullData != null && fullData.iDataLen > 0 && camera.capture.get()) {
                        byte[] data = fullData.pcPicData.getByteArray(0, fullData.iDataLen);
                        FaceInfo faceInfo = new FaceInfo();
                        faceInfo.faceData = data;
                        faceInfo.cameraIP = camera.ip;
                        faceInfo.cameraAlias = camera.alias;
                        camera.captureQueue.offer(faceInfo);
                    }
                }
                return 0;
            }
        };
        IntByReference relsize = new IntByReference();
        int iRet = NetClient.getInstance().StartRecvNetPicStream(camera.logonId, picPara, picPara.size(), relsize);
    }
```

#### 遇到的问题

1. **java.lang.UnsatisfiedLinkError: Unable to load library 'NVSSDK':**

   这个错误是无法找到对应的dll库，也就是我前面提到的NVSSDK.dll。出现这个情况的有以下三种可能

   - jdk版本的位数和dll库的位数不一致。比如你的jdk版本是32位的，但是你的dll库是64位，就会出现这个错误，反过来也一样。解决办法是更换同样位数的库或者jdk

   - 还有一种你的环境变量中的path没有包含你要用的dll库路径。在JNA中查找动态库的顺序是，Java虚拟机中的jna.library.path，然后是Java虚机的java.library.path，最后才是系统环境变量下path。所以只要在这个三个路径只要有一个路径下有包含你的dll库路径就行。

     ```java
     System.setProperty("jna.library.path", dllPath);
     ```

2. 在使用过程中指针和结构体之间的相互转换，发现值没有复制到Java过来。

   - 结构体的构造方法,添加useMemory(pointer);和read();

     ```java
       public static class FacePicStream extends Structure{
     		public FacePicStream(Pointer pointer) {
                 super(pointer);
                 useMemory(pointer);
                 read();
             }    
       }
     ```

   - 使用

     ```java
     Pointer _tInfo;
     NVSSDK.FacePicStream picStream = new NVSSDK.FacePicStream(_tInfo);
     ```
   
3. Java中的Structure和C结构体中的顺序写错了。如果是同样类型的不会出现编译错误，但是取到的值准确，类型不同则出现编译错误。

   