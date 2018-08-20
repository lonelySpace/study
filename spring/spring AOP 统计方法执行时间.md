#spring AOP 统计方法的执行时间
1.代码
```java
/**
 * @program: service
 * @description: 方法执行时间统计
 * @author: liujiawei
 * @create: 2018-08-20 15:47
 **/
@Aspect
@Component
public class ExectueTimeInterceptor {

    private static final Logger logger = LoggerFactory.getLogger(ExectueTimeInterceptor.class);

    public static final String POINT = "execution (* com.mashanglc.service.*.biz.*.*(..))";

    @Around(POINT)
    public Object timeAround(ProceedingJoinPoint joinPoint) {
        Object retObj = null;
        Object[] args = joinPoint.getArgs();
        long start = MyDateTimeUtils.getCurrentTime();
        try {
            retObj = joinPoint.proceed(args);
        } catch (Throwable throwable) {
            logger.error("统计某方法执行耗时环绕通知出错", throwable);
        }
        long end = MyDateTimeUtils.getCurrentTime();
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        String methodName = signature.getDeclaringTypeName() + "." + signature.getName();
        logger.info("methodName={}, 耗时{}ms", methodName, end - start);
        return retObj;
    }
}
```
2.spring.xml
```xml
<!--添加这段配置-->
<aop:aspectj-autoproxy />
```