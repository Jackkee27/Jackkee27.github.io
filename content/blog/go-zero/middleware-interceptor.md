---
title: Middleware & Interceptor
type: blog
prev: blog/go-zero
next: blog/go-zero
date: 2025-04-30
authors:
  - name: Jackkee Li
    link: https://github.com/Jackkee27
sidebar:
  open: true
---

这篇文章介绍了关于在 Go Zero 中使用 middleware 和 interceptor 的一些方法。

<!--more-->

## Middleware 中间件：API 层

- 全局中间件
  
    ```go
    // in main pkg
    server.Use(your_middlewares)
    ```
    
- 非全局中间件
    1. 在 API 文件的 server 中加入 `middleware` 字段；
    2. 在 `internal/middleware/` 下写完中间件的逻辑；
    3. 在 `internal/svc` 下添加中间件。
    
    ```go
    // in xxx.api
    @server (
        middleware: AuthMiddleware
    )
    service user {
        // user 服务下的handler将适用AuthMiddleware
        @handler userInfo
        get /api/user/:id (UserInfoReq) returns (UserInfoResp)
    }
    
    // in internal/middleware/authmiddleware.go
    type AuthMiddleware struct {}
    
    func NewAuthMiddleware() *AuthMiddleware {
    		return &AuthMiddleware{}
    }
    
    func (m *AuthMiddleware) Handle(next http.HandlerFunc) http.HandlerFunc {
    		return func(w http.ResponseWriter, r *http.Request) {
    				roleStr := r.Header.Get("Role")
    				role, _ := strconv.Atoi(roleStr)
    				if ctype.Role(role) != ctype.RoleAdmin {
    						response.Response(r, w, nil, errors.New("permission denied"))
    						return
    				}
    				next(w, r)
    		}
    }
    
    // in internal/svc/servicecontext.go
    type ServiceContext struct {
    		// ...
    		AuthMiddleware rest.Middleware
    }
    func NewServiceContext(c config.Config) *ServiceContext {
        // ...
    		return &ServiceContext{
    	      // ...
    				AuthMiddleware: middleware.NewAuthMiddleware().Handle,
    		}
    }
    
    ```
    

## Intercepter 拦截器：gRPC 层

{{< callout type="warning" >}}
API 层和 RPC 层之间不能直接使用 context 来传数据，需要使用 metadata 来处理。

这里仅以最普通的一元拦截器 `UnaryInterceptors` 为例。
{{< /callout >}}







[gRPC（六）进阶：拦截器 interceptor | Go 技术论坛](https://learnku.com/articles/73106)

- Server：
  
    在 gRPC Server 中，使用 `AddUnaryInterceptors` 方法添加拦截器。这个方法需要实现 `grpc.UnaryServerInterceptor` 接口。
    
    ```go
    // gRPC server in main pkg
    func main() {
    		defer s.Stop()
    		s.AddUnaryInterceptors(interceptors.LogInterceptor)
    		s.Start()
    }
    
    // 实现grpc.UnaryServerInterceptor接口
    func LogInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
        // 从传递过来的metadata中获取数据，重新赋值给ctx
        remoteAddr := metadata.ValueFromIncomingContext(ctx, "remoteAddr")
        userID := metadata.ValueFromIncomingContext(ctx, "userID")
        if len(remoteAddr) > 0 {
            ctx = context.WithValue(ctx, "remoteAddr", remoteAddr[0])
        }
        if len(userID) > 0 {
            ctx = context.WithValue(ctx, "userID", userID[0])
        }
    
        return handler(ctx, req)
    }
    ```
    
- Client：
  
    在 `svcContext` 中进行 gRPC 依赖注入时，需要传递`zrpc.WithUnaryClientInterceptor`。这个函数的参数需要实现 `grpc.UnaryClientInterceptor` 接口。
    
    在调用 RPC 方法时，需要传递对应的 context。
    
    ```go
    // gRPC client in svc
    func NewServiceContext(c config.Config) *ServiceContext {
    		// init operation
        return &ServiceContext{
        Config:   c,
        // interceptors.ClientInfoInterceptor
        UserRpc:  users.NewUsers(zrpc.MustNewClient(c.UserRpc, zrpc.WithUnaryClientInterceptor(interceptors.ClientInfoInterceptor))),
        GroupRpc: groups.NewGroups(zrpc.MustNewClient(c.GroupRpc, zrpc.WithUnaryClientInterceptor(interceptors.ClientInfoInterceptor))),
        }
    }
    
    // 实现 grpc.UnaryClientInterceptor 接口
    func ClientInfoInterceptor(ctx context.Context, method string, req, reply any, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
        // 调用grpc服务之前，先构建metadata，从ctx中获取要传递给grpc server的值
        md := metadata.New(map[string]string{
            "remoteAddr": ctx.Value("remoteAddr").(string),
            "userID":     ctx.Value("userID").(string),
        })
        ctx = metadata.NewOutgoingContext(ctx, md)
    
        return invoker(ctx, method, req, reply, cc, opts...)
    }
    ```