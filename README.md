# 树熊最佳实践
1. 查询数据列表
2. 文件上传
3. EasyUI
4. Jquery AutoComplete
5. 表单验证(后端)

#一些通用约定
1. 除非必要, 否则使用有符号数.
2. 任何报文均有包头(包括响应报文), ACK除外.
3. 除0x开头的, 其他数字均代表10进制数

#容错机制
1. 系统机错误:发现失败, 若干次失败后(如send返回-1, 可配置), 则重新发起三次握手.
2. 服务器响应出错: 若干次未收到服务器响应(可配置), 重新发起三次握手.

#交互
1. 探针在上电重启后会向服务器发送握手报文.
2. 探针每隔一定时间会向服务器发送数据报文.

#数据包列表
|序号|数据包|命令|发起方|接收方|描述|
|--|--|--|--|--|--|
|3|更新探针配置请求包|0x66|探针|服务端|更新配置和上传报文合在一起了(上传频率一样)|
|4|更新探针配置应答包|103|服务端|探针|	
|5|握手报文请求包|0x68|探针|服务端	
|6|握手报文响应包|105|服务端|探针|

#包头
|Bytes|数据类型|	字段名|描述|值|
|--|--|--|--|--|
|2|	Short|	magic|	树熊探针协议识别码|	0xefd1|
|1|	Byte|	code|	探针报文类型识别码数据报文:见数据包列表||	
|1	|Byte|	subCode|	子识别码|	|
|2	|Short|	version|	协议版本号|	0x0001|
|2	|Short	|length|	报文总长度(包头+包体)||	
|2	|Short	|venderId|	设备厂商ID十进制(即factoryId)|	1001|
|2	|Short|	productId|	厂商产品ID|	1|
|12|	Char(12)|	sn|	探针序列号十进制字符串	||
|6|	Byte(6)|	probeMac|	探针mac	||
|2|	Short|	reserve|	保留字(四字节对齐)||

##更新探针配置&上传报文请求包(102)

|Bytes|数据类型|字段名|描述|值|
|--|--|--|--|--|
|8|Long|configVersionTimestamp|"探针设备的配置版本号(用时间戳表示),如果客户端上报的时间戳和服务器保留的不一致,则下发更新配置"|初值为0|
|2|Short|count|子报文的数目|>=0|
|2|Short|subPacketLength|单个子报文长度|0x0018|
|6|Byte[6]|devMac|被探测设备的mac|
|2|Short|flag|bitMap，映射于（探测模式中）消息的类型，在定位模式中此字段暂时没有含义用户进入mask：2#001，用户离开mask：2#010|
|8|Long|timestamp|采样时间，毫秒级别|
|1|Byte|rssi|以dBm为单位的有符号整型值，以二进制的补码表示|
|1|Byte|channel|接收终端报文的信道|
|1|Byte|isAssociated|终端是否关联AP|关联：0x01，未关联：0x00|
|1|Byte|reserved|pad字段，留空|
|2|Short|frameControl|802.11协议报文的"Frame,Control"字段|
|2|Short|probeTimes|此次上报周期内探测到的次数|


##更新探针配置&上传报文应答包(103)
|Bytes|	数据类型|	描述|
|--|--|--|
|1|	Byte|	是否需要更新配置(0:不更新, 1:更新)|
|2|	Short|	探针配置JSON字符串长度, 可能是0|
|N|	Char(N)|探针配置JSON字符串```javascript{clientLeaveInterval: 100,code: 0,dataUploadInterval: 20,enable: "Y",inetAddress: "115.29.235.33:12092",latitude: "",longitude: "",message: "",mode: "probe",needRestart: "N",needUpgrade: "N",rssi: -55,success: "Y",telnet: "Y",upgradeUrl: ""timeStamp:1440666818char8}```|
|2|	Short|	保留|


##握手报文请求包(104)
|Bytes|数据类型|字段名|描述|值|
|--|--|--|--|--|
|2|Short|protocolVersion|协议版本|1|
|2|Short|固件版本号长度|||
|N|Char(N)|firmwareVersion|固件版本|V0.0.1.3|
|2|Short|硬件版本号长度|||
|N|Char(N)|hardwareVersion|硬件版本号|1.0|
|2|Short|设备类型长度|||
|N|Char(N)|productType|设备类型|P1或P1S|

##握手报文应答包(105)
|Bytes|数据类型|描述|
|--|--|--|
|1|Byte|成功 1,失败 0|
|2|Short|错误码(若有)|


#附录
##json返回值格式
|字段|类型|描述|样值|
|--|--|--|--|
|clientLeaveInterval|int|离开间隔|100|
|code|int|未知|0|
|dataUploadInterval|int|上传间隔|20|
|enable||string|关闭/开启探针|Y|
|inetAddress|string|未知|115.29.235.33:12092|
|latitude|string|未知|
|longitude|string|未知|
|message|string|响应消息|
|mode|string|探针模式|probe|
|needRestart|string|是否需要重启|N|
|needUpgrade|string|是否需要更新|N|
|rssi|int|信号强度|-55|
|success|string|成功/失败|Y|
|telnet||string|是否开启telnet|Y|
|upgradeUrl|string|更新url|
|timeStamp|long|服务配置时间|1440666818|
