## Logstash y Kibana:

Fichero logstash.conf. Por defecto se busca en /usr/share/logstash/pipeline. El pipeline indica el procesado de datos en tres partes:
* input
* filter
* output
#### Input:
https://www.elastic.co/guide/en/logstash/current/input-plugins.html
Plug-ins para recogida de datos: 
* tcp
* udp
* http
* file
* jdbc
* beats
* se añade un campo codec para indicar el formato en el que se reciben.
```
input {
  http {
    port => '9563’
    codec => "json"
  }
}
input {
    file {
      path => “/home/unai/access_log”
      start_position => “beginning”
    }
}

input {
  jdbc {
    jdbc_connection_string => “jdbc:mysql//localhost:3306/nombre-tabla”
    jdbc_user => “usuario-mysql”
    jdbc_password => “contraseña-mysql”
    jdbc_driver_library => “ruta-conector-java-mysql.jar”
    jdbc_driver_class => “com.mysql.jdbc.Driver”
    schedule => "* * * * *"
    statement => “SELECT * FROM tabla”
  }
}
```
#### Filter:
https://www.elastic.co/guide/en/logstash/current/filter-plugins.html
* json
* csv
* grok
```
filter {
  json {
    source => "message”
    target => "doc"
  }
}

filter {
  csv {
    separator => ","
    skip_header => "true”
    columns => ["Nombre", "Autor", "Isbn"]
  }
}

filter {
  if [version] == "3.11" {
    drop {}
  }
  mutate {
    remove_field => ["version", "nombre"]
  }
}
```
#### Output:
https://www.elastic.co/guide/en/logstash/current/output-plugins.html
```
output {
  elasticsearch {
    hosts => [ “instancia-es:9200” ]
    index => “mi-índice”
  }
  stdout {
    codec => rubydebug
  }
}
```
#### Configuración:
Se pueden procesar múltiples entradas de manera selectiva:
```
input {
  tcp {
    port => 5045
    type => 'datos-tcp’ }
  udp {
    port => 5045
    type => 'datos-udp’ }
}
filter {
  if [type] == 'datos-tcp’ {
    grok {
    match => ["message" , " … "] }
  }
  else if [type] == 'datos-udp’ {
    mutate {
```

#### Ejercicio 1:
Configurar pipeline para recibir http en puerto 9900 json. índice logs-ej1. 
```
input {
    http {
        port => 9900
        codec => "json"
    }
}

output {
    elasticsearch {
        hosts => ["elasticsearch:9200"]
        index => "logs-ej1"
    }
}

curl -X POST -H "Content-Type: application/json" -d '{"timestamp": "Nov 28 07:55:04","device" : "server","process" : "kernel","event-number" : "1150.308049","message" : "port 2(veth78fa91b) entered blocking state"}' http://localhost:9900

curl -X POST -H "Content-Type: application/json" -d '{"timestamp": "Nov 28 07:55:05","device" : "server","process" : "systemd-networkd","event-number" : "730","message" : "Gained IPv6LL"}' http://localhost:9900

curl -X GET "localhost:9200/logs-ej1/_search?pretty"
```
#### Grok:
https://grokdebugger.com/
```
input {
…
}
filter {
  grok {
    match => { "message" => "%{SYSLOGTIMESTAMP:tiempo} %{LOGLEVEL:nivel} %{GREEDYDATA:mensaje}" }
  }
}
```

#### Ejercicio 2:
Parsear y almacenar en un índice de elasticsearch líneas de log. Recibir http puerto 9901. Índice: logs-apache.
```
%{IP:cliente} - - %{DATE:fecha} - %{INT:estado} %{WORD:metodo} %{URI:direccion}

input{
    http{
        port => 9901
    }
}
filter{
    grok{
        match => { "message" => "%{IP:cliente} - - %{DATE:fecha} - %{INT:estado} %{WORD:metodo} %{URI:direccion}"}
    }
}
output{
    elasticsearch{
        hosts => ["http://elasticsearch:9200"]
        index => "logs-apache"
    }
}

curl -X POST "localhost:9901" -d "124.173.67.77 - - 23/07/2016 - 0400 GET http://www.059boss.com/index.php"
curl -X POST "localhost:9901" -d "195.182.131.107 - - 23/07/2016 - 0400 GET http://asconprofi.ru/common/proxy.php"
curl -X POST "localhost:9901" -d "155.94.224.168 - - 23/07/2016 - 0400 GET http://www.daqimeng.com/user/login"
curl -X POST "localhost:9901" -d "119.29.32.85 - - 23/07/2016 - 0400 GET http://www.tianx.top"

curl -X GET "localhost:9200/logs-apache/_search?pretty"
```

