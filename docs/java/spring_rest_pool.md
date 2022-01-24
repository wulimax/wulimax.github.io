###  spring  apache client 和 okhttp连接池配置

```java
import java.util.concurrent.TimeUnit;
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "rest.config")
public class RestTemplateProperties {

    public static final TimeUnit DEFAULT_TIME_TO_LIVE_UNIT;

    private int connectTimeout = 2000;
    private int readTimeout = 5000;

    private int maxTotal = 200;
    private int maxPerRoute = 50;
    private int timeToLive = 900;
    private TimeUnit timeToLiveUnit;

    public RestTemplateProperties() {
        this.timeToLiveUnit = DEFAULT_TIME_TO_LIVE_UNIT;
    }

    public int getConnectTimeout() {
        return connectTimeout;
    }

    public void setConnectTimeout(int connectTimeout) {
        this.connectTimeout = connectTimeout;
    }

    public int getReadTimeout() {
        return readTimeout;
    }

    public void setReadTimeout(int readTimeout) {
        this.readTimeout = readTimeout;
    }

    public int getMaxTotal() {
        return maxTotal;
    }

    public void setMaxTotal(int maxTotal) {
        this.maxTotal = maxTotal;
    }

    public int getMaxPerRoute() {
        return maxPerRoute;
    }

    public void setMaxPerRoute(int maxPerRoute) {
        this.maxPerRoute = maxPerRoute;
    }

    public int getTimeToLive() {
        return timeToLive;
    }

    public void setTimeToLive(int timeToLive) {
        this.timeToLive = timeToLive;
    }

    public TimeUnit getTimeToLiveUnit() {
        return timeToLiveUnit;
    }

    public void setTimeToLiveUnit(TimeUnit timeToLiveUnit) {
        this.timeToLiveUnit = timeToLiveUnit;
    }

    static {
        DEFAULT_TIME_TO_LIVE_UNIT = TimeUnit.SECONDS;
    }
}
```

```java
import java.util.concurrent.TimeUnit;
import okhttp3.ConnectionPool;
import okhttp3.OkHttpClient;
import org.apache.http.conn.HttpClientConnectionManager;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.impl.nio.client.HttpAsyncClientBuilder;
import org.apache.http.impl.nio.conn.PoolingNHttpClientConnectionManager;
import org.apache.http.impl.nio.reactor.DefaultConnectingIOReactor;
import org.apache.http.nio.conn.NHttpClientConnectionManager;
import org.apache.http.nio.reactor.ConnectingIOReactor;
import org.apache.http.nio.reactor.IOReactorException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.HttpComponentsAsyncClientHttpRequestFactory;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.http.client.OkHttp3ClientHttpRequestFactory;
import org.springframework.web.client.AsyncRestTemplate;
import org.springframework.web.client.RestTemplate;

@Configuration
@EnableConfigurationProperties(RestTemplateProperties.class)
public class RestTemplateAutoConfiguration {

    public RestTemplateAutoConfiguration() {
    }

    @Configuration
    @ConditionalOnProperty({"rest.okhttp.enabled"})
    protected static class OkHttpRestTemplateConfiguration {

        public OkHttp3ClientHttpRequestFactory okHttpRequestFactory(RestTemplateProperties templateProperties) {
            OkHttpClient.Builder okHttpClientBuilder = new OkHttpClient().newBuilder();
            okHttpClientBuilder.connectTimeout(templateProperties.getConnectTimeout(), TimeUnit.MILLISECONDS);
            okHttpClientBuilder.readTimeout(templateProperties.getReadTimeout(), TimeUnit.MILLISECONDS);
            okHttpClientBuilder.connectionPool(
                    new ConnectionPool(templateProperties.getMaxTotal(), templateProperties.getTimeToLive(),
                            templateProperties.getTimeToLiveUnit()));
            OkHttp3ClientHttpRequestFactory factory = new OkHttp3ClientHttpRequestFactory(okHttpClientBuilder.build());

            return factory;
        }

        @Bean
        @LoadBalanced
        public RestTemplate okRestTemplate(RestTemplateProperties templateProperties) {
            OkHttp3ClientHttpRequestFactory okHttpRequestFactory = okHttpRequestFactory(templateProperties);
            return new RestTemplate(okHttpRequestFactory);
        }

        @Bean
        @LoadBalanced
        public AsyncRestTemplate asyncRestTemplate(RestTemplateProperties templateProperties) {
            OkHttp3ClientHttpRequestFactory okHttpRequestFactory = okHttpRequestFactory(templateProperties);
            return new AsyncRestTemplate(okHttpRequestFactory);
        }
    }


    @Configuration
    @ConditionalOnProperty(value = {"rest.httpclient.enabled"}, matchIfMissing = true)
    protected static class HttpClientRestTemplateConfiguration {

        private static final Logger log = LoggerFactory.getLogger(HttpClientRestTemplateConfiguration.class);

        public HttpComponentsClientHttpRequestFactory httpClientRequestFactory(
                RestTemplateProperties templateProperties) {
            HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
            factory.setConnectTimeout(templateProperties.getConnectTimeout());
            factory.setReadTimeout(templateProperties.getReadTimeout());
            return factory;
        }

        public PoolingHttpClientConnectionManager poolingHttpClientConnectionManager(
                RestTemplateProperties templateProperties) {
            PoolingHttpClientConnectionManager poolManager = new PoolingHttpClientConnectionManager();
            poolManager.setMaxTotal(templateProperties.getMaxTotal());
            poolManager.setDefaultMaxPerRoute(templateProperties.getMaxPerRoute());
            poolManager.setValidateAfterInactivity(templateProperties.getTimeToLive());
            return poolManager;
        }

        public HttpClientBuilder httpClientBuilder(RestTemplateProperties templateProperties) {
            HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();
            HttpClientConnectionManager connectionManager = poolingHttpClientConnectionManager(templateProperties);
            httpClientBuilder.setConnectionManager(connectionManager);
            return httpClientBuilder;
        }

        @Bean
        @LoadBalanced
        public RestTemplate restTemplate(RestTemplateProperties templateProperties) {
            HttpComponentsClientHttpRequestFactory requestFactory = httpClientRequestFactory(templateProperties);
            requestFactory.setHttpClient(httpClientBuilder(templateProperties).build());
            return new RestTemplate(requestFactory);
        }

        public HttpComponentsAsyncClientHttpRequestFactory asyncRequestFactory(
                RestTemplateProperties templateProperties) {
            HttpComponentsAsyncClientHttpRequestFactory asyncFactory = new HttpComponentsAsyncClientHttpRequestFactory();
            asyncFactory.setConnectTimeout(templateProperties.getConnectTimeout());
            asyncFactory.setReadTimeout(templateProperties.getReadTimeout());
            return asyncFactory;
        }

        public PoolingNHttpClientConnectionManager nHttpClientConnectionManager(
                RestTemplateProperties templateProperties) {
            ConnectingIOReactor ioReactor = null;
            try {
                ioReactor = new DefaultConnectingIOReactor();
            } catch (IOReactorException e) {
                log.error("构建异步连接失败", e);
            }
            assert ioReactor != null;
            PoolingNHttpClientConnectionManager poolNManager = new PoolingNHttpClientConnectionManager(ioReactor);
            poolNManager.setMaxTotal(templateProperties.getMaxTotal());
            poolNManager.setDefaultMaxPerRoute(templateProperties.getMaxPerRoute());
            poolNManager.closeIdleConnections(templateProperties.getTimeToLive(), TimeUnit.SECONDS);
            return poolNManager;
        }

        public HttpAsyncClientBuilder httpAsyncClientBuilder(RestTemplateProperties templateProperties) {
            HttpAsyncClientBuilder httpAsyncClientBuilder = HttpAsyncClientBuilder.create();
            NHttpClientConnectionManager nConnectionManager = nHttpClientConnectionManager(templateProperties);
            httpAsyncClientBuilder.setConnectionManager(nConnectionManager);
            return httpAsyncClientBuilder;
        }

        @Bean
        @LoadBalanced
        public AsyncRestTemplate asyncRestTemplate(RestTemplateProperties templateProperties) {
            HttpComponentsAsyncClientHttpRequestFactory asyncFactory = asyncRequestFactory(
                    templateProperties);
            asyncFactory.setAsyncClient(httpAsyncClientBuilder(templateProperties).build());
            return new AsyncRestTemplate(asyncFactory);
        }
    }
}

```

```yaml
#自定义RestTemplata超时与连接池配置
#rest.httpclient.enabled：httpclient 客户端配置；true 开启，false 关闭；
#rest.okhttp.enabled：okhttp 客户端配置；true 开启，false 关闭；
rest:
  #设置 httpclient 组件 client 客户端；enabled=true 开启，enabled=false 关闭；
  httpclient:
    enabled: true
  #设置 okhttp 组件 client 客户端；enabled=true 开启，enabled=false 关闭；
  okhttp:
    enabled: false
  config:
    # 请求超时时间，默认值2000；
    # 限制请求时间内必须请求到服务，并不限制服务处理的返回时间
    connect-timeout: 2000
    # 读取超时时间，默认值5000；
    # 请求处理的超时时间 下级服务响应最大时间，超出时间消费方返回 timeout
    read-timeout: 5000
    # 连接池：整个连接池最大连接数，默认值 200；
    # 连接池中最多持有连接数，超过的连接进入队列等待
    max-total: 100
    # 连接池：每个路由的最大连接数，默认值 50；
    # 每个目标服务器最多持有连接数，所以 max-connections-per-route 的配置要比 max-connections 小
    max-per-route: 25
    # 连接池：连接存活时间，默认值 900（秒）；
    # 连接时指定的连接存活时间；
    time-to-live: 900
    # 连接池：连接存活时间单位，默认值 seconds；
    # 单位类型：DAYS、HOURS、MINUTES、SECONDS、MILLISECONDS、MICROSECONDS、NANOSECONDS
    # 建议这个参数采用默认，不用修改配置
    time-to-live-unit: days
```

feign连接池配置

```yaml
feign:
  httpclient:
    #开启 httpclient 组件 client 客户端
    #enabled: true
    # 连接池：最大连接数，默认值 200；
    # 连接池中最多持有连接数，超过的连接进入队列等待
    # httpclient 与 okhttp 共用
    max-connections: 100
    # 连接池：每个路由的最大连接数，默认值 50；
    # 每个目标服务器最多持有连接数，所以 max-connections-per-route 的配置要比 max-connections 小
    max-connections-per-route: 25
    # 连接池：连接存活时间，默认值 900（秒）；
    # 连接时指定的连接存活时间；
    time-to-live: 300
    #连接池：连接存活时间单位，默认值 seconds；建议这个参数采用默认，不用修改配置
    #单位类型：milliseconds、microseconds、minutes、hours、days、nanoseconds、seconds
    time-to-live-unit: seconds

  okhttp:
    #开启 okhttp 组件 client 客户端
    enabled: true
```

