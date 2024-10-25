# Comparação entre Spring e Quarkus

Este repositório contém oito aplicações que permitem uma comparação entre os modelos **imperativo** e **reativo** para **Spring** e **Quarkus** em dois ambientes: **JVM** e **Nativo**. O objetivo é avaliar o desempenho e o uso de recursos entre as duas tecnologias e modelos em cenários variados. Para isso, um ambiente de testes foi configurado utilizando **Kubernetes**, com suporte para **Minikube** (opcional), onde as aplicações serão executadas e as métricas serão coletadas.

| Tecnologia | Modelo    | Ambiente |
|------------|-----------|----------|
| Spring     | Imperativo| JVM (RedHat) |
| Spring     | Imperativo| Nativo (GraalVM) |
| Spring     | Reativo   | JVM (RedHat) |
| Spring     | Reativo   | Nativo (GraalVM) |
| Quarkus    | Imperativo| JVM (RedHat) |
| Quarkus    | Imperativo| Nativo (Mandrel) |
| Quarkus    | Reativo   | JVM (RedHat) |
| Quarkus    | Reativo   | Nativo (Mandrel) |

## Pré-requisitos

Antes de iniciar os testes, certifique-se de que os seguintes pré-requisitos estejam atendidos:

- **Docker** instalado para construir as imagens.
- **kubectl** instalado e configurado para gerenciamento do cluster Kubernetes.
- **JMeter** instalado e configurado para testes de stress na aplicação. O JMeter será utilizado para medir o pico de uso de memória, CPU e TPS (transações por segundo) máximos da aplicação.
- **Minikube** (opcional): caso opte por utilizar Minikube, será necessário carregar a imagem do PostgreSQL conforme instruído abaixo.

> **Nota:** Minikube não é um requisito obrigatório. Caso esteja utilizando Minikube, lembre-se de carregar previamente a imagem do PostgreSQL.

## Configurando o Ambiente de Testes

### Passo 1 (Opcional): Carregar a Imagem do PostgreSQL no Minikube

Para garantir que o PostgreSQL esteja disponível para as aplicações em um ambiente Minikube, carregue a imagem necessária com o seguinte comando:

```bash
minikube image load postgres:15.4
```

### Passo 2: Construir a Imagem da Aplicação

Construa a imagem Docker da aplicação que será testada. Neste exemplo, estamos utilizando o Spring rodando em JVM. Use o comando abaixo para construir a imagem:


```bash
docker build -f docker/Dockerfile.jvm -t spring.project.imperative:redhat-21 .
```

### Passo 3: Aplicar os Manifestos Kubernetes

Para subir a aplicação e o banco de dados no cluster Kubernetes, aplique os manifestos com o comando:

```bash
kubectl apply -f spring-project-imperative-redhat-ns.yaml \
-f init-sql-configmap.yaml \
-f postgres-configmap.yaml \
-f postgres-deployment.yaml \
-f postgres-service.yaml \
-f spring-project-imperative-redhat-deployment.yaml \
-f spring-project-imperative-redhat-srv.yaml
```

### Passo 4: Verificar se a Aplicação Está Rodando

```bash
kubectl get pods
```

A saída esperada deve mostrar o status dos pods como Running 1/1, indicando que estão prontos para o próximo passo.

```bash
NAME                                                    READY   STATUS    RESTARTS   AGE
postgres-deployment-XXXXXXXX-XXXXX                      1/1     Running   0          2m
spring-project-imperative-redhat-deployment-XXXXXXXXX   1/1     Running   0          2m
```

## Coleta de Métricas

As métricas a serem coletadas para comparação entre Spring e Quarkus incluem:

**Artifact Size**: Tamanho final do artefato (JAR) gerado após a construção.
**Build Time**: Tempo necessário para construir a aplicação.
**Startup Time**: Tempo que a aplicação leva para iniciar e ficar disponível.
**Memory After Startup**: Consumo de memória logo após a inicialização.
**Get (MEM, CPU, TPS)**: Uso de memória, CPU e TPS (transações por segundo) para operações de leitura.
**Post (MEM, CPU, TPS)**: Uso de memória, CPU e TPS para operações de escrita.

## Configuração de Teste de Stress com JMeter

Para medir as métricas de uso de recursos sob carga, utilize o JMeter com a seguinte configuração:

**Threads**: 100
**Ramp-Up**: 5 segundos
**Duração**: 300 segundos (5 minutos)

Execute o teste de stress com essas configurações para identificar o pico de uso de memória, CPU e TPS da aplicação. Certifique-se de que a aplicação esteja em estado estável antes de iniciar a execução dos testes.

## Passos para Coleta de Métricas no Container da Aplicação

Primeiramente, execute o comando abaixo para entrar no terminal do container da aplicação:

```bash
docker exec -it nomeDoContainer /bin/bash
```

Uma vez dentro do container, siga os passos a seguir para coletar as métricas:

1. **Artifact Size**

```bash
ls -lah
```
2. **Build Time**

Execute o comando de build da aplicação para medir o tempo de construção:

```bash
mvn install
```
3. **Startup Time**

Para verificar o tempo de startup da aplicação, consulte o log da aplicação:

```bash
kubectl logs nomeDoPod -n namespace
```
4. **Memory After Startup**

Para verificar o uso de memória logo após a inicialização:

```bash
ps -eo size,pid,user,command --sort -size | awk '{ hr=$1/1024 ; printf("%13.2f Mb ",hr) } { for ( x=4 ; x<=NF ; x++ ) { printf("%s ",$x) } print "" }'
```
5. **GET e POST**

Durante o teste de carga com JMeter, capture o consumo de CPU e MEM a cada segundo, por 300 segundos:

```bash
top -b -d 1 -n 300 > top-5minutes.txt
```

* Para encontrar o maior pico de CPU:

  ```bash
  grep -E "^[[:space:]]*[0-9]+" top-5minutes.txt | awk '{print $1, $9}' | sort -k2 -nr | head -n 1
  ```

* Para encontrar o maior pico de MEM:

  ```bash
  grep -E "^[[:space:]]*[0-9]+" top-5minutes.txt | awk '{print $1, $10}' | sort -k2 -nr | head -n 1
  ```
