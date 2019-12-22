# 系统日志实现

* `syslog` 

  最初的系统日志项目, 定义了最基础的日志协议`syslog`

  配置位于`/etc/syslog.conf`

* `rsyslog` 

  兼容`syslog`协议, 使用类似`syslog`的配置语法, 并提供新的特性

  配置位于`/etc/rsyslog.conf`

  > 听说也使用`syslog`的配置? 

* `syslog-ng` 

  兼容`syslog`协议, 使用全新的配置语法, 并提供新的特性

# 使用

> 见`man syslog` , 我已经不会C了, 简单介绍

无论使用的何种系统日志, 都支持`syslog`协议, 因此程序中日志的使用应该是日志实现无关的.

函数原型如下:

```c
#include <syslog.h>

void openlog(const char *ident, int option, int facility);
void syslog(int priority, const char *format, ...);
void closelog(void);

void vsyslog(int priority, const char *format, va_list ap);
```

这里仅介绍`facility`和`level`参数, 因此SSHD配置中用到了

* `facility`

  指定程序的类型, 能够让日志系统区别对待不同程序的日志. 常见值如下:

  ```config
  LOG_AUTH       security/authorization messages
  
  LOG_AUTHPRIV   security/authorization messages (private)
  
  LOG_CRON       clock daemon (cron and at)
  
  LOG_DAEMON     system daemons without separate facility value
  
  LOG_FTP        ftp daemon
  
  LOG_KERN       kernel messages (these can't be generated from user processes)
  
  LOG_LOCAL0 through LOG_LOCAL7
  reserved for local use
  
  LOG_LPR        line printer subsystem
  
  LOG_MAIL       mail subsystem
  
  LOG_NEWS       USENET news subsystem
  
  LOG_SYSLOG     messages generated internally by syslogd(8)
  
  LOG_USER (default)
  generic user-level messages
  
  LOG_UUCP       UUCP subsystem
  ```

* `level`

  日志级别, 依据重要性的降序, 列举如下:

  LOG_EMERG      system is unusable

  ```config
  LOG_ALERT      action must be taken immediately
  
  LOG_CRIT       critical conditions
  
  LOG_ERR        error conditions
  
  LOG_WARNING    warning conditions
  
  LOG_NOTICE     normal, but significant, condition
  
  LOG_INFO       informational message
  
  LOG_DEBUG      debug-level message
  ```
  

# 参考

[What is the difference between syslog, rsyslog and syslog-ng?](https://serverfault.com/questions/692309/what-is-the-difference-between-syslog-rsyslog-and-syslog-ng)