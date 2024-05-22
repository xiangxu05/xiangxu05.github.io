---
layout: post
title: 嵌入式｜基于ESP32实现的串口服务器
categories: [Embedded]
description: 基于ESP32实现的串口服务器，启动于APSTA模式，同时包括一个设置页面，可以对ESP32相关参数进行设置。
keywords: Embedded, WiFi, WebServer
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

手上有一块ESP32板子，这段时间以来组装设备为了更便携一些，上位机与设备、设备与设备间通讯均用到了串口服务器，但是成品串口服务器总有接口限制，传输上线限制等问题，同时也没法做到需要的多口模数转换等功能因此自己摸索了一下，模数转换功能还没做。使用了Arduino环境，ESP_dev_module库。实现代码在地址：[xiangxu05/ESP32_WIFI_Serial: 基于ESP32实现的串口服务器](https://github.com/xiangxu05/ESP32_Wifi_Serial)

## 0x00 WIFI设置

设置ESP32的WIFI，主要用到的就是<WiFi.h>中的方法，以下对使用到的方法进行简单介绍。WiFi.h头文件中定义了一个*WiFiClass*类，并存在一个定义了的全局变量*extern WiFiClass WiFi*。这个类继承了*WiFiGenericClass、WiFiSTAClass、WiFiScanClass、WiFiAPClass*，因此可以使用这几个类的方法，设置一个APSTA模式的WiFi便使用了其中的方法。

使用前记得包含该库。

```c++
#include <WiFi.h>
```

1. 设置WIFI模式

   ```c++
   static bool mode(wifi_mode_t);
   
   @param wifi_mode_t           WIFI模式，模式列表:
           WIFI_OFF             不作定义
           WIFI_STA             定义为STA模式,相当于无线终端，不接受无线的接入
           WIFI_AP              定义为AP模式,提供无线接入服务，允许其它设备通过WIFI接入
           WIFI_AP_STA          定义为STA和AP共存模式
   
   @return                     成功返回1,失败返回0
   ```

   要使用APSTA模式，因此需要将WIFI设置为WIFI_AP_STA：

   ```c++
   WiFi.mode(WIFI_AP_STA);
   ```

2. WIFI-AP设置

   ```c++
   bool softAP(const char* ssid, const char* passphrase = NULL, int channel = 1, int ssid_hidden = 0, int max_connection = 4, bool ftm_responder = false);
   
    @param ssid                         指向SSID字符串的指针(最大63字节)。
    @param passphrase                   可选，默认 = NULL，用于WPA2加密的WIFI密码，最少8字节，开放WIFI可以用NULL。
    @param channel                      可选，默认 = 1，WIFI频道号码，1-13。
    @param ssid_hidden                  可选，默认 = 0，网络隐藏(0 = 广播SSID,1 = 隐藏SSID)。
    @param max_connection               可选，默认 = 4， 最大同时连接的客户端, 1 - 4。
    @ftm_responder                      可选，默认 = false，一种高速传输模式可以d在高带宽且低延迟的情况下与另一个 ESP32 设备进行通信。
    @return                             成功返回true,不成功返回false
   ```

   可以通过这个方法来对ESP32的AP参数进行设置。

   ```c++
   WiFi.softAP(ssid_name, security_key, 3, 1);
   ```

   接着还需要对AP的内部IP、掩码、网关进行设置。

   ```c++
   bool softAPConfig(IPAddress local_ip, IPAddress gateway, IPAddress subnet, IPAddress dhcp_lease_start = (uint32_t) 0);
   
   @param local_ip                配置AP的IP地址
   @param gateway                 配置AP的网关IP地址
   @param subnet                  配置AP的子网掩码
   @dhcp_lease_start              配置AP的DHCP租约开始
   @return                        成功返回true,失败返回false
   ```

   此时仅对AP的local_ip、gateway、subnet进行配置。

   ```c++
   WiFi.softAPConfig(lan_ip, lan_gateway, lan_subnet);
   ```

3. WIFI-STA设置

   ```c++
   wl_status_t begin(const char* wpa2_ssid, wpa2_auth_method_t method, const char* wpa2_identity=NULL, const char* wpa2_username=NULL, const char *wpa2_password=NULL, const char* ca_pem=NULL, const char* client_crt=NULL, const char* client_key=NULL, int32_t channel=0, const uint8_t* bssid=0, bool connect=true);
   
   @param wpa2_ssid				用于连接的WiFi网络的SSID。
   @param method					使用的WPA2认证方法。
   @param wpa2_identity			可选,用于WPA2 Enterprise认证的身份。
   @param wpa2_username			可选,用于WPA2 Enterprise认证的用户名。
   @param wpa2_password			可选,用于WPA2 Enterprise认证的密码。
   @param ca_pem					可选,CA证书，用于验证服务器的证书。
   @param client_crt				可选,客户端证书。
   @param client_key				可选,客户端私钥。
   @param channel					可选,指定要连接的WiFi信道，默认为0（自动选择）。
   @param bssid					可选,指定要连接的接入点的BSSID（MAC地址）。
   @param connect					可选,是否立即连接到指定的网络，默认为true。
   @return							返回连接状态，类型为 wl_status_t。
   
   wl_status_t begin(const char* ssid, const char *passphrase = NULL, int32_t channel = 0, const uint8_t* bssid = NULL, bool connect = true);
   
   @param ssid						用于连接的WiFi网络的SSID。
   @param passphrase				可选,用于连接WiFi网络的密码。
   @param channel					可选,指定要连接的WiFi信道，默认为0（自动选择）。
   @param bssid					可选,指定要连接的接入点的BSSID（MAC地址）。
   @param connect					可选,是否立即连接到指定的网络，默认为true。
   @return							返回连接状态，类型为 wl_status_t。
   
   wl_status_t begin(char* ssid, char *passphrase = NULL, int32_t channel = 0, const uint8_t* bssid = NULL, bool connect = true);
   
   @param ssid						用于连接的WiFi网络的SSID。
   @param passphrase				可选,用于连接WiFi网络的密码。
   @param channel					可选,指定要连接的WiFi信道，默认为0（自动选择）。
   @param bssid					可选,指定要连接的接入点的BSSID（MAC地址）。
   @param connect					可选,是否立即连接到指定的网络，默认为true。
   @return							返回连接状态，类型为 wl_status_t。
   
   wl_status_t begin();			无参数，该方法尝试连接到上一次配置的WiFi网络。
   
   @return							返回连接状态，类型为 wl_status_t。
   ```

   此时，仅传入对应的ssid与password就行。

   ```c++
   WiFi.begin(ssid, password);
   ```

   至此，可以写一个配置函数如下：

   ```c++
   int WIFI_SET_APSTA(String ssid_name,String security_key,String ssid,String password,String local_ip,String subnet,String gateway)
   
   {
    long iRetval = -1; //标志是否设置成功，这边仅对AP启动判断，因为AP模式关乎到是否能够进行连接设置。
    WiFi.mode(WIFI_AP_STA);//设置为AP_STA模式
   
    IPAddress lan_gateway,lan_subnet,lan_ip;
    lan_ip.fromString(local_ip);
    lan_gateway.fromString(gateway);
    lan_subnet.fromString(subnet);
    //这边IPAddress的定义也可以用如下方法，参数间用逗号隔开
    //IPAddress local_IP(192,168,1,1);
    //IPAddress subnet(255,255,255,0);
    //IPAddress gateway(192,168,1,1);
       
    WiFi.softAPConfig(lan_ip, lan_gateway, lan_subnet);
    WiFi.softAP(ssid_name, security_key, 3, 1);//对AP进行配置并启动
       
    iRetval = WiFi.softAP(ssid_name, security_key);
       
    //输出提示
    if (iRetval) {
   ​    Serial.println("Soft AP started successfully.");
   ​    Serial.print("SSID: ");
   ​    Serial.println(ssid_name);
   ​    Serial.print("IP Address: ");
   ​    Serial.println(WiFi.softAPIP());
     } else {
   ​    Serial.println("Failed to start Soft AP.");
   ​    return WIFIAPUNBUILED;
     }
   
     // 连接到WiFi网络
     WiFi.begin(ssid, password);
     Serial.println("Connecting to WiFi");
     while (WiFi.status() != WL_CONNECTED) {
   ​    Serial.print(".");
   ​    delay(500);
     }
   
     Serial.println();
     Serial.print("WiFi connected. Local IP address: ");
     Serial.println(WiFi.localIP());
     return iRetval;
   
   }
   ```

4. WiFiServer

   要作为TCP Server使用，就需要使用到WiFiServer类，因此我们需要首先定义一个该对象。

   ```c++
   WiFiServer server;
   ```

   首先启动该对象。

   ```c++
   void begin(uint16_t port=0);
   
   @param port						可选，服务器监听的端口号，默认为0。如果设置为0，服务器将选择一个默认端口。
   @return							无返回值。
   
   void begin(uint16_t port, int reuse_enable);
   
   @param port						服务器监听的端口号。
   @param  reuse_enable			是否启用端口重用，通常用于服务器在重启时可以立即绑定到同一端口。非零值表示启用，零值表示禁用。
   @return							无返回值。
   ```

   简单的，此时仅传入一个开放端口。

   ```c++
   server.begin((uint16_t)server_port.toInt());
   ```

   该对象有一个方法会用到。

   ```c++
   WiFiClient available();
   
   @return							返回一个WiFiClient类型
   ```

5. WiFiClient

   使用到了三个方法。

   ```c++
   int available();
   
   @return							可用返回1,不可用返回0
   
   uint8_t connected();
   
   @return							返回 1 表示当前 WiFiClient 对象与服务器或客户端保持连接。返回 0 表示连接已经断开。
   
   int read();
   
   @return							返回一个收到的数据，不能收到数返回-1。
   ```

   至此，可以实现一个监听串口与TCP端口的方法。

   ```c++
   static void TCP_Server_user()
   
   {
    String RxBuff;
    String txBuff;
   
    if(connected == 0){
     Serial.println("\nwaiting for connect...");
     connected = 1;
    }
   
    client=server.available();
    
    if(client)
    {
     Serial.println("get client,welcome!");
     
     while(client.connected()||client.available()||Serial.available())
     {
      if(client.available())
      { 
   ​    while(client.available()){
   ​     char c=client.read();
   ​     RxBuff +=c;
   ​    }
   
   ​    if(RxBuff == "AT_DigitalUp"){
   
   ​     digitalWrite(18, HIGH);
   ​     digitalWrite(19, HIGH);
   
   ​    }else if(RxBuff == "AT_DigitalDown"){//这部分是额外的设备控制，可以删掉。
   
   ​     digitalWrite(18, LOW);
   ​     digitalWrite(19, LOW);
   
   ​    }else{
   
   ​     Serial.print(RxBuff);
   
   ​    }
   ​    RxBuff="";
      }
   
      if(Serial.available()){
   ​    while(Serial.available()){
   
   ​     char c=Serial.read();
   ​     txBuff+=c;
   
   ​    }
   
   ​    if(txBuff == "AT_DigitalUp"){
   
   ​     digitalWrite(18, HIGH);
   ​     digitalWrite(19, HIGH);
   
   ​    }else if(txBuff == "AT_DigitalDown"){
   
   ​     digitalWrite(18, LOW);
   ​     digitalWrite(19, LOW);
   
   ​    }
   ​    else{
   
   ​     client.print(txBuff);
   
   ​    }
   ​    txBuff="";
      }
     }
   
     client.stop();
     connected = 0;
     Serial.println("no clinet now.");
    }
   }
   ```

## 0x01 NVS非易失寄存器

对于传统的单片机来说我们如果要固化保存小批量的数据的话通常会使用EEPROM，在Arduino core for the ESP32中也有相关的功能。不过对于ESP32来说官方还提供了一种叫做 Preferences 的功能，这个功能也可以用来固化保存数据，并且使用上比EEPROM更加方便。

为了实现对WIFI以及TCP监听端口设置，因此需要一个网页来进行，同时网页将更改存储于NVS的参数。ESP32官方在Flash上建立了一个叫做nvs的分区，而Preferences功能就是建立在该分区上的。Arduino core for the ESP32中默认分区（ Partition Scheme: “Default 4MB with spiffs (1.2MB APP /1.5MB SPIFFS)” ）情况下nvs分区的大小为 20480 字节，实际可存放的数据大小要小于这个值（ 单个数据来说最大为496K或者97%的nvs分区大小 ）。

Preferences中数据以键值对（key - value）的方式存储。在键值对之上还有一层命名空间（namespace），不同命名空间中可以有相同的键名存在。在Preferences中命名空间和键名均为字符串，并且长度不大于15个字节。

使用前记得包含该库。

```c++
#include <Preferences.h>
```

Preferences支持多种存储的数据类型，为了统一格式因此我们均使用String类型存储，功能实现用到了以下几个方法。

```c++
bool begin(const char *name, bool readOnly);

@name: 用于标识命名空间的字符串。这个命名空间将包含一组相关的键值对。
@readOnly: 如果设置为 `true`，则以只读模式打开首选项；如果设置为 `false`，则以读写模式打开首选项。
@return: 成功返回 `true`，失败返回 `false`。

void clear();

无参数。这个方法会清除当前命名空间中的所有键值对。
@return: 无返回值。

void end();

无参数。这个方法会结束对当前命名空间的操作，释放相关资源。
@return: 无返回值。

String getString(const char* key, const String& defaultValue = String());

@key: 要读取的字符串键。
@defaultValue: （可选）如果指定的键不存在，返回的默认字符串值。
@return: 返回与键关联的字符串值。如果键不存在，则返回 `defaultValue`。

bool putString(const char* key, const String& value);

@key: 要写入的字符串键。
@value: 要写入的字符串值。
@return: 成功返回 `true`，失败返回 `false`。
```

因此在单片机启动时，做了如下操作以实现默认值启动WIFI。

```c++
Preferences preferences; //全局变量，不在setup()中
// 打开 NVS
 preferences.begin("settings", false);

 String ssid_name = preferences.getString("SSID_NAME", "ESP_TCPserverV1");
 String security_key = preferences.getString("SECURITY_KEY", "12345678.");
 String sta_ssid = preferences.getString("ssid", "znlhsys_2.4G");
 String sta_password = preferences.getString("password", "znlhsysbistu");
 String local_ip = preferences.getString("LOCAL_IP", "192.168.1.1");
 String subnet = preferences.getString("SUBNET", "255.255.255.0");
 String gateway = preferences.getString("GATEWAY", "192.168.1.1");
 String server_port = preferences.getString("SERVERPORT", "8899");

 // 关闭 NVS
 preferences.end();

 WIFI_SET_APSTA(ssid_name,security_key,sta_ssid,sta_password,local_ip,subnet,gateway)
```

后续仅需要修改这些键值对，即可在设备重启时修改WIFI参数。

## 0x02 WebServer

ESP32提供了一个简便使用的WebServer，包含于头文件<WebServer.h>中，因此使用时记得包含库。

```c++
#include <WebServer.h>
```

该对象存在以下几个方法供使用。

```c++
WebServer webserver(uint16_t port);

@port: 用于初始化 `WebServer` 对象的端口号。服务器将在该端口上监听HTTP请求。
@return: 无返回值（这是构造函数，用于创建 `WebServer` 对象）。

void on(const Uri &uri, THandlerFunction handler);

@uri: 用于匹配请求路径的字符串。例如，`"/"` 表示根路径。
@handler: 与该路径关联的处理函数。当该路径收到HTTP请求时，将调用此函数。
@return: 无返回值。

void onNotFound(THandlerFunction fn);

@fn: 处理未找到页面的函数。当请求的路径未被任何处理函数匹配时，将调用此函数。
@return: 无返回值。

void begin();

无参数。这个方法启动Web服务器，开始监听传入的HTTP请求。
@return: 无返回值。

void handleClient();

无参数。这个方法处理传入的客户端请求。应在 `loop()` 函数中周期性地调用，以确保服务器能够响应客户端请求。
@return: 无返回值。

void send(int code, const char* content_type, const String& content);

@code: HTTP状态码（例如，200表示成功，404表示未找到）。
@content_type: 响应内容的MIME类型（例如，`"text/html"`，`"application/json"`）。
@content: 要发送的响应内容。
@return: 无返回值。
```

特别的是该对象返回给Client的页面以String类型输入，因此需要将HTML文件以字符串类型定义。

实现的思路则是，首先定义一个根路径，处理逻辑是访问时读取NVS存储器中的参数，并返回页面。提交时转到/submit页面，将发送来的参数存入存储器，并重新指向根目录。

因此对WebServer设置。

```c++
 webserver.on("/", handleRoot);
 webserver.on("/submit", handleSubmit);
 webserver.onNotFound([](){webserver.send(200,"text/html;charset=utf-8","没有找到页面！");});
 webserver.begin();
```

以下是处理函数：

```c++
void handleRoot(){

 preferences.begin("settings", false);
 String sta_ssid = preferences.getString("ssid", "znlhsys_2.4G");
 String sta_password = preferences.getString("password", "znlhsysbistu");
 String ssid_name = preferences.getString("SSID_NAME", "ESP_TCPserverV1");
 String security_key = preferences.getString("SECURITY_KEY", "12345678.");
 String local_ip = preferences.getString("LOCAL_IP", "192.168.1.1");
 String subnet = preferences.getString("SUBNET", "255.255.255.0");
 String gateway = preferences.getString("GATEWAY", "192.168.1.1");
 String server_port = preferences.getString("SERVERPORT", "8899");
 preferences.end();

 // 格式化HTML页面
 const int bufferSize = 4096;
 char html[bufferSize];

 // 使用 snprintf 格式化HTML内容
 snprintf(html, bufferSize, root_html,
​      sta_ssid.c_str(), sta_password.c_str(), ssid_name.c_str(), security_key.c_str(),
​      local_ip.c_str(), subnet.c_str(), gateway.c_str(), server_port.c_str());
 webserver.send(200, "text/html", html);

}

void handleSubmit() {

 if (webserver.hasArg("ssid") && webserver.hasArg("password") && webserver.hasArg("apSSID") &&
   webserver.hasArg("apPassword") && webserver.hasArg("localIP") && webserver.hasArg("subnetMask") &&
   webserver.hasArg("gateWay") && webserver.hasArg("tcpPort")) {
  
  preferences.begin("settings", false);
  // 存储每个参数到Preferences
  preferences.putString("ssid", webserver.arg("ssid"));
  preferences.putString("password", webserver.arg("password"));
  preferences.putString("SSID_NAME", webserver.arg("apSSID"));
  preferences.putString("SECURITY_KEY", webserver.arg("apPassword"));
  preferences.putString("LOCAL_IP", webserver.arg("localIP"));
  preferences.putString("SUBNET", webserver.arg("subnetMask"));
  preferences.putString("GATEWAY", webserver.arg("gateWay"));
  preferences.putString("SERVERPORT", webserver.arg("tcpPort"));
  preferences.end();

  // 重定向回根页面
  webserver.sendHeader("Location", "/");
  webserver.send(303);

 } else {

  webserver.send(400, "text/plain", "Missing required parameters");

 }


}
```



## 0x03 循环

为了实现既能应对串口、TCP端口消息，又能解决网页的显示，因此在loop函数中分别调用了两个handle。

```c++
void loop()
{
	TCP_Server_user();
  	webserver.handleClient();
}
```

## 0x04 小结

以上，就实现了一个简单的串口服务器，同时这个方法还存在如下问题：

- 对于串口通讯来说，还少了一个设置通讯速率的参数，实现的串口端口也较少。
- 多端口对多TCP连接的逻辑还没实现。
- 直接在loop函数中使用两个handle，在这个逻辑里会导致TCP连接后，网页不响应的问题。
- 两个功能在同一个方法中实现，效率不高，可以引入freertos，这个本来也要做的，在代码中也可以看到相关注释，暂时先这样后续再完善。
