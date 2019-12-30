# WebSecurityConfigurerAdapter与ResourceServerConfigurerAdapter

## 1.1、spring security的过滤器链

spring security自己有一个叫FilterChainProxy代理类，该类实现了servlet接口。FilterChainProxy内部有一个List filterChains,在spring 的体系里有个order值（int型）越小优先级越高，filterChains是一个依据order的降序集合，优先级高的在前面，而SecurityFilterChain是一个接口也是一个chain，每个chain里有若干个filter，在spring security里一个请求只会被一个filter chain进行处理，也就是spring security通过遍历filterChains这个集合时，只要找到能处理该请求（servlet-path匹配，即uri中去掉context-path的部分）的filter chain就不再进行其他的filter chain匹配。

```java
private List<SecurityFilterChain> filterChains;

private List<Filter> getFilters(HttpServletRequest request) {
    Iterator var2 = this.filterChains.iterator();

    SecurityFilterChain chain;
    do {
        if (!var2.hasNext()) {
            return null;
        }

        chain = (SecurityFilterChain)var2.next();
    } while(!chain.matches(request)); // 找到匹配的chain后终止循环

    return chain.getFilters(); 
}
```

## 1.2、二者顺序

默认的WebSecurityConfigurerAdapter的order是100

```java
@Order(100)
public abstract class WebSecurityConfigurerAdapter
```

而ResourceServerConfigurerAdapter实现类的@EnableResourceServer里导入了ResourceServerConfiguration，

```java
@Import({ResourceServerConfiguration.class})
public @interface EnableResourceServer {
```

该类里定义了order为3

```java
public class ResourceServerConfiguration extends WebSecurityConfigurerAdapter implements Ordered {
    private int order = 3;
```

所以ResourceServerConfigurerAdapter的实现类优先级比另外一个的更高，在请求匹配的情况下以它为准，而WebSecurityConfigurerAdapter的实现类会失效。 如果想让WebSecurityConfigurerAdapter比ResourceServerConfigurerAdapter优先级高的话，只须要让前者的@Order值比后者的@Order值更小就行了。

```java
@Order(1)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
```

每声明一个\*Adapter的实现类，都会产生一个filterChain。前面讲到一个request（匹配url）只能被一个filterChain处理，所以有二个Adapter的时候，在请求都匹配的情况下，优先级较低的会失效。

## 1.3、二者同时生效

根本在于让不同的Adapter匹配不同request（url） 实现时将细粒度较粗的优先级设低

```java
@EnableWebSecuritypublic 
class MultiHttpSecurityConfig {
        @Configuration
        @EnableResourceServer                                                   
        public static class ResourceServerConfig extends ResourceServerConfigurerAdapter {

                @Override
                protected void configure(HttpSecurity http) throws Exception {
                        http
                                .antMatcher("/auth/**")       // 指定该Adapter只处理/auth/**的请求
                                .authorizeRequests()   // 对该路径做更细的权限控制
                                .antMatchers("/admin/**").hasAuthority("ROLE_ADMIN")
                                .antMatchers("/user/**").hasAuthority("ROLE_USER")
                                .anyRequest().authenticated();
                }
        }

        @Configuration
        public static class WebSecurityConfig extends WebSecurityConfigurerAdapter {
                protected void configure(HttpSecurity http) throws Exception {
                        http
                                .authorizeRequests()  // 因为它的顺序在上面的Adapter之后，所以实际是对/auth/**外的所有请求做权限的控制
                                .antMatchers("/actutor/**").permitAll(
                                .antMatchers("/admin/**").hasAuthority("ROLE_ADMIN")
                                .anyRequest()
                                .authenticated();
                }
        }   
}
```

## 1.4、Tips

spring security集成Oauth2后有个默认的AuthorizationServerSecurityConfiguration，其order为0，该类是对/oauth/token、/oauth/token\_key、/oauth/check\_token三个url做处理的

```java
protected void configure(HttpSecurity http) throws Exception {
    AuthorizationServerSecurityConfigurer configurer = new AuthorizationServerSecurityConfigurer();
    FrameworkEndpointHandlerMapping handlerMapping = this.endpoints.oauth2EndpointHandlerMapping();
    http.setSharedObject(FrameworkEndpointHandlerMapping.class, handlerMapping);
    this.configure(configurer);
    http.apply(configurer);
    String tokenEndpointPath = handlerMapping.getServletPath("/oauth/token");
    String tokenKeyPath = handlerMapping.getServletPath("/oauth/token_key");
    String checkTokenPath = handlerMapping.getServletPath("/oauth/check_token");
    if (!this.endpoints.getEndpointsConfigurer().isUserDetailsServiceOverride()) {
        UserDetailsService userDetailsService = (UserDetailsService)http.getSharedObject(UserDetailsService.class);
        this.endpoints.getEndpointsConfigurer().userDetailsService(userDetailsService);
    }
    // /oauth/token的权限写死为fullyAuthenticated
    // /oauth/token_key、/oauth/check_token的权限则是可配的，默认为denyAll，可在实现AuthorizationServerConfigurerAdapter的配置类中修改
    ((RequestMatcherConfigurer)((HttpSecurity)((AuthorizedUrl)((AuthorizedUrl)((AuthorizedUrl)http.authorizeRequests().antMatchers(new String[]{tokenEndpointPath})).fullyAuthenticated().antMatchers(new String[]{tokenKeyPath})).access(configurer.getTokenKeyAccess()).antMatchers(new String[]{checkTokenPath})).access(configurer.getCheckTokenAccess()).and()).requestMatchers().antMatchers(new String[]{tokenEndpointPath, tokenKeyPath, checkTokenPath})).and().sessionManagement().sessionCreationPolicy(SessionCreationPolicy.NEVER);
    http.setSharedObject(ClientDetailsService.class, this.clientDetailsService);
}

public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
        public void configure(AuthorizationServerSecurityConfigurer oauthServer) throws Exception {
            oauthServer
                .tokenKeyAccess("permitAll()")
                .checkTokenAccess("isAuthenticated()");
        }
}
```

