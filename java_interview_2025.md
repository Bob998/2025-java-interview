
# IMO SMS Api文档
此技术文档提供有关 **Imo SMS API**的相关内容，包括鉴权，请求短信，发送回执等内容。
网关供应商按照此文档开发后，提供账号，密码给IMO侧，IMO即可接入该网关。

---

## **Table of Contents**
- [鉴权](#鉴权)
- [请求短信](#请求短信)
- [发送回执](#发送回执)

---
    
### **鉴权**
Imo SMS API使用**access token**来鉴权，建议网关控制token的过期时间为15分钟。

- **IP白名单**:
  ```
    - 5.150.157.32/27
    - 5.150.157.64/27
    - 5.150.157.96/27
    - 164.90.99.32/27
    - 164.90.101.32/27
    - 83.229.91.0/24
    - 199.30.242.0/24
  ```
- **请求参数**:
  - **时间戳**: 毫秒级时间
  - **账号**: 网关供应商生成的账号信息
  - **密码**: 网关供应商生成的密码信息
  
- **加密算法**:
    - `HMAC-SHA1`
    - `HMAC-SHA256`

- **生成规则**: 
  - `token = base64(HMAC(key=${password}, message="${userkey}:${password}:${timestamp}"))`

- **具体case**
- Python
  ```python
    import hmac
    import hashlib
    import base64
    import time

    def generate_hmac_token(userkey, password, timestamp=None):
        """
        :param userkey: str, 用户账号
        :param password: str, 密码，用作 HMAC key
        :param timestamp: int or None, 毫秒级时间戳，None 时使用当前时间
        :return: str, Base64 编码 token
        """
        timestamp = int(time.time() * 1000) if timestamp is None else int(timestamp)
        
        msg = f"{userkey}:{password}:{timestamp}".encode('utf-8')
        
        mac = hmac.new(password.encode('utf-8'), msg, hashlib.sha1)
        
        token = base64.b64encode(mac.digest()).decode('utf-8')
        
        return token

    if __name__ == "__main__":
        userkey = "userkey_example"
        password = "password_example"
        timestamp = "1740055707181"
        token = generate_hmac_token(userkey, password, timestamp)
        print(token)
  ```

### **请求短信**
网关供应商按照此标准开发，IMO 侧通过以下参数发送短信请求。
- **短信下发请求**
  - **Content-type**: `application/json`
  - **Authorization**:  <Bearer `${token}`>
  - **Body Data**:
    | 字段      | 类型    | 描述     | 是否必须             |最大值
    |---|---|---|---|---|
    | `to` | string | 接收短信的电话号码，必须符合 E.164 国际格式（示例: +8801866******）| 必须                     | 24 |
    | `sender_id`     | string  | 展示的发送者ID，例如品牌名或系统标识（示例: IMO）    | 必须                      | 128|
    | `channel`  | string  | 发送通道，用于区分短信发送的类型或来源渠道，例如不同短信服务提供商、网络类型或发送场景（如国际/国内通道）       | 必须                      |        128
    | `type`     | string  | 文案类型（示例: otp, marketing, notification），大小写敏感。其中otp为验证码类短信，marketing为营销类短信，notification为通知类短信 | 必须                 | 128 |
    | `text`     | string  | 短信文案内容，特殊字符需 UTF-8 编码 | 必须                      | 256 |
    | `timestamp`| long    | 毫秒级时间戳，用于校验 token 的有效性   | 必须                      | 20 |
    | `user_key`  | string  | 用户账号信息，用于校验 token 的合法性    | 必须                      | 128 |
    | `algorithm`| string  | 加密算法，大小写敏感，目前仅支持 `HMAC-SHA1`或 `HMAC-SHA256`    | 必须                      |32|
    | `callback_url` | string  | 回执通知接收 URL，用于接收短信发送结果回调              | 必须                      |   128 |
    | `custom`   | string  | 短信请求的透传字段，用于业务追踪或标识  | 可选                      |256|

  - **请求具体case**
    ```
    curl --location 'https://XXX.com/XXX' \
      --header 'Content-Type: application/json' \
      --header 'Authorization: Bearer ${token}' \
      --data '{
          "to": "+14155550000",
          "sender_id": ${sender_id},
          "channel": ${channel},
          "type": "otp",
          "text": "your verification code is 1234",
          "timestamp": 1740055707181,
          "user_key": ${user_key},
          "algorithm": "HMAC-SHA1",
          "callback_url": "https://imo.im/sms/sms_delivery/verification/gateway_name",
          "custom": "your custom data is 123"
      }'
    ```
- **请求返回**
  - **数据结构**
      |字段| 类型| 描述 |是否必须
      |---|---|---|---|
      |msg_id| string| 请求的唯一标识 ID，用于追踪短信发送请求的唯一性|必须
      |status| string| 请求状态，大小写敏感，目前仅支持success/not_auth/send_failed/unknown`。其中success为请求成功，not_auth为鉴权失败，send_failed为请求失败，unknown为未知状态 |必须
      |message| string| 请求描述，和status对应，可自定义|必须
      |custom| string| 透传字段，用于业务追踪或标识 |可选

  - **请求成功case**
    ```json
      {
        "msg_id": "dG1740399774467000000BW",
        "status": "success",
        "message": "请求成功",
        "custom": "your custom data is 123"
      }
    ```
  - **请求失败case**
    ```json
      {
        "msg_id": "",
        "status": "not_auth",
        "message": "鉴权失败"
      }
    ```

### **发送回执**
网关供应商需按照本标准开发，并通过以下参数向 IMO 侧发送回执。每个 msg_id 仅允许返回一条回执。若发送失败，可进行最多 3 次重试，超过次数后丢弃。建议在请求成功后的 10 分钟内 返回回执，逾期未返回的，IMO 侧将视为短信状态未知。
- **数据结构**  
  |字段| 类型| 描述|是否必须|最大值|
  |---|---|---|---|---|
  |to| string| 接收短信的手机号码，必须符合 国际号码 E.164 格式（示例: +8801866******）|必须|24
  |msg_id| string| 请求的唯一标识 ID，用于追踪短信发送请求的唯一性 |必须|128
  |status| string| 短信发送回执状态，大小写敏感，目前仅支持 `delivered` 或 `undelivered`。其中delivered为已送达， undelivered为未送达|必须|128
  |message| string| 回执状态的具体描述，和status对应，可自定义|必须|256
  |price| float | 短信请求的单价，默认为0.00，单位为`美元`（USD）|必须|128
  |count| float | 短信请求的条数，默认为1.00|必须|128
  |cost| float | 短信请求的成本，其值为price*count，默认为0.00，单位为`美元`（USD）|必须|128
  |mccmnc| string| 手机号码的 MCC/MNC 信息（移动国家码/移动网络码），用于运营商识别|可选|32
  |custom| string| 透传字段，用于业务追踪或标识|可选|256
   
- **DLR具体case**
  ```
    curl --location 'https://imo.im/sms/sms_delivery/verification/gateway_name' \
      --header 'Content-Type: application/json' \
      --data '{
          "to":"+8618855555555",
          "msg_id":"dG1740399774467000000BW",
          "status":"delivered",
          "message": "success",
          "price": 0.01,
          "count": 1.00,
          "cost": 0.01,
          "mccmnc": "",
          "custom": "your custom data is 123"
      }'
    ```
- **回执请求case**
  - **回执已送达case**
    ```json
      {
        "to":"+8618855555555",
        "msg_id":"dG1740399774467000000BW",
        "status":"delivered",
        "message": "已送达",
        "price": 0.01,
        "count": 1.00,
        "cost": 0.01,
        "mccmnc": "",
        "custom": "your custom data is 123"
      }
    ```
  - **回执无法送达case** 
    ```json
      {
        "to":"+8618855555555",
        "msg_id":"dG1740399774467000000BW",
        "status":"undelivered",
        "message": "无法送达",
        "price": 0.00,
        "count": 1.00,
        "cost": 0.00,
      }
    ```
- **请求返回**
  - **数据结构**
      |字段| 类型| 描述 |  
      |---|---|---|
      |status| 字符串| 请求状态，大小写敏感，目前仅支持`success`/`failed` |
      |message| 字符串| 请求描述，和status对应，可自定义|
  - **请求返回成功case**
    ```json
      {
        "status": "success",
        "message": "发送成功"
      }
    ```
  - **请求返回失败case**
    ```json
      {
        "status": "failed",
        "message": "发送失败"
      }
    ```