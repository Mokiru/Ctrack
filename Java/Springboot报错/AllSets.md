# Access to XMLHttpRequest at 'http://127.0.0.1:8080/login' from origin 'http://127.0.0.1:5500' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.

在 `Controller` 中 添加 注解 `@CrossOrigin` 允许跨域请求。