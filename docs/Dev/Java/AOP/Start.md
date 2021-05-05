# AOP start

## demo1

[Pixiv-Illustration-Collection-Backend](https://github.com/OysterQAQ/Pixiv-Illustration-Collection-Backend) commit 74653d11 (`FlowLimitListener.java`):

``` java
@Aspect
@Component
@Slf4j
public class FlowLimitListener {

    /**
     * 用来存放不同接口的RateLimiter(key为接口名称，value为RateLimiter)
     */
    private Map<String, RateLimiter> map = Maps.newConcurrentMap();
    private RateLimiter rateLimiter;

    @Pointcut(value = "@annotation(dev.cheerfun.pixivic.common.annotation.FlowLimit)")
    public void pointCut(){}

    @Around(value = "pointCut()")
    public Object handleRateLimiter(ProceedingJoinPoint joinPoint){
        Object obj=null;
        Signature signature = joinPoint.getSignature();
        MethodSignature methodSignature = (MethodSignature) signature;
        FlowLimit annotation = methodSignature.getMethod().getAnnotation(FlowLimit.class);
        double limitNum = annotation.limitNum();
        String functionName = methodSignature.getName();
        if (map.containsKey(functionName)) {
            rateLimiter = map.get(functionName);
        } else {
            //每秒允许多少请求limitNum
            map.put(functionName, RateLimiter.create(limitNum));
            rateLimiter = map.get(functionName);
        }

        try {
            if (rateLimiter.tryAcquire()) {
                //执行方法
                obj = joinPoint.proceed();
            } else {
                //拒绝了请求（服务降级）
                //TODO 拒绝后提示
                throw new VisitOftenException(CodeEnum.VISIT_OFTEN.getCode(),CodeEnum.VISIT_OFTEN.getMsg());
            }
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        return obj;
    }
}
```

## demo2

[@Aspect 注解使用详解_fz13768884254的博客-CSDN博客](https://blog.csdn.net/fz13768884254/article/details/83538709)
