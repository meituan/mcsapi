MOS云主机(MCS) API规范
======================

MOS云主机 (MCS, Meituan Compute Service)
提供EC2兼容的API访问接口，以方便用户自动化管理云主机。
为了访问MOS开放接口，MOS为每个用户分配访问的令牌和密码（ACCESS
KEY ID和SECRET），每个访问请求都需要携带ACCESS KEY
ID以及SECRET对请求数据的数字签名。用户可以在"`MOS管理界面 <https://mos.meituan.com>`_"
的"`帐户-个人设置 <https://mos.meituan.com/dashboard/account#profile>`_"
页面查询访问API的入口URL, ACCESS KEY ID以及SECRET。

API概述
-------

每个MOS区域(Region)有一个独立的API接口。

接口支持GET和POST两种方式。参数以key-value形式通过x-form-url-encoded格式编码提交。例如::

    https://10.168.44.160:8883?
        Action=DescribeTemplates&
        AWSAccessKeyId=4ba303cc17454cc7904e044db2a3c912&
        SignatureVersion=2&
        Timestamp=2013-07-20T15%3A02%3A17.000Z&
        SignatureMethod=HmacSHA256&
        Signature=q90E9%2BNtkVb9rJUPQJy75Ec%2F7j6HRCFfaSp2OhWYwuc%3D


请求参数
~~~~~~~~

一般，每个接口请求都包含如下参数

+------------------+----------+-----------------------------------------------------------+------------------------------------------------+
| 参数名称         | 可选     | 说明                                                      | 举例                                           |
+==================+==========+===========================================================+================================================+
| Action           | 必选     | API请求的类型名称。支持的Action及相关参数参见下文         |                                                |
|                  |          | "Action列表"                                              | "DescribeInstances"                            |
+------------------+----------+-----------------------------------------------------------+------------------------------------------------+
| AWSAccessKeyId   | 必选     | 访问API请求的ACCESS KEY ID                                | "4ba303cc17454cc7904e044db2a3c912"             |
+------------------+----------+-----------------------------------------------------------+------------------------------------------------+
| Timestamp        | 必选     | 该请求的时间戳，iso8601格式："YYYY-MM-DDTHH:MM:SS.MMMZ"   | "2013-07-20T02:41:04.000Z"                     |
+------------------+----------+-----------------------------------------------------------+------------------------------------------------+
| SignatureVersion | 必选     | 数字签名算法版本，目前支持AWS数字签名2.0版本，该值必须为2 | "2"                                            |
+------------------+----------+-----------------------------------------------------------+------------------------------------------------+
| SignatureMethod  | 必选     | 数字签名的Hash算法，可能的值为"HmacSHA256"和"HmacSHA1"，  |                                                |
|                  |          | 分别对应SHA256和SHA1算法                                  | "HmacSHA256"                                   |
+------------------+----------+-----------------------------------------------------------+------------------------------------------------+
| Signature        | 必选     | 请求内容的数字签名，具体签名生成算法见下文                | "m4XAVozDdWIduO8DkOj7fECeNZsUbX41Bv1atOSqMwA=" |
+------------------+----------+-----------------------------------------------------------+------------------------------------------------+
| Format           | 可选     | 返回内容的编码格式，可选值为xml和json，如果未指定，       |                                                |
|                  |          | 则默认为xml。如果返回为xml，则返回Content-Type为          |                                                |
|                  |          | application/xml，如果是json格式，则为application/json     | "json"                                         |
+------------------+----------+-----------------------------------------------------------+------------------------------------------------+


数字签名生成方法
~~~~~~~~~~~~~~~~

MOS开放API需要签名，遵循"`AWS API 2.0签名规范 <http://docs.aws.amazon.com/general/latest/gr/signature-version-2.html>`_"。
需要将请求内容按如下方法拼接，并按照指定Hash算法获得base64编码的签名字符串：

1. 全部字母大写的HTTP请求方法（GET或POST)，以换行(\\n)结束。例如::

    POST\n

2. 全部字母为小写的API服务主机名，以换行(\\n)结束。如果请求端口号为非默认端口（http非80端口，https非443），则需要加上端口，以冒号和host分隔。例如::

    mos-api.meituan.com\n

3. 请求绝对路径，以换行(\\n)结束。如果路径为空字符串，则为根路径"/"。例如::

    /\n

4. 所有请求参数按照请求参数名称的字典顺序排序，并以x-www-form-urlencoded编码拼接在一起。例如::

    AWSAccessKeyId=4ba303cc17454cc7904e044db2a3c912&Action=DescribeTemplates&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2013-07-20T15%3A28%3A04.000Z

完整的拼接的签名内容为::

    POST
    mos-api.meituan.com
    /
    AWSAccessKeyId=4ba303cc17454cc79............ersion=2&Timestamp=2013-07-20T15%3A28%3A04.000Z

5. 最后，以SECRET为Key对上述字符串以SHA256算法进行Hash后获得签名，结果为::

    EFVL9+vcx6tG9DCp04Ic+itrdCD/uZEhisS9amSSbig=


返回格式
~~~~~~~~

如果API请求成功，服务器返回HTTP 200，body包含返回数据。返回数据根据请求的Format参数进行格式化。

如果API请求失败，服务器返回HTTP 4xx，body包含错误信息，消息格式示例如下：

xml格式错误响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <ErrorResponse>
        <Error>
            <message>Not enough balance</message>
            <code>406</code>
        </Error>
    </ErrorResponse>

json格式错误响应

::

    {"ErrorResponse":
        {"Error":
            {"message": "Not enough balance",
             "code": 406
            }
        }
    }

其中，code为错误代码，message为错误消息。code和返回消息的HTTP错误代码相同。

模板API
-------

DescribeTemplates
~~~~~~~~~~~~~~~~~

列出所有用户可以使用的虚拟机模板，在创建虚拟机，更改虚拟机系统磁盘时，需要相关信息。

**请求参数：**

无

**输出字段：**

+--------------+--------+----------------------------+
| 字段名       | 类型   | 说明                       | 
+==============+========+============================+
| templateId   | string | 模板ID                     |
+--------------+--------+----------------------------+
| templateName | string | 模板名称                   |
+--------------+--------+----------------------------+
| size         | int    | 模板Image的字节大小(Bytes) |
+--------------+--------+----------------------------+
| checksum     | string | 模板Image的MD5 checksum    |
+--------------+--------+----------------------------+
| status       | string | 模板状态                   |
+--------------+--------+----------------------------+


**示例：**

请求URL

::

    https://10.168.44.160:8883?
        Action=DescribeTemplates&
        AUTHDATA

xml响应

::

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

::

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

套餐类型API
-----------

DescribeInstanceTypes
~~~~~~~~~~~~~~~~~~~~~

列出所有用户可以使用的虚拟机套餐类型，在创建虚拟机，更改虚拟机类型时，需要相关信息。

**请求参数：**

+------------------+---------+------+-----------------------------------------------+
| 参数名           | 类型    | 可选 | 说明                                          |
+==================+=========+======+===============================================+
| Limit            | integer | 可选 | 本次请求返回的数量                            |
+------------------+---------+------+-----------------------------------------------+
| Offset           | integer | 可选 | 本次请求返回的偏移量                          |
+------------------+---------+------+-----------------------------------------------+
| Filter.n.Name    | string  | 可选 | 过滤字段名称，n从1开始。支持字段名为：name    |
+------------------+---------+------+-----------------------------------------------+
| Filter.n.Value.m | string  | 可选 | 对应Filter.n.Name的过滤字段的匹配值，m从1开始 |
+------------------+---------+------+-----------------------------------------------+

**返回数据：**

返回InstanceTypeSet，包含如下子段：

+--------------+-------------+------------------------------+
| 字段名       | 类型        | 说明                         |
+==============+=============+==============================+
| InstanceType | complextype | 虚拟机类型定义               |
+--------------+-------------+------------------------------+
| Total        | integer     | 返回符合条件的虚拟机类型总量 |
+--------------+-------------+------------------------------+
| Limit        | integer     | 返回虚拟机类型的数量         |
+--------------+-------------+------------------------------+
| Offset       | integer     | 返回虚拟机类型的偏移量       |
+--------------+-------------+------------------------------+

InstanceType包含如下子段：

+-------------------+---------+--------------------------------------+
| 字段名            | 类型    | 说明                                 |
+===================+=========+======================================+
| instanceTypeId    | string  | 虚拟机类型ID                         |
+-------------------+---------+--------------------------------------+
| instanceType      | string  | 虚拟机类型名称                       |
+-------------------+---------+--------------------------------------+
| cpu               | integer | 该类型虚拟机CPU核数，单位为个        |
+-------------------+---------+--------------------------------------+
| memory            | integer | 该类型虚拟机内存大小，单位为MB       |
+-------------------+---------+--------------------------------------+
| volume            | integer | 该类型虚拟机虚拟存储大小，单位为MB   |
+-------------------+---------+--------------------------------------+
| internalBandwidth | integer | 该类型虚拟机内网接入带宽，单位为Mbps |
+-------------------+---------+--------------------------------------+
| externalBandwidth | integer | 该类型虚拟机外网接入带宽，单位为Mbps |
+-------------------+---------+--------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        Limit=1&
        Action=DescribeInstanceTypes&
        AUTHDATA

xml响应

::

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

::

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


帐户API
-------

GetBalance
~~~~~~~~~~

获得用户的当前帐户余额

**请求参数：**

无

**返回数据：**

+-----------+---------------+------------------------------------------------------+
| 字段名    | 类型          | 说明                                                 |
+===========+===============+======================================================+
| balance   | decimal(10,2) | 帐户余额                                             |
+-----------+---------------+------------------------------------------------------+
| timestamp | datetime      | 最后一次帐户余额发生变化的时间，iso8601格式。        |
|           |               | 如果该帐户从未发生过交易，则余额为0，无timestamp字段 |
+-----------+---------------+------------------------------------------------------+


**示例：**

请求URL

::

    https://10.168.44.160:8883?
        Action=GetBalance&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <GetBalanceResponse>
        <timestamp>2013-07-19T15:52:02Z</timestamp>
        <balance>16.66</balance>
    </GetBalanceResponse>

json响应

::

    {"GetBalanceResponse": 
        {"timestamp": "2013-07-19T15:52:02Z",
         "balance": 16.66
        }
    }

SSH密钥API
----------

DescribeKeyPairs
~~~~~~~~~~~~~~~~

列出用户所有的SSH Key pairs

**请求参数：**

+------------------+---------+------+-----------------------------------------------+
| 参数名           | 类型    | 可选 | 说明                                          |
+==================+=========+======+===============================================+
| Limit            | integer | 可选 | 本次请求返回的最多数量                        |
+------------------+---------+------+-----------------------------------------------+
| Offset           | integer | 可选 | 本次请求返回的偏移量                          |
+------------------+---------+------+-----------------------------------------------+
| Filter.n.Name    | string  | 可选 | 过滤字段名称，n从1开始。可能的值为：name      |
+------------------+---------+------+-----------------------------------------------+
| Filter.n.Value.m | string  | 可选 | 对应Filter.n.Name的过滤字段的匹配值，m从1开始 |
+------------------+---------+------+-----------------------------------------------+

**返回数据：**

返回KeyPairSet包含如下字段：

+---------+-------------+---------------------------+
| 字段名  | 类型        | 说明                      |
+=========+=============+===========================+
| KeyPair | complexType | 返回的SSH Key信息         |
+---------+-------------+---------------------------+
| Total   | integer     | 满足查询条件的SSH Key个数 |
+---------+-------------+---------------------------+
| Limit   | integer     | 实际返回的SSH Key个数     |
+---------+-------------+---------------------------+
| Offset  | integer     | 返回的偏移量              |
+---------+-------------+---------------------------+

KeyPair包含的字段：

+----------------+--------+-----------------------------------------+
| 字段名         | 类型   | 说明                                    |
+================+========+=========================================+
| keyId          | string | SSH Key的ID                             |
+----------------+--------+-----------------------------------------+
| keyName        | string | SSH Key的名称                           |
+----------------+--------+-----------------------------------------+
| keyFingerprint | string | SSH 公钥(public key)的指纹(fingerprint) |
+----------------+--------+-----------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        Action=DescribeKeyPairs&
        AUTHDATA

xml响应

::

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

::

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

ImportKeyPair
~~~~~~~~~~~~~

导入一个SSH Key

**请求参数：**

+-------------------+--------+------+---------------------+
| 参数名            | 类型   | 可选 | 说明                |
+===================+========+======+=====================+
| KeyName           | string | 必须 | SSH Key名称         |
+-------------------+--------+------+---------------------+
| PublicKeyMaterial | string | 必须 | SSH Key的public key |
+-------------------+--------+------+---------------------+

**返回数据：**

返回KeyPair包含的字段：

+----------------+--------+-----------------------------------------+
| 字段名         | 类型   | 说明                                    |
+================+========+=========================================+
| keyId          | string | SSH Key的ID                             |
+----------------+--------+-----------------------------------------+
| keyName        | string | SSH Key的名称                           |
+----------------+--------+-----------------------------------------+
| keyFingerprint | string | SSH 公钥(public key)的指纹(fingerprint) |
+----------------+--------+-----------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        KeyName=newkey&
        Action=ImportKeyPair&
        PublicKeyMaterial=ssh-rsa+AAAAB3Nza...OVL%2B2Y7R+qj%40dog%0A&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <ImportKeyPairResponse>
        <KeyPair>
            <keyId>0f4697a4-6439-4ae7-b6fe-be29ace2303c</keyId>
            <keyName>newkey</keyName>
            <keyFingerprint>0a:43:d9:7b:17:a1:24:26:9a:0e:ce:dc:f4:0a:03:44</keyFingerprint>
        </KeyPair>
    </ImportKeyPairResponse>

json响应

::

    {"ImportKeyPairResponse":
        {"KeyPair":
            {"keyId": "0f4697a4-6439-4ae7-b6fe-be29ace2303c",
             "keyName": "newkey",
             "keyFingerprint": "0a:43:d9:7b:17:a1:24:26:9a:0e:ce:dc:f4:0a:03:44"
            }
        }
    }

DeleteKeyPair
~~~~~~~~~~~~~

删除一个SSH Key

**请求参数：**

+---------+--------+------+-------------+
| 参数名  | 类型   | 可选 | 说明        |
+=========+========+======+=============+
| KeyName | string | 必须 | SSH Key名称 |
+---------+--------+------+-------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        KeyName=newkey&
        Action=DeleteKeyPair&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <DeleteKeyPairResponse>
        <return>True</return>
    </DeleteKeyPairResponse>

json响应

::

    {"DeleteKeyPairResponse":
        {"return": "True"
        }
    }

虚拟机API
---------

DescribeInstances
~~~~~~~~~~~~~~~~~

列出所有或指定的用户虚拟机实例。

**请求参数：**

+------------------+---------+------+--------------------------------------------------+
| 参数名           | 类型    | 可选 | 说明                                             |
+==================+=========+======+==================================================+
| InstanceId.n     | string  | 可选 | 指定虚拟机的ID，n从1开始                         |
+------------------+---------+------+--------------------------------------------------+
| InstanceName.n   | string  | 可选 | 指定虚拟机的Name，n从1开始                       |
+------------------+---------+------+--------------------------------------------------+
| Limit            | integer | 可选 | 本次请求返回的最多数量                           |
+------------------+---------+------+--------------------------------------------------+
| Offset           | integer | 可选 | 本次请求返回的偏移量                             |
+------------------+---------+------+--------------------------------------------------+
| Filter.n.Name    | string  | 可选 | 过滤字段名称，n从1开始。支持字段为：name, status |
+------------------+---------+------+--------------------------------------------------+
| Filter.n.Value.m | string  | 可选 | 对应Filter.n.Name的过滤字段的匹配值，m从1开始    |
+------------------+---------+------+--------------------------------------------------+

**返回数据：**

返回InstanceSet包含如下字段：

+----------+-------------+--------------------------+
| 字段名   | 类型        | 说明                     |
+==========+=============+==========================+
| Instance | complexType | 返回的虚拟机信息         |
+----------+-------------+--------------------------+
| Total    | integer     | 满足查询条件的虚拟机个数 |
+----------+-------------+--------------------------+
| Limit    | integer     | 实际返回的虚拟机个数     |
+----------+-------------+--------------------------+
| Offset   | integer     | 虚拟机的偏移量           |
+----------+-------------+--------------------------+

Instance包含的字段：

+----------------+---------+-----------------------------------------------+
| 字段名         | 类型    | 说明                                          |
+================+=========+===============================================+
| instanceId     | string  | 虚拟机的ID                                    |
+----------------+---------+-----------------------------------------------+
| instanceName   | string  | 虚拟机的名称                                  |
+----------------+---------+-----------------------------------------------+
| instanceType   | string  | 虚拟机的类型                                  |
+----------------+---------+-----------------------------------------------+
| instanceTypeId | string  | 虚拟机类型的ID                                |
+----------------+---------+-----------------------------------------------+
| status         | string  | 虚拟机的状态，可能值有running/ready/suspend等 |
+----------------+---------+-----------------------------------------------+
| cpu            | integer | 虚拟机的CPU核数                               |
+----------------+---------+-----------------------------------------------+
| memory         | integer | 虚拟机的内存大小，单位为MB                    |
+----------------+---------+-----------------------------------------------+
| volume         | integer | 虚拟机的总磁盘大小，单位为MB                  |
+----------------+---------+-----------------------------------------------+
| ipAddresses    | string  | 虚拟机的IP地址列表                            |
+----------------+---------+-----------------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        Limit=1&
        Offset=2&
        Action=DescribeInstances&
        AUTHDATA

xml响应

::

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

::

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

DescribeInstanceStatus
~~~~~~~~~~~~~~~~~~~~~~

获得指定虚拟机实例的状态。

**请求参数：**

+------------+--------+------+--------------+
| 参数名     | 类型   | 可选 | 说明         |
+============+========+======+==============+
| InstanceId | string | 必须 | 指定虚拟机ID |
+------------+--------+------+--------------+

**返回数据：**

返回InstanceStatus，包含status字段。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=testtest&
        Action=DescribeInstanceStatus&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <DescribeInstanceStatusResponse>
        <InstanceStatus>
            <status>ready</status>
        </InstanceStatus>
    </DescribeInstanceStatusResponse>

json响应

::

    {"DescribeInstanceStatusResponse": 
        {"InstanceStatus": 
            {"status": "ready"}
        }
    }

DescribeInstanceVolumes
~~~~~~~~~~~~~~~~~~~~~~~

列出指定虚拟机的所有虚拟磁盘的信息。

**请求参数：**

+------------+--------+------+--------------+
| 参数名     | 类型   | 可选 | 说明         |
+============+========+======+==============+
| InstanceId | string | 必须 | 指定虚拟机ID |
+------------+--------+------+--------------+

**返回数据：**

返回数据集InstanceVolumeSet，包含如下字段：

+----------------+-------------+----------------------+
| 字段名         | 类型        | 说明                 |
+================+=============+======================+
| InstanceVolume | complextype | 一个虚拟机磁盘的信息 |
+----------------+-------------+----------------------+

InstanceVolume包含如下字段信息：

+--------------+---------+---------------------------------------------------------+
| 字段名       | 类型    | 说明                                                    |
+==============+=========+=========================================================+
| instanceId   | string  | 虚拟机ID                                                |
+--------------+---------+---------------------------------------------------------+
| instanceName | string  | 虚拟机名称                                              |
+--------------+---------+---------------------------------------------------------+
| volumeId     | string  | 虚拟磁盘ID                                              |
+--------------+---------+---------------------------------------------------------+
| volumeName   | string  | 虚拟磁盘名称                                            |
+--------------+---------+---------------------------------------------------------+
| volumeSize   | integer | 磁盘大小，单位为MB                                      |
+--------------+---------+---------------------------------------------------------+
| cacheMode    | string  | 磁盘的缓存模式，可能值为none, writeback或writethrough,  |
|              |         | 缺省为none                                              |
+--------------+---------+---------------------------------------------------------+
| driver       | string  | 磁盘的驱动，可能值为virtio, ide和scsi，缺省为virtio     |
+--------------+---------+---------------------------------------------------------+
| index        | integer | 磁盘挂载在虚拟机上的序号，从0开始                       |
+--------------+---------+---------------------------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=testtest&
        Action=DescribeInstanceVolumes&
        AUTHDATA

xml响应

::

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

::

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

DescribeInstanceNetworkInterfaces
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

列出指定虚拟机实例的所有虚拟网络接口的信息。

**请求参数：**

+------------+--------+------+--------------+
| 参数名     | 类型   | 可选 | 说明         |
+============+========+======+==============+
| InstanceId | string | 必须 | 指定虚拟机ID |
+------------+--------+------+--------------+

**返回数据：**

返回数据集InstanceNetworkInterfaceSet，包含如下字段：

+--------------------------+-------------+--------------------------+
| 字段名                   | 类型        | 说明                     |
+==========================+=============+==========================+
| InstanceNetworkInterface | complextype | 一个虚拟机网络接口的信息 |
+--------------------------+-------------+--------------------------+

InstanceNetworkInterface包含如下信息：

+--------------+---------+-------------------------------------------+
| 字段名       | 类型    | 说明                                      |
+==============+=========+===========================================+
| instanceId   | string  | 虚拟机ID                                  |
+--------------+---------+-------------------------------------------+
| instanceName | string  | 虚拟机名称                                |
+--------------+---------+-------------------------------------------+
| networkId    | string  | 网络接口接入的虚拟机网络ID                |
+--------------+---------+-------------------------------------------+
| networkName  | string  | 网络接口接入的虚拟网络名称                |
+--------------+---------+-------------------------------------------+
| ipAddress    | string  | 网络接口的IP地址                          |
+--------------+---------+-------------------------------------------+
| macAddress   | string  | 网络接口的硬件地址                        |
+--------------+---------+-------------------------------------------+
| bandwidth    | integer | 网络接口带宽，单位为Mbps                  |
+--------------+---------+-------------------------------------------+
| driver       | string  | 驱动，可能值有virtio, e1000，缺省为virtio |
+--------------+---------+-------------------------------------------+
| index        | integer | 网络接口在虚拟机上的序号                  |
+--------------+---------+-------------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=testtest&
        Action=DescribeInstanceNetworkInterfaces&
        AUTHDATA

xml响应

::

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

::

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

GetPasswordData
~~~~~~~~~~~~~~~

获得指定虚拟机实例的初始帐户和密码信息。

**请求参数：**

+------------+--------+------+--------------+
| 参数名     | 类型   | 可选 | 说明         |
+============+========+======+==============+
| InstanceId | string | 必须 | 指定虚拟机ID |
+------------+--------+------+--------------+

**返回数据：**

+--------------+----------+------------------------------------------------------------------------+
| 字段名       | 类型     | 说明                                                                   |
+==============+==========+========================================================================+
| timestamp    | datetime | 指示初始帐号密码生成的时间                                             |
+--------------+----------+------------------------------------------------------------------------+
| account      | string   | 虚拟机的初始帐号                                                       |
+--------------+----------+------------------------------------------------------------------------+
| passwordData | string   | 虚拟机的初始帐号密码数据，如果虚拟机未使用SSH keypair，                |
|              |          | 则该数据为明文密码，否则，该数据为keypair公钥加密，                    |
|              |          | 需要使用该keypair的对应key文件解密                                     |
+--------------+----------+------------------------------------------------------------------------+
| keypairId    | string   | 如果虚拟机使用了keypair，则为该虚拟机使用的keypair的ID；否则无此字段   |
+--------------+----------+------------------------------------------------------------------------+
| keypairName  | string   | 如果虚拟机使用了keypair，则为该虚拟机使用的keypair的名称；否则无此字段 |
+--------------+----------+------------------------------------------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=GetPasswordData&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <GetPasswordDataResponse>
        <timestamp>2013-07-22T02:48:56Z</timestamp>
        <account>cirros</account>
        <passwordData>jwFN2C3Ngmgu</passwordData>
    </GetPasswordDataResponse>

json响应

::

    {"GetPasswordDataResponse": 
        {"timestamp": "2013-07-22T02:48:56Z", 
         "account": "cirros", 
         "passwordData": "jwFN2C3Ngmgu"
        }
    }

GetInstanceContractInfo
~~~~~~~~~~~~~~~~~~~~~~~

获得指定虚拟机实例的合同时间信息。

**请求参数：**

+------------+--------+------+--------------+
| 参数名     | 类型   | 可选 | 说明         |
+============+========+======+==============+
| InstanceId | string | 必须 | 指定虚拟机ID |
+------------+--------+------+--------------+

**返回数据：**

返回如下字段：

+-----------+----------+------------------------------------------+
| 字段名    | 类型     | 说明                                     |
+===========+==========+==========================================+
| startedAt | datetime | 虚拟机租约开始时间                       |
+-----------+----------+------------------------------------------+
| expireAt  | datetime | 虚拟机租约到期时间                       |
+-----------+----------+------------------------------------------+
| extendTo  | datetime | 如果未按期续费，虚拟机过期后保留截止时间 |
+-----------+----------+------------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=GetInstanceContractInfo&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <GetInstanceContractInfoResponse>
        <startedAt>2013-07-22T03:00:00Z</startedAt>
        <extendTo>2013-07-26T03:00:00Z</extendTo>
        <expireAt>2013-07-25T03:00:00Z</expireAt>
    </GetInstanceContractInfoResponse>

json响应

::

    {"GetInstanceContractInfoResponse":
        {"startedAt": "2013-07-22T03:00:00Z",
         "extendTo": "2013-07-26T03:00:00Z",
         "expireAt": "2013-07-25T03:00:00Z"
        }
    }

StartInstance
~~~~~~~~~~~~~

启动指定虚拟机实例。虚拟机在ready状态时才能成功启动。

**请求参数：**

+------------+--------+------+----------------------+
| 参数名     | 类型   | 可选 | 说明                 |
+============+========+======+======================+
| InstanceId | string | 必须 | 启动的虚拟机ID或名称 |
+------------+--------+------+----------------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=StartInstance&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <StartInstanceResponse>
        <return>True</return>
    </StartInstanceResponse>

json响应

::

    {"StartInstanceResponse":
        {"return": "True"
        }
    }

StopInstance
~~~~~~~~~~~~

停止指定虚拟机实例。只有虚拟机在running状态时才能成功停止虚拟机。如果指定强制停止，则虚拟机进程立即退出，可能会造成虚拟机内部数据丢失。否则，虚拟机将试图软关机，30秒超时后，如果虚拟机实例还未停止，则强制停止。

**请求参数：**

+------------+---------+------+------------------+
| 参数名     | 类型    | 可选 | 说明             |
+============+=========+======+==================+
| InstanceId | string  | 必须 | 指定的虚拟机ID   |
+------------+---------+------+------------------+
| Force      | boolean | 可选 | 是否强制立即停止 |
+------------+---------+------+------------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=StopInstance&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <StopInstanceResponse>
        <return>True</return>
    </StopInstanceResponse>

json响应

::

    {"StopInstanceResponse":
        {"return": "True"
        }
    }

RebootInstance
~~~~~~~~~~~~~~

重启指定虚拟机实例。

**请求参数：**

+------------+--------+------+--------------+
| 参数名     | 类型   | 可选 | 说明         |
+============+========+======+==============+
| InstanceId | string | 必须 | 指定虚拟机ID |
+------------+--------+------+--------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=RebootInstance&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <RebootInstanceResponse>
        <return>True</return>
    </RebootInstanceResponse>

json响应

::

    {"RebootInstanceResponse":
        {"return": "True"
        }
    }

RebuildInstanceRootImage
~~~~~~~~~~~~~~~~~~~~~~~~

重置指定虚拟机实例的的系统磁盘镜像。

**请求参数：**

+------------+--------+------+----------------------------------------------+
| 参数名     | 类型   | 可选 | 说明                                         |
+============+========+======+==============================================+
| InstanceId | string | 必须 | 需要重置系统盘镜像的InstanceId               |
+------------+--------+------+----------------------------------------------+
| ImageId    | string | 可选 | 指定系统盘的源模板镜像ID，如果不指定该参数， |
|            |        |      | 则使用原来的模板镜像重置系统盘               |
+------------+--------+------+----------------------------------------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=RebuildInstanceRootImage&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <RebuildInstanceRootImageResponse>
        <return>True</return>
    </RebuildInstanceRootImageResponse>

json响应

::

    {"RebuildInstanceRootImageResponse":
        {"return": "True"
        }
    }

CreateInstance
~~~~~~~~~~~~~~

创建虚拟机实例。*注意：该操作涉及帐户扣费，请保证帐户有足够余额，否则将创建失败*。

**请求参数：**

+--------------+--------+------+------------------------------------------------------------------+
| 参数名       | 类型   | 可选 | 说明                                                             |
+==============+========+======+==================================================================+
| ImageId      | string | 必须 | 镜像模板ID                                                       |
+--------------+--------+------+------------------------------------------------------------------+
| InstanceType | string | 必须 | 创建的虚拟机类型ID或名称                                         |
+--------------+--------+------+------------------------------------------------------------------+
| Duration     | string | 可选 | 创建的虚拟机的时间，格式为数字\+H/M，例如1H, 72H或者1M。缺省为1M |
+--------------+--------+------+------------------------------------------------------------------+
| InstanceName | string | 可选 | 指定创建的虚拟机的名称                                           |
+--------------+--------+------+------------------------------------------------------------------+
| KeyName      | string | 可选 | 指定创建的虚拟机使用的SSH Keypair                                |
+--------------+--------+------+------------------------------------------------------------------+

**返回数据：**

如果成功返回生成的Instance信息，包含如下字段：

+----------------+---------+-----------------------------------------------+
| 字段名         | 类型    | 说明                                          |
+================+=========+===============================================+
| instanceId     | string  | 虚拟机的ID                                    |
+----------------+---------+-----------------------------------------------+
| instanceName   | string  | 虚拟机的名称                                  |
+----------------+---------+-----------------------------------------------+
| instanceType   | string  | 虚拟机的类型                                  |
+----------------+---------+-----------------------------------------------+
| instanceTypeId | string  | 虚拟机类型的ID                                |
+----------------+---------+-----------------------------------------------+
| status         | string  | 虚拟机的状态，可能值有running/ready/suspend等 |
+----------------+---------+-----------------------------------------------+
| cpu            | integer | 虚拟机的CPU核数                               |
+----------------+---------+-----------------------------------------------+
| memory         | integer | 虚拟机的内存大小，单位为MB                    |
+----------------+---------+-----------------------------------------------+
| volume         | integer | 虚拟机的总磁盘大小，单位为MB                  |
+----------------+---------+-----------------------------------------------+
| ipAddresses    | string  | 虚拟机的IP地址列表                            |
+----------------+---------+-----------------------------------------------+

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        ImageId=d1620e45-c561-42e7-a2a4-53ae0a389bb9&
        Duration=72H&
        InstanceType=small_net&
        Action=CreateInstance&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <CreateInstanceResponse>
        <Instance>
            <instanceId>022a58da-5cee-4589-9e6a-f54fa1abd269</instanceId>
            <instanceName>system</instanceName>
            <instanceType>small_net</instanceType>
            ...
        </Instance>
    </CreateInstanceResponse>

json响应

::

    {"CreateInstanceResponse": 
        {"Instance": 
            {"instanceId": "022a58da-5cee-4589-9e6a-f54fa1abd269", 
             "instanceName": "system", 
             "instanceType": "small_net",
             ...
            }
        }
    }

TerminateInstance
~~~~~~~~~~~~~~~~~

删除指定虚拟机实例。

**请求参数：**

+------------+--------+------+----------------------+
| 参数名     | 类型   | 可选 | 说明                 |
+============+========+======+======================+
| InstanceId | string | 必须 | 删除的虚拟机ID或名称 |
+------------+--------+------+----------------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=TerminateInstance&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <TerminateInstanceResponse>
        <return>True</return>
    </TerminateInstanceResponse>

json响应

::

    {"TerminateInstanceResponse":
        {"return": "True"
        }
    }

RenewInstance
~~~~~~~~~~~~~

续期指定虚拟机实例。*注意：该操作涉及帐户扣费，请保证帐户有足够余额，否则将续期失败*。

**请求参数：**

+------------+--------+------+----------------------------------------------------+
| 参数名     | 类型   | 可选 | 说明                                               |
+============+========+======+====================================================+
| InstanceId | string | 必须 | 续期的虚拟机ID或名称                               |
+------------+--------+------+----------------------------------------------------+
| Duration   | string | 可选 | 指定续期时间，格式为数字\+H/M（小时/月），例如1H， |
|            |        |      | 72H或者1M。如果不指定，缺省为1M                    |
+------------+--------+------+----------------------------------------------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Duration=72H&
        Action=RenewInstance&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <RenewInstanceResponse>
        <return>True</return>
    </RenewInstanceResponse>

json响应

::

    {"RenewInstanceResponse":
        {"return": "True"
        }
    }

ChangeInstanceType
~~~~~~~~~~~~~~~~~~

更改指定虚拟机实例的类型。*注意：该操作涉及帐户扣费，请保证帐户有足够余额，否则将更改失败*。

+--------------+--------+------+-------------------------------------------------------------+
| 参数名       | 类型   | 可选 | 说明                                                        |
+==============+========+======+=============================================================+
| InstanceId   | string | 必须 | 更改类型的虚拟机ID或名称                                    |
+--------------+--------+------+-------------------------------------------------------------+
| InstanceType | string | 必须 | 更改的虚拟机类型                                            |
+--------------+--------+------+-------------------------------------------------------------+
| Duration     | string | 可选 | 更改后虚拟机的租期时间，格式为数字+H/M（小时/月），例如1H， |
|              |        |      | 72H或者1M。如果不指定，缺省为1M                             |
+--------------+--------+------+-------------------------------------------------------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=ChangeInstanceType&
        InstanceType=small_net&
        Duration=1M&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <ChangeInstanceTypeResponse>
        <return>True</return>
    </ChangeInstanceTypeResponse>

json响应

::

    {"ChangeInstanceTypeResponse":
        {"return": "True"
        }
    }

GetInstanceMetadata
~~~~~~~~~~~~~~~~~~~

获取指定虚拟机实例的元数据。

**请求参数：**

+------------+--------+------+----------------------------+
| 参数名     | 类型   | 可选 | 说明                       |
+============+========+======+============================+
| InstanceId | string | 必须 | 获取元数据的虚拟机ID或名称 |
+------------+--------+------+----------------------------+

**返回数据：**

返回InstanceMetadata数据集，包含所有key-value的元数据。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Action=GetInstanceMetadata&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <GetInstanceMetadataResponse>
        <InstanceMetadata>
            <os_version>2011.08</os_version>
            <os_name>Linux</os_name>
            <os_distribution>Cirros</os_distribution>
        </InstanceMetadata>
    </GetInstanceMetadataResponse>

json响应

::

    {"GetInstanceMetadataResponse": 
        {"InstanceMetadata": 
            {"os_version": "2011.08",
             "os_name": "Linux",
             "os_distribution": "Cirros"
            }
        }
    }

PutInstanceMetadata
~~~~~~~~~~~~~~~~~~~

设置指定虚拟机实例的元数据。

**请求参数：**

+------------+--------+------+----------------------------------+
| 参数名     | 类型   | 可选 | 说明                             |
+============+========+======+==================================+
| InstanceId | string | 必须 | 设置元数据的虚拟机ID或名称       |
+------------+--------+------+----------------------------------+
| Name.n     | string | 必须 | 指定第n个元数据的key，n从1开始   |
+------------+--------+------+----------------------------------+
| Value.n    | string | 必须 | 指定第n个元数据的value，n从1开始 |
+------------+--------+------+----------------------------------+

**返回数据：**

成功则返回值return为True；否则返回错误信息。

**示例：**

请求URL

::

    https://10.168.44.160:8883?
        InstanceId=system&
        Name.1=test7d&
        Value.1=1&
        Action=PutInstanceMetadata&
        AUTHDATA

xml响应

::

    <?xml version="1.0" encoding="utf-8"?>
    <PutInstanceMetadataResponse>
        <return>True</return>
    </PutInstanceMetadataResponse>

json响应

::

    {"PutInstanceMetadataResponse":
        {"return": "True"
        }
    }

