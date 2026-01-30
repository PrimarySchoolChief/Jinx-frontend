# 端口转发管理面板后端 API 文档

本文档详细描述了端口转发管理面板前端所使用的后端 API。后端将采用 Go + Gin + Gorm 技术栈实现。

## 通用响应结构

所有 API 响应都将遵循以下 JSON 结构：

```json
{
  "Ok": true,      // 布尔值，表示请求是否成功
  "Msg": "操作成功", // 字符串，包含成功或错误信息
  "Data": {}       // 对象，请求成功时返回的数据，具体结构取决于端点
}
```

当 `Ok` 为 `false` 时，`Msg` 字段将包含具体的错误信息。

## 认证和用户管理

### 1. 用户登录

- **端点**: `POST /ajax/login`
- **描述**: 用户通过用户名和密码登录系统。
- **请求体**: `application/x-www-form-urlencoded` 或 `application/json`
  ```json
  {
    "username": "string", // 用户名
    "password": "string"  // 密码
  }
  ```
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "登录成功",
      "Data": null
    }
    ```
  - 失败:
    ```json
    {
      "Ok": false,
      "Msg": "用户名或密码错误",
      "Data": null
    }
    ```

### 2. 用户注册

- **端点**: `POST /ajax/register`
- **描述**: 用户注册新账户。
- **请求体**: `application/x-www-form-urlencoded` 或 `application/json`
  ```json
  {
    "username": "string",    // 用户名
    "password": "string",    // 密码
    "recaptcha": "string" // reCAPTCHA 验证码 (如果启用)
  }
  ```
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "注册成功",
      "Data": null
    }
    ```
  - 失败:
    ```json
    {
      "Ok": false,
      "Msg": "注册失败，原因...",
      "Data": null
    }
    ```

### 3. 获取注册状态

- **端点**: `GET /ajax/register`
- **描述**: 检查注册功能是否启用，以及是否需要 reCAPTCHA。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": {
        "register": true,     // 布尔值，表示注册是否启用
        "reCAPTCHA": false,   // 布尔值，表示是否需要 reCAPTCHA
        "siteKey": "string"   // reCAPTCHA 的站点密钥 (如果需要)
      }
    }
    ```

### 4. 获取用户信息

- **端点**: `GET /ajax/userInfo`
- **描述**: 获取当前登录用户的详细信息。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": {
        "permission": 1,        // 用户权限等级
        "permission_id": 1,     // 权限 ID
        "traffic_used": 1024,   // 已用流量 (字节)
        "traffic": 1073741824,  // 总流量 (字节)
        "speed": 100,           // 速度限制 (Mbps)
        "maxconn": 1000,        // 最大连接数
        "reset_date": "2026-02-01", // 流量重置日期
        "expire_date": "2027-01-01", // 账户到期日期
        "rule": 5,              // 常规转发规则数量
        "nat_rule": 3           // NAT 转发规则数量
      }
    }
    ```

## 端口转发规则 (常规)

### 1. 获取所有常规端口转发规则

- **端点**: `GET /ajax/forward_rule`
- **描述**: 获取当前用户的所有常规端口转发规则。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": [
        // ForwardRule 对象数组
      ]
    }
    ```

### 2. 获取单个常规端口转发规则

- **端点**: `GET /ajax/forward_rule?id={id}`
- **描述**: 根据 ID 获取单个常规端口转发规则的详细信息。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": {
        // ForwardRule 对象
      }
    }
    ```

### 3. 创建常规端口转发规则

- **端点**: `POST /ajax/forward_rule`
- **描述**: 创建新的常规端口转发规则。
- **请求体**: `application/json`
  ```json
  {
    "name": "string",          // 规则名称
    "mode": 0,                 // 转发模式 (0: 单转发, 1: 负载均衡, 2: 故障转移)
    "protocol": "tcpudp",      // 协议 ("tcpudp", "http", "https", "host", "secure", "securex", "tls")
    "bind": "string",          // 绑定地址/域名 (根据协议不同)
    "targets": [               // 目标服务器列表
      {
        "Host": "string",      // 目标主机
        "Port": 8080           // 目标端口
      }
    ],
    "outbound": "string",      // 出站设置
    "proxy_protocol": 0,       // Proxy Protocol (0: 关闭, 1: v1, 2: v2)
    "conf": {},                // 配置 (键值对对象)
    "node_id": 1,              // 节点 ID
    "dest_node": 0,            // 目标节点 ID (如果协议为 Secure/SecureX/TLS)
    "dest_device": 0           // 目标设备 ID (如果协议为 Secure/SecureX/TLS)
  }
  ```
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "添加成功",
      "Data": null
    }
    ```

### 4. 更新常规端口转发规则

- **端点**: `PUT /ajax/forward_rule?id={id}`
- **描述**: 更新现有的常规端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **请求体**: `application/json`
  ```json
  {
    "name": "string",          // 规则名称
    "mode": 0,                 // 转发模式
    "targets": [               // 目标服务器列表
      {
        "Host": "string",
        "Port": 8080
      }
    ],
    "outbound": "string",
    "proxy_protocol": 0,
    "conf": {}
  }
  ```
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "修改成功",
      "Data": null
    }
    ```

### 5. 删除常规端口转发规则

- **端点**: `DELETE /ajax/forward_rule?id={id}`
- **描述**: 根据 ID 删除常规端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "删除成功",
      "Data": null
    }
    ```

### 6. 重启常规端口转发规则

- **端点**: `GET /ajax/forward_rule/restart?id={id}`
- **描述**: 根据 ID 重启常规端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "重启成功",
      "Data": null
    }
    ```

### 7. 启用常规端口转发规则

- **端点**: `GET /ajax/forward_rule/start?id={id}`
- **描述**: 根据 ID 启用常规端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "启用成功",
      "Data": null
    }
    ```

### 8. 停止常规端口转发规则

- **端点**: `GET /ajax/forward_rule/stop?id={id}`
- **描述**: 根据 ID 停止常规端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "暂停成功",
      "Data": null
    }
    ```

### 9. 获取常规端口转发规则统计

- **端点**: `GET /ajax/forward_rule/statistics`
- **描述**: 获取常规端口转发规则的流量统计信息。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": [
        {
          "rule_id": 1,     // 规则 ID
          "traffic": 123456 // 流量 (字节)
        }
      ]
    }
    ```

### 10. 调试常规端口转发规则

- **端点**: `GET /ajax/forward_rule/debug?id={id}`
- **描述**: 获取常规端口转发规则的调试信息。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": {
        "InBound": {
          "Ok": true,
          "Data": {
            "Timestarp": 1678886400, // 时间戳
            "Error": 0,              // 错误码
            "Status": "Active",      // 规则状态
            "MaxConn": 0,            // 最大连接数
            "Connected": 5,          // 已连接数
            "Targets": {
              "target_host:target_port": {
                "Ok": true,
                "Data": {
                  "ip_address": "status"
                }
              }
            }
          }
        },
        "ShowOutbound": false, // 是否显示出站信息
        "OutBound": null       // 出站调试信息 (如果 ShowOutbound 为 true)
      }
    }
    ```

## 端口转发规则 (NAT)

### 1. 获取所有 NAT 端口转发规则

- **端点**: `GET /ajax/nat_forward_rule`
- **描述**: 获取当前用户的所有 NAT 端口转发规则。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": [
        // NatForwardRule 对象数组
      ]
    }
    ```

### 2. 获取单个 NAT 端口转发规则

- **端点**: `GET /ajax/nat_forward_rule?id={id}`
- **描述**: 根据 ID 获取单个 NAT 端口转发规则的详细信息。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": {
        // NatForwardRule 对象
      }
    }
    ```

### 3. 创建 NAT 端口转发规则

- **端点**: `POST /ajax/nat_forward_rule`
- **描述**: 创建新的 NAT 端口转发规则。
- **请求体**: `application/json`
  ```json
  {
    "name": "string",          // 规则名称
    "protocol": "tcpudp",      // 协议 ("tcpudp", "http", "https")
    "bind": "string",          // 绑定地址/域名 (根据协议不同)
    "targets": [               // 目标服务器列表 (仅支持一个目标)
      {
        "Host": "string",      // 目标主机
        "Port": 8080           // 目标端口
      }
    ],
    "proxy_protocol": 0,       // Proxy Protocol (0: 关闭, 1: v1, 2: v2)
    "conf": {},                // 配置 (键值对对象)
    "node_id": 1,              // 节点 ID
    "dest_device": 1           // 目标设备 ID
  }
  ```
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "添加成功",
      "Data": null
    }
    ```

### 4. 更新 NAT 端口转发规则

- **端点**: `PUT /ajax/nat_forward_rule?id={id}`
- **描述**: 更新现有的 NAT 端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **请求体**: `application/json`
  ```json
  {
    "name": "string",          // 规则名称
    "targets": [               // 目标服务器列表 (仅支持一个目标)
      {
        "Host": "string",
        "Port": 8080
      }
    ],
    "proxy_protocol": 0,
    "conf": {}
  }
  ```
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "修改成功",
      "Data": null
    }
    ```

### 5. 删除 NAT 端口转发规则

- **端点**: `DELETE /ajax/nat_forward_rule?id={id}`
- **描述**: 根据 ID 删除 NAT 端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "删除成功",
      "Data": null
    }
    ```

### 6. 重启 NAT 端口转发规则

- **端点**: `GET /ajax/nat_forward_rule/restart?id={id}`
- **描述**: 根据 ID 重启 NAT 端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "重启成功",
      "Data": null
    }
    ```

### 7. 启用 NAT 端口转发规则

- **端点**: `GET /ajax/nat_forward_rule/start?id={id}`
- **描述**: 根据 ID 启用 NAT 端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "启用成功",
      "Data": null
    }
    ```

### 8. 停止 NAT 端口转发规则

- **端点**: `GET /ajax/nat_forward_rule/stop?id={id}`
- **描述**: 根据 ID 停止 NAT 端口转发规则。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "暂停成功",
      "Data": null
    }
    ```

### 9. 获取 NAT 端口转发规则统计

- **端点**: `GET /ajax/nat_forward_rule/statistics`
- **描述**: 获取 NAT 端口转发规则的流量统计信息。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": [
        {
          "rule_id": 1,     // 规则 ID
          "traffic": 123456 // 流量 (字节)
        }
      ]
    }
    ```

### 10. 调试 NAT 端口转发规则

- **端点**: `GET /ajax/nat_forward_rule/debug?id={id}`
- **描述**: 获取 NAT 端口转发规则的调试信息。
- **参数**:
  - `id`: 规则 ID (查询参数)
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": {
        "InBound": {
          "Ok": true,
          "Data": {
            "Timestarp": 1678886400, // 时间戳
            "Error": 0,              // 错误码
            "Status": "Active",      // 规则状态
            "MaxConn": 0,            // 最大连接数
            "Connected": 5,          // 已连接数
            "Targets": {
              "target_host:target_port": {
                "Ok": true,
                "Data": {
                  "ip_address": "status"
                }
              }
            }
          }
        }
      }
    }
    ```

## 节点管理

### 1. 获取所有节点

- **端点**: `GET /ajax/node`
- **描述**: 获取所有可用节点的信息。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": [
        {
          "id": 1,
          "name": "节点A",
          "addr": "192.168.1.1",
          "speed": 1.0,
          "traffic": 1.0,
          "port_range": "10000-20000",
          "reseved_ports": "80,443",
          "reseved_target_ports": "",
          "icp": false,
          "tls_verify": false,
          "tls_verify_host": false,
          "blocked_protocol": "",
          "blocked_hostname": "",
          "blocked_path": "",
          "note": "这是一个示例节点",
          "protocol": "tcpudp|http|https",
          "outbounds": ["outbound1", "outbound2"],
          "permission": 1
        }
      ]
    }
    ```

### 2. 获取所有 NAT 节点

- **端点**: `GET /ajax/nat_node`
- **描述**: 获取所有可用于 NAT 转发的节点信息。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": [
        {
          "id": 1,
          "name": "NAT节点A",
          "addr": "192.168.1.2",
          "speed": 1.0,
          "traffic": 1.0,
          "port_range": "10000-20000",
          "reseved_ports": "",
          "reseved_target_ports": "",
          "icp": false,
          "tls_verify": false,
          "tls_verify_host": false,
          "blocked_protocol": "",
          "blocked_hostname": "",
          "blocked_path": "",
          "note": "这是一个示例NAT节点",
          "nat_protocol": "tcpudp|http|https",
          "permission": 4
        }
      ]
    }
    ```

## 设备管理

### 1. 获取所有隧道设备

- **端点**: `GET /ajax/tunnel_device`
- **描述**: 获取所有隧道设备的信息。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": [
        {
          "id": 1,
          "name": "隧道设备A"
        }
      ]
    }
    ```

### 2. 获取所有 NAT 设备

- **端点**: `GET /ajax/nat_device`
- **描述**: 获取所有 NAT 设备的信息。
- **响应**:
  - 成功:
    ```json
    {
      "Ok": true,
      "Msg": "获取成功",
      "Data": [
        {
          "id": 1,
          "name": "NAT设备A"
        }
      ]
    }
    ```
