# Recuperação do frontend Health InfoCharts no Azure

O workflow foi ajustado para contornar temporariamente o conflito React 16 x MUI 5 no npm, usando `npm install --legacy-peer-deps`. Isso não comprova compatibilidade integral das bibliotecas; o build pode revelar problemas adicionais. O pipeline usa Node 22 **apenas no build**. O modelo React e as rotas existentes não foram migrados.

## Pré-requisito obrigatório antes do merge

Em **GitHub → PHV00/iblueit-health-Infocharts → Settings → Secrets and variables → Actions → Variables**, crie uma variável de repositório:

```
REACT_APP_BASE_URL_I_BLUE_IT=https://<DOMINIO-REAL-DO-BACKEND>/api
```

O domínio deve ser o **Default domain** da Azure Function App `iblueit-server-side`. Não adicione login, senha ou connection string do MongoDB. A aplicação React incorpora `REACT_APP_*` **no momento do build**, não em tempo de execução. Esta URL não é um segredo.

O workflow falha propositalmente se a variável estiver ausente. Para a build local, copie `src/providers/_config.template.js` como `src/providers/_config.js` e defina `REACT_APP_BASE_URL_I_BLUE_IT` no ambiente ou em `.env` (ignorado pelo Git).

## Passos na Azure

1. No backend **Function App**, confirme a rota `POST /api/login` e o acesso ao Atlas.
2. Em **Function App → API → CORS**, adicione a origem pública **HTTPS do frontend**, sem barra final ou caminho. Não use `*` com credenciais.
3. Confirme o destino do workflow em **Azure App Service / Web App**: `iblueit-health-Infocharts`, incluindo o Default domain e as configurações do serviço.
4. Revise que o deployment do GitHub Actions tenha identidade OIDC autorizada para publicar no recurso.
5. O projeto React usa `react-scripts build` e produz assets estáticos em `build/`. O **App Service precisa servir estes arquivos** como SPA; se o deploy tiver sucesso mas aparecer a página padrão ou erro 404 nas rotas do navegador, configurar a inicialização/hospedagem estática será um passo separado. Para Linux App Service, verificar a stack e o comando de startup adequado, conforme o plano; não presumir o SO.
6. Após criar a variável no GitHub, **mesclar o PR em `main`** para disparar novo run do workflow. Não usar Re-run de uma execução antiga (repetiria o código do commit anterior).

## Como validar

- GitHub Actions: `Install legacy dependencies`, `Prepare API configuration`, `Build production frontend`, `Run non-interactive tests`, `Deploy`.
- No navegador: abrir o frontend e inspecionar Network para `POST https://<DOMINIO-REAL-DO-BACKEND>/api/login`.
- `404`: rota/endereço incorreto; `CORS`: origem não autorizada no backend; `500`: logs da API e Atlas; `401/403`: validação/autenticação.
- Não exponha senhas, dados de pacientes nem credenciais MongoDB nos logs de diagnóstico.

## Riscos pendentes

O código ainda usa React 16 e Material UI 5 com peer dependency incompatível; `--legacy-peer-deps` contorna a resolução, não torna essa combinação oficialmente compatível. O frontend também tem dependências antigas e deve ser migrado numa etapa posterior com testes.
