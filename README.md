# POC - Configuring Keycloak as External Key manager inside WSO2 API Manager

# Setup
Para iniciar a execução do projeto primeiro temos que subir as aplicações no Docker. Para isso basta executar o comando no terminal: docker-compose up -d

# WSO2 API manager

### Version 3.2.0

### Credentials
- user: admin
- pass: admin

### URLs
- Carbon: https://localhost:9443/carbon
- Publisher: https://localhost:9443/publisher
- Dev portal: https://localhost:9443/devportal
- Admin: https://localhost:9443/admin

# Keycloak

### Version 20.0.5

### Credentials 
- user: admin
- pass: admin

### URLs
- https://localhost:8080


# MySQL

### Version 5.7

### Credentials 
- MYSQL_ROOT_PASSWORD=root_pwd
- MYSQL_USER=admin
- MYSQL_PASSWORD=pa55word
- MYSQL_DATABASE=keycloak


# Tutorial 1 - Step by Step - WSO2 + Keycloak OPENID

Primeira parte do tutorial será realizar a configuração no Keycloak do client que o WSO2 irá usar com OPENID

1. Open Keycloak http://localhost:8080/admin/{{realm}}/console
1. Keycloak -> Client Scopes -> Create a new scope with the values below:
    - name: default
    - protocol: openid-connect
    - display on consent screen: true
    - include in token scope: true

2. Keycloak -> Client -> Criar novo client para WSO2 API mananger
3. Fill the fields with the values below:
    - Client Id: apim-client
    - Enabled: true
    - Client protocol: openid-connect
    - Client Authentication / Access type: ON / confidential
    - Standard flow enabled: true
    - Direct access grant flow enabled: true
    - Service accounts enabled: true
    - Valid redirect uris: https://wso2-api-manager:9443 
    - Advanced Settings -> Access toke lifespan: 1000 hours - Only development POC
    - Client Scopes -> Add scope default to Assigned Default Client Scopes
    - Service Account Roles -> realm-admin, offline_acces, uma_authorization to Assigned Roles
4. Essa ultima etapa é importante caso tu esteja rodando o keycloak via docker, no docker o host geralmente é o nome do service. No nosso caso o nome do service é keycloak. Portanto, precisamos configurar no Realm Settings -> Frontend URL -> http://keycloak:8080, para quando for gerado o token do keycloak ele retornar dentro do token jwt o host correto senão o WSO2 irá invalidar o token.

Nessa etapa iremos configurar o WSO2 com um novo Identity Server que será o Keycloak com OPENID
Isso se replica para todos os realms, cada realm será um Key manager diferente no WSO2

4. Open WSO2 APi Mananger / Admin Portal -> https://localhost:9443/admin
5. Open Key Managers -> Add Key Manager
6. Fill the fields with the values below:
    - Name: keycloak
    - Description: Keycloak
    - Key Manager Type: Keycloak
    - Well-know URL: http://keycloak:8080/realms/{{realm}}/.well-known/openid-configuration, after we must click in Import to load all data.
    - UserInfo Endpoint: http://keycloak:8080/realms/{{realm}}/protocol/openid-connect/userinfo
    - Client Id: apim-client
    - Client Secret: open apim-client secret, copy the secret from keycloak realm and paste here
    - Save

Depois de todas as etapas de configuração agora é criar a api no wso2 e usar que estará pronta toda a parte de autorização

# Create a API and invoke with Keycloak token - Client to Server communication
1. Open WSO2 API Manager / Publisher -> https://localhost:9443/publisher
2. Deploy Sample API -> Publish - This will create a Pizza API as example to be used
3. Runtime Configurations -> Application Level Security -> Key Manager Configuration -> Allow Selected -> Keycloak e Resident Key Manager
    OBSERVAÇÃO:
    - A opção default é Allow All, isso quer dizer que qualquer client id e secret gerado em qualquer key manager vai ser valido pra essa api. Caso queira especificar qual o Key Manager que pode ser usado é só selecionar Allow selected e escolher.
    - No nosso caso a api só poderá ser acessar pelo WSO2 Key manager ou Keycloak key manager
4. Open WSO2 API Manager / Dev Portal -> https://localhost:9443/devportal/apis
5. Open Applications -> Add new application
    - Application Name: Myapp
    - Per Token Quota: Unlimited # ONLY FOR POC
6. Select Production Keys -> OAuth2 Tokens -> Select Keycloak
    OBSERVAÇÂO:
    - Aqui conforme configuramos anteriormente, teremos possibilidade de gerar o token pelo Key manager do keycloak ou pelo Key manager do WSO2.
    - In grant type select -> Code, Refresh token, Password, Device Code. Unselect the others type.
    - If select Code grant type we must put a Callback URL. Here we will use https://www.keycloak.org/app/ because we will test the client use this keycloak.org/app application. Callback URL is a redirection URI in the client application which is used by the authorization server to send the client's user-agent (usually web browser) back after granting access.
    - Click in Generate Keys. Copy the client id e secret and use in the client application to request a token.
7. Do the same step above to Sandbox Keys
8. Go to APIs -> Select Pizza API -> Subscriptions -> Select MyApp and add to subscriptions
9. Now you can generate a token in Keycloak and use to Try Out the request to Pizza API

# Create a API and invoke with Keycloak token - Server to Server communication
@todo

################################################################################################################

# Tutorial 2 - Step by Step - WSO2 + Keycloak SAML - Federated or Third Party Key Manager?
@todo

################################################################################################################

# References
- https://apim.docs.wso2.com/en/latest/administer/key-managers/configure-keycloak-connector/
- https://www.youtube.com/watch?v=xuZ6DPhXNX8
- https://www.chakray.com/how-use-keycloak-as-wso2-api-manager-identity-provider/
- https://athiththan11.medium.com/wso2-api-manager-keycloak-saml-sso-164dea896bbf