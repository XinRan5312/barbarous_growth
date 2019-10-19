----------------------------
ActiveMQ-JMS				|
----------------------------
	# JMS,Java Message Service 是JAVAEE中的一种技术规范
	# 当前CORBA,RCOM,RMI等RPC中间件技术已经应用在各个领域了.但是对于复杂度,规模越来越高的分布式系统.这些技术也显示出局限性
		* 同步通信,客户端发出调用后,必须等到响应才能继续执行
		* 客户端和服务端耦合度太高,服务端可客户端都必须正常运行
		* 点对点通信,客户端一次调用只能把消息发送给一个目标对象
	
	# 面向消息中间件,就较好的解决了上面的问题,发送者把消息发送给消息服务器.消息服务器把消息存起来,在合适的时候发送给接受者
		* 发送和接收是异步的,发送者无需等等
		* 两个的生命周期也不一定相同,发送消息的时候,接收者不一定要运行.
		* 一对多通信,一个消息可以有多个接收者
	
	# JAVA消息服务-JMS,定义了JAVA中访问消息中间件的接口,没有做实现.'JMS'只是接口
		* ActiveMQ
		* RocetMQ
		* RabbitMQ
			...都是JMS的实现
	
	# JMS术语
		Provider				生产者
		Consumer				消费者
		PTP						点对点模型
			* 一个消息只能被一个消费者所消费
			* 消费者和生产者之间没有时间上的关联性,他们都是面向队列.来达到解耦

		Pub/Sub					发布/订阅模型
			* 一个消息可以被多个消费者消费(广播,谁都能收到)
			* 消费者和生产者有时间关联性,消费者只能收到订阅之后的消息
			* 持久订阅与非持久订阅
				1,持久订阅.当消费者宕机,重启后.可以收到宕机期间生产者的广播信息
				2,非持久的.消费者只有在线,才能收到消息.宕机后,生产者的所有消息都会丢失

		Queue					队列目标
		Topic					主题目标
		Message					消息
		ConnectionFactory		连接工厂,JMS用它创建连接
		Connection				与MQ服务的虚拟连接
		Destination				消息目的地抽象
		Session					会话,一个发送或者接收的线程,构建于Connection上
		Acknowledge				签收
		Transaction				事务,是属于 Provider 的一个属性,本地事务


	# JMS消息类型
		* JMS定义了五种不同的消息正文格式,以及调用的消息类型.允许你发送并接收一些不同形式的数据,提供现有消息格式的一些级别的兼容性

		StreamMessage		JAVA原始值的数据流
		MapMessage			一套名称-值对
		TextMessage			一个字符串对象
		ObjectMessage		一个序列化的JAVA对象
		BytesMessage		一个未解释字节的数据流

	
	# JMS消息结构
		* JSM消息都由三个部分组成
		1,消息头	
			* 每个消息头字段都有相应的getter和setter方法
			* 消息头包含了消息的识别信息和路由信息,消息头包含一些标准的属性如下
			JMSDestination			//send方法设置
				* 消息发送的目的地,可以是Topic或者Queue

			JMSDeliveryMode			//send方法设置
				* 传送模式,有两种
					1,持久模式:服务出现故障,这条消息会保存到物理介质,恢复后重新发送
					2,非持久式:服务出现故障,消息丢失,它仅仅存在于内存

			JMSExpiration			//send方法设置
				* 消息的过期时间,如果该值为 0,则表示消息不会过期.如果在指定时间内还没有被消费,则自动删除

			JMSPriority				//send方法设置
				* 设置消息的优先级,消息只是理论上按照优先级进行推送.只是理论
				* 0-4 普通级别	5-9 紧急级别
				* 默认为:4

			JMSMessageID			//send方法设置
				* 每个消息的唯一标识,由  Provider 自动生成

			JMSTimestamp			//由客户端设置
				* 消息时间戳,在消息发送出去的时候.会自动的设置

			JMSCorrelationID		//由客户端设置
				* 用来连接另一个消息
				* 典型的应用场景就是:在回复消息中中,连接原消息的 JMSMessageID 值
				* 这个值不仅仅可以是 JMSMessageID ,由开发者设置

			JMSReplyTo				//由客户端设置
				* 提供当前消息的回复地址,由开发者设置

			JMSType					//由客户端设置
				* 消息类型的识别符,由开发者设置
				
			JMSRedelivered			//由JMS Provider设置
				* 如果说消费者收到了一个设置了 JMSRedelivered 属性的消息,则表示消费者在早些时候已经消费过这个消息.但是没有签收(acknowledged)
				* 如果消息未ack,被重新传送,则 JMSRedelivered=true 反之 JMSRedelivered=false
				* 系统自动设置,不需要自己关心

		2,消息属性	:如果需要除消息头字段以外的值,那么可以使用消息属性
			* 通常来说,消息属性包含一下三种类型
			1,应用程序设置和添加属性
				message.setStringProperty(name, value);
				message.setBooleanProperty(name, value);
				message.setByteProperty(name, value);
				message.setDoubleProperty(name, value);
				message.setFloatProperty(name, value);
				message.setIntProperty(name, value);

			2,JMS定义属性
				* 以 "JMSX" 作为属性名的前缀,
				* connection.getMetaData().getJMSXPropertyNames();  //返回所有支持JMSX属性的名字
				JMSXUserID		
					* 发送消息的用户标识,发送时提供商设置
				JMSXAppID
					* 发送消息的应用标识,发送时提供商设置
				JMSXDeliveryCount
					* 转发消息重试次数,第一次是1,第二次是2.由提供商设置
				JMSXGroupID
					* 消息所在消息组的标识,由客户端设置
				JMSXGroupSeq
					* 组内消息的序列,第一个是1,第二个是2,由客户端设置
				JMSXProducerTXID
					* 产生消息的事务,的事务标识.发送时,提供商设置
				JMSXConsumerTXID
					* 消费消息的事务,的事务标识.接受时,提供商设置
				JMSXRevTimestamp
					* JMS转发消息到消费者的时候,接收时,提供商设置
				JMSXState
					* 假设存在一个消息仓库.里面存储了多所有消息的一个拷贝.
					* 每个拷贝有消息的状态信息:准备,发送.... ...
				

			3,JMS供应商特定的属性
				尽量少用.
					

		3,消息体	:封装具体的消息数据,有很多种.(看上面)


	# 消费者两种方式消费消息
		1,同步消费:通过调用消费者的receive方法,阻塞线程.直到有消息到达
		2,异步消费:消费者注册监听,当消息到达,会触发监听事件
	
	
	# JMS的可靠性机制
		1 消息的ACK机制
			* JMS消息只有在被确实后,才认为别已经成功的消费了.消息的成功消费包含三个阶段
				1,消费者接收消息
				2,消费者处理消息
				3,消息被确认
			* 事务性会话中,当消费者执行commnit操作的时候,确认自动提交
			* 非事务性会话中,取决于创建 Session 时设置的应答模式
				static final int AUTO_ACKNOWLEDGE = 1;
					* 消费者从receive或者onMessage方法返回的时候,就确定该消息被消费

				static final int CLIENT_ACKNOWLEDGE = 2;
					* 消费者需要手动的调用 message 的 acknowledge 方法,来确定消息的成功消费
					* 这种模式中,确定是在会话层上进行的,确认一个被消费的消息.将会自动确认所有已经被会话消费的消息
					* 例如:消费者消息10个消息,消费到第五个的时候,执行确定.那么这10个都会自动确定

				static final int DUPS_OK_ACKNOWLEDGE = 3;
					* 该选择只是会话迟钝的确定消息的提交,如果JMS Provider 失败,那么可能会导致一些重复的消息.如果是重复的消息,JMS Provider 必须把消息头的 JMSRedelivered 字段值设置为 true
					* 只是说性能好点点,不会时时的去确定消息

				static final int SESSION_TRANSACTED = 0;
		
		2,消息的持久化
			* JMS支持一下两种消息'提交模式'
				PERSISTENT
					* 要求 Provider 持久化保存消息,保证该消息不会因为 JMS Provider 的失败而丢失
				NON_PERSISTENT
					* 不要求保存
		
		3,持久订阅
			* 首先消息生产者必须使用 PERSISTENT 来发送消息.
			* 消费者可以通过会话的 createDurableSubscriber(); 来创建一个持久订阅,方法第一个参数就是指定 topic,第二参数是订阅的名称
			* JMS Provider 会存储发布到持久订阅对应的 topic 上的消息。如果最初创建持久订阅的消费者或者其他任何消费者,使用相同的连接工厂和连接的客户ID,相同的主题和相同的订阅名.再次调用会话上的createDurableSubscriber();方法,那么该持久订阅就会被激活.
			* JMS Provider 会向消费者发送,'消费者挂机'时,所发布的消息
			* 持久订阅在某个时刻,只能有一个激活的订阅者.持久订阅在创建之后会一直保留,直到应用程序调用会话上的 ubsubscribe 方法

		4,本地事务
			* 在JMS 客户端,可以使用本地事务来组合消息的发送和接收
			* JMS的Session 接口提供了commnit和rollback方法.
			* 事务的提交意味着生产的所有消息被发送,消费的所有消息被确定
			* 事务的回滚意味着生产的所有消息被销毁,消费的所所有消息被恢复,且会重新的投递.除非它们已经过期
			* 事务性的会话总是牵涉到事务处理中,commnit或rollback方法一旦被调用.一个事务就结束了.而另一个事务被开始,关闭事务性会话将回滚其中的事务。
			* 需要注意,如果是使用请求/回复 机制,也就是发送一个消息,同时希望在同一个事务中等待接收该消息的回复.那么程序会被挂起,因为直到事务提交,发送操作才会真正执行

		
	
	# JMS PTP模型
		* PTP是基于队列的,生产者发送消息到队列,消费者从队列获取消息
		* PTP的特点
			1,如果在Session关闭的时候,有一些消息已经被收到,但还没有签收.那么当消费者下次连接到相同的队列的时候,这些消息还会被再次接收
			2,如果用户在 receive 方法中设定了消息的选择条件,那么不符合条件的消息会被留在队列中,不会被接收到
			3,队列可以长久的保存消息,直到消费者消费.消费不需要因为担心消息会丢失,而时刻保持连接激活的状态.充分提现异步传输的优势

	# JMS PUB/SUB模型
		* 这种模式定义了如果向一个内容节点发布和订阅消息,这些节点称为 topic
		* 主题可以被认为是消息的传输中介,发布者发布消息到主题,订阅者从主题获取消息,主题使得消息订阅者和发布者保持互相独立,不需要接触即可保证消息的传送
		* 特点
			1,消息分为持久性订阅和非持久性订阅
				* 非持久性:消费者宕机期间的消息,会丢失.只有保持正常连接状态才可以获取到消息
				* 持久性的:持久性订阅的时候,消费者会向JMS注册一个自己身份的ID,如果自己宕机了.那么 Provider 会把这个ID宕机期间的消息全部保存.当这个ID的消费者恢复连接以后,就可以获取到宕机期间丢失的所有消息
			2,如果消费在receive 方法中设定了消息选择条件,那么不符合条件的信息不会被接收
			3,非持久订阅状态下,不能恢复或者重新派送一个未签收的消息.只有持久订阅才能恢复或重新派送一个未签收的消息
			4,当所有的消息必须被接收,则使用持久订阅.当丢失消息能够容忍,则使用非持久订阅

	# 消息临时目的地
		* 可以通过会话 createTemporaryQueue 方法和 createTemporaryTopic 来创建临时目的地
		* 它们的存在时间,只限于创建它们的连接所保持的时间.
		* 只有创建该临时目的地的连接上的消费者才能够从临时目的地中获取到消息
		* '一般的连接,创建了目的地后会保存在MQ,程序结束还在.下次还可以用.临时的就是,下次重启程序就没了。。MQ不会保存'
	

	# JMS API结构
				
							ConnectionFactory
									|
								Connection
									|
			MessageProducer	-	Session	-	MessageConsumer
				|								|
			Destination						Destination
		
	* 消息接收者
		1,创建连接工厂
		2,通过工厂获取连接
		3,通过连接创建会话
		4,通过会话创建队列/主题
		5,通过会话创建消息接收者
		6,消息接收者阻塞,收到消息,获取到message
		7,获取数据

	* 消息生产者
		1,创建连接工厂
		2,通过工厂获取连接
		3,通过连接创建会话
		4,通过会话创建队列/主题

		5,通过会话创建消息生产者对象,绑定队列/主题
			* 也可以不绑定队列/主题,在执行发送的时候绑定
		6,设置消息生产者对象的部分属性
			* 也可以在执行发送的时候设置
		7,创建消息
			* 消息类型有N多种,根据需要创建
		8,通过消息生产者执行发送操作
			* 可以在这里绑定队列/主题,各种属性
		9,关闭资源,仅仅需要关闭连接即可,会自动关闭相关的连接