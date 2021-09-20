加入依赖
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
```


jwt工具类
```java
package com.example.demo.utils;

import io.jsonwebtoken.*;
import org.springframework.util.StringUtils;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class JwtUtil {
    //过期时间30分钟
    private static long tokenExpiration = 30 * 60 * 1000;
    //签名密钥
    private static String tokenSignKey = "83c85bd9-e97a-479f-96f4-b347f5490bcb";

    /**
     * 根据用户信息生成token
     *
     * @param userId
     * @param userName
     * @return token
     */
    public static String createToken(String userId, String userName) {
        String token = Jwts.builder()
                /*设置名字*/
                .setSubject("userId")
                /*签发时间*/
                .setIssuedAt(new Date())
                /*过期时间*/
                .setExpiration(new Date(System.currentTimeMillis() + tokenExpiration))
                /*设计用户信息*/
                .claim("userId", userId)
                .claim("userName", userName)
                /*设置密钥*/
                .signWith(SignatureAlgorithm.HS512, tokenSignKey)
                .compressWith(CompressionCodecs.GZIP)
                .compact();

        return token;

    }

    /**
     * 根据token字符串返回用户ID
     *
     * @param token
     * @return
     */
    public static String getUserId(String token) {
        if (StringUtils.isEmpty(token)) return null;
        String userId = (String) getClaim(token).get("userId");
        return userId;
    }

    /**
     * 根据token字符串返回用户Name
     *
     * @param token
     * @return
     */
    public static String getUserName(String token) {
        if (StringUtils.isEmpty(token)) return null;
        String userName = (String) getClaim(token).get("userName");
        return userName;
    }

    /**
     * 获取Claims 部分内容（即要传的信息）
     *
     * @param token
     * @return
     */
    public static Claims getClaim(String token) {
        Claims claims = null;
        try {
            claims = Jwts.parser()
                    .setSigningKey(tokenSignKey)
                    .parseClaimsJws(token)
                    .getBody();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
        return claims;
    }

    public static boolean validationToken(String token) {
        if (StringUtils.isEmpty(token)) return false;
        String hello = "hello";
        return false;
    }

    /**
     * 获取jwt发布时间
     */
    public static Date getIssuedAt(String token) {
        return getClaim(token).getIssuedAt();
    }

    /**
     * 获取jwt失效时间
     */
    public static Date getExpiration(String token) {
        return getClaim(token).getExpiration();
    }

    /**
     * 验证token是否有效
     *
     * @param token
     * @return true:有效   false:无效
     */
    public static boolean validation(String token) {
        if (StringUtils.isEmpty(token)) return false;
        Claims claims = null;
        try {
            claims = Jwts.parser().setSigningKey(tokenSignKey).
                    parseClaimsJws(token).getBody();
        } catch (Exception e) {
            return false;
        }
        Date expiration = claims.getExpiration();
        /*当前时间是否在过期时间之前*/
        return new Date().before(expiration);
    }

}

```


拦截器
```java
import com.alibaba.fastjson.JSON;
import com.example.demo.response.ResponseDTO;
import com.example.demo.utils.JwtUtil;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;

public class MyLoginIntercepte implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("token");
        if (JwtUtil.validation(token)) {
            return true;
        }
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json; charset=utf-8");
        Object json = JSON.toJSON(new ResponseDTO("4000", "token无效"));
        PrintWriter writer = response.getWriter();
        writer.write(json.toString());
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}

```


配置拦截器
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyLoginIntercepte())
                .addPathPatterns("/**")  //所有请求都被拦截包括静态资源
                .excludePathPatterns("/user/login"); //放行的请求
    }
}
```
