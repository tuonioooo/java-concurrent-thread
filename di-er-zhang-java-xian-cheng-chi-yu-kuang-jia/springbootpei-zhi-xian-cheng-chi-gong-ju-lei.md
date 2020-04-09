# SpringBoot配置线程池工具类

* 配置类

```text
package com.artron.ise.api.utils.pool;

import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;
import java.util.concurrent.ThreadPoolExecutor;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-5-10
 * Time: 16:55
 * info: Springboot 配置线程池
 */
@Configuration
@EnableAsync //开启异步任务支持
public class ThreadPoolOfAsyncConfig implements AsyncConfigurer {

    private static final int CORE_POLL_SIZE = 150;

    private static final int MAX_POLL_SIZE = 200;

    private static final int QUEUE_CAPACITY = 1000;

    private static final int KEEP_ALIVE_SECONDS = 60;

    @Bean("async_executor")
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(CORE_POLL_SIZE);//核心线程数
        taskExecutor.setMaxPoolSize(MAX_POLL_SIZE);//最大线程数
        taskExecutor.setQueueCapacity(QUEUE_CAPACITY);//队列最大长度
        taskExecutor.setKeepAliveSeconds(KEEP_ALIVE_SECONDS);//线程池维护线程所允许的空闲时间
        taskExecutor.setWaitForTasksToCompleteOnShutdown(true);//用来设置线程池关闭的时候等待所有任务都完成再继续销毁其他的Bean
        taskExecutor.setAwaitTerminationSeconds(KEEP_ALIVE_SECONDS);//设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是阻塞住
        taskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());//线程池对拒绝任务(无线程可用)的处理策略
        taskExecutor.initialize();
        return taskExecutor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return null;
    }
}
```

> 注意：这里已经在配置类里加了@EnableAsync注解，就不需要在启动类加了，我看其他文章说还得在启动类加一次，不需要

* 异步接口使用

```text
package com.artron.ise.api.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.AsyncResult;
import org.springframework.stereotype.Service;

import java.util.concurrent.Future;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-5-10
 * Time: 17:00
 * info:
 */
@Service
public class AsyncService {

    private static Logger logger = LoggerFactory.getLogger(AsyncService.class);

    @Async("async_executor")//调用指定配置线程池名称，如果名称不存在会报异常
    public Future<String> configName(){
        logger.info("configName，线程池名称：" + Thread.currentThread().getName());
        return new AsyncResult<>("configName，线程池名称：" + Thread.currentThread().getName());

    }

    @Async //如果不指定配置线程池名称，会找 getAsyncExecutor方法配置的默认线程池
    public Future<String> noConfigName(){
        logger.info("noCofigName，线程池名称：" + Thread.currentThread().getName());
        return new AsyncResult<>("noCofigName，线程池名称：" + Thread.currentThread().getName());

    }
}
```

* controller测试

```text
package com.artron.ise.api.controller;

import com.alibaba.fastjson.JSONObject;
import com.artron.ise.api.service.AsyncService;
import com.artron.ise.api.utils.pool.ThreadPoolConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.stream.IntStream;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-5-10
 * Time: 16:39
 * info:
 */

@RestController
@RequestMapping(value = "/rest")
public class RestController1 {

    @Autowired
    private AsyncService asyncService;

    @RequestMapping(value = "/test")
    public String parseData(Model model) throws ExecutionException, InterruptedException {
        Future<String> future1 = asyncService.configName();
        Future<String> future2 = asyncService.noConfigName();
        String str1 = future1.get();//阻塞，为了显示结果
        String str2 = future2.get();//阻塞，为了显示结果
        JSONObject result = new JSONObject();
        result.put("asyncService.configName", str1);
        result.put("asyncService.noConfigName", str2);

        //控制台打印其实是异步的，可以运行测试

        return JSONObject.toJSONString(result);
    }

}
```

