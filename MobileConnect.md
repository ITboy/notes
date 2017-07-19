# Mobile Connect

## Mepin

### Mepin的几个http接口的理解
* create/access code, get/access code, get/mepin_id是登录的流程
* service provider得到mepin_id后，把mepin_id作为用户id存放用户相关的信息
* create transaction, confirm是控制授权的流程