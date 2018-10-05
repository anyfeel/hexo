---
title: Nginx 基于权重的平滑轮询算法
date: 2018-03-19 12:43:14
tags: Nginx
---

## Nginx 基于权重的轮询算法

Nginx基于权重的轮询算法的实现可以参考它的一次代码提交 [Upstream: smooth weighted round-robin balancing](https://github.com/phusion/nginx/commit/27e94984486058d73157038f7950a0a36ecc6e35)

它不但实现了基于权重的轮询算法，而且还实现了平滑的算法。所谓平滑，就是在一段时间内，不仅服务器被选择的次数的分布和它们的权重一致，而且调度算法还比较均匀的选择服务器，而不会集中一段时间之内只选择某一个权重比较高的服务器。如果使用随机算法选择或者普通的基于权重的轮询算法，就比较容易造成某个服务集中被调用压力过大。

举个例子，比如权重为 {a:5, b:1, c:1} 的一组服务器，Nginx 的平滑的轮询算法选择的序列为{ a, a, b, a, c, a, a },这显然要比{ c, b, a, a, a, a, a } 序列更平滑，更合理，不会造成对a服务器的集中访问。

## Lua 实现
每次需要遍历所有的 servers 列表，返回 best server

<!-- more -->

```
local ceil = math.ceil

local _M = { _VERSION = "0.11" }

--[[
parameters:
    - (table) servers
    - (function) peer_cb(index, server)
return:
    - (table) server
    - (string) error
--]]
function _M.next_round_robin_server(servers, peer_cb)
    local srvs_cnt = #servers

    if srvs_cnt == 1 then
        if peer_cb(1, servers[1]) then
            return servers[1], nil
        end

        return nil, "round robin: no servers available"
    end

    -- select round robin server
    local best
    local max_weight
    local weight_sum = 0
    for idx = 1, srvs_cnt do
        local srv = servers[idx]
        -- init round robin state
        srv.weight = srv.weight or 1
        srv.effective_weight = srv.effective_weight or srv.weight
        srv.current_weight = srv.current_weight or 0

        if peer_cb(idx, srv) then
            srv.current_weight = srv.current_weight + srv.effective_weight
            weight_sum = weight_sum + srv.effective_weight

            if srv.effective_weight < srv.weight then
                srv.effective_weight = srv.effective_weight + 1
            end

            if not max_weight or srv.current_weight > max_weight then
                max_weight = srv.current_weight
                best = srv
            end
        end
    end

    if not best then
        return nil, "round robin: no servers available"
    end

    best.current_weight = best.current_weight - weight_sum

    return best, nil
end


function _M.free_round_robin_server(srv, failed)
    if not failed then
        return
    end

    srv.effective_weight = ceil((srv.effective_weight or 1) / 2)
end


return _M

```
