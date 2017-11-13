# moligine
a  waf module for nginx,with a web terminal by flask

 moligine 是基于openresty，通过lua脚本实现的waf模块
 具有以下特点：
 1，提供了一基于flask的web终端，用于统一配置，后续版本将集成ELK，用于防护日志分析
 2，使用redis存储配置和waf规则，便于反向代理集群统一管理
可以用来设置黑白名单（ip，url黑白名单）设置各种防护规则如：SQL注入、XSS、Shell注入，上传特殊文件，cc等
建议使用时先不开启防护，只记录日志，一段时间以后，对日志进行分析，再开启防护，防止误杀
#客户端安装,建议使用root用户，否则请sudo 提权
1，请先安装openresty，我比较懒，直接使用oneinstack一键安装 参照 https://oneinstack.com/
2，将 dkjson.lua 上传到 lualib目录下，如果你使用oneinstack默认安装 这个目录在 /usr/local/openresty/lualib/
3，将 lua.tar.gz 上传到nginx config目录下 请上传到 /usr/local/openresty/nginx/conf
4，创建waf规则目录 每次Nginx reload，就会在nginx 启动阶段从redis中读取配置文件，并存储到本地目录 mkdir -p /usr/local/openresty/nginx/conf/waf/wafconf/5，创建默认日志目录 mkdir -p /var/log/moligine6，将改目录的属主赋予nginx worker进程用户 chown -R www.www /var/log/moligine7，修改redis连接地址 修改init.lua中  local rediscmd = "redis-cli -h 192.168.1.197 -p 6379 " 修改其中的ip和端口 如果你觉得有安全性问题，可以在安装redis的时候加上口令，并修改这里的连接字符串
#redis 安装
刚刚我们配置从192.168.1.197 redis获取数据那么这里我们需要在192.168.1.197上安装redis，这里省略若干字
#web端安装
1，virtual_env安装Python 2.7环境，并pip安装 flask，redis等扩展
2，修改配置文件app/etc/config.py redis_conf = {    'host':'192.168.1.197',    'port':6379,    'db':0 } 写上redis相关的配置3，首次运行需要在redis中添加初始数据，建议在run.py中添加
#############################################################
globalconfig = { 'attacklog':r'off', 'enableDefend':r'off', 'RulePath':r'/usr/local/nginx/conf/waf/wafconf/', 'logdir':r'/usr/local/nginx/logs/hack/', 'CookieMatch':r'off', 'postMatch':r'off', 'ipBlockModule':r'off', 'servernameBlackModule':r'off', 'snUrlBlackModule':r'off', 'CCDeny':r'off', 'CCrate':r'10/2', 'html':'''  <h1>access deny by moligine</h1> '''}
r.set("globalconfig",json.dumps(globalconfig))
############################################################
添加到run.py最后一行代码之前初始数据添加完以后记得删除或者注销以上代码！！！

#nginx.conf配置
请在nginx.conf段添加以下配置
##################################################################  
lua_need_request_body off;  
lua_shared_dict limit 10m;  
init_by_lua_file  /usr/local/openresty/nginx/conf/lua/init.lua;  
access_by_lua_file /usr/local/openresty/nginx/conf/lua/waf.lua;  
client_body_in_file_only on;  
lua_code_cache on;
##################################################################
可参照remoteCmd.py脚本可参照remoteCmd.py脚本可参照remoteCmd.py脚本可参照remoteCmd.py脚本可参照remoteCmd.py脚本可参照remoteCmd.py脚本

有问题可以联系作者（如果你能找到联系方式的话）
你尽管问，反正我也帮不了你
