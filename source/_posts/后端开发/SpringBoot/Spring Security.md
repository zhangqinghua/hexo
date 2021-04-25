---
title: Spring Security

categories:
- 后端开发
- SpringBoot

date: 2021-02-21
---
示例一：

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()           // 确保对我们应用程序的任何请求都需要对用户进行身份验证
        .anyRequest()
        .authenticated()
        .and()
        .formLogin()                   // 允许用户使用基于表单的登录进行身份验证
        .and()
        .httpBasic();                  // 允许用户使用 HTTP Basic 身份验证进行身份验证
}
```
示例一：
```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .anyRequest()
        .authenticated()
        .and()
        .formLogin()
        .loginPage("/login")           // 指定登录页面的位置。
        .permitAll();                  // 授予所有用户(即未经身份验证的用户)访问我们登录页面的权限。
}
```

示例一：

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()                                                  // http.authorizeRequests()方法有多个子级，每个匹配器均按声明 Sequences 考虑。        
        .antMatchers("/resources/**", "/signup", "/about").permitAll()        // 如果 URL 以“/resources /”开头，等于“/signup”或等于“/about”，则任何用户都可以访问请求。          
        .antMatchers("/admin/**").hasRole("ADMIN")                            // 任何以“/admin /”开头的 URL 都将限于角色为“ ROLE_ADMIN”的用户。
        .antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")  // 任何以“/db /”开头的 URL 都要求用户同时具有“ ROLE_ADMIN”和“ ROLE_DBA”
        .anyRequest().authenticated()                                         // 尚未匹配的任何 URL 仅要求对用户进行身份验证      
        .and()
        // ...
        .formLogin();
}
```

示例一：

```java
protected void configure(HttpSecurity http) throws Exception {
    http.logout()                                   // 提供注销支持。使用WebSecurityConfigurerAdapter时将自动应用。                          
        .logoutUrl("/my/logout")                    // 触发注销的 URL 发生(默认为/logout)。如果启用了 CSRF 保护(默认)，则请求也必须是 POST。                          
        .logoutSuccessUrl("/my/index")              // 发生注销后重定向到的 URL。默认值为/login?logout。                         
        .logoutSuccessHandler(logoutSuccessHandler) // 让我们指定一个自定义LogoutSuccessHandler。如果指定此选项，则logoutSuccessUrl()将被忽略。                           
        .invalidateHttpSession(true)                // 指定注销时是否使HttpSession无效。默认情况下，这是“ true”。在幕后配置SecurityContextLogoutHandler。                          
        .addLogoutHandler(logoutHandler)            // 添加LogoutHandler。默认情况下，将SecurityContextLogoutHandler添加为最后的LogoutHandler。                            
        .deleteCookies(cookieNamesToClear)          // 允许指定成功注销后将删除的 cookie 名称。这是显式添加CookieClearingLogoutHandler的快捷方式。                         
        .and()
        // ...
}
```