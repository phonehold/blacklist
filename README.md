### 黑名单功能


对于Cache 节点，可能会有一些非法的内容需要禁止用户访问或删除，如果源站在可控范围内，那么删除源站对应的url 然后再purge 可以解决。如果源站不可控，那么就有必要在Cache 上做一些操作来禁止用户访问，从而达到网监的要求。

环境要求：

	trafficserver
	
	ts-lua
	
	memcached
	
	lua-memcached
	
	phpMemcachedAdmin  //memcached web管理界面


对于 ats ts-lua lua-memcached 安装就不在说了。

贴一下主要实现的lua 代码吧。很简单

**blacklist.lua**


```
local HOSTNAME = ''

function __init__(argtb)

    if (#argtb) < 1 then
        print(argtb[0], 'hostname parameter required!!')
        return -1
    end

    HOSTNAME = argtb[1]
end
time=ts.now()

body= "Access Deny From " ..  HOSTNAME .. time .."\n"
function send_response()
    ts.client_response.header['BlackList'] = ts.ctx['black']
end


function do_remap()
        local Memcached = require('Memcached')
        local m = Memcached.Connect()
        local req_url = ts.client_request.get_url()
        --ts.debug(req_url)
        local obj = m:get(req_url)
        --ts.debug(obj)
        if obj == nil then

        else
                ts.http.set_resp(403, body)
                ts.ctx['black']= obj
        end
        ts.hook(TS_LUA_HOOK_SEND_RESPONSE_HDR, send_response)
end
```
