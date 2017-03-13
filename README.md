# AutoReply
##利用python的itchat库，图灵机器人的API接口，实现一个可以自（you）动（qu）回复的微信助手
###代码如下：
```python
#coding:utf-8
import re
import time

import itchat
from itchat.content import *

import json
import requests
import urllib
import urllib3

KEY = '你的key'  
url = 'http://www.tuling123.com/openapi/api'

#设置全局变量
a = 0

@itchat.msg_register([TEXT, PICTURE, MAP, CARD, NOTE, SHARING, RECORDING, ATTACHMENT, VIDEO])
def text_reply(msg):
    #设置全局变量
    global a
    a += 1
    if msg['Type'] == 'Text':
        reply_content = msg['Text']
    elif msg['Type'] == 'Picture':
        reply_content = r"图片: " + msg['FileName']
    elif msg['Type'] == 'Card':
        reply_content = r" " + msg['RecommendInfo']['NickName'] + r" 的名片"
    elif msg['Type'] == 'Map':
        x, y, location = re.search("<location x=\"(.*?)\" y=\"(.*?)\".*label=\"(.*?)\".*", msg['OriContent']).group(1,
                                                                                                                    2,
                                                                                                                    3)
        if location is None:
            reply_content = r"位置: 纬度->" + x.__str__() + " 经度->" + y.__str__()
        else:
            reply_content = r"位置: " + location
            
    elif msg['Type'] == 'Note':
        reply_content = r"通知"
    elif msg['Type'] == 'Sharing':
        reply_content = r"分享"
    elif msg['Type'] == 'Recording':
        reply_content = r"语音"
    elif msg['Type'] == 'Attachment':
        reply_content = r"文件: " + msg['FileName']
    elif msg['Type'] == 'Video':
        reply_content = r"视频: " + msg['FileName']
    else:
        reply_content = r"消息"

    #调用图灵机器人
    req_info = reply_content.encode('utf-8')
    query = {'key': KEY, 'info': req_info}
    headers = {'Content-type': 'text/html', 'charset': 'utf-8'}
    r = requests.get(url, params=query, headers=headers)
    res = r.text
    tuRingRobot = json.loads(res).get('text').replace('<br>', '\n')

    friend = itchat.search_friends(userName=msg['FromUserName'])
    itchat.send(r"Friend:%s                              "
                r"Time:%s    "
                r" Message:%s" % (friend['RemarkName'], time.ctime(), reply_content),
                toUserName='filehelper')
    if a == 1:
        itchat.send(r"我已经收到您发送的消息，我PP会让我爹在第一时间回复您的。     " 
                    r"时间：%s"
                    r"     --微信助手 PP"% (time.ctime()),
                    toUserName=msg['FromUserName'])
    else:
        itchat.send(r'%s'
                    r"     --微信助手 PP"% tuRingRobot,
                    toUserName=msg['FromUserName'])


itchat.auto_login()
itchat.run()
```
##### 昨天发现自动回复有个BUG
##### 预设的功能是，对每一个对话，第一句回复的都是
    我已经收到您发送的消息，我PP会让我爹在第一时间回复您的。    时间：     --微信助手 PP
##### 但是测试发现，这段小程只满足微信账号登陆之后，第一条聊天信息的回复内容是上面的内容，其他的回复内容均来自图灵机器人
#####希望大家能继续完善这段小程序,简在你我，功在编程中国
