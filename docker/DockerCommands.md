### Running Just HBase

    $ docker run -d --name=hbase -p 8080:8080 -p 8085:8085 -p 9090:9090 -p 9095:9095 -p 2181:2181 -p 16010:16010 -v /data/hbase:/data datuh/pyml:1.0
