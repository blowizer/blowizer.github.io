---
layout:     post
title:      "Loadrunner 测试"
subtitle:   "奇门遁甲"
date:       2022-06-09
author:     "blowizer"
image:      "img/post-bg-coffee.jpeg"
tags: 
    - 测试
categories: Notes
---

# 创建脚本
## 创建
![创建socket script](https://img-blog.csdnimg.cn/755315fc6b354e5188c22021469069ac.png#pic_center)
## 录制
- 截取报文
![在这里插入图片描述](https://img-blog.csdnimg.cn/32d8310350104421aeced18fbca2be6f.png)
- 
- 替换空格和换行![](https://img-blog.csdnimg.cn/6438a7e269654c49ac6e6df09280b55c.png)
- 创建socket 
![](https://img-blog.csdnimg.cn/cf42d18fb16c4c3db1c1e9772f280e24.png)
- loadrunner gennerator 中ctrl+r 录制发送报文 生成如下内容
> 参数化流水号,右健所选内容,命名为seqNum_6 格式为%lu6
![在这里插入图片描述](https://img-blog.csdnimg.cn/f303cd697db94445a5a7a9cc7208b4bd.png)
在Action.c 中添加代码
```c
/*********************************************************************
 * Created by Windows Sockets Recorder
 *
 * Created on: Wed Jun 16 11:06:06
 *********************************************************************/

#include "lrs.h"


Action()
{
   
    int rc=0;

	char *Data, *p,*q;
	int Size;
	char index[2];
	char hexStr[20]={0};
	char strByte[20]={0};
	unsigned char hexByte[20]={0};
	unsigned char *unitptr;
	char rSeqNum[20] = {0};
	int len = 0;
	int i,j,k,ret;
	int z = 0;

 
    //设定开始事务
 
    len = strlen(lr_eval_string("{seqNum_6}"));
  
    p = lr_eval_string("{seqNum_6}") ;
    ret = str2hex(rSeqNum,6,p);
    lr_output_message("str=%s, len = %d",p, ret);
    lr_output_message("str=%s, len = %d",rSeqNum, ret);
    HexStrTobyte(p,hexByte,&len);
    for (i =0;i < len ; i++){
    	lr_output_message("str=%02x,",hexByte[i]);
    }
    q = hexStr;
  
   lr_output_message("str=%s",p);
   len = strlen(p);
   for (j=0; j<len/2 ; j++){
   		memcpy(q,"\\x",2);  
   		memcpy(q+2, p,2);  		
   		p+=2;
   		q+=4;	
   }
   
  lr_output_message("str=%s",hexStr);
  lr_save_string(hexStr,"seqN");
  
 #if 1
    lrs_startup(257);
    //创建socketS
    lr_start_transaction("create_socket");
	//
    rc = lrs_create_socket("socket0", "TCP", "RemoteHost=127.0.0.1:32001",  LrsLastArg);
	//判断套接字创建是否成功
 
	if (rc==0){
	
		lr_end_transaction("create_socket", LR_PASS); 
    	lr_output_message("Socket was successfully send ");
		
	}
	else{
		lr_end_transaction("create_socket", LR_FAIL); 
    	lr_output_message("An error occurred while closing the socket, Error Code: %d", rc);
    	
	}
	//判断socket是否链接成功的事务，0表示创建成功 
	lr_start_transaction("send_socket");   
	//发送安全会话建立请求buf0为在data.ws中定义的发送变量     
	lrs_send("socket0", "buf0", LrsLastArg);
 	//设置连接超时时间为1秒
	lrs_set_recv_timeout(45,0);

	if (rc==0){
	
		lr_end_transaction("send_socket", LR_PASS); 
    	lr_output_message("Socket was successfully send ");
		
	}
	else{
		lr_end_transaction("send_socket", LR_FAIL); 
    	lr_output_message("An error occurred while send the socket, Error Code: %d", rc);
    	
	}
	//设置接收超时时间为1秒
    //lrs_set_recv_timeout2(45,0);
	lr_output_message("Socket was successfully send");

	//接收消息，存放在buf1中，buf1是在data.ws中定义的接收数组，注意数组长度一定要大于等于实际接收长度
    lrs_receive("socket0", "buf1", LrsLastArg); 
    //把Socket最后接收的字节数组，长度放在recvlen中，内容放在recvbuf中
    //lrs_get_last_received_buffer("socket0",&recvbuf,&recvlen);
	
	 lr_start_transaction("received_socket");
	//lrs_save_param("socket0", NULL, "F39", 39, 2);
	//lr_message ("F39: %s", lr_eval_string("{F39}"));
    lrs_get_last_received_buffer("socket0",&Data,&Size);
	//lr_message("报文内容:%s",Data);
	//lr_message("应答码第39位:%x",Data[39]);
	//lr_message("应答码第40位:%x",Data[40]);
		//获取流水号000000
    if(Size == 116){
	
		lr_output_message("接收报文长度 116 == %d", Size);
		lr_end_transaction("received_socket", LR_PASS);
	}else{
		
		lr_output_message("接收报文长度不够116 != %d", Size);
		lr_end_transaction("received_socket", LR_FAIL);
	}

	//memcpy(F39,lr_eval_string("{F39}"),2);
	//lr_output_message("recvbuf: %X",recvbuf);
	 lr_start_transaction("ITMP-9121-A0-33");	
	//获取流水号000000
    //if(Data[32]==0x45&&Data[33]==0x34){
    memcpy(rSeqNum, Data+31,3);
    if(!memcmp(Data+31,hexByte,3)){
    	
		lr_log_message("流水号位：%3.3s",Data+32);
		lr_output_message("交易流水一致");
		lr_end_transaction("ITMP-9121-A0-33", LR_PASS);
	}else{
		lr_log_message("流水号4位：%x",Data[33]);
		lr_output_message("交易流水不一致");
		lr_end_transaction("ITMP-9121-A0-33", LR_FAIL);
	}
	 lr_start_transaction("ITMP-9121-A0");
	
	//获取应答码000000
    if(Data[39]==0x41&&Data[40]==0x30){
		lr_output_message("交易成功");
		lr_end_transaction("ITMP-9121-A0", LR_PASS);
	}else{
		
		lr_output_message("交易失败");
		lr_end_transaction("ITMP-9121-A0", LR_FAIL); 
	}
	//关闭socket
	lrs_close_socket("socket0");
	//lrs_free_buffer(lseqNum);
	#endif
	return 0;
} 
void test_printf5(char *str, int length)
{
    int iii;
    printf("the str is:");
    for (iii = 0; iii < length; iii++)
    {
        printf("%02x ", str[iii]);
    }
    printf("\n");
}
/*str to hex,
字符串转换为16进制
利用snprintf 函数
int snprintf(char* dest_str,size_t size,const char* format,...);
功能:将可变参数以format的格式转换到dest_str中，并可以用%s打印出来。
1、dest_str:目标字符串，即输出字符串
2、size: 转换字符控制长度。
(1) 如果格式化后的字符串长度 < size，则将此字符串全部复制到str中，并给其后添加一个字符串结束符('\0')；
(2) 如果格式化后的字符串长度 >= size，则只将其中的(size-1)个字符复制到str中，并给其后添加一个字符串结束符('\0')，返回值为欲写入的字符串长度。
3、format :转换为某个格式，例如：%2d,%04X,%02X。
4、...:需要转换的字符串
*/
int str2hex(char *out_string, int length, char *in_string)
{
    int index;
    char *fmt = "%02x";

    for (index = 0; index < length; index++)
    {
        snprintf((char *)&out_string[index << 1], 3, fmt, in_string[index]);
    }
    return (length << 1);
}


int HexStrTobyte(char *str, unsigned char *out, unsigned int *outlen)
{
    char *p = str;
    char high = 0, low = 0;
    int tmplen = strlen(p), cnt = 0;
    tmplen = strlen(p);
    while (cnt < (tmplen / 2))
    {
        high = ((*p > '9') && ((*p <= 'F') || (*p <= 'f'))) ? *p - 48 - 7 : *p - 48;
        low = (*(++p) > '9' && ((*p <= 'F') || (*p <= 'f'))) ? *(p)-48 - 7 : *(p)-48;
        out[cnt] = ((high & 0x0f) << 4 | (low & 0x0f));
        p++;
        cnt++;
    }
    if (tmplen % 2 != 0)
        out[cnt] = ((*p > '9') && ((*p <= 'F') || (*p <= 'f'))) ? *p - 48 - 7 : *p - 48;

    if (outlen != NULL)
        *outlen = tmplen / 2 + tmplen % 2;
    return tmplen / 2 + tmplen % 2;
}
int byteToHexStr(unsigned char byte_arr[], int arr_len, char *HexStr, int *HexStrLen)
{
    int i, index = 0;
    for (i = 0; i < arr_len; i++)
    {
        char hex1;
        char hex2;
        int value = byte_arr[i];
        int v1 = value / 16;
        int v2 = value % 16;
        if (v1 >= 0 && v1 <= 9)
            hex1 = (char)(48 + v1);
        else
            hex1 = (char)(55 + v1);
        if (v2 >= 0 && v2 <= 9)
            hex2 = (char)(48 + v2);
        else
            hex2 = (char)(55 + v2);
        if (*HexStrLen <= i)
        {
            return -1;
        }
        HexStr[index++] = hex1;
        HexStr[index++] = hex2;
    }
    *HexStrLen = index;
    return 0;
}



```