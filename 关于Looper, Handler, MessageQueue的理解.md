# 关于Looper, Handler, MessageQueue的个人理解
* 类比于我们常用的消息中间件RabbitMQ
    * Looper就相当于RabbitMQ
    * MessageQueue相当于RabbitMQ中具体的队列
    * Handler相当于RabbitMQ的生产者和消费者(类似RabbitMQ的生产者和消费者注册到一个队列上)

## Looper
* 消息队列的管理者, 管理整个队列的运行和操作, 比如分发消息等
* 这个类本身不提供队列的功能, 它是用来管理队列的

## MessageQueue
* 消息队列本身, 它主要是用来保存消息的, 它用有增加消息, 取得消息等功能

## Handler
### 消息队列的生产者和消费者
1. 关于生产者
    * 当Handler作为生产者时, 通过调用post/sendMessage来生产新消息
    * 作为生产者时, 调用方一般是与Handler关联线程的其他线程, 其他线程通过Handler来生产消息(也就是任务), 最后转到与Handler相关联的线程执行
2. 关于消费者
    * 当Handler作为消费者时, 通过Looper的调度来取得消息, 进行消费
    * 消费过程一定是在与Handler相关联的线程执行的

## 其他
* 类似与RabbitMQ, 同一个队列可以有多个生产者和消费者, 同一个线程下也可以有多个Handler
* 由于Message.target是产生此消息的生产者, 所以通过Message.target来执行具体的任务可以保证生产消息的Handler与消费消息的Handler是同一个Handler