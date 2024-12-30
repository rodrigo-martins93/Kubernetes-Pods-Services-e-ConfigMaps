# Documentação do Projeto Kubernetes

## Descrição
Este projeto define um conjunto de recursos Kubernetes para configurar uma aplicação composta por três componentes principais:
- **Portal de Notícias**: Interface pública acessada pelos usuários.
- **Sistema de Notícias**: Camada intermediária responsável pela lógica de negócios.
- **Banco de Dados**: Base de dados MySQL para armazenar as informações.

## Recursos Criados
Os seguintes recursos estão definidos no arquivo YAML:

### 1. ConfigMaps
Os ConfigMaps armazenam dados de configuração que são usados pelos pods.

#### db-configmap
Configuração do banco de dados MySQL:
```yaml
MYSQL_ROOT_PASSWORD: q123456
MYSQL_DATABASE: empresa
MYSQL_PASSWORD: "123456"
```

### portal-configmap
Configuração do Portal de Notícias:
```yaml
IP_SISTEMA: http://<IP_DO_NODE>:30001
```

### sistema-configmap
Configuração do Sistema de Notícias:
```yaml
HOST_DB: svc-db-noticias:3306
USER_DB: root
PASS_DB: q123456
DATABASE_DB: empresa
```
---

### 2.Pods

Os <mark>Pods</mark> representam os containers da aplicação.

**portal-noticias**
Pod para o Portal de Notícias:

- **Imagem:** <mark>aluracursos/portal-noticias:1</mark>
- **Porta:** <mark>80</mark>
- **Configuração:** <mark>Referência ao portal-configmap</mark>

**db-noticias**
Pod para o Banco de Dados:

- **Imagem:** <mark>aluracursos/mysql-db:1</mark>
- **Porta:** <mark>3306</mark>
- **Configuração:** <mark>Referência ao db-configmap</mark>

**sistema-noticias**
Pod para o Sistema de Notícias:

- **Imagem:** <mark>aluracursos/sistema-noticias:1</mark>
- **Porta:** <mark>80</mark>
- **Configuração:** <mark>Referência ao sistema-configmap</mark>

---

### 3. Services<mark></mark>

Os <mark>Services</mark> expõem os Pods e gerenciam o acesso entre eles.

**svc-db-noticias**
Serviço para o Banco de Dados:

- **Tipo:** <mark>ClusterIP</mark>
- **Porta:** <mark>3306</mark>
- **Selector:** <mark>app: db-noticias</mark>
 
**svc-portal-noticias**
Serviço para o Portal de Notícias:

- **Tipo:** <mark>NodePort</mark>
- **Porta:** <mark>80</mark>
- **NodePort:** <mark>30000</mark>
- **Selector:** <mark>app: portal-noticias</mark>

**svc-sistema-noticias**
Serviço para o Sistema de Notícias:

- **Tipo:** <mark>NodePort</mark>
- **Porta:** <mark>80</mark>
- **NodePort:** <mark>30001</mark>
- **Selector:** <mark>app: sistema-noticias</mark>

---

### Fluxo de Comunicação

1. O **Portal de Notícias** (expõe NodePort na porta <mark>30000</mark>) acessa o Sistema de Notícias pela URL configurada no <mark>ConfigMap</mark> (<mark>http://<IP_DO_NODE>:30001</mark>).

2. O **Sistema de Notícias** utiliza as credenciais e o host do banco de dados configurados no <mark>sistema-configmap</mark> para se conectar ao serviço do banco (<mark>svc-db-noticias</mark>).

3. O **Banco de Dados** (exposto pelo <mark>svc-db-noticias</mark>) é acessado pelo **Sistema de Notícias** para armazenar e recuperar dados.

---

## Passo a Passo para Implementação

1. Configuração do Cluster Kubernetes

    Certifique-se de que você possui um cluster Kubernetes em execução e que o <mark>kubectl</mark> está configurado corretamente para interagir com ele.

2. Execute o seguintes comandos individualmente para aplicar os recursos no cluster:
```
kubectl apply -f db-configmap.yaml
kubectl apply -f portal-configmap.yaml
kubectl apply -f sistema-configmap.yaml
kubectl apply -f portal-noticias-pod.yaml
kubectl apply -f db-noticias-pod.yaml
kubectl apply -f sistema-noticias-pod.yaml
kubectl apply -f svc-db-noticias.yaml
kubectl apply -f svc-portal-noticias.yaml
kubectl apply -f svc-sistema-noticias.yaml
```

## 3. **Verificar os Recursos Criados**

Após aplicar o arquivo YAML, você pode verificar se os recursos foram criados corretamente:

- **ConfigMaps:**

```
kubectl get configmaps
```

- **Pods:**

```
kubectl get pods
```

- **Services:**

```
kubectl get services
```

## 4. **Testar o Acesso**

- Localize o IP do nó onde o cluster está em execução. Você pode fazer isso com o seguinte comando:
```
kubectl get nodes -o wide
```

- Substitua <IP_DO_NODE> pelos valores encontrados para acessar os serviços:
    - **Portal de Notícias:** <mark>http://<IP_DO_NODE>:30000</mark>
    - **Sistema de Notícias:** <mark>http://<IP_DO_NODE>:30001</mark>




