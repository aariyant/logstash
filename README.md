# Logstash
---
## Installing Logstash

<details open>
<summary><b><font size="4">Linux</font></b></summary>
<p>

__Install Logstash using APT :__
```shell
# Download and install the Public Signing Key:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# You may need to install the apt-transport-https package on Debian before proceeding:
sudo apt-get install apt-transport-https

# Save the repository definition to /etc/apt/sources.list.d/elastic-7.x.list:
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

# Run sudo apt-get update and the repository is ready for use. You can install it with:
sudo apt-get update && sudo apt-get install logstash

# To start
sudo systemctl start logstash.service
```

__Install Logstash using YUM :__
```shell
# Download and install the public signing key:
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

# Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example logstash.repo
echo "[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md" | sudo tee /etc/yum.repos.d/logstash.repo

# And your repository is ready for use. You can install it with:
sudo yum install logstash

# To start
sudo systemctl start logstash.service
```

__Install Logstash using Binary :__
1. Open this url - https://www.elastic.co/downloads/logstash.
2. If you want to select specific version, you can search from this url - https://www.elastic.co/downloads/past-releases#logstash
3. Select your platform and download TARG.GZ, DEB, ZIP, or RPM package.
4. Extract the package to a local folder.

</p>
</details>

<details open>
<summary><b><font size="4">macOS</font></b></summary>
<p>

__Install Logstash using Binary :__
1. Open this url - https://www.elastic.co/downloads/logstash.
2. If you want to select specific version, you can search from this url - https://www.elastic.co/downloads/past-releases#logstash
3. Select your platform and download TARG.GZ, DEB, ZIP, or RPM package.
4. Extract the package to a local folder.

__Install with homebrew :__
```shell
# To install with Homebrew, you first need to tap the Elastic Homebrew repository:
brew tap elastic/tap

# After you’ve tapped the Elastic Homebrew repo, you can use brew install to install the default distribution of Logstash:
brew install elastic/tap/logstash-full

# Starting Logstash with Homebrew
brew services start elastic/tap/logstash-full
logstash
```

</p>
</details>

<details open>
<summary><b><font size="4">Windows</font></b></summary>
<p>

__Configuring JDK in Windows__
1. Download and install Oracle Java Development Kit v8. Choose all defaults.
2. Click Start, search for "Environment Variables" and open the system properties applet. The advanced tab of the "System Properties" applet should appear.
3. Click the Environment Variables button.
4. Under System variables click "New".
5. Enter the variable name "JAVA_HOME" (without quotes) and browse to the JDK install directory and click OK. It will look like this:

__Install Logstash using Binary :__
1. Download the Logstash ZIP package from here - https://www.elastic.co/downloads/logstash. 
2. If you want to select specific version, you can search from this url - https://www.elastic.co/downloads/past-releases#logstash
3. Extract the ZIP contents to a local folder. For this example I will extract the contents to C:\logstash\

</p>
</details>

<details open>
<summary><b><font size="4">Docker</font></b></summary>
<p>

1. Make sure you already installed docker compose, if not please follow this url [install-docker-compose](https://docs.docker.com/compose/install)
2. Create file with name `docker-compose.yaml` and paste this configuration :
    ```yaml
    version: '3'
    services:
      logstash:
        image: logstash:7.17.23
        container_name: logstash
        environment:
          - ELASTICSEARCH_HOST=http://192.168.31.102:9200
          - MYSQL_HOST=mysql
        ports:
          - "5044:5044"
          - "9600:9600"
        volumes:
          - ./config/pipeline:/usr/share/logstash/pipeline
          - ./config/logstash.yml:/usr/share/logstash/config/logstash.yml
          - ./config/pipelines.yml:/usr/share/logstash/config/pipelines.yml
          - ./data:/usr/share/logstash/data
          - ./sample-files:/tmp/log-files/
          - ./database/mysql-connector-java-8.0.22.jar:/home/mysql-connector-java-8.0.22.jar
        networks:
          - logstash
    
    networks:
      logstash:
        driver: bridge
    ```
3. Save it and execute this command :
    ```shell
    docker-compose up -d
    ```
> __NOTE:__  Dont forget to update the `environment` values.

</p>
</details>

---

## Processing First Event
To run a logstash, we need to configure a pipeline. The pipeline consists of three parts, inputs, outputs, and optionally filters.

To define or configure pipeline, we can writing a configuration file consisting of the three parts of a pipeline. But to make sure the logstash can run normally, we can run the logstash without editing the pipeline configuration file. We can define a pipeline configuration as an argument when start logstash.

__Example :__
```shell
./bin/logstash -e "input { stdin {} } output { stdout {} }"
```

__Expected response :__
```shell
[2024-08-12T21:38:35,064][INFO ][logstash.javapipeline    ][main] Pipeline started {"pipeline.id"=>"main"}
The stdin plugin is now waiting for input:
[2024-08-12T21:38:35,069][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
```

If success logstash will show log like that, and we can move configuration into pipeline configuration file at `config/pipeline/first-event.conf`

And run it again with this following command :
```shell
./bin/logstash -f config/pipeline/first-event.conf
```

### Handling JSON Input
What if i want handle an incoming json input instead of just some arbitrary and unstructured text ? By using JSON, we can have different fields and thereby work with structured data.

__Example :__
```
{ "first": "Hello", "second": "World" }
```

__Expected response :__
```
{
       "message" => "{ \"first\": \"Hello\", \"second\": \"World\" }",
          "host" => "ip-192-168-10-109",
    "@timestamp" => 2024-08-12T15:24:46.271Z,
      "@version" => "1"
}
```
Based on the response, that is not the response what we looking for. Because we dont see any new fields in the output and we have no way actually doing something with the json. To fix that, we can use `codec` plugin to parsing the json.

```
input {
  codec => json
}
```

__Expected response :__
```
{
      "@version" => "1",
        "second" => "World",
    "@timestamp" => 2024-08-12T15:32:57.308Z,
          "host" => "ip-192-168-10-109",
         "first" => "Hello"
}
```

### Outputing Events to File
Logstash also can send the output into multiple target. Example, we can add file output alongside the existing one. To do that, we can use pluging called `file`.

__Example :__
```
output {
    stdout {}
    file {
        path => "output.txt"
    }
}
```

### Working with HTTP Input
With HTTP input, we can send an event to logstash through http. Logically the plugin is named `http`. By default, we can add `http` plugin without any options. It will running to the localhost  and listen on port `8080`.
__Example :__
```
input {
    stdin {
        codec => json
    }
    http {
        host => "0.0.0.0"
        port => 8081
    }
}
```
`http` plugin will automatically check the content type from `Content-Type` header when user send a request to logstash.

__Example :__
```shell
curl 127.0.0.1:8081 -H 'Content-Type: application/json' -d '{ "first": "hello", "second": "world" }'
```

__Expected response :__
```
{
    "@timestamp" => 2024-08-12T15:55:53.921Z,
      "@version" => "1",
          "host" => "127.0.0.1",
        "second" => "world",
       "headers" => {
           "http_version" => "HTTP/1.1",
           "request_path" => "/",
        "http_user_agent" => "curl/8.6.0",
         "content_length" => "39",
            "http_accept" => "*/*",
           "content_type" => "application/json",
         "request_method" => "POST",
              "http_host" => "127.0.0.1:8081"
    },
         "first" => "hello"
}
```
### Filtering Events
Filter plugin will performs some kind of processing on events before handling them off to the configured outputs. One of the very common filter plugin to use is one named `mutate`. The mutate filter can perform various actions, such as renaming or copying fields, lower casing and upper casing strings, replacing values, and more.

For example, we can convert some field value from one data type to another.

__Example :__
```
filter {
    mutate {
        convert => { "count" => "integer" }
    }
}
```

#### Common Filter Options
| option    | purpose |
|-----------|---------|
| add_field | adds one or more fields to the event |
| remove_field | removes one or more fields from the event |
| add_tag | adds one or more tags to the event |
| remove_tag | removes one or more tags from the event |


## Parsing Request with Grok
Grok is a powerful tool for extracting structured data from unstructured text. Grok syntax is composed of reusable elements called Grok patterns that enable parsing for data such as timestamps, IP addresses, hostnames, log levels, and more.The prebuilt patterns make Grok easier to use than defining new regular expressions to extract structured data, especially for long text strings.

Grok patterns follow the syntax: `%{PATTERN TO MATCH:output label}`. For more detail we can check [here](https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1/grok-patterns)

### Ingest data from file
Logstash support to read an input from one or many file in one time. To do this, we can use `path` plugin.

__Example :__
```
input {
    file {
        path => "/path/access.log"
        start_position => "beginning"
    }
}
```

For example, we have some log like this, we can parsing it with grok plugin for filter each field.

__Example :__
```shell
123.116.207.53 - - [14/Aug/2024:22:57:02 +0700] "GET /id/necessitatibus/vitae.json HTTP/1.0" 200 4969 "https://garcia.com/explore/main/list/index/" "Mozilla/5.0 (Windows 95) AppleWebKit/5321 (KHTML, like Gecko) Chrome/13.0.852.0 Safari/5321"

%{IPORHOST:remote_ip} - %{DATA:user_name} \[%{HTTPDATE:access_time}\] \"%{WORD:http_method} %{DATA:url} HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{NUMBER:body_sent_bytes} \"%{DATA:referrer}\" \"%{DATA:agent}\"
```

```
input {
    file {
        path => "/path/access.log"
        start_position => "beginning"
    }

    http {

    }
}

filter {
    grok {
        match => {
            'message' => '%{IPORHOST:remote_ip} - %{DATA:user_name} \[%{HTTPDATE:access_time}\] \"%{WORD:http_method} %{DATA:url} HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{NUMBER:body_sent_bytes} \"%{DATA:referrer}\" \"%{DATA:agent}\"'
        }
    }
}

output {
    stdout {
    }
}
```

## Conditional Statement
|Group | Operators |
|------|-----------|
|equality | ==, !=, <, >, <=, >=|
|regexp | =~, !~ |
|inclusion | in, not in |

__Example :__
```
input {
  file {
    path => "/path/access.log"
    start_position => "beginning"
  }
  http {

  }
}

filter {
  if [headers][request_path] =~ "error" or [path] =~ "errors" {
    mutate {
      replace => { type => "error" }
    }
  } else {
    mutate {
      replace => { type => "access" }
    }
    grok {
      match => { 'message' => '%{IPORHOST:remote_ip} - %{DATA:user_name} \[%{HTTPDATE:access_time}\] \"%{WORD:http_method} %{DATA:url} HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{NUMBER:body_sent_bytes} \"%{DATA:referrer}\" \"%{DATA:agent}\"' }
    }
    
    mutate {
      convert => {
        "response" => "integer"
        "bytes" => "integer"
      }
    }
  }
}

output {
  stdout {
  }
}
```

## Sending Log to Elasticsearch
To sending output into elasticsearch, we can set the output like this :
__Example :__
```
output {
  elasticsearch {
    hosts => [ "localhost:9200" ]
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```

### Ingesting Another Sources

#### Ingesting CSV File
1. At first, please download the sample csv file "[csv-file](files/csv-file.csv)"
2. Then, create some configuration file inside logstash configuration directory (Example : __./config/pipeline__) and give name `csv-file.conf`
3. For example, you can use this configuration :
    ```shell
    input {
        file {
            path => "{path-to-your-file}"
            start_position => "beginning"
            sincedb_path => "/dev/null"
        }
    }

    filter {
        csv {
            separator => ","
            columns => [
                "locationGroupIdentifier",
                "location.locationIdentifier",
                "locationGroupType",
                "locationGroupName",
                "sourceLink"
            ]
        }
    }

    output {
        elasticsearch {
            hosts => ["${ELASTICSEARCH_HOST}"]
            index => "{index-name}"
        }
        stdout {}
    }
    ```
    Please dont forget to replace the following placeholders :
    - `{path-to-your-file}` replace with actual patch of your csv file
    - `${ELASTICSEARCH_HOST}` replace with your elasticsearch host and port
    - `{index-name}` replace with the desired index name in elasticsearch
4. Save and restart the logstash
5. To check if ingest success, you can check via elasticsearch or kibana.
    ```shell
    # Via elasticsearch
    curl -XGET "${ELASTICSEARCH_HOST}/{index-name}/_search"

    # Via kibana
    GET {index-name}/_search
    ```

#### Ingesting from MySQL

1. Please download sample sql file from "[here](files/database/product.sql)" or bring your own sql.
2. Create configuration file with this configuration and save it into __./config/pipeline__ with name `mysql.conf` :
    ```shell
    input{
        jdbc {
            jdbc_driver_library => "/home/mysql-connector-java-8.0.22.jar"
            jdbc_driver_class => "com.mysql.jdbc.Driver"
            jdbc_connection_string => "jdbc:mysql://${MYSQL_HOST}:3306/product?zeroDateTimeBehavior=convertToNull"
            jdbc_user => 'root'
            jdbc_password => ''
            statement => "Select product.*, DATE_FORMAT(updatedAt, '%Y-%m-%d %T') as lastTransaction from product where updatedAt > :sql_last_value"
            tracking_column => "lastTransaction"
            tracking_column_type => "timestamp"
            use_column_value => true
            lowercase_column_names => false
            clean_run => true
            schedule => "*/15 * * * * *"
        }
    }
    filter {
        ruby {
            code => "
                if event.get('productStatusFK')
                    productStatusFK = event.get('productStatusFK').to_i
                    if productStatusFK == 0
                        event.set('productStatusFK', 'passive')
                    elsif productStatusFK == 1
                        event.set('productStatusFK', 'active')
                    end
                end
            "
        }
        mutate {
            remove_field => ["@version", "@timestamp"]
        }
    }
    output { 
        elasticsearch {
            hosts => [ "${ELASTICSEARCH_HOST}" ]
            document_id => '%{productPK}'
            index => "product"
            doc_as_upsert => true
            action => "update"
            codec => "json"
            manage_template => true
            template_overwrite => true
        }
    }
    ```
    Please dont forget to replace the following placeholders :
    - `{path-to-your-file}` replace with actual patch of your nginx log file
    - `${ELASTICSEARCH_HOST}` replace with your elasticsearch host and port
3. Save and restart the logstash

### Running Multiple Logstash Pipeline
If you want to store multiple pipeline, you can add `pipelines.yml` configuration to prevent each log merger, this configuration can you save in __./config/__ with name `pipelines.yml`.

Example :
```yml
- pipeline.id: csv-file
  path.config: "config/pipeline/csv-file.conf"

- pipeline.id: mysql
  path.config: "config/pipeline/mysql.conf"
  pipeline.workers: 1
```

# Beats
## Installing Beats
### Install Filebeat

<details open>
<summary><b><font size="4">Linux</font></b></summary>
<p>

__Install using APT :__
```shell
# Download and install the Public Signing Key:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# You may need to install the apt-transport-https package on Debian before proceeding:
sudo apt-get install apt-transport-https

# Save the repository definition to /etc/apt/sources.list.d/elastic-7.x.list:
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

# Run sudo apt-get update and the repository is ready for use. You can install it with:
sudo apt-get update && sudo apt-get install filebeat

# To start
sudo systemctl start filebeat
```

__Install using YUM :__
```shell
# Download and install the public signing key:
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

# Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example logstash.repo
echo "[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md" | sudo tee /etc/yum.repos.d/logstash.repo

# And your repository is ready for use. You can install it with:
sudo yum install filebeat

# To start
sudo systemctl start filebeat
```

__Install using Binary :__
1. Open this url - https://www.elastic.co/guide/en/beats/filebeat/7.17/filebeat-installation-configuration.html#installation.
2. Select your platform and download TARG.GZ, DEB, ZIP, or RPM package.
3. Extract the package to a local folder.
4. Go to extracted folder and run logstash `./filebeat`

</p>
</details>

<details open>
<summary><b><font size="4">macOS</font></b></summary>
<p>

```shell
# To install with Homebrew, you first need to tap the Elastic Homebrew repository:
brew tap elastic/tap

# After you’ve tapped the Elastic Homebrew repo, you can use brew install to install the default distribution of Logstash:
brew install elastic/tap/filebeat-full

# Starting Logstash with Homebrew
brew services start elastic/tap/filebeat-full
```

</p>
</details>

<details open>
<summary><b><font size="4">Windows</font></b></summary>
<p>

1. Download the Filebeat Windows zip file from [here](https://www.elastic.co/downloads/beats/filebeat).
2. Extract the contents of the zip file into C:\Program Files.
3. Rename the filebeat-<version>-windows directory to Filebeat.
4. Open a PowerShell prompt as an Administrator (right-click the PowerShell icon and select Run As Administrator).
5. From the PowerShell prompt, run the following commands to install Filebeat as a Windows service:
    ```shell
    PS > cd 'C:\Program Files\Filebeat'
    PS C:\Program Files\Filebeat> .\install-service-filebeat.ps1
    ```

</p>
</details>

<details open>
<summary><b><font size="4">Docker</font></b></summary>
<p>

1. Make sure you already installed docker compose, if not please follow this url [install-docker-compose](https://docs.docker.com/compose/install)
2. Add this configuration into current docker compose file :
    ```yaml
      filebeat:
        image: elastic/filebeat:7.17.23
        container_name: filebeat
        command: ["filebeat", "-e", "--strict.perms=false"]
        environment:
          - LOGSTASH_HOST=logstash:5044
        volumes:
          - ./beats-data/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
          - ./sample-files:/tmp/log-files/
        networks:
          - logstash    
    ```
3. Save it and execute this command :
    ```shell
    docker-compose up -d
    ```
> __NOTE:__  Dont forget to update the `environment` values.

</p>
</details>

---

### Install Hearbeat

<details open>
<summary><b><font size="4">Linux</font></b></summary>
<p>

__Install using APT :__
```shell
# Download and install the Public Signing Key:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# You may need to install the apt-transport-https package on Debian before proceeding:
sudo apt-get install apt-transport-https

# Save the repository definition to /etc/apt/sources.list.d/elastic-7.x.list:
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

# Run sudo apt-get update and the repository is ready for use. You can install it with:
sudo apt-get update && sudo apt-get install heartbeat-elastic

# To start
sudo systemctl start heartbeat-elastic
```

__Install using YUM :__
```shell
# Download and install the public signing key:
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

# Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example logstash.repo
echo "[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md" | sudo tee /etc/yum.repos.d/logstash.repo

# And your repository is ready for use. You can install it with:
sudo yum install heartbeat-elastic

# To start
sudo systemctl start heartbeat-elastic
```

</p>
</details>

<details open>
<summary><b><font size="4">Docker</font></b></summary>
<p>

1. Make sure you already installed docker compose, if not please follow this url [install-docker-compose](https://docs.docker.com/compose/install)
2. Add this configuration into current docker compose file :
    ```yaml
      heartbeat:
        image: elastic/heartbeat:7.17.23
        container_name: heartbeat
        environment:
          - LOGSTASH_HOST=logstash:5044
          - ELASTICSEARCH_HOST=http://192.168.31.102:9200
        volumes:
          - ./beats-data/heartbeat/heartbeat.yml:/usr/share/heartbeat/heartbeat.yml:ro
        networks:
          - logstash  
    ```
3. Save it and execute this command :
    ```shell
    docker-compose up -d
    ```
> __NOTE:__  Dont forget to update the `environment` values.

</p>
</details>

---

### Install APM Server
Elastic APM is an application performance monitoring system built on the Elastic Stack. It allows you to monitor software services and applications in real time, by collecting detailed performance information on response time for incoming requests, database queries, calls to caches, external HTTP requests, and more.

![elasticsearch-apm-server](https://www.elastic.co/guide/en/apm/server/7.11/images/apm-architecture-diy.png)

To download and install APM Server, use the commands below that work with your system

<details open>
<summary><b><font size="4">Linux</font></b></summary>
<p>

__Install using APT :__
```shell
# Download and install the Public Signing Key:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# You may need to install the apt-transport-https package on Debian before proceeding:
sudo apt-get install apt-transport-https

# Save the repository definition to /etc/apt/sources.list.d/elastic-7.x.list:
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

# Run sudo apt-get update and the repository is ready for use. You can install it with:
sudo apt-get update && sudo apt-get install apm-server

# To start
sudo systemctl start apm-server
```

__Install using YUM :__
```shell
# Download and install the public signing key:
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

# Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example logstash.repo
echo "[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md" | sudo tee /etc/yum.repos.d/logstash.repo

# And your repository is ready for use. You can install it with:
sudo yum install apm-server

# To start
sudo systemctl start apm-server
```

</p>
</details>

<details open>
<summary><b><font size="4">macOS</font></b></summary>
<p>

```shell
# To install with Homebrew, you first need to tap the Elastic Homebrew repository:
brew tap elastic/tap

# After you’ve tapped the Elastic Homebrew repo, you can use brew install to install the default distribution of Logstash:
brew install elastic/tap/filebeat-full

# Starting Logstash with Homebrew
brew services start elastic/tap/apm-server-full
```

</p>
</details>

<details open>
<summary><b><font size="4">Windows</font></b></summary>
<p>

1. Download the Filebeat Windows zip file from [here](https://www.elastic.co/downloads/apm/apm-server).
2. Extract the contents of the zip file into C:\Program Files.
3. Rename the `apm-server-<version>-windows` directory to APM-Server.
4. Open a PowerShell prompt as an Administrator (right-click the PowerShell icon and select Run As Administrator).
5. From the PowerShell prompt, run the following commands to install Filebeat as a Windows service:
    ```shell
    PS > cd 'C:\Program Files\APM-Server'
    PS C:\Program Files\APM-Server> .\install-service-apm-server.ps1
    ```

</p>
</details>

<details open>
<summary><b><font size="4">Docker</font></b></summary>
<p>

1. Make sure you already installed docker compose, if not please follow this url [install-docker-compose](https://docs.docker.com/compose/install)
2. Add this configuration into current docker compose file :
    ```yaml
      apm-server:
        image: elastic/apm-server:7.17.23
        container_name: apm-server
        environment:
          - ELASTICSEARCH_HOST=http://192.168.31.102:9200
        volumes:
          - ./apm-server/apm-server.yml:/usr/share/apm-server/apm-server.yml
        ports:
          - "8200:8200"
          - "6060:6060"
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - DAC_OVERRIDE
          - SETGID
          - SETUID
        networks:
          - logstash
    ```
3. Save it and execute this command :
    ```shell
    docker-compose up -d
    ```

> __NOTE:__  Dont forget to update the `environment` values.

</p>
</details>

---

## Setup Beats
### Setup Filebeat

Open the filebeat configuration file:
```shell
sudo vi filebeat.yml
```
Filebeat supports numerous outputs, but you’ll usually only send events directly to elasticsearch or to logstash for additional processing. In this tutorial, we'll use logstash to perform additional processing on the data collected by filebeat.

```yaml
output.logstash:
  hosts: "${LOGSTASH_HOST}"
```

#### Setup Filebeat for tailing log
Filebeat can be use to tailing log, you can add list of the file you wanna tail to filebeat configuration file.

```shell
filebeat.inputs:
  - type: filestream
    enabled: true
    id: nginx
    tags: ["nginx"]
    paths:
    - /tmp/log-files/nginx*.log
```

In logstash, you can use this configuration to accept connection from filebeat. For example, save it into __./config/pipeline/__ with name `beats.conf` :
```shell
input {
    beats {
        port => 5044
    }
}

filter {
    grok {
        match => {
            'message' => '%{IPORHOST:remote_ip} - %{DATA:user_name} \[%{HTTPDATE:access_time}\] \"%{WORD:http_method} %{DATA:url} HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{NUMBER:body_sent_bytes} \"%{DATA:referrer}\" \"%{DATA:agent}\"'
        }
    }
}

output {
    elasticsearch {
        hosts => ["${ELASTICSEARCH_HOST}"]
        index => "filebeat-nginx-%{+YYYY.MM.dd}"
    }
}
```

Dont forget to add new pipeline configuration in `pipelines.yaml`
```yaml
- pipeline.id: beats
  path.config: "/usr/share/logstash/pipeline/beats.conf"
```

### Setup Heartbeat
Heartbeat is a lightweight shipper that can periodically check the status of your services and determine whether they are available. Heartbeat currently supports monitors for checking hosts via :

- ICMP
    Use the icmp monitor when you simply want to check whether a service is available
- TCP
    Use the tcp monitor to connect via TCP. You can optionally configure this monitor to verify the endpoint by sending and/or receiving a custom payload
- HTTP
    Use the http monitor to connect via HTTP. You can optionally configure this monitor to verify that the service returns the expected response, such as a specific status code, response header, or content

Heartbeat also support numerous outputs, for this tutorial we'll send event to elasticsearch and logstash, But first, we'll send an event into logstash.



#### Output to Logstash

For this example, configuration file is located at __beats-data/hearbeat/heartbeat.yaml__. Open the Heartbeat configuration file, and add this output configuration :

```yaml
output.logstash:
  hosts: "${LOGSTASH_HOST}"
```

For example, we want to monitor `github.com` and `google.com` using HTTP. We can set the requirement like request header, payload, method, and etc. Or we can set the expected response from the service we monitor.

```yaml
heartbeat.monitors:
- type: http
  schedule: '@every 10s'
  urls:
    - https://github.com
    - https://google.com
  check.request:
    method: GET
    headers:
        'Content-Type': 'application/json'
  check.response:
    status: 200
```

Save it and restart the heartbeat service.

#### Output to Elasticsearch
Change configuration from previous `output.logstash` into `output.elasticsearch` or we can disable it by adding a `#` sign in front of the config we want to disable.
```yaml
#ouput.logstash:
#  host: "${LOGSTASH_HOST}"

output.elasticsearch:
  hosts: ["${ELASTICSEARCH_HOST}"]
```

Same as before we will try to monitor `github.com` and `google.com`, but now we'll use icmp to get the status.
```yaml
heartbeat.monitors:
...
- type: icmp
  hosts: ["github.com", "google.com"]
  schedule: '*/5 * * * * * *'
```

If we directly set the output into elasticsearch, heartbeat will automatically create a new index.
![hearbeat-create-new-index](images/SCR-20240815-smpd.png)

### Setup APM Server
open the apm-server configuration file located at __./apm-server/apm-server.yml__ and set output to elasticsearch :
```yaml
apm-server:
  host: "0.0.0.0:8200"
output.elasticsearch:
  hosts: ["${ELASTICSEARCH_HOST}"]
```
Save it and restart the apm server.

#### Create sample app for apm
Currently to get the data from application, elastic provide apm agent to be integrated into the application including :
- APM Android Agent
- APM Go Agent
- APM iOS Agent
- APM Java Agent
- APM .NET Agent
- APM Node.js Agent
- APM PHP Agent
- APM Python Agent
- APM Ruby Agent

In this example, we'll use python agent to demonstrate how apm server works.
Example :
```python
import sqlite3
import redis
import os
from dotenv import load_dotenv
from flask import Flask, request
from elasticapm.contrib.flask import ElasticAPM


load_dotenv()
APM_HOST = os.getenv("APM_HOST", "http://localhost:8200")
REDIS_HOST = os.getenv("REDIS_HOST", "localhost")
REDIS_PORT = os.getenv("REDIS_PORT", 6379)
REDIS_DB = os.getenv("REDIS_DB", 0)
ENVIRONMENT = os.getenv("ENVIRONMENT", "development")


print(APM_HOST, REDIS_HOST, REDIS_PORT, REDIS_DB, ENVIRONMENT)


app = Flask(__name__)

app.config['ELASTIC_APM'] = {
    'SERVICE_NAME': 'PYTHON APM',
    'SERVER_URL': APM_HOST,
    'ENVIRONMENT': ENVIRONMENT,
    'DEBUG': 'DEBUG'
}

apm = ElasticAPM(app)

#app = Flask(__name__) 

redis_conn = redis.Redis(host = REDIS_HOST, port = REDIS_PORT, db = REDIS_DB)

@app.route("/log", methods=["POST"])
def log():
    message = request.form["message"]

    sqlite_conn = sqlite3.connect("logs.db")
    cursor = sqlite_conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS logs (message text)")
    cursor.execute("INSERT INTO logs (message) values (?)", (message,))
    sqlite_conn.commit()
    sqlite_conn.close()

    redis_conn.rpush("logs", message)

    return "Logged: {}".format(message)

if __name__ == "__main__":
    app.run(debug=True)
```

To run application and send logs to databases, run this command :
```shell
curl -X POST -d "message=test_message" http://localhost:5000/log
```
