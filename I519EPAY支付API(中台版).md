# I519 行動付API(中台版)

## 總覽

此文件描述了所有i519對配合之商家之間資料交換的API。

所有通訊是透過 HTTP 並使用指定的 HTTP Method 來完成。Request 是 JSON 格式，內容類型都是 `application/json`。

## Base URL

```
https://{evn}.i519.com.tw/payment/
```

- `{evn}` 為環境參數： `test` 為測試環境, `cloud` 為正式環境。

## Token

Client 每次呼叫 i519 API 前, 必須取得 access token, 之後將 token 連同資料一起呼叫 API. 

![Token](token.jpg)

每次核發的 token 時限為 30 秒, 時限內可不限次數的呼叫 i519 API,  超過時限則 client 必須重新取得 token.

### API

```
POST {baseUrl}/api/token
```

### Body

Reuest body 必須是 JSON 格式。下列為支援的欄位:

```json
{
	"channel": "<<註冊後由i519提供>>", 
	"client": "<<註冊後由i519提供>>", 
	"password": "<<註冊後由i519提供>>"
}
```

- `channel` - 用戶端通路代號
- `client` -  用戶端識別碼
- `password` - 密碼

### Output

```json
{
	"token": "generated-token"
}
```

- `token` - 核發 token

### HTTP Status Code

- 在取得 Token 過程中, 發生任何例外 (如 password 錯誤) 導致失敗, i519 將會回傳 `401`
- 在跟任何 API 串接時,   Client 傳入 Token 驗證失敗 (如 token 已過期), i519 也將回傳 `401`

## 資料上傳

### API

```
POST {baseUrl}/payment/api/details?t={token}
```

### Body

Reuest body 必須是 JSON 格式。下列為支援的欄位：

| Name | 名稱 | 說明 | 最大長度 | 必填 | 型態 |
|------|-----|---------|-----|------|-------|
| `channelCode` | 通路代號 | 註冊後由i519提供 | 50 | Y | C | |
| `mobile` | 消費者手機號碼 | | 12 | Y | C |
| `email` | 消費者email | | 50 | N | C |
| `payNo` | 交易序號 | 各商家可對應銷帳使用之號碼 (唯一不可重複) | 20 | Y | C |
| `payType` | 繳費模式 | `EPAY` | 10 | Y | C |
| `payAmount` | 交易金額 | 數字，不包含 `-` | 10 | Y | I |
| `payExpiryDate` | 繳費期限 | `yyyy-MM-dd HH:mm[:ss]` | 19 | Y | C |
| `remark` | 交易內容摘要 |若payType選擇`EPAY`，中台請輸入`EPAY;`開頭的文字。 | 255 | N | C |
| `username` | 經手人帳號 | 經手人(業績歸屬)之使用者帳號 | 50 | Y | C |
| `returnUrl` | 已繳費通知url | 客戶端系統用來接收已繳費通知的rul，若不帶此項或帶`null`則套用預設url | 255 | N | C |

> 型態: `C`文字；`D`日期；`I`整數；`B`布林

```json
[{
  "channelCode": "a.test.channel.code",
  "mobile": "0988-888-888",
  "email": "support@i519.com.tw",
  "payNo": "A001",
  "payType": "EPAY",
  "payAmount": 1000,
  "remark": "EPAY;A1111",
  "payExpiryDate": "2016-11-03 11:15",
   "username": "test"
}, {
	...
}]
```

### Output

| Name | 名稱 | 說明 |
|-----|------|------|
| `code` | 代碼 | |
| `result` | payNo |新增的交易序號 |
| `message` | 訊息array | 訊息補充說明 |


以 json 格式回傳

```json
{
 "code": "...",
 "result": [
        {
            "payNo": "...",
            "pinCode": "..."
        }
    ],
 "message": [
 		"...",
 		"..."
 	]
}
```

#### 訊息代碼清單

| Type | Code | 說明 | 問題描述 |
|------|------|------|--------|
| INFO | `I000` | 處理成功 | |
| ERROR | `E001` | 傳送資料無內文 | Input jsonData無內容 |
| ERROR | `E002` | 傳送資料格式錯誤 | Input jsonData中，任一欄位格式有誤 |
| ERROR | `E003` | 傳送資料內容錯誤 | 資料內容正確性驗證不通過 |
| ERROR | `E999` | 未知錯誤 | |

### 程式範例

貴公司開發人員可以參考以下程式碼, 但套用前請先調整`url`及`data`的內容:

```html
<!doctype html public "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
    <head>
        <script src="https://code.jquery.com/jquery-3.1.1.min.js" integrity="sha256-hVVnYaiADRTO2PzUGmuLJr8BLUSjGIZsDYGmIJLv2b8=" crossorigin="anonymous"></script>
    </head>

    <body>
        <button type="button"">Click to send data to i519</button>
        <script type="text/javascript">
            $("button").click(function(){
                $.ajax({
                    method: "POST",
                    url: "https://{evn}.i519.com.tw/payment/api/details",
                    contentType: "application/json; charset=UTF-8",
                    data: JSON.stringify([{
                          "channelCode": "a.test.channel.code",
                          "mobile": "0988-888-888",
                          "email": "support@i519.com.tw",
                          "payNo": "A001",
                          "payType": "CSTORE",
                          "collectAgc": "FAMIPORT",
                          "payAmount": 1000,
                          "payExpiryDate": "2017-11-03 11:15",
                          "username": "test"
                        }, {
                          "channelCode": "a.test.channel.code",
                          "mobile": "0988-888-888",
                          "email": "support@i519.com.tw",
                          "payNo": "A002",
                          "payType": "ATM",
                          "payAmount": 2000,
                          "payExpiryDate": "2017-11-03 11:15",
                          "username": "test"
                        }]),
                    dataType: "json"
                }).done(function(data, status, xhr){
                    console.log("response status code: " + xhr.status);
                    if (xhr.status !== 200) {
                        console.error("Something went wrong..");
                    }
                });
            });
        </script>
    </body>
</html>
```

## 接收交易結果(只適用於EPAY)

### API

```
POST {baseUrl}/payment/api/payback?t={token}
```

### Body

Reuest body 必須是 JSON 格式。下列為支援的欄位：

| Name | 名稱 | 說明 | 最大長度 | 必填 | 型態 |
|------|-----|---------|-----|------|-------|
| `channelCode` | 通路代號 | 註冊後由i519提供 | 50 | Y | C | |
| `mobile` | 消費者手機號碼 | | 12 | Y | C |
| `payNo` | 交易序號 | 各商家可對應銷帳使用之號碼 (唯一不可重複) | 20 | Y | C |
| `payType` | 繳費模式 |  `EPAY` | 10 | Y | C |
| `collectAgc` | 代收機構 | `null`, 若payType選擇`EPAY`才**可且需**填入: `EZWallet`, `LPM`, `JKO`,`PlusPay`,`PXPay` | 10 | Y | C |
| `payAmount` | 交易金額 | 數字，不包含 `-` | 10 | Y | I |
| `payExpiryDate` | 繳費期限 | `yyyy-MM-dd HH:mm[:ss]`| 19 | Y | C |
| `remark` | 交易內容摘要 | 可將ID回寫(第一碼英文+末3碼)| 255 | N | C | 
| `paymentStatus` | 繳費狀態 | `UNPAID`, `PAID`| 50 | Y | C |
| `payDate` | 繳費日期 | `yyyy-MM-dd HH:mm:ss`| 19 | Y | C |

> 型態: `C`文字；`D`日期；`I`整數；`B`布林

```json
[{
  "channelCode": "a.test.channel.code",
  "mobile": "0988-888-888", 
  "payNo": "A001",
  "payType": "EPAY",
  "collectAgc": "EZWallet",
  "payAmount": 1000,
  "payExpiryDate": "2016-11-03 11:15",
  "remark": "A789",
  "paymentStatus": "PAID",
  "payDate": "2016-11-03 11:15:23"
}, {
	...
}]
```

### Output

| Name | 名稱 | 說明 |
|-----|------|------|
| `code` | 代碼 | |
| `result` | payNo | |
| `message` | 訊息array | 訊息補充說明 |


以 json 格式回傳

```json
{
 "code": "...",
 "result": [
        {
            "payNo": "..."
        }
    ],
 "message": [
 		"...",
 		"..."
 	]
}
```

#### 訊息代碼清單

| Type | Code | 說明 | 問題描述 |
|------|------|------|--------|
| INFO | `I000` | 處理成功 | |
| ERROR | `E001` | 傳送資料無內文 | Input jsonData無內容 |
| ERROR | `E002` | 傳送資料格式錯誤 | Input jsonData中，任一欄位格式有誤 |
| ERROR | `E003` | 傳送資料內容錯誤 | 資料內容正確性驗證不通過 |
| ERROR | `E999` | 未知錯誤 | |
