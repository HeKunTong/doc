##方法列表

###getTrackerToken()
获取链追踪器token
```php
TrackerManager::getInstance()->getTracker()->getTrackerToken()
TrackerManager::getInstance()->getTracker(\Swoole\Coroutine::getuid())->getTrackerToken()
```
###addAttribute()
添加参数
```php
TrackerManager::getInstance()->getTracker()->addAttribute('user','用户名1');
TrackerManager::getInstance()->getTracker()->addAttribute('name','这是昵称');
```
###getAttribute()
获取参数
```php
TrackerManager::getInstance()->getTracker()->getAttribute('user');
TrackerManager::getInstance()->getTracker()->getAttribute('name');
```
###getAttributes()
获取所有的参数
```php
TrackerManager::getInstance()->getTracker()->getAttribute()
```
###getStacks()
获取调用栈
```php
TrackerManager::getInstance()->getTracker()->getStacks()
```
###addCaller()
添加一个跟踪点,返回TrackerCaller对象
```php
$caller = TrackerManager::getInstance()->getTracker()->addCaller('CurlBaiDu','wd=easyswoole');
$caller->endCall();//结束跟踪
```
###__toString()
字符串输出对象  

###toString()
按分类转为字符串
```php
TrackerManager::getInstance()->getTracker()->toString()
```
###stackToString()
调用栈转为字符串
```php
TrackerManager::getInstance()->getTracker()->stackToString()
```