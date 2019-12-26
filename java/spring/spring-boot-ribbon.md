## spring-boot-ribbon

##### Spring  怎么运用：RestTemplate

```
 @LoadBalanced
 RestTemplate restTemplate;
 
 RestTemplate中进行http请求时，这个请求就会被拦截器拦截进行【ClientHttpRequestInterceptor】
 List<ClientHttpRequestInterceptor> interceptors 
 
 实现：LoadBalancerInterceptor 类
  ribbon就是通过这个拦截器进行拦截请求，然后实现负载均衡调用

```

##### 拦截器定义在

```
org.springframework.cloud.client.loadbalancer.LoadBalancerAutoConfiguration.LoadBalancerInterceptorConfig#ribbonInterceptor

@Configuration
@ConditionalOnMissingClass("org.springframework.retry.support.RetryTemplate")
static class LoadBalancerInterceptorConfig {
   @Bean
    //定义ribbon的拦截器
   public LoadBalancerInterceptor ribbonInterceptor(
         LoadBalancerClient loadBalancerClient,
         LoadBalancerRequestFactory requestFactory) {
      return new LoadBalancerInterceptor(loadBalancerClient, requestFactory);
   }

   @Bean
   @ConditionalOnMissingBean
    //定义注入器，用来将拦截器注入到RestTemplate中，跟上面配套使用
   public RestTemplateCustomizer restTemplateCustomizer(
         final LoadBalancerInterceptor loadBalancerInterceptor) {
      return restTemplate -> {
               List<ClientHttpRequestInterceptor> list = new ArrayList<>(
                       restTemplate.getInterceptors());
               list.add(loadBalancerInterceptor);
               restTemplate.setInterceptors(list);
           };
   }
}

```

