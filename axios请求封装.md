#  请求封装

1. 定义一个基础继承类
   - 类有个$http属性，该属性是一个实例
     - **实例**需要配置请求拦截，以及响应拦截
       - 目前只配置响应拦截就可以
         - code 为 0 正常响应
         - code为其他则调用一个展示弹窗，展示报错信息
     - 实例配置域名
     - 实例配置默认请求头
   - 提供四个方法
     - get获取数据
     - post设置数据
     - put更新数据
     - delete删除数据
   - 提供修改默认配置的方法
     - 实例方法中有default配置
     - 可以传递通用配置
     - 根据不同的方法，使用不同的配置