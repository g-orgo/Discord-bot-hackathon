# 🦅 Raptor Audit — Resumo para Stakeholders

**Data:** 6 de abril de 2026

Fizemos uma revisão completa dos quatro projetos ativos do Raptor: o bot do Discord, o servidor de IA, o servidor de autenticação e o frontend web.

## O que encontramos

### 🔒 Segurança — Brute-force nos endpoints de login *(corrigido)*
O servidor de autenticação não tinha limite de tentativas de login. Isso significava que alguém poderia tentar milhares de senhas automaticamente sem ser bloqueado — um problema clássico de segurança. Adicionamos um limite de 10 tentativas por IP a cada 15 minutos. Quem ultrapassar esse limite recebe erro 429 e precisa aguardar.

### 🌐 Configuração — URL de CORS hardcoded no servidor LLM *(corrigido)*
O servidor de IA tinha o endereço do frontend (`localhost:5173`) fixo no código. Isso faria qualquer deploy em produção parar de funcionar silenciosamente para usuários no browser. Tornamos o valor configurável via variável de ambiente.

### 🧹 Limpeza — Imports não usados no bot do Discord *(corrigido)*
Quatro funções estavam sendo importadas mas não usadas no arquivo principal do bot. Removemos para manter o código limpo e sem ruído.

## O que está bem ✅

- Autenticação com JWT funcionando corretamente
- Senhas sempre armazenadas com hash (bcrypt) — nunca em texto puro
- Frontend sem URLs hardcoded — tudo via proxy
- Rota de Personalidade protegida: só acessa quem está logado
- Modelos de dados tipados no servidor de IA (Pydantic)
- Validação de entrada nos endpoints de registro/login

## Próximos passos sugeridos

- Documentar a variável `CORS_ORIGIN` para deploys em produção
- Em produção, colocar o servidor LLM atrás de um proxy reverso para que a porta 8000 não fique exposta publicamente
