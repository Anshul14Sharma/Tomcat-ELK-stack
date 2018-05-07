## ELK stack ##
This repo aims to provide the tomcat logs configuration for ELK stack.  

ELK stack version 6.2.4  
Ubuntu 17.10  

## Installation Intruction ##  
"ELK" is the acronym for three open source projects: Elasticsearch, Logstash, and Kibana. Elasticsearch is a search and analytics engine. Logstash is a server-side data processing pipeline that ingests data from multiple sources simultaneously, transforms it, and then sends it to a "stash" like Elasticsearch. Kibana lets users visualize data with charts and graphs in Elasticsearch.  

The Elastic Stack is the next evolution of ELK.  

#### ELASTICSEARCH: ####
The open source, distributed, RESTful, JSON-based search engine. Easy to use, scalable and flexible, it earned hyper-popularity among users and a company formed around it, you know, for search.  
#### LOGSTASH: ####  
Logstash is an open source, server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then sends it to your favorite “stash.” (Ours is Elasticsearch, naturally.)  
#### KIBANA: ####
Kibana lets you visualize your Elasticsearch data and navigate the Elastic Stack.  
#### FILEBEAT (Lightweight Shipper for Logs): ####
Filebeat helps you keep the simple things simple by offering a lightweight way to forward and centralize logs and files.  

#### GET STARTED WITH ELK STACK(6.2.4): ####
##### STEP 1: #####
* Download Elasticsearch and Unzip:  
https://www.elastic.co/downloads/elasticsearch  
* Download Logstash and Unzip:  
https://www.elastic.co/downloads/logstash  
* Download Kibana and Unzip:  
https://www.elastic.co/downloads/kibana  
* Download Filebeats and Unzip:  
https://www.elastic.co/downloads/beats/filebeat

##### STEP 2: #####
* INSTALL X-PACK INTO ELASTICSEARCH:  
* Go to the elasticsearch folder:  
 ``./bin/elasticsearch-plugin install x-pack``  
* Start Elasticsearch:  
``./bin/elasticsearch``  
* Generate default passwords:  
``./bin/x-pack/setup-passwords interactive``  
* Note the passwords for elastic and kibana users.  
* INSTALL X-PACK INTO KIBANA:  
* Go to the kibana folder:  
``./bin/kibana-plugin install x-pack``  
* Add credentials to the kibana.yml file:  
``elasticsearch.username: "kibana"``  
``elasticsearch.password:  "<pwd>"``  
* <pwd> is the password for the kibana set previously for elasticsearch.  
* Start kibana:  
``./bin/kibana``  
##### STEP 3: #####
* CONFIGURING FILEBEAT TO SEND LOGS LINES TO LOGSTASH:  
* Go to the filebeat directory:  
    ``cd filebeat-6.2.4``  
* edit the filebeat.yml configuration file with the following:  
    ``nano filebeat.yml``  
``filebeat.prospectors:  
  type: log  
  enabled: true  
  paths:  
  -/var/log/*.log # Absolute path to the file or files from where you want to import logs into logstash.  
  ##Multiline options  
  multiline.pattern: ^[0-9]{2}-(?:Jan(?:uary)?|Feb(?:ruary)?|Mar(?:ch)?|Apr(?:il)?|May|Jun(?:e)?|Jul(?:y)?|Aug(?:ust)?|Sep(?:tember)?|Sept|Oct (?:ober)?|Nov(?:ember)?|Dec(?:ember)?)-[0-9]{4} (this will match the timestamp, which is begininng of every newline)  
  multiline.negate: true  
  multiline.match: after  
  #--------------------Logsatsh output--------------------------  
  output.logstash:  
	hosts: ["localhost:5044"]``
* Save your changes.  
* At the data source machine, run Filebeat with the following command:  
``./filebeat -e -c filebeat.yml ``  
* Filebeat will attempt to connect on port 5044. Until Logstash starts with an active Beats plugin, there won’t be any answer on that port, so any messages you see regarding failure to connect on that port are normal for now.  
* Filebeat stores the state of each file it harvests in the registry. However, if you do need to force Filebeat to read the log file from   scratch. To do this, go to the terminal window where Filebeat is running and press Ctrl+C to shut down Filebeat. Then delete the Filebeat   registry file. For example, run:  
``sudo rm data/registry``  
Or take help about multiline option from [filebeat multiline.](https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html)  
##### STEP 4: #####
* STASHING YOUR FIRST EVENT:  
First, let’s test your Logstash installation by running the most basic Logstash pipeline.  
A Logstash pipeline has two required elements,input and output, and one optional element, filter. The input plugins consume data from a   source, the filter plugins modify the data as you specify, and the output plugins write the data to a destination.  
To test your Logstash installation, run the most basic Logstash pipeline. For example:  
``cd logstash-6.2.4``  
``./bin/logstash -e  ‘input { stdin {} } output { stdout {} }’``  
* After starting Logstash, wait until you see "Pipeline main started" and then enter hello world at the command prompt.  
you will see the following output:
``helloworld  
{  
      "@version" => "1",  
    "@timestamp" => 2018-05-07T06:55:29.608Z,  
       "message" => "helloworld",  
          "host" => "anshul-HP-Notebook"  
}``  
* CONFIGURING LOGSTASH FOR FILEBEAT INPUT:  
* Next, you have to create Logstash configuration pipeline that uses Beats input plugin to receive events from Beats.  
``cd logstash-6.2.4``  
``nano first_pipeline.conf``  
* [first_pipeline.conf](https://raw.githubusercontent.com/Anshul14Sharma/Tomcat-ELK-stack/master/first-pipeline.conf)  
* Save your changes.  
* Now, to match the pattern defined in filter we have to provide a file with all grok patterns in it.  
* Create the directory patterns and inside that create file grok-patterns.txt and add the contents from the link below to the file grok-patterns.txt  
[grok-pattens](https://raw.githubusercontent.com/Anshul14Sharma/Tomcat-ELK-stack/master/grok-patterns.txt)  
* To verify your configuration, run the following command:  
``./bin/logstash -f first_pipeline.conf –config.test_and_exit``  
* The –config.test_and_exit option parses your configuration file and reports any errors.  
* If the configuration file passes the configuration test, start Logstash with the following command:  
``./bin/logstash -f first_pipeline.conf –config.reload.automatic``  
* The –config.reload.automatic option enables automatic config reloading so that you don’t have to stop and restart Logstash every time you modify the configuration file.  

##### STEP 5: #####  
* Configure elasticsearch:  
* Go to config folder of elasticsearch  
``cd elasticsearch-6.2.4/config``  
``nano elasticsearch.yml``  
* Edit the following lines to the file:  
`` cluster.name: Vega # Cluster name for your elasticsearch  
 node.name: Virgo # Node name for your elasticsearch   
 network.host: localhost  
 http:port: 9200``
* Save your changes.  
* Run elasticsearch using:  
``./bin/elasticsearh``  
* Point your browser at ``http://localhost:9200`` or ``curl -u username:password -XGET ‘localhost:9200?pretty’``  
* Following output will appear:  
``{  
  "name" : "Virgo",  
  "cluster_name" : "Vega",  
  "cluster_uuid" : "p6yFcOePQjWzVJ1NCjcawQ",  
  "version" : {  
    "number" : "6.2.4",  
    "build_hash" : "ccec39f",  
    "build_date" : "2018-04-12T20:37:28.497551Z",  
    "build_snapshot" : false,  
    "lucene_version" : "7.2.1",  
    "minimum_wire_compatibility_version" : "5.6.0",  
    "minimum_index_compatibility_version" : "5.0.0"  
  },  
  "tagline" : "You Know, for Search"  
}``  
##### STEP 6: #####
* Configuring kibana:  
* Go to the kibana directory  
* Open config/kibana.yml in an editor  
``nano config/kibana.yml``  
* Edit file with the following lines:  
``elasticsearch.url: "http://localhost:9200"  
elasticsearch.username: "elastic"  
elasticsearch.password: "elastic"``  
* Save your changes.  
* Run kibana using:  
``./bin/kibana``  
* Point your browser at ``http://localhost:5601``

