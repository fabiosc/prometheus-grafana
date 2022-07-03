# Prometheus e Grafana no Docker

Projeto baseado nos artigos e vídeos disponíveis na internet
- [Configurando o Prometheus para aplicações java com SpringBoot](https://grafana.com/blog/2022/05/04/how-to-capture-spring-boot-metrics-with-the-opentelemetry-java-instrumentation-agent/)
- [Dashboard de métricas com Spring Boot Actuator, Prometheus e Grafana](https://www.youtube.com/watch?v=K_EI1SxVQ5Q)
- [Observabilidade para aplicações baseadas em Java Spring utilizando OpenTelemetry](https://reachmnadeem.wordpress.com/2021/03/04/observability-for-java-spring-based-applications-using-opentelemetry/)
- [Autenticação básica para Prometheus](https://computingforgeeks.com/secure-prometheus-server-with-basic-password-authentication/)
- [Definir senha para usuário Admin no Grafana](https://grafana.com/docs/grafana/next/setup-grafana/configure-grafana/#security)

## Executando

1. [Antes de executar](#antes-de-executar)
    * [Arquivo prometheus.yml](#arquivo-prometheusyml)
    * [Arquivo web.yml](#arquivo-webyml)
    * [Arquivo grafana.ini](#arquivo-grafanaini)
    * [Preparando Aplicação Java](#preparando-aplicacao-java)
1. [Execução](#execução)
    * [Acessando a UI do Prometheus](#acessando-a-ui-do-prometheus)
    * [Acessando a UI do Grafana](#acessando-a-ui-do-grafana)

## Antes de executar
O arquivo atentar-se aos arquivos de configuração:
- grafana.ini, com as configurações do Grafana
- prometheus.yml, com as configurações do Prometheus
- web.yml, com as configurações de acesso ao Prometheus (ui e/ou apis)

### ARQUIVO prometheus.yml
* No arquivo prometheus.yml estão as configurações do Prometheus, assim como as configurações dos jobs para buscar as métricas de cada aplicação que será monitorada
* Nesse arquivo atentar-se a configuração do job da aplicação fictícia ```api-produtos```, pois nela está presente o exemplo de como configurar o Prometheus para ficar escutando as metricas da aplicação ao rodar na mesma máquina onde o container do Prometheus está sendo executado.
```
  - job_name: 'produtos'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
        labels:
          application: 'api-produtos'
```

### ARQUIVO web.yml
* Esse arquivo nesse primeiro momento só possui a configuração para não permitir o acesso sem login a UI do Prometheus e ao consumo das APIs de métricas do Prometheus

### ARQUIVO grafana.ini
* Esse arquivo contém as configurações iniciais do Grafana, para esse nosso exemplo, foi  alterada a senha do usuário admin, para iniciar com uma senha já definida no arquivo gafana.ini. 
No arquivo grafana.ini encontrar a chave ```[security]``` e no campo ```admin_password``` está setada a senha do usuário admin para acessar o grafana

### PREPARANDO APLICACAO JAVA
* Para exemplo utilizamos uma aplicação Java com Spring Boot e na aplicação é necessário adicionar como dependência o Spring Actuator e o Micrometer para gerar as métricas para o Prometheus. E também configurar o application.yml para expor os endpoints de monitoramento.
```
        <!-- Exemplo dependencia Micrometer utilizado no pom.xml-->
		<dependency>
			<groupId>io.micrometer</groupId>
			<artifactId>micrometer-registry-prometheus</artifactId>
			<scope>runtime</scope>
		</dependency>

        <!-- Exemplo dependencia Spring Actuator utilizado no pom.xml-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
```
```
#Exemplo para expor os endpoints de health check e das metricas do prometeus no arquivo application.yml
management:
  endpoints:
    web:
      exposure:
        include: 'health,prometheus'
        
  endpoint:
    health:
      enabled: true
      show-details: always
      
    prometheus:
      enabled: true
```

## Execução
No diretório (pasta) onde o arquivo docker-compose.yml encontra-se executar o comando: 
### docker-compose up -d

### Acessando a UI do Prometheus
Para acessar o Prometheus, basta acessar no browser a URL http://localhost:9090, o usuário de acesso, é "admin" e a senha a que foi definida no arquivo web.yml é "changeme" que foi criptografa utilizando BCrypt-Generator.com (https://bcrypt-generator.com/)

### Acessando a UI do Grafana
Para acessar o Grafana, basta acessar no browser a URL http://localhost:3000, o usuário de acesso, é "admin" e a senha a que foi definida no arquivo grafana.ini é "changeme" e está presente no bloco ```[security]``` do arquivo grafana.ini, no campo ```admin_password```