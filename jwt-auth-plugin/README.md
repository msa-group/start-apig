# 概述

jwt-auth 插件实现了基于 JWT 进行认证鉴权的功能，支持从 HTTP 请求的 URL 参数、请求头、Cookie 字段解析 JWT，同时验证该 Token 是否有权限访问。相较于配置 JWT 认证鉴权，jwt-auth 插件额外提供了调用方身份识别的能力，支持对不同调用方配置不同的 JWT 凭证。

## 场景概述

Json Web Token（JWT）一般可以用作独立的身份验证令牌，可以包含用户标识、用户角色和权限等信息，以便于从资源服务器获取资源，也可以增加一些额外的其他业务逻辑所必须的声明信息，特别适用于分布式站点的登录场景。

## JWT 验证流程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant APIGateway as API网关
    participant BackendService as 后端服务

    Client->>APIGateway: HTTP Request (携带JWT Token)
    APIGateway->>APIGateway: 根据密钥验证签名,检查过期时间,验证issuer

    activate APIGateway

    alt Token验证成功
        APIGateway-->>Client: 200
        APIGateway->>BackendService: 转发请求(携带Token)
        BackendService-->>APIGateway: 业务响应
        APIGateway-->>Client: 响应数据
    else Token验证失败
        APIGateway->>APIGateway: 错误日志记录
        APIGateway-->>Client: 401 Unauthorized
    end

    deactivate APIGateway
```

## 部署

1. 创建密钥对，可以在线生成：https://mkjwk.org/ ，本工程以对称密钥为例
   ![在线生成密钥](image.png)
2. 实现 token 颁发的认证服务

- python 示例

```python
import jwt

# 签发方
ISSUER = 'httpbin.org'
# sk(密钥)，本工程以对称密钥为例，第1步的key中的k值为sk经过base64URL编码生成，在此decode为原始数据即为密钥
jwk_key = '6fYeKzXqP2FKV9INaq62tGG5Ypc3hLQZzsao3ULosHGGFuECmnspfcZT6ran3KlKEMvo_eR7eohjzbdmuPe6A3k2CRW3ZEr8LKQtHjlNY-fmjWX6rcyiRX26U3bPfEmrQWmlOVghDaAqPw0AUg0HfjgAGQrXw4iuyxqOI37tNi9jpIxYdOiKjSmUQVJ6rHDeqRT9KL6-dd9BPZ2hvSw_iEqr_R568qPvZv8oz3TQSveeavHFkakFIXfRfhvueeA5uGWD3MzIuY6a81uqFZ1JiMIy__BNIrEVO1oaTSf1J0XxFnqm7NtYtNDS0uX6NVVRZ-EGquAM8lg6xwNPDfeBjA'
default_secret = base64.urlsafe_b64decode(pad_base64url(jwk_key))
SECRET_KEY = os.environ.get("SECRET_KEY") or default_secret

# 以flask框架为例，生成签发token的方法
@app.route('/create-jwt', methods=['POST'])
def create_jwt():
    token = jwt.encode({
        'exp': datetime.datetime.utcnow() + datetime.timedelta(hours=1),# 过期时间
        'iat': datetime.datetime.utcnow(), # 签发时间
        'iss': ISSUER, # 签发方
    }, SECRET_KEY)
return jsonify(code=200, msg="ok", data={"token": token})
```

- nodejs 示例

```
const express = require('express');
const jwt = require('jsonwebtoken');
const base64url = require('base64url');

const app = express();
app.use(express.json());

const ISSUER = 'httpbin.org';
const JWK_KEY = '6fYeKzXqP2FKV9INaq62tGG5Ypc3hLQZzsao3ULosHGGFuECmnspfcZT6ran3KlKEMvo_eR7eohjzbdmuPe6A3k2CRW3ZEr8LKQtHjlNY-fmjWX6rcyiRX26U3bPfEmrQWmlOVghDaAqPw0AUg0HfjgAGQrXw4iuyxqOI37tNi9jpIxYdOiKjSmUQVJ6rHDeqRT9KL6-dd9BPZ2hvSw_iEqr_R568qPvZv8oz3TQSveeavHFkakFIXfRfhvueeA5uGWD3MzIuY6a81uqFZ1JiMIy__BNIrEVO1oaTSf1J0XxFnqm7NtYtNDS0uX6NVVRZ-EGquAM8lg6xwNPDfeBjA';

function padBase64url(input) {
    let str = input;
    while (str.length % 4 !== 0) {
        str += '=';
    }
    return str;
}

const defaultSecret = base64url.decode(padBase64url(JWK_KEY));
const SECRET_KEY = process.env.SECRET_KEY || defaultSecret;

app.post('/create-jwt', (req, res) => {
    const token = jwt.sign({
        exp: Math.floor(Date.now() / 1000) + 3600,
        iat: Math.floor(Date.now() / 1000),
        iss: ISSUER
    }, SECRET_KEY, { algorithm: 'HS256' });

    res.json({ code: 200, msg: "ok", data: { token } });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

- java 示例

```
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Base64;
import java.util.Date;
import javax.crypto.SecretKey;

@RestController
public class JwtController {

    private static final String ISSUER = "httpbin.org";
    private static final String JWK_KEY = "6fYeKzXqP2FKV9INaq62tGG5Ypc3hLQZzsao3ULosHGGFuECmnspfcZT6ran3KlKEMvo_eR7eohjzbdmuPe6A3k2CRW3ZEr8LKQtHjlNY-fmjWX6rcyiRX26U3bPfEmrQWmlOVghDaAqPw0AUg0HfjgAGQrXw4iuyxqOI37tNi9jpIxYdOiKjSmUQVJ6rHDeqRT9KL6-dd9BPZ2hvSw_iEqr_R568qPvZv8oz3TQSveeavHFkakFIXfRfhvueeA5uGWD3MzIuY6a81uqFZ1JiMIy__BNIrEVO1oaTSf1J0XxFnqm7NtYtNDS0uX6NVVRZ-EGquAM8lg6xwNPDfeBjA";

    private SecretKey getSecretKey() {
        String envSecret = System.getenv("SECRET_KEY");
        if (envSecret != null) {
            return Keys.hmacShaKeyFor(envSecret.getBytes());
        }

        String padded = JWK_KEY;
        while (padded.length() % 4 != 0) {
            padded += "=";
        }
        byte[] decoded = Base64.getUrlDecoder().decode(padded);
        return Keys.hmacShaKeyFor(decoded);
    }

    @PostMapping("/create-jwt")
    public ResponseEntity<Map<String, Object>> createJwt() {
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);

        String token = Jwts.builder()
                .setExpiration(new Date(nowMillis + 3600 * 1000))
                .setIssuedAt(now)
                .setIssuer(ISSUER)
                .signWith(getSecretKey())
                .compact();

        Map<String, Object> response = new HashMap<>();
        response.put("code", 200);
        response.put("msg", "ok");
        response.put("data", Collections.singletonMap("token", token));

        return ResponseEntity.ok(response);
    }
}
```

- golang 示例

```
package main

import (
	"encoding/base64"
	"os"
	"time"
	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt/v5"
)

const ISSUER = "httpbin.org"
const JWK_KEY = "6fYeKzXqP2FKV9INaq62tGG5Ypc3hLQZzsao3ULosHGGFuECmnspfcZT6ran3KlKEMvo_eR7eohjzbdmuPe6A3k2CRW3ZEr8LKQtHjlNY-fmjWX6rcyiRX26U3bPfEmrQWmlOVghDaAqPw0AUg0HfjgAGQrXw4iuyxqOI37tNi9jpIxYdOiKjSmUQVJ6rHDeqRT9KL6-dd9BPZ2hvSw_iEqr_R568qPvZv8oz3TQSveeavHFkakFIXfRfhvueeA5uGWD3MzIuY6a81uqFZ1JiMIy__BNIrEVO1oaTSf1J0XxFnqm7NtYtNDS0uX6NVVRZ-EGquAM8lg6xwNPDfeBjA"

func padBase64URL(s string) string {
	for len(s)%4 != 0 {
		s += "="
	}
	return s
}

func getSecretKey() []byte {
	if envSecret := os.Getenv("SECRET_KEY"); envSecret != "" {
		return []byte(envSecret)
	}

	padded := padBase64URL(JWK_KEY)
	decoded, _ := base64.RawURLEncoding.DecodeString(padded)
	return decoded
}

func main() {
	r := gin.Default()

	r.POST("/create-jwt", func(c *gin.Context) {
		token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(time.Hour)),
			IssuedAt:  jwt.NewNumericDate(time.Now()),
			Issuer:    ISSUER,
		})

		tokenString, _ := token.SignedString(getSecretKey())

		c.JSON(200, gin.H{
			"code": 200,
			"msg":  "ok",
			"data": gin.H{"token": tokenString},
		})
	})

	r.Run(":8080")
}
```

3. 在云原生 API 网关配置 jwt-auth 插件
   本示例 `jwt-auth` 插件的配置如下：

```
consumers:
- name: test
  issuer: httpbin.org
  jwks: |
    {
      "keys": [
        {
          "kty": "oct",
          "use": "sig",
          "k": "6fYeKzXqP2FKV9INaq62tGG5Ypc3hLQZzsao3ULosHGGFuECmnspfcZT6ran3KlKEMvo_eR7eohjzbdmuPe6A3k2CRW3ZEr8LKQtHjlNY-fmjWX6rcyiRX26U3bPfEmrQWmlOVghDaAqPw0AUg0HfjgAGQrXw4iuyxqOI37tNi9jpIxYdOiKjSmUQVJ6rHDeqRT9KL6-dd9BPZ2hvSw_iEqr_R568qPvZv8oz3TQSveeavHFkakFIXfRfhvueeA5uGWD3MzIuY6a81uqFZ1JiMIy__BNIrEVO1oaTSf1J0XxFnqm7NtYtNDS0uX6NVVRZ-EGquAM8lg6xwNPDfeBjA",
          "alg": "HS256"
        }
      ]
    }

```

## 使用

1. 发起未经鉴权的请求
   首先，尝试不带任何鉴权信息发起请求：

```
curl 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/jwt-test'
```

预期返回结果：

```
Jwt is missing
```

说明：Jwt is missing 表示未提供有效的鉴权信息，访问被网关拒绝。

2. 获取鉴权信息

```
curl -X POST 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/create-jwt'
```

预期返回结果：

```
{
  "code": 200,X-Mse-Consumer
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3Mzk4NDgxMDQsImlhdCI6MTczOTg0NDUwNCwiaXNzIjoiaHR0cGJpbi5vcmcifQ.IjedzubW-ROlBclWFwMaDixvR3N91-w7vnLbDspC4iI"
  },
  "msg": "ok"
}
```

说明：data.token 为获取到的 JWT Token。

3. 发起带鉴权 Header 的请求，在请求头中添加 Authorization：

```
curl -iv -X GET 'http://env-cum46mum1hkh1e2psje0-cn-hangzhou.alicloudapi.com/robots.txt' -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3Mzk1MjYzMTUsImlhdCI6MTczOTUyMjcxNSwiaXNzIjoiaHR0cGJpbi5vcmcifQ.r1mxEOlgJTt06bU4VzvhAkpdO1UmIDq55IPTc5oYLLM'
```

预期返回结果：
![Alt text](image-1.png)
