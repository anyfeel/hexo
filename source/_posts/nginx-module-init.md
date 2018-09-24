---
title: Nginx 模块初始化过程
date: 2018-09-24 09:50:39
tags: Nginx
---

## 初始化核心函数
`ngx_init_cycle()(ngx_cycle.c:275)` -> `ngx_conf_parse()(ngx_conf_file.c:)` -> `ngx_conf_handler()`

## 初始化步骤
- modules 和序号绑定关系
```
ngx_int_t
ngx_preinit_modules(void)
{
    ngx_uint_t  i;

    for (i = 0; ngx_modules[i]; i++) {
        ngx_modules[i]->index = i;
        ngx_modules[i]->name = ngx_module_names[i];
    }

    ngx_modules_n = i;
    ngx_max_module = ngx_modules_n + NGX_MAX_DYNAMIC_MODULES;

    return NGX_OK;
}
```

- `ngx_init_cycle` 初始化核心模块
```
...

for (i = 0; cycle->modules[i]; i++) {
	if (cycle->modules[i]->type != NGX_CORE_MODULE) {
		continue;
	}

	module = cycle->modules[i]->ctx;

	if (module->create_conf) {
		rv = module->create_conf(cycle);
		if (rv == NULL) {
			ngx_destroy_pool(pool);
			return NULL;
		}
		cycle->conf_ctx[cycle->modules[i]->index] = rv;
	}
}

...

for (i = 0; cycle->modules[i]; i++) {
	if (cycle->modules[i]->type != NGX_CORE_MODULE) {  // NGX_CORE_MODULE 核心模块
		continue;
	}

	module = cycle->modules[i]->ctx;

	if (module->init_conf) {
		if (module->init_conf(cycle,
							  cycle->conf_ctx[cycle->modules[i]->index]) // 初始化
			== NGX_CONF_ERROR)
		{
			environ = senv;
			ngx_destroy_cycle_pools(&conf);
			return NULL;
		}
	}
}
...
```

- 调用 ngx_events_block 来解析具体模块如配置和指令, 以 event 模块为例
```
static char *
ngx_events_block(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    ...

    *(void **) conf = ctx;

    for (i = 0; cf->cycle->modules[i]; i++) {
        if (cf->cycle->modules[i]->type != NGX_EVENT_MODULE) {
            continue;
        }

        m = cf->cycle->modules[i]->ctx;

        if (m->create_conf) {
            (*ctx)[cf->cycle->modules[i]->ctx_index] =
                                                     m->create_conf(cf->cycle);  // 为每个 event 模块分配内存
            if ((*ctx)[cf->cycle->modules[i]->ctx_index] == NULL) {
                return NGX_CONF_ERROR;
            }
        }
    }

    pcf = *cf;
    cf->ctx = ctx;
    cf->module_type = NGX_EVENT_MODULE;
    cf->cmd_type = NGX_EVENT_CONF;

    rv = ngx_conf_parse(cf, NULL);  // 解析 event 模块，如 `work_connections`

    *cf = pcf;

    if (rv != NGX_CONF_OK) {
        return rv;
    }

    for (i = 0; cf->cycle->modules[i]; i++) {
        if (cf->cycle->modules[i]->type != NGX_EVENT_MODULE) {
            continue;
        }

        m = cf->cycle->modules[i]->ctx;

        if (m->init_conf) {
            rv = m->init_conf(cf->cycle,
                              (*ctx)[cf->cycle->modules[i]->ctx_index]); // 和指令默认值对比，设置指令的值
            if (rv != NGX_CONF_OK) {
                return rv;
            }
        }
    }

    return NGX_CONF_OK;
}
```
