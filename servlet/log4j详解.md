# log4j优点
相比于System.out.println这样的日志输出，log4j的最大优势就是能够对日志进行定制化处理，如哪些等级的日志可以输出，输出到控制台还是输出到日志文件，是每天输出一个日志文件，还是日志达到一定大小之后进行输出等等，这些都是可以通过用户的配置进行定制的。

log4j的配置一般在程序启动的时候就同步完成了初始化操作，推荐使用配置文件的方式进行配置

# Logger 层级
 - log4j的logger层级是通过logger的名字来进行设置的。例如com.foo就是com.foo.bar的父级logger，子集可以继承父级的logger配置。
 - 程序中通过LogManager.getLogger("logger名字")来获取logger，所有的logger都是单例的。LogManager.getLogger()会使用类名作为logger名称去获取logger，此时logger的继承优势就能体现出来了，例如我们只需配置包层级的logger，包里面的类通过类名获取的logger就都会使用包层级的logger配置
 - 顶级logger 顶级logger默认是一直存在，且是所有logger的父级，有两种获取顶级logger的方式
    ```
    Logger logger = LogManager.getLogger(LogManager.ROOT_LOGGER_NAME);
    
    Logger logger = LogManager.getRootLogger();
    ```

# LoggerContext
loggerContext扮演日志系统的锚点的角色，在一个应用中根据运行环境不同，可能有多个激活的loggerContext

# Configuration
每个loggerContext都有一个激活的configuration，每个configuration包含所有的 *Appenders（附加器--输出目标，输出格式等），context级别的filter（过滤器--输出条件筛选），loggerConfig（日志配置--输出器名称，使用那个附加器等）还包含了对StrSubstitutor的引用*

# Logger
Logger 只配置一个名字和与LoggerConfig的关联
  ## 获取logger
  - 通过调用静态方法LogManager.getLogger("logger名字")来获取
    ```
    Logger x = LogManager.getLogger("wombat");
    Logger y = LogManager.getLogger("wombat");
    ```
    x 和 y 其实是一个logger对象

  - LogManager.getLogger()会使用类名作为logger名称去获取logger

# LoggerConfig
 - ## LogLevel
  
   log4j的内置输出等级有8个，分别是

   > 1. ALL
   > 2. TRACE
   > 3. DEBUG
   > 4. INFO  
   > 5. WARN
   > 6. ERROR
   > 7. FATAL
   > 8. OFF
   
   只有对应级别以上的日志内容才能进行输出

- ## Filter
    除了通过日志等级来对日志进行过滤之外，log4j还提供了Filter（过滤器）来进行过滤。filter可以在下面这些节点上进行过滤
    1. 控制转交给LoggerConfig之前
    2. 控制转交给LoggerConfig之后，使用任何Appender进行输出之前
    3. 控制转交给LoggerConfig之后，使用指定Appender进行输出之前
   
  和防火墙的过滤机制类似，log4j的过滤输出也有三种结果：Accept，Deny，Neutral。Accept表示其他的Filter都不应该再被调用，事件继续传递。Deny表示事件会被立即忽略，控制权返还给调用者。Neutral表示事件需要被传递给其他Filter进行处理。

- ## Appender
   附加器的作用是让各个logger可以有选择的开启或者关闭打印。log4j允许打印到多个目标。在log4j中，一个输出目标就被成为一个Appender。目前支持的appender有
   > 1. console
   > 2. files
   > 3. remote socket servers
   > 4. Apache Flume
   > 5. JMS
   > 6. remote UNIX Syslog daemons
   > 7. various database APIs

   一个Logger可以添加多个Appender，可以通过调用addLoggerAppender方法来将Appender附加到当前的Configuration。如果没有任何一个Logger实例的名字和LoggerConfig中定义的名字一样，那么就会创建一个Logger，同时Appender也会被附加到这个Logger上，并且通知所有的Logger更新它们的LoggerConfig引用

   一个Logger的每一个log请求都会被提交给这个Logger配置的的所有Appender，还有这个Logger的父级Logger配置的所有Appender。也就是说，Appender也会在logger的继承中被继承下来。例如，如果RootLogger中添加了一个Console Appender（控制台附加器），那么所有的可打印的日志（通过Level和Filter的筛选）至少都会在控制台输出。此时如果有个file Appender（文件附加器）被添加到了另一个名字叫做FileLogger的Logger的配置中,那么FileLogger和它的子级Logger都会将可打印的日志输出到控制台，并存储到指定路径的文件中。如果不希望Appender有这种继承关系，可以在配置Logger时添加additivity="false"

- ## Layout
  通常我们不仅希望能够定制输出的位置，也希望能够定制输出的格式。在log4j中通过将layout和Appender进行关联来实现这样的定制需求。Layout负责根据用户的定制需求来格式化日志内容。作为log4j的标准发布内容之一，PatternLayout可以让用户像使用C语言的printf函数的格式一样来定制日志的输出格式。

For example, the PatternLayout with the conversion pattern "%r [%t] %-5p %c - %m%n" will output something akin to:

176 [main] INFO  org.foo.Bar - Located nearest gas station.
The first field is the number of milliseconds elapsed since the start of the program. The second field is the thread making the log request. The third field is the level of the log statement. The fourth field is the name of the logger associated with the log request. The text after the '-' is the message of the statement.

Log4j comes with many different Layouts for various use cases such as JSON, XML, HTML, and Syslog (including the new RFC 5424 version). Other appenders such as the database connectors fill in specified fields instead of a particular textual layout.

Just as importantly, log4j will render the content of the log message according to user specified criteria. For example, if you frequently need to log Oranges, an object type used in your current project, then you can create an OrangeMessage that accepts an Orange instance and pass that to Log4j so that the Orange object can be formatted into an appropriate byte array when required.

