# Analytics on Stock Market Data Live Stream using AWS

![image](https://github.com/user-attachments/assets/3dc5010a-defa-4445-b38c-423d2301ab0d)

### Data 

 Live Market Data:
 1. During Market Hours
 - Quote Data for AAPL
 - Bar Data for AAPL, MSFT, GOOG, SPY, META, DIA, TSLA
 2. Outside Market Hours 
 - Quote Data for FAKEPACA
 - Bar Data for FAKEPACA 

Websocket : 
- wss://stream.data.alpaca.markets/v2/iex
- wss://stream.data.alpaca.markets/v2/test

### AWS

1. Create EMR Cluster
- Configure Cluster
- Connect to Primary node using ssh


2. Edit EC2 Security Groups - give access to local machine's ip


3. SSH EMR Cluster on local machine
- Navigate to the aws-key.pem directory
- ssh into the emr cluster


4. Set up kafka
- pip install kafka-python
- pip install boto3
- pip install websocket-client streamlit watchdog plotly
-  wget https://downloads.apache.org/kafka/3.5.2/kafka_2.13-3.5.2.tgz
- tar -xzf kafka_2.13-3.5.2.tgz
- cd kafka_2.13-3.5.2
- nano config/server.properties
- look for advertized listeners and change it to ec2's host ip for master node [ ec2's host ip for master node - ip-10-x-x.ec2.internal ]
- look for zookeeper connect and change it to ec2's host ip for master node
- bin/kafka-server-start.sh config/server.properties


5. Set up Producer.py
- vim producer.py
- update kafka-bootstrap-server with [ip-10-x-x.ec2.internal]
- bin/kafka-topics.sh --create --bootstrap-server ip-10-0-x-x.ec2.internal:9092 --replication-factor 1 --partitions 1 --topic symbol_topic
- bin/kafka-topics.sh --create --bootstrap-server ip-10-0-x-x.ec2.internal:9092 --replication-factor 1 --partitions 1 --topic symbol_topic2
- spark-submit producer.py



6. Set up Consumer.py
- vim consumer.py
- update kafka-bootstrap-server with [ip-10-x-x.ec2.internal]
- bin/kafka-topics.sh --create --bootstrap-server ip-10-0-x-x.ec2.internal:9092 --replication-factor 1 --partitions 1 --topic visual_topic
- bin/kafka-topics.sh --create --bootstrap-server ip-10-0-x-x.ec2.internal:9092 --replication-factor 1 --partitions 1 --topic visual_topic2
- mkdir -p /home/hadoop/consumer1
- nano /home/hadoop/consumer1/log4j.properties
- spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.2.1 \
             --conf "spark.driver.extraJavaOptions=-Dlog4j.configuration=file:/home/hadoop/consumer1/log4j.properties" \
             --conf "spark.executor.extraJavaOptions=-Dlog4j.configuration=file:/home/hadoop/consumer1/log4j.properties" \
             consumer1.py
- vim consumer2.py
- update kafka-bootstrap-server with [ip-10-x-x.ec2.internal]
- spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.2.1 consumer2.py

7. Set up Visualizer (Viz.py)
- ~/.local/bin/streamlit run viz.py --server.port 8501 --server.address 0.0.0.0
- ec2-> Security Groups -> Edit Inbound Rules -> add 8501, 0.0.0.0/0 ->Save rules


8. Set up Log Visualizer (log_viz.py)
-  sudo sysctl fs.inotify.max_user_watches=524288
- ~/.local/bin/streamlit run log_viz.py --server.port 8502 --server.address 0.0.0.0
- ec2-> Security Groups -> Edit Inbound Rules -> add 8502, 0.0.0.0/0 ->Save rules


9. Ganglia (data monitoring)
    - look for ganglia application under emr, enable ssh configuration 
    - ssh tunneling to port 8050 [ ssh -i ./aws-master-node.pem -ND 8050 hadoop@ec2-44-201-28-163.compute-1.amazonaws.com]
    - copy uri into a new tab
    - create a SOCKS5 proxy on the browser
    - proxy onto SOCKS5
    - reload

Demo:
https://stevens.zoom.us/rec/share/y5wKK1-hd9FhHAZ8MyxMaIcaGh0pxRAR-ZCoWAw1HWT0OsA-6SaX33J9n9Lo3TFr.p_vYE-iL_XVfAdoA
