# MOS云主机(MCS) API规范 #


MOS云主机(MCS, Meituan Compute Service)提供EC2兼容的API访问接口，以方便用户自动化管理云主机。为了访问MOS开放接口，MOS为每个用户分配访问的令牌和密码（ACCESS KEY ID和SECRET），每个访问请求都需要携带ACCESS KEY ID以及SECRET对请求数据的数字签名。用户可以在[MOS管理界面](https://mos.meituan.com)的[帐户-个人设置](https://mos.meituan.com/dashboard/account#profile)页面查询访问API的入口URL, ACCESS KEY ID以及SECRET。

## API介绍 ##

每个MOS区域(Region)有一个独立的API入口。

接口支持GET和POST两种方式。参数以key-value形式通过x-form-url-encoded格式编码提交。例如：

    https://10.168.44.160:8883?
        Action=DescribeTemplates&
        AWSAccessKeyId=4ba303cc17454cc7904e044db2a3c912&
        SignatureVersion=2&
        Timestamp=2013-07-20T15%3A02%3A17.000Z&
        SignatureMethod=HmacSHA256&
        Signature=q90E9%2BNtkVb9rJUPQJy75Ec%2F7j6HRCFfaSp2OhWYwuc%3D


一般，每个接口请求都包含如下参数：

<table>
  <tbody>
    <tr>
      <th>参数名称</th>
      <th>是否可选</th>
      <th>说明</th>
      <th>举例</th>
    </tr>
    <tr>
      <td>Action</td>
      <td>必选</td>
      <td>API请求的类型名称。支持的Action及相关参数参见下文"Action列表"。</td>
      <td>"DescribeInstances"</td>
    </tr>
    <tr>
      <td>AWSAccessKeyId</td>
      <td>必选</td>
      <td>访问API请求的ACCESS KEY ID</td>
      <td>"4ba303cc17454cc7904e044db2a3c912"</td>
    </tr>
    <tr>
      <td>Timestamp</td>
      <td>必选</td>
      <td>该请求的时间戳，iso8601格式："YYYY-MM-DDTHH:MM:SS.MMMZ"</td>
      <td>"2013-07-20T02:41:04.000Z"</td>
    </tr>
    <tr>
      <td>SignatureVersion</td>
      <td>必选</td>
      <td>数字签名算法版本，目前支持AWS数字签名2.0版本，该值必须为2</td>
      <td>"2"</td>
    </tr>
    <tr>
      <td>SignatureMethod</td>
      <td>必选</td>
      <td>数字签名的Hash算法，可能的值为"HmacSHA256"和"HmacSHA1"，分别对应SHA256和SHA1算法</td>
      <td>"HmacSHA256"</td>
    </tr>
    <tr>
      <td>Signature</td>
      <td>必选</td>
      <td>请求内容的数字签名，具体签名生成算法见下文。</td>
      <td>"m4XAVozDdWIduO8DkOj7fECeNZsUbX41Bv1atOSqMwA="</td>
    </tr>
    <tr>
      <td>Format</td>
      <td>可选</td>
      <td>返回内容的编码格式，可选值为xml和json，如果未指定，则默认为xml。如果返回为xml，则返回Content-Type为application/xml，如果是json格式，则为application/json。</td>
      <td>"json"</td>
    </tr>
  </tbody>
</table>

### 数字签名生成方法 ###

MOS开放API需要签名，遵循[AWS API 2.0签名规范](http://docs.aws.amazon.com/general/latest/gr/signature-version-2.html)。需要将请求内容按如下方法拼接，并按照指定Hash算法获得base64编码的签名字符串：

1\. 全部字母大写的HTTP请求方法（GET或POST)，以换行(\n)结束。例如：

    POST\n


2\. 全部字母为小写的API服务主机名，以换行(\n)结束。如果请求端口号为非默认端口（http非80端口，https非443），则需要加上端口，以冒号和host分隔。例如：

    mos-api.meituan.com\n


3\. 请求绝对路径，以换行(\n)结束。如果路径为空字符串，则为根路径"/"。例如：

    /\n


4\. 所有请求参数按照请求参数名称的字典顺序排序，并以x-www-form-urlencoded编码拼接在一起。例如：

    AWSAccessKeyId=4ba303cc17454cc7904e044db2a3c912&Action=DescribeTemplates&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2013-07-20T15%3A28%3A04.000Z


完整的拼接的签名内容为：

    POST
    mos-api.meituan.com
    /
    AWSAccessKeyId=4ba303cc17454cc7904e044db2a3c912&Action=DescribeTemplates&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2013-07-20T15%3A28%3A04.000Z


5\. 最后，以SECRET为Key对上述字符串以SHA256算法进行Hash后获得签名，结果为：

    EFVL9+vcx6tG9DCp04Ic+itrdCD/uZEhisS9amSSbig=


### 返回格式 ###

如果API请求成功，服务器返回HTTP 200，body包含返回数据。返回数据根据请求的Format参数进行格式化。

如果API请求失败，服务器返回HTTP 4xx，body包含错误信息，消息格式示例如下：

xml格式错误响应

    <?xml version="1.0" encoding="utf-8"?>
    <ErrorResponse>
        <Error>
            <message>Not enough balance</message>
            <code>406</code>
        </Error>
    </ErrorResponse>

json格式错误响应

    {"ErrorResponse":
        {"Error":
            {"message": "Not enough balance",
             "code": 406
            }
        }
    }

其中，code为错误代码，message为错误消息。code和返回消息的HTTP错误代码相同。


## 模板API ##

### DescribeTemplates ###

列出所有用户可以使用的虚拟机模板，在创建虚拟机，更改虚拟机系统磁盘时，需要相关信息。

**请求参数：**

无

**输出字段：**

<table>
    <tbody>
      <tr>
        <th>字段名</th>
        <th>类型</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>templateId</td>
        <td>string</td>
        <td>模板ID</td>
      </tr>
      <tr>
        <td>templateName</td>
        <td>string</td>
        <td>模板名称</td>
      </tr>
      <tr>
        <td>size</td>
        <td>int</td>
        <td>模板Image的字节大小(Bytes)</td>
      </tr>
      <tr>
        <td>checksum</td>
        <td>string</td>
        <td>模板Image的MD5 checksum</td>
      </tr>
      <tr>
        <td>status</td>
        <td>string</td>
        <td>模板状态</td>
      </tr>
    </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        Action=DescribeTemplates&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <DescribeTemplatesResponse>
        <TemplateSet>
            <Template>
                <status>active</status>
                <checksum>952a921243eecf2f457b82051e880558</checksum>
                <templateId>019c6db6-55fa-443d-ac0c-182e3379d175</templateId>
                <size>187367424</size>
                <templateName>turnkey-core-12.0-squeeze-x86.qcow2</templateName>
            </Template>
        </TemplateSet>
    </DescribeTemplatesResponse>

json响应

    {"DescribeTemplatesResponse": 
        {"TemplateSet": 
            {"Template": [
                {"status": "active",
                 "checksum": "952a921243eecf2f457b82051e880558",
                 "templateName": "turnkey-core-12.0-squeeze-x86.qcow2",
                 "templateId": "019c6db6-55fa-443d-ac0c-182e3379d175",
                 "size": 187367424
                }
             ]
            }
        }
    }


## 套餐类型API ##

### DescribeInstanceTypes ###

列出所有用户可以使用的虚拟机套餐类型，在创建虚拟机，更改虚拟机类型时，需要相关信息。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>Limit</td>
      <td>integer</td>
      <td>可选</td>
      <td>本次请求返回的数量</td>
    </tr>
    <tr>
      <td>Offset</td>
      <td>integer</td>
      <td>可选</td>
      <td>本次请求返回的偏移量</td>
    </tr>
    <tr>
      <td>Filter.n.Name</td>
      <td>string</td>
      <td>可选</td>
      <td>过滤字段名称，n从1开始。可能的值为：name</td>
    </tr>
    <tr>
      <td>Filter.n.Value.m</td>
      <td>string</td>
      <td>可选</td>
      <td>对应Filter.n.Name的过滤字段的匹配值，m从1开始</td>
    </tr>
  </tbody>
</table>

**返回数据：**

返回InstanceTypeSet，包含如下子段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceType</td>
      <td>complextype</td>
      <td>虚拟机类型定义</td>
    </tr>
    <tr>
      <td>Total</td>
      <td>integer</td>
      <td>返回符合条件的虚拟机类型总量</td>
    </tr>
    <tr>
      <td>Limit</td>
      <td>integer</td>
      <td>返回虚拟机类型的数量</td>
    </tr>
    <tr>
      <td>Offset</td>
      <td>integer</td>
      <td>返回虚拟机类型的偏移量</td>
    </tr>
  </tbody>
</table>

InstanceType包含如下子段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>instanceTypeId</td>
      <td>string</td>
      <td>虚拟机类型ID</td>
    </tr>
    <tr>
      <td>instanceType</td>
      <td>string</td>
      <td>虚拟机类型名称</td>
    </tr>
    <tr>
      <td>cpu</td>
      <td>integer</td>
      <td>该类型虚拟机CPU核数，单位为个</td>
    </tr>
    <tr>
      <td>memory</td>
      <td>integer</td>
      <td>该类型虚拟机内存大小，单位为MB</td>
    </tr>
    <tr>
      <td>volume</td>
      <td>integer</td>
      <td>该类型虚拟机虚拟存储大小，单位为MB</td>
    </tr>
    <tr>
      <td>internalBandwidth</td>
      <td>integer</td>
      <td>该类型虚拟机内网接入带宽，单位为Mbps</td>
    </tr>
    <tr>
      <td>externalBandwidth</td>
      <td>integer</td>
      <td>该类型虚拟机外网接入带宽，单位为Mbps</td>
    </tr>
  </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        Limit=1&
        Action=DescribeInstanceTypes&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <DescribeInstanceTypesResponse>
        <InstanceTypeSet>
            <Total>7</Total>
            <Limit>1</Limit>
            <InstanceType>
                <instanceTypeId>8e845438-2f6d-4c87-9216-88da6692dc2b</instanceTypeId>
                <internalBandwidth>200</internalBandwidth>
                <externalBandwidth>2</externalBandwidth>
                <cpu>1</cpu>
                <volume>1024</volume>
                <memory>128</memory>
                <instanceType>small_net_2</instanceType>
            </InstanceType>
        </InstanceTypeSet>
    </DescribeInstanceTypesResponse>

json响应

    {"DescribeInstanceTypesResponse": 
        {"InstanceTypeSet": 
            {"Total": 7, 
             "Limit": 1, 
             "InstanceType": [
                {"instanceTypeId": "8e845438-2f6d-4c87-9216-88da6692dc2b",
                 "internalBandwidth": 200,
                 "externalBandwidth": 2,
                 "instanceType": "small_net_2", 
                 "volume": 1024, 
                 "memory": 128, 
                 "cpu": 1,
                }
             ]
            }
        }
    }


## 帐户API ##

### GetBalance ###

获得用户的当前帐户余额

**请求参数：**

无

**返回数据：**

<table>
    <tbody>
      <tr>
        <th>字段名</th>
        <th>类型</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>balance</td>
        <td>decimal(10,2)</td>
        <td>帐户余额</td>
      </tr>
      <tr>
        <td>timestamp</td>
        <td>datetime</td>
        <td>最后一次帐户余额发生变化的时间，如果该帐户从未发生过交易，则余额为0，无timestamp字段</td>
      </tr>
    </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        Action=GetBalance&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <GetBalanceResponse>
        <timestamp>2013-07-19T15:52:02Z</timestamp>
        <balance>16.66</balance>
    </GetBalanceResponse>

json响应

    {"GetBalanceResponse": 
        {"timestamp": "2013-07-19T15:52:02Z",
         "balance": 16.66
        }
    }


## SSH密钥API ##

### DescribeKeyPairs ###

列出用户所有的SSH Key pairs

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>Limit</td>
      <td>integer</td>
      <td>可选</td>
      <td>本次请求返回的最多数量</td>
    </tr>
    <tr>
      <td>Offset</td>
      <td>integer</td>
      <td>可选</td>
      <td>本次请求返回的偏移量</td>
    </tr>
    <tr>
      <td>Filter.n.Name</td>
      <td>string</td>
      <td>可选</td>
      <td>过滤字段名称，n从1开始。可能的值为：name</td>
    </tr>
    <tr>
      <td>Filter.n.Value.m</td>
      <td>string</td>
      <td>可选</td>
      <td>对应Filter.n.Name的过滤字段的匹配值，m从1开始</td>
    </tr>
  </tbody>
</table>

**返回数据：**

返回KeyPairSet包含如下字段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>KeyPair</td>
      <td>complexType</td>
      <td>返回的SSH Key信息</td>
    </tr>
    <tr>
      <td>Total</td>
      <td>integer</td>
      <td>满足查询条件的SSH Key个数</td>
    </tr>
    <tr>
      <td>Limit</td>
      <td>integer</td>
      <td>实际返回的SSH Key个数</td>
    </tr>
    <tr>
      <td>Offset</td>
      <td>integer</td>
      <td>返回的偏移量</td>
    </tr>
  </tbody>
</table>

KeyPair包含的字段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>keyId</td>
      <td>string</td>
      <td>SSH Key的ID</td>
    </tr>
    <tr>
      <td>keyName</td>
      <td>string</td>
      <td>SSH Key的名称</td>
    </tr>
    <tr>
      <td>keyFingerprint</td>
      <td>string</td>
      <td>SSH 公钥(public key)的指纹(fingerprint)</td>
    </tr>
  </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        Action=DescribeKeyPairs&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <DescribeKeyPairsResponse>
        <KeyPairSet>
            <KeyPair>
                <keyId>cb97eb8b-de94-4148-849f-2b931cfce97a</keyId>
                <keyName>testkey</keyName>
                <keyFingerprint>0a:43:d9:7b:17:a1:24:26:9a:0e:ce:dc:f4:0a:03:44</keyFingerprint>
            </KeyPair>
            <KeyPair>
                <keyId>b7bfd341-e6d1-4971-8c45-d3ed6f97a846</keyId>
                <keyName>mackey</keyName>
                <keyFingerprint>18:0e:d1:45:82:54:78:be:60:f1:a6:8f:cf:64:88:1e</keyFingerprint>
            </KeyPair>
        </KeyPairSet>
    </DescribeKeyPairsResponse>

json响应

    {"DescribeKeyPairsResponse": 
        {"KeyPairSet": 
            {"KeyPair": [
                {"keyId": "cb97eb8b-de94-4148-849f-2b931cfce97a",
                 "keyName": "testkey",
                 "keyFingerprint": "0a:43:d9:7b:17:a1:24:26:9a:0e:ce:dc:f4:0a:03:44"
                },
                {"keyId": "b7bfd341-e6d1-4971-8c45-d3ed6f97a846",
                 "keyName": "mackey",
                 "keyFingerprint": "18:0e:d1:45:82:54:78:be:60:f1:a6:8f:cf:64:88:1e"
                }
             ]
            }
        }
    }

### ImportKeyPair ###

导入一个SSH Key

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>KeyName</td>
      <td>string</td>
      <td>必须</td>
      <td>SSH Key名称</td>
    </tr>
    <tr>
      <td>PublicKeyMaterial</td>
      <td>string</td>
      <td>必须</td>
      <td>SSH Key的public key</td>
    </tr>
  </tbody>
</table>

**返回数据：**

返回KeyPair包含的字段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>keyId</td>
      <td>string</td>
      <td>SSH Key的ID</td>
    </tr>
    <tr>
      <td>keyName</td>
      <td>string</td>
      <td>SSH Key的名称</td>
    </tr>
    <tr>
      <td>keyFingerprint</td>
      <td>string</td>
      <td>SSH 公钥(public key)的指纹(fingerprint)</td>
    </tr>
  </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        KeyName=newkey&
        Action=ImportKeyPair&
        PublicKeyMaterial=ssh-rsa+AAAAB3Nza...OVL%2B2Y7R+qj%40dog%0A&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <ImportKeyPairResponse>
        <KeyPair>
            <keyId>0f4697a4-6439-4ae7-b6fe-be29ace2303c</keyId>
            <keyName>newkey</keyName>
            <keyFingerprint>0a:43:d9:7b:17:a1:24:26:9a:0e:ce:dc:f4:0a:03:44</keyFingerprint>
        </KeyPair>
    </ImportKeyPairResponse>

json响应

    {"ImportKeyPairResponse":
        {"KeyPair":
            {"keyId": "0f4697a4-6439-4ae7-b6fe-be29ace2303c",
             "keyName": "newkey",
             "keyFingerprint": "0a:43:d9:7b:17:a1:24:26:9a:0e:ce:dc:f4:0a:03:44"
            }
        }
    }

### DeleteKeyPair ###

删除一个SSH Key

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>KeyName</td>
      <td>string</td>
      <td>必须</td>
      <td>SSH Key名称</td>
    </tr>
  </tbody>
</table>

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        KeyName=newkey&
        Action=DeleteKeyPair&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <DeleteKeyPairResponse>
        <return>True</return>
    </DeleteKeyPairResponse>

json响应

    {"DeleteKeyPairResponse":
        {"return": "True"
        }
    }


## 虚拟机API ##

### DescribeInstances ###

列出所有或指定的用户虚拟机实例。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId.n</td>
      <td>string</td>
      <td>可选</td>
      <td>指定虚拟机的ID，n从1开始</td>
    </tr>
    <tr>
      <td>InstanceName.n</td>
      <td>string</td>
      <td>可选</td>
      <td>指定虚拟机的Name，n从1开始</td>
    </tr>
    <tr>
      <td>Limit</td>
      <td>integer</td>
      <td>可选</td>
      <td>本次请求返回的最多数量</td>
    </tr>
    <tr>
      <td>Offset</td>
      <td>integer</td>
      <td>可选</td>
      <td>本次请求返回的偏移量</td>
    </tr>
    <tr>
      <td>Filter.n.Name</td>
      <td>string</td>
      <td>可选</td>
      <td>过滤字段名称，n从1开始。可能的值为：name, status</td>
    </tr>
    <tr>
      <td>Filter.n.Value.m</td>
      <td>string</td>
      <td>可选</td>
      <td>对应Filter.n.Name的过滤字段的匹配值，m从1开始</td>
    </tr>
  </tbody>
</table>

**返回数据：**

返回InstanceSet包含如下字段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>Instance</td>
      <td>complexType</td>
      <td>返回的虚拟机信息</td>
    </tr>
    <tr>
      <td>Total</td>
      <td>integer</td>
      <td>满足查询条件的虚拟机个数</td>
    </tr>
    <tr>
      <td>Limit</td>
      <td>integer</td>
      <td>实际返回的虚拟机个数</td>
    </tr>
    <tr>
      <td>Offset</td>
      <td>integer</td>
      <td>虚拟机的偏移量</td>
    </tr>
  </tbody>
</table>

Instance包含的字段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>instanceId</td>
      <td>string</td>
      <td>虚拟机的ID</td>
    </tr>
    <tr>
      <td>instanceName</td>
      <td>string</td>
      <td>虚拟机的名称</td>
    </tr>
    <tr>
      <td>instanceType</td>
      <td>string</td>
      <td>虚拟机的类型</td>
    </tr>
    <tr>
      <td>instanceTypeId</td>
      <td>string</td>
      <td>虚拟机类型的ID</td>
    </tr>
    <tr>
      <td>status</td>
      <td>string</td>
      <td>虚拟机的状态，可能值有running/ready/suspend等</td>
    </tr>
    <tr>
      <td>cpu</td>
      <td>integer</td>
      <td>虚拟机的CPU核数</td>
    </tr>
    <tr>
      <td>memory</td>
      <td>integer</td>
      <td>虚拟机的内存大小，单位为MB</td>
    </tr>
    <tr>
      <td>volume</td>
      <td>integer</td>
      <td>虚拟机的总磁盘大小，单位为MB</td>
    </tr>
    <tr>
      <td>ipAddresses</td>
      <td>string</td>
      <td>虚拟机的IP地址列表</td>
    </tr>
  </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        Limit=1&
        Offset=2&
        Action=DescribeInstances&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <DescribeInstancesResponse>
        <InstanceSet>
            <Instance>
                <status>ready</status>
                <instanceId>027ff1d8-e3a0-4e2e-a1e1-03d6ee03c353</instanceId>
                <instanceType>small</instanceType>
                <volume>59</volume>
                <memory>128</memory>
                <instanceName>testtest</instanceName>
                <cpu>1</cpu>
                <ipAddresses>10.168.44.230</ipAddresses>
            </Instance>
            <Total>6</Total>
            <Limit>1</Limit>
            <Offset>2</Offset>
        </InstanceSet>
    </DescribeInstancesResponse>

json响应

    {"DescribeInstancesResponse": 
        {"InstanceSet":
            {"Instance": [
                {"status": "ready",
                 "instanceId": "027ff1d8-e3a0-4e2e-a1e1-03d6ee03c353",
                 "cpu": 1,
                 "volume": 59,
                 "memory": 128,
                 "instanceName": "testtest",
                 "instanceType": "small",
                 "ipAddresses": "10.168.44.230",
                }
             ], 
             "Total": 6,
             "Limit": 1,
             "Offset": 2
            }
        }
    }

### DescribeInstanceStatus ###

获得指定虚拟机实例的状态。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>指定虚拟机ID</td>
    </tr>
  </tbody>
</table>

**返回数据：**

返回InstanceStatus，包含status字段。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=testtest&
        Action=DescribeInstanceStatus&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <DescribeInstanceStatusResponse>
        <InstanceStatus>
            <status>ready</status>
        </InstanceStatus>
    </DescribeInstanceStatusResponse>

json响应

    {"DescribeInstanceStatusResponse": 
        {"InstanceStatus": 
            {"status": "ready"}
        }
    }

### DescribeInstanceVolumes ###

列出指定虚拟机的所有虚拟磁盘的信息。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>指定虚拟机ID</td>
    </tr>
  </tbody>
</table>

**返回数据：**

返回数据集InstanceVolumeSet，包含如下字段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceVolume</td>
      <td>complex type</td>
      <td>一个虚拟机磁盘的信息</td>
    </tr>
  </tbody>
</table>

InstanceVolume包含如下字段信息：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>instanceId</td>
      <td>string</td>
      <td>虚拟机ID</td>
    </tr>
    <tr>
      <td>instanceName</td>
      <td>string</td>
      <td>虚拟机名称</td>
    </tr>
    <tr>
      <td>volumeId</td>
      <td>string</td>
      <td>虚拟磁盘ID</td>
    </tr>
    <tr>
      <td>volumeName</td>
      <td>string</td>
      <td>虚拟磁盘名称</td>
    </tr>
    <tr>
      <td>volumeSize</td>
      <td>integer</td>
      <td>磁盘大小，单位为MB</td>
    </tr>
    <tr>
      <td>cacheMode</td>
      <td>string</td>
      <td>磁盘的缓存模式，可能值为none, writeback, writethrough, 缺省为none</td>
    </tr>
    <tr>
      <td>driver</td>
      <td>string</td>
      <td>磁盘的驱动，可能值为virtio, ide和scsi，缺省为virtio</td>
    </tr>
    <tr>
      <td>index</td>
      <td>integer</td>
      <td>磁盘挂载在虚拟机上的序号，从0开始</td>
    </tr>
  </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=testtest&
        Action=DescribeInstanceVolumes&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <DescribeInstanceVolumesResponse>
        <InstanceVolumeSet>
            <InstanceVolume>
                <index>1</index>
                <instanceId>027ff1d8-e3a0-4e2e-a1e1-03d6ee03c353</instanceId>
                <volumeName>vdisk_testtest_1371493324.491348</volumeName>
                <driver>virtio</driver>
                <volumeId>0fccde09-74af-4504-9c89-52016510e9d7</volumeId>
                <cacheMode>none</cacheMode>
                <volumeSize>20</volumeSize>
                <instanceName>testtest</instanceName>
            </InstanceVolume>
            <InstanceVolume>...</InstanceVolume>
        </InstanceVolumeSet>
    </DescribeInstanceVolumesResponse>

json响应

    {"DescribeInstanceVolumesResponse": 
        {"InstanceVolumeSet": 
            {"InstanceVolume": [
                {"index": 1, 
                 "instanceId": "027ff1d8-e3a0-4e2e-a1e1-03d6ee03c353", 
                 "volumeName": "vdisk_testtest_1371493324.491348", 
                 "driver": "virtio", 
                 "volumeId": "0fccde09-74af-4504-9c89-52016510e9d7", 
                 "cacheMode": "none", 
                 "volumeSize": 20, 
                 "instanceName": "testtest"
                },
                {...}
             ]
            }
        }
    }

### DescribeInstanceNetworkInterfaces ###

列出指定虚拟机实例的所有虚拟网络接口的信息。

**请求参数：**

<table>
    <tbody>
      <tr>
        <th>参数名</th>
        <th>类型</th>
        <th>可选</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>InstanceId</td>
        <td>string</td>
        <td>必须</td>
        <td>指定虚拟机ID</td>
      </tr>
    </tbody>
</table>

**返回数据：**

返回数据集InstanceNetworkInterfaceSet，包含如下字段：

<table>
    <tbody>
      <tr>
        <th>字段名</th>
        <th>类型</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>InstanceNetworkInterface</td>
        <td>complex type</td>
        <td>一个虚拟机网络接口的信息</td>
      </tr>
    </tbody>
  </table>
  <p>InstanceNetworkInterface包含如下信息：</p>
  <table>
    <tbody>
      <tr>
        <th>字段名</th>
        <th>类型</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>instanceId</td>
        <td>string</td>
        <td>虚拟机ID</td>
      </tr>
      <tr>
        <td>instanceName</td>
        <td>string</td>
        <td>虚拟机名称</td>
      </tr>
      <tr>
        <td>networkId</td>
        <td>string</td>
        <td>网络接口接入的虚拟机网络ID</td>
      </tr>
      <tr>
        <td>networkName</td>
        <td>string</td>
        <td>网络接口接入的虚拟网络名称</td>
      </tr>
      <tr>
        <td>ipAddress</td>
        <td>string</td>
        <td>网络接口的IP地址</td>
      </tr>
      <tr>
        <td>macAddress</td>
        <td>string</td>
        <td>网络接口的硬件地址</td>
      </tr>
      <tr>
        <td>bandwidth</td>
        <td>integer</td>
        <td>网络接口带宽，单位为Mbps</td>
      </tr>
      <tr>
        <td>driver</td>
        <td>string</td>
        <td>驱动，可能值有virtio, e1000，缺省为virtio</td>
      </tr>
      <tr>
        <td>index</td>
        <td>integer</td>
        <td>网络接口在虚拟机上的序号</td>
      </tr>
    </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=testtest&
        Action=DescribeInstanceNetworkInterfaces&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <DescribeInstanceNetworkInterfacesResponse>
        <InstanceNetworkInterfaceSet>
            <InstanceNetworkInterface>
                <networkId>40480c6f-2c7e-4ba8-b040-92a64a948c90</networkId>
                <index>0</index>
                <instanceId>027ff1d8-e3a0-4e2e-a1e1-03d6ee03c353</instanceId>
                <instanceName>testtest</instanceName>
                <driver>virtio</driver>
                <bandwidth>10</bandwidth>
                <networkName>public</networkName>
                <ipAddress>10.168.44.229</ipAddress>
                <macAddress>00:22:34:84:24:60</macAddress>
            </InstanceNetworkInterface>
        </InstanceNetworkInterfaceSet>
    </DescribeInstanceNetworkInterfacesResponse>

json响应

    {"DescribeInstanceNetworkInterfacesResponse": 
        {"InstanceNetworkInterfaceSet": 
            {"InstanceNetworkInterface": [
                {"networkId": "40480c6f-2c7e-4ba8-b040-92a64a948c90",
                 "index": 0,
                 "instanceId": "027ff1d8-e3a0-4e2e-a1e1-03d6ee03c353",
                 "networkName": "public",
                 "driver": "virtio",
                 "bandwidth": 10,
                 "instanceName": "testtest",
                 "ipAddress": "10.168.44.229",
                 "macAddress": "00:22:34:84:24:60"
                },
                {...}
             ]
            }
        }
    }

### GetPasswordData ###

获得指定虚拟机实例的初始帐户和密码信息。

**请求参数：**

<table>
    <tbody>
      <tr>
        <th>参数名</th>
        <th>类型</th>
        <th>可选</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>InstanceId</td>
        <td>string</td>
        <td>必须</td>
        <td>指定虚拟机ID</td>
      </tr>
    </tbody>
</table>

**返回数据：**

<table>
    <tbody>
      <tr>
        <th>字段名</th>
        <th>类型</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>timestamp</td>
        <td>datetime</td>
        <td>指示初始帐号密码生成的时间</td>
      </tr>
      <tr>
        <td>account</td>
        <td>string</td>
        <td>虚拟机的初始帐号</td>
      </tr>
      <tr>
        <td>passwordData</td>
        <td>string</td>
        <td>虚拟机的初始帐号密码数据，如果虚拟机未使用keypair，则该数据为明文密码，否则，该数据为keypair公钥加密，需要使用该keypair的对应key文件解密</td>
      </tr>
      <tr>
        <td>keypairId</td>
        <td>string</td>
        <td>如果虚拟机使用了keypair，则为该虚拟机使用的keypair的ID；否则无此字段</td>
      </tr>
      <tr>
        <td>keypairName</td>
        <td>string</td>
        <td>如果虚拟机使用了keypair，则为该虚拟机使用的keypair的名称；否则无此字段</td>
      </tr>
    </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=GetPasswordData&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <GetPasswordDataResponse>
        <timestamp>2013-07-22T02:48:56Z</timestamp>
        <account>cirros</account>
        <passwordData>jwFN2C3Ngmgu</passwordData>
    </GetPasswordDataResponse>

json响应

    {"GetPasswordDataResponse": 
        {"timestamp": "2013-07-22T02:48:56Z", 
         "account": "cirros", 
         "passwordData": "jwFN2C3Ngmgu"
        }
    }

### GetInstanceContractInfo ###

获得指定虚拟机实例的合同时间信息。

**请求参数：**

<table>
    <tbody>
      <tr>
        <th>参数名</th>
        <th>类型</th>
        <th>可选</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>InstanceId</td>
        <td>string</td>
        <td>必须</td>
        <td>指定虚拟机ID</td>
      </tr>
    </tbody>
</table>

**返回数据：**

返回如下字段：

<table>
    <tbody>
      <tr>
        <th>字段名</th>
        <th>类型</th>
        <th>说明</th>
      </tr>
      <tr>
        <td>started_at</td>
        <td>datetime</td>
        <td>虚拟机租约开始时间</td>
      </tr>
      <tr>
        <td>expire_at</td>
        <td>datetime</td>
        <td>虚拟机租约到期时间</td>
      </tr>
      <tr>
        <td>extend_to</td>
        <td>datetime</td>
        <td>如果未按期续费，虚拟机过期后保留时间</td>
      </tr>
    </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=GetInstanceContractInfo&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <GetInstanceContractInfoResponse>
        <started_at>2013-07-22T03:00:00Z</started_at>
        <extend_to>2013-07-26T03:00:00Z</extend_to>
        <expire_at>2013-07-25T03:00:00Z</expire_at>
    </GetInstanceContractInfoResponse>

json响应

    {"GetInstanceContractInfoResponse":
        {"started_at": "2013-07-22T03:00:00Z",
         "extend_to": "2013-07-26T03:00:00Z",
         "expire_at": "2013-07-25T03:00:00Z"
        }
    }


### StartInstance ###

启动指定虚拟机实例。虚拟机在ready状态时才能成功启动。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>指定虚拟机ID</td>
    </tr>
  </tbody>
</table>

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=StartInstance&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <StartInstanceResponse>
        <return>True</return>
    </StartInstanceResponse>

json响应

    {"StartInstanceResponse":
        {"return": "True"
        }
    }

### StopInstance ###

停止指定虚拟机实例。只有虚拟机在running状态时才能成功停止虚拟机。如果指定强制停止，则虚拟机进程立即退出，可能会造成虚拟机内部数据丢失。否则，虚拟机将试图软关机，30秒超时后，如果虚拟机实例还未停止，则强制停止。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>停止的虚拟机ID</td>
    </tr>
    <tr>
      <td>Force</td>
      <td>boolean</td>
      <td>可选</td>
      <td>是否强制立即停止</td>
    </tr>
  </tbody>
</table>

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=StopInstance&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <StopInstanceResponse>
        <return>True</return>
    </StopInstanceResponse>

json响应

    {"StopInstanceResponse":
        {"return": "True"
        }
    }

### RebootInstance ###

重启指定虚拟机实例。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>重启的虚拟机ID</td>
    </tr>
  </tbody>
</table>

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=RebootInstance&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <RebootInstanceResponse>
        <return>True</return>
    </RebootInstanceResponse>

json响应

    {"RebootInstanceResponse":
        {"return": "True"
        }
    }


### RebuildInstanceRootImage ###

重置指定虚拟机实例的的系统磁盘镜像。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>需要重置系统盘镜像的InstanceId</td>
    </tr>
    <tr>
      <td>ImageId</td>
      <td>string</td>
      <td>可选</td>
      <td>指定充值系统盘的源模板镜像ID，如果不指定该参数，则重置为原来的操作系统</td>
    </tr>
  </tbody>
</table>

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=RebuildInstanceRootImage&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <RebuildInstanceRootImageResponse>
        <return>True</return>
    </RebuildInstanceRootImageResponse>

json响应

    {"RebuildInstanceRootImageResponse":
        {"return": "True"
        }
    }

### CreateInstance ###

创建虚拟机实例。*注意：该操作涉及帐户扣费，请保证帐户有足够余额，否则将创建失败*。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>ImageId</td>
      <td>string</td>
      <td>必须</td>
      <td>镜像模板ID</td>
    </tr>
    <tr>
      <td>InstanceType</td>
      <td>string</td>
      <td>必须</td>
      <td>创建的类型</td>
    </tr>
    <tr>
      <td>Duration</td>
      <td>string</td>
      <td>可选</td>
      <td>创建的虚拟机的时间，格式为数字+H/M，例如1H, 72H或者1M。缺省为1M</td>
    </tr>
    <tr>
      <td>InstanceName</td>
      <td>string</td>
      <td>可选</td>
      <td>指定创建的虚拟机的名称</td>
    </tr>
    <tr>
      <td>KeyName</td>
      <td>string</td>
      <td>可选</td>
      <td>指定创建的虚拟机使用的SSH Key</td>
    </tr>
  </tbody>
</table>

**返回数据：**

如果成功返回生成的Instance信息，包含如下字段：

<table>
  <tbody>
    <tr>
      <th>字段名</th>
      <th>类型</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>instanceId</td>
      <td>string</td>
      <td>虚拟机ID</td>
    </tr>
    <tr>
      <td>instanceName</td>
      <td>string</td>
      <td>虚拟机名称</td>
    </tr>
    <tr>
      <td>instanceType</td>
      <td>string</td>
      <td>虚拟机类型</td>
    </tr>
  </tbody>
</table>

**示例：**

请求URL

    https://10.168.44.160:8883?
        ImageId=d1620e45-c561-42e7-a2a4-53ae0a389bb9&
        Duration=72H&
        InstanceType=small_net&
        Action=CreateInstance&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <CreateInstanceResponse>
        <Instance>
            <instanceId>022a58da-5cee-4589-9e6a-f54fa1abd269</instanceId>
            <instanceName>system</instanceName>
            <instanceType>small_net</instanceType>
        </Instance>
    </CreateInstanceResponse>

json响应

    {"CreateInstanceResponse": 
        {"Instance": 
            {"instanceId": "022a58da-5cee-4589-9e6a-f54fa1abd269", 
             "instanceName": "system", 
             "instanceType": "small_net"
            }
        }
    }


### TerminateInstance ###

销毁指定虚拟机实例。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>销毁虚拟机的ID</td>
    </tr>
  </tbody>
</table>

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=TerminateInstance&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <TerminateInstanceResponse>
        <return>True</return>
    </TerminateInstanceResponse>

json响应

    {"TerminateInstanceResponse":
        {"return": "True"
        }
    }


### RenewInstance ###

续期指定虚拟机实例。*注意：该操作涉及帐户扣费，请保证帐户有足够余额，否则将续期失败*。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>需要续期的虚拟机ID</td>
    </tr>
    <tr>
      <td>Duration</td>
      <td>string</td>
      <td>可选</td>
      <td>指定续期时间，格式为数字+H/M（小时/月），例如1H, 72H或者1M。如果不指定，缺省为1M</td>
    </tr>
  </tbody>
</table>


**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Duration=72H&
        Action=RenewInstance&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <RenewInstanceResponse>
        <return>True</return>
    </RenewInstanceResponse>

json响应

    {"RenewInstanceResponse":
        {"return": "True"
        }
    }

### ChangeInstanceType ###

更改指定虚拟机实例的类型。*注意：该操作涉及帐户扣费，请保证帐户有足够余额，否则将更改失败*。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>更改类型的虚拟机ID</td>
    </tr>
    <tr>
      <td>InstanceType</td>
      <td>string</td>
      <td>必须</td>
      <td>更改的虚拟机类型</td>
    </tr>
    <tr>
      <td>Duration</td>
      <td>datetime</td>
      <td>可选</td>
      <td>更改后虚拟机的租期时间，格式为数字+H/M（小时/月），例如1H, 72H或者1M。如果不指定，缺省为1M
      </td>
    </tr>
  </tbody>
</table>

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=ChangeInstanceType&
        InstanceType=small_net&
        Duration=1M&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <ChangeInstanceTypeResponse>
        <return>True</return>
    </ChangeInstanceTypeResponse>

json响应

    {"ChangeInstanceTypeResponse":
        {"return": "True"
        }
    }


### GetInstanceMetadata ###

获取指定虚拟机实例的元数据。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>指定虚拟机ID</td>
    </tr>
  </tbody>
</table>

**返回数据：**

返回InstanceMetadata数据集，包含所有key-value的元数据。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=GetInstanceMetadata&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <GetInstanceMetadataResponse>
        <InstanceMetadata>
            <os_version>2011.08</os_version>
            <os_name>Linux</os_name>
            <os_distribution>Cirros</os_distribution>
        </InstanceMetadata>
    </GetInstanceMetadataResponse>

json响应

    {"GetInstanceMetadataResponse": 
        {"InstanceMetadata": 
            {"os_version": "2011.08",
             "os_name": "Linux",
             "os_distribution": "Cirros"
            }
        }
    }


### PutInstanceMetadata ###

设置指定虚拟机实例的元数据。

**请求参数：**

<table>
  <tbody>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>可选</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>InstanceId</td>
      <td>string</td>
      <td>必须</td>
      <td>指定虚拟机ID</td>
    </tr>
    <tr>
      <td>Name.n</td>
      <td>string</td>
      <td>必须</td>
      <td>指定第n个元数据的key，n从1开始</td>
    </tr>
    <tr>
      <td>Value.n</td>
      <td>string</td>
      <td>必须</td>
      <td>指定第n个元数据的value，n从1开始</td>
    </tr>
  </tbody>
</table>

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

    https://10.168.44.160:8883?
        InstanceId=system&
        Name.1=test7d&
        Value.1=1&
        Action=PutInstanceMetadata&
        AUTHDATA

xml响应

    <?xml version="1.0" encoding="utf-8"?>
    <PutInstanceMetadataResponse>
        <return>True</return>
    </PutInstanceMetadataResponse>

json响应

    {"PutInstanceMetadataResponse":
        {"return": "True"
        }
    }
