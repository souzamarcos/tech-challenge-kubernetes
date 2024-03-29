# Tech Challenge Kubernetes
Repositório com a configuração do kubernetes

# Repositórios relacionados
* [Serviço de produto](https://github.com/souzamarcos/tech-challenge-ms-product)
* [Serviço de pedidos](https://github.com/souzamarcos/tech-challenge-ms-order)
* [Serviço de pagamento](https://github.com/souzamarcos/tech-challenge-ms-payment)
* [Serviço de cliente](https://github.com/souzamarcos/tech-challenge-ms-customer)
* [Serviço de notificação](https://github.com/souzamarcos/tech-challenge-ms-notification)
* [Infraestrutura Terraform](https://github.com/souzamarcos/tech-challenge-terraform)
* [Configuração do Kubernetes](https://github.com/souzamarcos/tech-challenge-kubernetes)
* [Lambda de Autenticação](https://github.com/souzamarcos/tech-challenge-authentication-lambda)

# Deploy
Processo de deploy do kubernetes nos ambientes de [Produção](#produção) ou [Local](#local).


## Produção
Para atualizar o ambiente de prod basta atualizar os arquivos na pasta [prod](/prod/) e mergear a PR na branch `main`.


### Configuração Inicial
É necessário as seguintes etapas para configurar o cluster Kubernetes para o funcionamento da aplicação:

1 - Criação das secrets com as informações de conexão com as basse de dados RDS de cada microserviço. Obs: credenciais omitidas por questão de segurança 

```bash
kubectl create secret generic db-product-secret --from-literal=url=jdbc:mysql://<HOST>:3306/dbProduct --from-literal=username=user --from-literal=password=password
kubectl create secret generic db-order-secret --from-literal=url=jdbc:mysql://<HOST>:3306/dbOrder --from-literal=username=user --from-literal=password=password
kubectl create secret generic db-payment-secret --from-literal=url=jdbc:mysql://<HOST>:3306/dbPayment --from-literal=username=user --from-literal=password=password
```

### Configurando o kong

1 - Execute os comandos abaixo:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml
kubectl apply -f prod/kong/k8s-gateway-class.yaml
```

2 - Agora instale o kong no kubernetes via helm:
- 2.1 - Adicionando o charts do kong
```bash
helm repo add kong https://charts.konghq.com
helm repo update
```

- 2.2 - Instalando o kong
```bash
helm install kong kong/ingress -n kong --create-namespace 
```

## Local
Para configurar a aplicação no kubernetes local **execute os comandos abaixo na raiz do projeto**:

1 - Iniciar as base de dados mysql de cada serviço com o comando presente no README de cada um deles.

``` bash
docker-compose -f docker-compose-without-application.yml up --build
```

2 - Procurar o IP da máquina através do comando:

Windows
```bash
ipconfig
```
Linux | Mac
```bash
ifconfig
```

3 - Crie as as secrets e defina a URL da base de dados. **No comando abaixo substitua o texto `<HOST>` pelo ip da máquina consultado na etapa acima**. Caso decida usar uma base MySql em outro local, coloque o endereço da mesma.
```bash
kubectl create secret generic db-product-secret --from-literal=url=jdbc:mysql://<HOST>:3306/dbProduct --from-literal=username=user --from-literal=password=password
kubectl create secret generic db-order-secret --from-literal=url=jdbc:mysql://<HOST>:3306/dbOrder --from-literal=username=user --from-literal=password=password
kubectl create secret generic db-payment-secret --from-literal=url=jdbc:mysql://<HOST>:3306/dbPayment --from-literal=username=user --from-literal=password=password
```

4 - Aplique os outros recursos do kubernetes
```bash
kubectl apply -f prod -R
```

A aplicação estará disponível no endereço [http://localhost/swagger](http://localhost/swagger).


> Obs: Caso queira remover todos os recursos criados execute os comandos abaixo:
>```bash
>kubectl apply -f prod -R
>kubectl delete secret db-product-secret
>kubectl delete secret db-order-secret
>kubectl delete secret db-payment-secret
>```
