# Instalação Dynatrace (APM - Kubernetes)
## Ambiente Managed (on premise)

## 1) Valide se está liberado as porta 443 do lado dos nodes Kubernetes, e porta 9999 no ActiveGate  

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

## Parte Kubernetes 



