1、Qt与JS通信的两种方式

​	第一种：QWebChannel+QWebEnginePage;这种方式在测试访问静态网页的时候没有问题，但是在访问vue写的相关的网页时，会出现js函数找不到的问题；所以推荐使用第二种方式

JsContext.h

```c++
class JsContext : public QObject
{
    Q_OBJECT
public:
    JsContext(QObject *parent = nullptr);


    void sendMsgToWeb(const QString& method, const QString& msg);

    bool requestLocalServer(QString req_type, QString &reply_data);
    bool requestLocalServer2(QString req_type, QString &reply_data);

    static void *recvForServerKey(void *arg);
signals:
    void recvdMsg(const QString& msg);
    void webToVideoUrl(QString url);

    void sendWebMsg(QString msg);
    void sendWebKey(QString skey);

public slots://客户端接收web数据的接口
    void onLogin(const QString& username, const QString& passwd);
    void onLiveVideo(const QString& url);
    void onGetKey();

private:
    pthread_t m_pid;
    QWebEngineView *m_webView;
    QWebEnginePage *m_page;
};


```

JsContext.cpp

```
#include "JsContext.h"
#include <QMessageBox>
#include <zmq.h>
#include "json/json.h"
#include "configmanager.h"

#if defined(Q_OS_UNIX)
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#endif

JsContext::JsContext(QWebEngineView *webView, QObject *parent) : QObject(parent)
{
    m_webView = webView;
    m_page = m_webView->page();
}

void JsContext::onLogin(const QString& username, const QString& passwd)
{

}

void JsContext::onLiveVideo(const QString& url)
{
    emit webToVideoUrl(url);
}

void JsContext::onGetKey()
{
    QMessageBox::information(NULL, "222", "222", QMessageBox::Yes, QMessageBox::Yes);
    pthread_create(&m_pid, NULL, recvForServerKey, this);
}

//直接调用JS的函数runJavaScript
void JsContext::sendMsgToWeb(const QString& method, const QString& msg)
{
    if(0 == method.compare("key"))
    {
        m_page->runJavaScript(QString("recvKey('%1');").arg(msg));
    } else
    {
        m_page->runJavaScript(QString("recvMessage('%1');").arg(msg));
    }
}
```

QWeb.html

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webchannel测试</title>
</head>
<body>
    <p>webchannel test</p>
    <script type="text/javascript" src="qwebchannel.js"></script>
    <script type="text/javascript" src="msgutils2.js"></script>
    <input id="待发送消息" type="text" name="msgText" />
    <input type="button" value="发送消息到浏览器" onclick="onBtnSendMsg()" />
</body>
</html>
```

msgutils.js

```javascript
var context;
// 初始化
function init()
{
    if (typeof qt != 'undefined')
    {
        new QWebChannel(qt.webChannelTransport, function(channel)
        {
        context = channel.objects.context;
        }
        );
    }
    else
    {
        alert("qt对象获取失败！");
    }
}
// 向qt发送消息
function sendMessage(msg)
{
    if(typeof context == 'undefined')
    {
        alert("context对象获取失败！");
    }
    else
    {
        context.onLiveVideo(msg);
    }
}

function recvMessage(msg)
{
    alert("接收到Qt发送的消息11：" + msg);
}

function recvkey(msg)
{
    alert("key:" + msg);
}
// 控件控制函数
function onBtnSendMsg()
{
    var cmd = document.getElementById("待发送消息").value;
    sendMessage(cmd);   
}
init();
```



第二种方法：只用QWebChannel

JsContext.h

```c++
#include <QObject>
#include <QWebEnginePage>
#include <pthread.h>

class JsContext : public QObject
{
    Q_OBJECT
public:
    JsContext(QObject *parent = nullptr);


    void sendMsgToWeb(const QString& method, const QString& msg);

    bool requestLocalServer(QString req_type, QString &reply_data);
    bool requestLocalServer2(QString req_type, QString &reply_data);

    static void *recvForServerKey(void *arg);
signals:
    void recvdMsg(const QString& msg);
    void webToVideoUrl(QString url);

    void sendWebMsg(QString msg);
    void sendWebKey(QString skey);

public slots://客户端接收web数据的接口
    void onLogin(const QString& username, const QString& passwd);
    void onLiveVideo(const QString& url);
    void onGetKey();
};
```

JsContext.cpp

```c++
#include "JsContext.h"
#include <QMessageBox>
#include <zmq.h>
#include "json/json.h"
#include "configmanager.h"

#if defined(Q_OS_UNIX)
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#endif

JsContext::JsContext(QWebEngineView *webView, QObject *parent) : QObject(parent)
{
}

void JsContext::onLogin(const QString& username, const QString& passwd)
{

}

void JsContext::onLiveVideo(const QString& url)
{
    emit webToVideoUrl(url);
}

void JsContext::onGetKey()
{
    QMessageBox::information(NULL, "222", "222", QMessageBox::Yes, QMessageBox::Yes);
    pthread_create(&m_pid, NULL, recvForServerKey, this);
}

void JsContext::sendMsgToWeb(const QString& method, const QString& msg)
{
    if(0 == method.compare("key"))
    {
        QString skey = msg.toHtmlEscaped();
        emit sendWebKey(skey);
    } else
    {
        QString smsg = msg.toHtmlEscaped();
        emit sendWebMsg(smsg);
    }
}
```

msgutils2.js

```javascript
var context;
// 初始化

var recvMessage = function(msg) {
      alert("接收到Qt发送的消息：" + msg);
}

function init()
{
    if (typeof qt != 'undefined')
    {
        new QWebChannel(qt.webChannelTransport, function(channel)
        {
        context = channel.objects.context;
		
		context.sendWebMsg.connect(recvMessage);
        }
        );
    }
    else
    {
        alert("qt对象获取失败！");
    }
}
// 向qt发送消息
function sendMessage(msg)
{
    if(typeof context == 'undefined')
    {
        alert("context对象获取失败！");
    }
    else
    {
        context.onMsg(msg, "1234", "5678");
    }
}

function recvMessage1(msg)
{
    alert("接收到Qt发送的消息：" + msg);
}

function recvkey(msg)
{
    alert("key:" + msg);
}
// 控件控制函数
function onBtnSendMsg()
{
    var cmd = document.getElementById("待发送消息").value;
    sendMessage(cmd);   
}
init();
```



存在的问题，网页图片显示不出来。（目前只在银河麒麟v4上出现）

原因：不支持canvar， 要用svg (这个时在前端修改)   *Echarts*

