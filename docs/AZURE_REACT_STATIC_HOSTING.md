# Hospedagem React SPA — Azure App Service Linux

## Causa
O workflow compilava o React, mas publicava o repositório inteiro (`package: .`). O OneDeploy podia transferir com sucesso sem configurar um servidor para servir os arquivos de `build/`.

## Correção do workflow
- Verifica a existência de `build/index.html`.
- Usa somente `build/` como artifact do build e como package do deploy.

## Configurações da Azure
Se a Web App do frontend for Linux com stack Node.js/PM2, defina em Configuration → General settings / Startup Command:

```bash
pm2 serve /home/site/wwwroot --no-daemon --spa
```

O comando disponibiliza index.html na raiz e fallback SPA para rotas do React Router. Confirme previamente Linux, Node.js/PM2 e que `/home/site/wwwroot/index.html` exista após o deploy. Container customizado ou Windows exigem outra configuração.

Mantenha no GitHub Actions Repository Variables a configuração REACT_APP_BASE_URL_I_BLUE_IT=https://iblueit-server-side-bbfqegaygagse8b5.switzerlandnorth-01.azurewebsites.net/api.

Na Function App do backend, em CORS, libere a origem HTTPS exata da Web App do frontend.

## Validação
1. Mesclar PR na main para disparar novo deploy.
2. Verificar build e publish bem sucedidos.
3. Confirmar `index.html` na raiz de `/home/site/wwwroot`.
4. Acessar domínio HTTPS do FRONTEND, e não a URL de Functions do backend.
5. Testar rotas internas da SPA e login, inspecionando a requisição POST `/api/login`.

Não copie credenciais do Atlas para o frontend.
