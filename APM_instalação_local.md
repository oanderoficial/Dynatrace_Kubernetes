# Instalação Dynatrace (APM - Kubernetes)
## Ambiente Managed (on premise)

## 1) Valide se está liberado a porta 443 do lado dos nodes Kubernetes, e a porta 9999 no ActiveGate  

<img width="439" height="103" alt="image" src="https://github.com/user-attachments/assets/9d589a30-69e0-4cc0-84db-d8f59ccd7340" />

## 2) Faça a instalação do OneAgent direto nos nós 

<p>Para isso vamos gerar o instalador na console UI</p>

<img width="460" height="278" alt="Captura de tela 2025-07-29 191610" src="https://github.com/user-attachments/assets/d89efdb3-b219-4fd2-8f23-1963dd1f557e" />
<br><br>
<strong> ⚠️ Aqui será necessário que você gere um token com as devidas permissões habilitadas </strong>
<br><br>
<img width="478" height="406" alt="Captura de tela 2025-07-29 193001" src="https://github.com/user-attachments/assets/6bdccbf2-0e8d-42c3-8bda-feb47b7af53d" />
<br><br>

<strong> Em "Access Tokens" clique em "Generate access token"</strong>
<br><br>
<img width="1094" height="458" alt="image" src="https://github.com/user-attachments/assets/67c79d57-d235-4d3e-aa69-476c7bb20a14" />
<br><br>
<p> Você vai criar seu token, não coloque data de expiração para esse token expecífico. Marque todas as permissões necessárias para o OneAgent. </p>
<br>
<strong> Depois de validar se está tudo certo, clique em "Generate Token".</strong>
<br><br>
<img width="284" height="74" alt="Captura de tela 2025-07-29 194110" src="https://github.com/user-attachments/assets/f1933fa5-2dd7-43f7-a7d9-c35a57ad2d61" />
<br><br>
<strong> Copie o token gerado e cole em um local de segurança, vamos precisar dele para a próxima parte.</strong>
<br><br> 
<img width="920" height="276" alt="Captura de tela 2025-07-29 194503" src="https://github.com/user-attachments/assets/3d1189fe-26a1-435d-83ea-aaa1ac9a2dad" />
<br><br>
<strong> Volte a aba "Download Dynatrace OneAgent for Linux", vamos inserir o token que acabamos de criar.</strong>
<br><br> 
<img width="497" height="407" alt="Captura de tela 2025-07-29 195459" src="https://github.com/user-attachments/assets/4e089d67-29a4-47ab-9ada-ef3218cce789" />
<br><br> 
<strong> Após inserir, scrole com o mouse para baixo, você vai ver três passos que precisam ser feito do lado do Sistema operacional lá nos nodes. </strong>
<br><br> 
<img width="432" height="328" alt="image" src="https://github.com/user-attachments/assets/25199c1b-2e50-4189-bfb8-146c2bb4acb6" />
<br><br> 

<strong> 2. Download do instalador:</strong>
* Baixa o instalador do Dynatrace OneAgent com wget, utilizando um token de API e parâmetros como arquitetura e zona de rede.

<strong> 3. Verificação da assinatura:</strong>
* Baixa o certificado raiz da Dynatrace e verifica a assinatura do instalador com openssl, garantindo sua autenticidade.

<strong> 4. Execução do instalador como root:</strong>
* Executa o script de instalação com /bin/sh, configurando o modo de monitoramento como fullstack, habilitando acesso a logs de aplicativos e definindo a zona de rede.

<strong> 5. Reinício de processos:</strong>
* Orienta a reiniciar os processos que devem ser monitorados para que o agente comece a coletar dados.
<br><br>

<strong> Depois desse processo realizado em todos os nodes você já deve começar a visualizar os hosts na console UI </strong>
<br><br>
<img width="1084" height="325" alt="Captura de tela 2025-07-29 200722" src="https://github.com/user-attachments/assets/f714ec45-826f-4d6e-82b3-c666c65756a0" />
<br><br>

## 3) Parte Kubernetes 

<strong> Primeiro vamos fazer a instalação do Dynatrace operator na ultima versão 1.6.1 direto do repositório oficial </strong>
<br><br> 

```bash
helm install dynatrace-operator oci://public.ecr.aws/dynatrace/dynatrace-operator \
  --set csidriver.enabled="false" \
  --create-namespace \
  --namespace dynatrace \
  --atomic
```

#### Explicação dos parâmetros

- `helm install dynatrace-operator oci://...`  
  Instala o Dynatrace Operator a partir de um repositório OCI (container registry).

- `--set csidriver.enabled="false"`  
  Desativa o driver CSI, usado para gerenciamento de volumes, caso não seja necessário.

- `--create-namespace`  
  Cria automaticamente o namespace `dynatrace` se ele ainda não existir no cluster.

- `--namespace dynatrace`  
  Define o namespace `dynatrace` como o local onde os recursos serão instalados.

- `--atomic`  
  Garante que a instalação seja atômica: se algum passo falhar, tudo será revertido automaticamente.

#### Criar o secret

<strong> Depois de instalado, vamos criar o secret </strong>

```bash
kubectl create secret generic productionk8s \
  --from-literal="apiToken=SEU_API_TOKEN" \
  --from-literal="paasToken=SEU_PAAS_TOKEN" \
  -n dynatrace
```

<p> Substitua <strong> SEU_API_TOKEN </strong> e <strong> SEU_PAAS_TOKEN </strong> pelos valores reais gerados na interface do Dynatrace. </p>


#### Verificar versões suportadas pelo CRD

```
kubectl get crd dynakubes.dynatrace.com -o=jsonpath='{.spec.versions[*].name}'
```
<br> 
<img width="295" height="51" alt="image" src="https://github.com/user-attachments/assets/7cc04a89-9704-4391-8e0e-893d805138ee" />


#### Configuração do Dynakube 

```yaml
apiVersion: dynatrace.com/v1beta1
kind: DynaKube
metadata:
  name: productionk8s
  namespace: dynatrace
  annotations:
    feature.dynatrace.com/metadata-enrichment: "true"
    feature.dynatrace.com/automatic-kubernetes-api-monitoring: "true"
spec:
  apiUrl: https://SEU-AMBIENTE-DYNATRACE.NET/e/1004594-4d8d-4671d-86f4-3189005dd8da35/api
  skipCertCheck: true
  tokens: productionk8s
  networkZone: productionk8s

  activeGate:
    capabilities:
      - routing
      - kubernetes-monitoring
      - dynatrace-api
    group: productionk8s
    resources:
      requests:
        cpu: 500m
        memory: 512Mi
      limits:
        cpu: 1000m
        memory: 1.5Gi

```

### DynaKube Manifest - Ambiente `productionk8s`

Este manifesto define os componentes necessários para integrar o cluster Kubernetes ao Dynatrace usando o operador oficial.

#### Estrutura do Manifesto

- **apiVersion**: `dynatrace.com/v1beta1`  
  Versão da API do CRD (Custom Resource Definition) do Dynatrace.

- **kind**: `DynaKube`  
  Tipo de recurso customizado que representa a configuração da integração Dynatrace no cluster.

- **metadata.name**: `productionk8s`  
  Nome da instância do DynaKube.

- **metadata.namespace**: `dynatrace`  
  Namespace onde os recursos Dynatrace serão instalados.

#### Annotations

- `feature.dynatrace.com/metadata-enrichment: "true"`  
  Habilita enriquecimento automático de metadados das entidades monitoradas.

- `feature.dynatrace.com/automatic-kubernetes-api-monitoring: "true"`  
  Ativa a monitoração automática da API do Kubernetes.

#### Spec

- **apiUrl**  
  URL do ambiente Dynatrace usado para se conectar à API.

- **skipCertCheck: true**  
  Ignora verificação de certificado SSL (útil para ambientes internos ou certificados customizados).

- **tokens: productionk8s**  
  Nome do `Secret` contendo os tokens `apiToken` e `paasToken`.

- **networkZone: productionk8s**  
  Zona de rede lógica usada para segmentação de comunicação dentro do Dynatrace.

#### activeGate

- **capabilities**  
  Lista de funcionalidades ativadas:
  - `routing`: roteamento de tráfego para o Dynatrace
  - `kubernetes-monitoring`: monitoramento da API do Kubernetes
  - `dynatrace-api`: acesso à API Dynatrace

- **resources**  
  Requisições e limites de CPU e memória para o ActiveGate.


> **Importante**: certifique-se de criar o `Secret` chamado `productionk8s` no namespace `dynatrace` contendo os tokens `apiToken` e `paasToken`.

#### Aplique a configuração 

```bash
kubectl apply -f dynakube.yaml
```
OU: 

```ps1
.\kubectl.exe --kubeconfig=.\kubeconfig-homolog.yaml apply -f C:\Users\ANDESI3\Downloads\kubectl\dynakube.yaml
```

#### Instrumentação (auto-injection)

<strong> Para facilitar essa parte, foi realizado a criação de um script para auxiliar a criação das labels </strong> 

```ps1
# Caminho do kubectl e do kubeconfig
$kubectl = ".\kubectl.exe"
$kubeconfig = "--kubeconfig=.\kubeconfig-homolog.yaml"

# Lista de namespaces do sistema a ignorar
$excludedNamespaces = @(
    "kube-system", "kube-public", "kube-node-lease",
    "cattle-system", "cattle-monitoring-system",
    "cattle-fleet-system", "cattle-impersonation-system",
    "calico-system", "tigera-operator",
    "longhorn-system", "cert-manager",
    "local-path-storage", "system-upgrade",
    "default"
)

# Obter todos os namespaces
$namespaces = & $kubectl $kubeconfig get ns -o jsonpath="{.items[*].metadata.name}" | Out-String
$namespaces = $namespaces.Trim().Split()

foreach ($ns in $namespaces) {
    if ($excludedNamespaces -notcontains $ns) {
        Write-Host "Aplicando label em namespace: $ns"
        & $kubectl $kubeconfig label ns $ns dynatrace.com/injection=true --overwrite
    } else {
        Write-Host "Ignorando namespace de sistema: $ns"
    }
}
```

<strong> Para validar se  se o label (dynatrace.com/injection=true) foi realmente aplicado aos namespaces corretos, você pode seguir alguns dos passos abaixo: </strong>

```bash
kubectl get ns --show-labels
```
ou: 

```ps1
.\kubectl.exe --kubeconfig=.\kubeconfig-homolog.yaml get ns --show-labels | findstr "dynatrace.com/inject=true"
```

<img width="746" height="154" alt="image" src="https://github.com/user-attachments/assets/8e3f2576-235a-4067-91ce-69a86e066ae0" />

#### Deep Monitoring 

<strong> ⚠️ Um ponto importante não esqueça de habilitar o "Deep Monitoring" na console UI, se faz necessário para que funcione os traces distribuídos </strong>
<br>
<img width="608" height="366" alt="Captura de tela 2025-07-29 211540" src="https://github.com/user-attachments/assets/116f7175-e4ca-4ce5-a601-3516a6ae9b01" />

