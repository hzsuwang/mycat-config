## mycat-config mycat读写分离配置
-----------------------------------------------------
        http://www.mycat.io/  mycat下载地扯

### mycat schema.xml 配置
-------------------------------------------------------
        <?xml version="1.0"?>
        <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
        <mycat:schema xmlns:mycat="http://io.mycat/">
            <!-- 定义MyCat的逻辑库 -->
            <schema name="test_schema" checkSQLschema="false" sqlMaxLimit="100" dataNode="qingkaNode"></schema>
    <!-- 定义MyCat的数据节点 -->
    <dataNode name="testNode" dataHost="dtHost" database="test" />
    <!-- 定义数据主机dtHost，连接到MySQL读写分离集群 ,schema中的每一个dataHost中的host属性值必须唯一-->
    <!-- dataHost实际上配置就是后台的数据库集群，一个datahost代表一个数据库集群 -->
    <!-- balance="1"，全部的readHost与stand by writeHost参与select语句的负载均衡-->
    <!-- writeType="0"，所有写操作发送到配置的第一个writeHost，这里就是我们的hostmaster，第一个挂了切到还生存的第二个writeHost-->
    <dataHost name="dtHost" maxCon="500" minCon="20" balance="1" writeType="0" dbType="mysql" dbDriver="native" switchType="2" slaveThreshold="100">
        <!--心跳检测 -->
        <heartbeat>show slave status</heartbeat>
        <!--配置后台数据库的IP地址和端口号，还有账号密码 -->
        <writeHost host="hostMaster" url="192.168.20.135:3306" user="root" password="root123" /><!-- 主库 -->
        <writeHost host="hostSlave" url="192.168.20.134:3306" user="root" password="root123" /><!-- 备库 -->
    </dataHost>
        </mycat:schema>


### mycat server.xml 配置
---------------------------------------------------------
        <?xml version="1.0" encoding="UTF-8"?>
        <!-- - - Licensed under the Apache License, Version 2.0 (the "License"); 
	- you may not use this file except in compliance with the License. - You 
	may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0 
	- - Unless required by applicable law or agreed to in writing, software - 
	distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT 
	WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the 
	License for the specific language governing permissions and - limitations 
	under the License. -->
        <!DOCTYPE mycat:server SYSTEM "server.dtd">
        <mycat:server xmlns:mycat="http://io.mycat/">
	<system>
	<property name="useSqlStat">0</property>  <!-- 1为开启实时统计、0为关闭 -->
	<property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为关闭 -->

		<property name="sequnceHandlerType">2</property>
      <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
        <!--  <property name="fakeMySQLVersion">5.6.20</property>--> <!--设置模拟的MySQL版本号-->
	<!-- <property name="processorBufferChunk">40960</property> -->
	<!-- 
	<property name="processors">1</property> 
	<property name="processorExecutor">32</property> 
	 -->
		<!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena-->
		<property name="processorBufferPoolType">0</property>
		<!--默认是65535 64K 用于sql解析时最大文本长度 -->
		<!--<property name="maxStringLiteralLength">65535</property>-->
		<!--<property name="sequnceHandlerType">0</property>-->
		<!--<property name="backSocketNoDelay">1</property>-->
		<!--<property name="frontSocketNoDelay">1</property>-->
		<!--<property name="processorExecutor">16</property>-->
		<!--
			<property name="serverPort">8066</property> <property name="managerPort">9066</property> 
			<property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property> 
			<property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
		<!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
		<property name="handleDistributedTransactions">0</property>
		
			<!--
			off heap for merge/order/group/limit      1开启   0关闭
		-->
		<property name="useOffHeapForMerge">1</property>

		<!--
			单位为m
		-->
		<property name="memoryPageSize">1m</property>

		<!--
			单位为k
		-->
		<property name="spillsFileBufferSize">1k</property>

		<property name="useStreamOutput">0</property>

		<!--
			单位为m
		-->
		<property name="systemReserveMemorySize">384m</property>


		<!--是否采用zookeeper协调切换  -->
		<property name="useZKSwitch">true</property>
		<property name="charset">utf8mb4</property>
	</system>
	
	<!-- 全局SQL防火墙设置 -->
	<!-- 
	<firewall> 
	   <whitehost>
	      <host host="127.0.0.1" user="mycat"/>
	      <host host="127.0.0.2" user="mycat"/>
	   </whitehost>
       <blacklist check="false">
       </blacklist>
	</firewall>
	-->
	
    <!-- 用户1，对应的MyCat逻辑库连接到的数据节点对应的主机为主从复制集群 -->
    <user name="mtest">
        <property name="password">root123</property>
        <property name="schemas">test_schema</property>
    </user>

    <!-- 用户2，只读权限-->
    <user name="stest">
        <property name="password">root123</property>
        <property name="schemas">test_schema</property>
        <property name="readOnly">true</property>
    </user>
        </mycat:server>

----------------------------------------------------------------
mysql控制台连接管理数据库:
        
        ./mysql -umtest -proot123 -hdb135 -P8066 -Dtest_schema  

