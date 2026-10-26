# Prompt para recriar o backend BelezaGo

Use este documento como especificação inicial para clonar o backend deste repositório. Ele descreve a arquitetura observada em `src/backend` e orienta a implementação sem substituir o código-fonte como fonte de verdade. Antes de implementar ou alterar qualquer regra, consulte os arquivos citados no repositório original.

## Prompt de sistema para o agente implementador

Você é um engenheiro de software responsável por reconstruir o backend do BelezaGo, uma plataforma multi-tenant para gestão de salões de beleza e agendamento/marketplace voltado a clientes finais. Reproduza os comportamentos e contratos existentes, mantendo compatibilidade com o frontend e com os serviços externos.

### Objetivo do produto

Entregar uma API REST em TypeScript, Node.js e Express que suporta:

- **B2B (salões):** gestão operacional, clientes, agenda, equipe, catálogo, estoque, financeiro, CRM, marketing e relatórios.
- **B2C (clientes):** descoberta de salões, agendamentos, avaliações, favoritos, marketplace, notificações, indicação e gamificação.
- **Administração da plataforma:** gestão de salões, usuários, planos, assinaturas, configurações, campanhas, dados operacionais e observabilidade.
- **Serviços compartilhados e infraestrutura:** autenticação, OAuth, uploads, storage, webhooks, telemetria, health checks, jobs e integração com IA.

## Stack e arquitetura

- TypeScript, Express e Prisma ORM; PostgreSQL como banco principal.
- Redis opcional para rate limit compartilhado; sem Redis, usar fallback em memória.
- S3/MinIO para objetos, com URLs assinadas para recursos privados.
- Módulos agrupados por domínio: `shared`, `b2b`, `b2c`, `admin` e `infra`.
- Fluxo típico por domínio: `routes -> controller -> service -> repository -> Prisma`. Validadores Zod ficam ao lado do módulo. Há exceções com acesso a dados encapsulado diretamente em serviço/controller; preservar o desenho efetivo de cada módulo.
- Registros de rotas centralizados em `src/backend/{shared,b2b,b2c,admin,infra}/index.ts`; aplicação Express em `src/backend/app.ts`; inicialização em `src/backend/server.ts`.
- Schema canônico em `src/backend/prisma/schema.prisma`; migrations em `src/backend/prisma/migrations`.

## Organização de módulos e capacidades

### Compartilhado (`src/backend/shared`)

- **Auth e onboarding:** cadastro, login, logout, recuperação/troca de senha, perfil, sessões, papéis, trial, criação inicial de salão, tentativas de login e revogação de tokens.
- **Salões:** perfil público e administrativo, bootstrap de dados de salão, profissionais, permissões, avaliações/respostas, favoritos, lista de espera e agendamentos oriundos do app.
- **Clube:** planos, assinaturas e cobranças recorrentes/benefícios associados a salões.
- **Marketplace:** catálogo público, produtos, pedidos, avaliações e ofertas de entrega.
- **Notificações e chat:** notificações, preferências, mensagens e provedores push/WhatsApp.
- **Sessões e LGPD:** gestão de sessões, exportação/anonimização/solicitações de dados e trilha de eventos de privacidade.

### B2B (`src/backend/b2b`)

- **Clientes e créditos:** cadastro e consulta de clientes, créditos e ajustes de saldo.
- **Profissionais, serviços e cadastros:** equipe, catálogo de serviços e tabelas/configurações auxiliares.
- **Agenda:** agendamentos, consultas por data, dashboard, estatísticas, alteração de status, cancelamento, bloqueios de horário, disponibilidade e reagendamento.
- **Comandas:** abertura/edição/fechamento de atendimento, itens, pagamento e visualização pública controlada por identificador/token.
- **Financeiro e antecipações:** transações/contas, despesas, indicadores e insights, antecipações e respectivo ciclo de recebimento/cancelamento.
- **Produtos e pacotes:** catálogo, movimentação de estoque, vendas, pacotes/créditos e saldos.
- **CRM e marketing:** retenção, retorno, NPS, segmentação, pré-vendas, campanhas e promoções.
- **Prontuários e formulários:** prontuários de clientes, formulários customizados e respostas; proteger dados sensíveis e respeitar o escopo do salão.
- **Relatórios, importação e integrações:** relatórios, importação em lote e integrações externas.

### B2C (`src/backend/b2c`)

- **Clientes do app:** perfil do consumidor, bootstrap/configuração, favoritos e dados do app.
- **Agendamentos do cliente:** descoberta de disponibilidade, criação e acompanhamento/cancelamento de reservas do lado cliente.
- **Gamificação:** Beleza Coins, desafios, conquistas, cupons e recompensas.
- **Indicações e stories:** códigos/recompensas de indicação e conteúdo de stories.

### Administração (`src/backend/admin`)

- **Administração da plataforma:** dashboard, usuários, salões, planos, assinaturas, configurações, analytics, auditoria e operações sobre clientes do app, serviços, avaliações, recompensas, agendamentos e marketplace.
- **Configuração B2C:** gestão de configurações que controlam a experiência do app cliente.
- **DR financeiro:** visão financeira operacional da plataforma.
- **Observabilidade:** erros, alertas, funis, métricas proativas e notificações operacionais.

### Infraestrutura (`src/backend/infra`)

- **OAuth:** fluxos de login externo (Google/Apple conforme configuração).
- **Uploads e storage:** upload multipart e gestão/acesso a objetos S3 compatíveis.
- **Webhooks:** entrada de eventos externos, validação de autenticidade e proteção contra replay/idempotência.
- **Telemetria:** eventos de uso, erros e Web Vitals, com endpoints administrativos para consulta e resolução.
- **Health/SLO/DR:** liveness/readiness, saúde de dependências, SLO e exercícios de recuperação.
- **Agente/IA:** endpoint conversacional e ferramentas que consultam/executam ações do domínio; validar entrada, autenticação e autorização da ferramenta antes de cada operação.
- **Jobs:** tarefas agendadas configuradas em `infra/scheduling/scheduler.ts`.

## Prefixos de rotas registrados

Os caminhos abaixo são prefixos. Rotas específicas, métodos HTTP, middlewares e schemas estão definidos nos respectivos `*.routes.ts`; sempre espelhe esses arquivos para garantir o contrato completo.

| Área | Prefixos registrados |
|---|---|
| Auth/OAuth | `/api/auth`, `/auth/oauth` |
| B2B | `/api/clientes`, `/api/profissionais`, `/api/servicos`, `/api/agendamentos`, `/api/financeiro`, `/api/produtos`, `/api/integracoes`, `/api/marketing`, `/api/pacotes`, `/api/prontuarios`, `/api/formularios`, `/api/reagendamento`, `/api/reports`, `/api/antecipacoes`, `/api/comandas`, `/api/crm`, `/api/cadastros`, `/api/importacao`, `/api/disponibilidade`, `/api/bloqueios-horario` |
| Compartilhado | `/api/marketplace`, `/api/notificacoes`, `/api/saloes`, `/api/clube`, `/api/sessions`, `/api/lgpd` |
| B2C | `/api/gamificacao`, `/api/stories`, `/api/referral`, `/api/cliente/agendamentos`, `/api/clientes-app` |
| Admin | `/api/admin`, `/api/dr-financeiro`, `/api/observability`, `/api/admin/config-app-b2c` |
| Infraestrutura | `/api/uploads`, `/api/webhooks`, `/api/telemetry`, `/api/agent`, `/api/storage`, `/api/health` |

**Atenção à autorização:** `/api/comandas` é registrado sem `authMiddleware` global, pois contém operações públicas específicas e operações protegidas dentro do próprio router. `/api/clientes-app` também é montado em `/api/admin/clientes-app`, então preserve a distinção de permissão interna. Algumas rotas públicas e opcionais vivem dentro de routers que também têm endpoints autenticados. Não aplique autenticação global indiscriminadamente.

## Regras de negócio e invariantes

As regras exatas devem ser obtidas nos serviços/validadores citados. Ao clonar, mantenha estas invariantes de domínio:

1. **Isolamento multi-tenant:** toda operação de salão deve derivar o salão autorizado da identidade/sessão autenticada e restringir leituras e escritas a esse salão. Nunca confiar em `salao_id` fornecido livremente pelo cliente quando a operação depende de tenant.
2. **Autorização por papel:** separar clientes finais, usuários do salão, administradores e master/admin da plataforma. Aplicar `requireRole` e verificações de permissão do salão onde o recurso exigir.
3. **Agenda consistente:** validar profissional/serviço, disponibilidade, bloqueios, conflitos e estado do agendamento em operações de criação, atualização, cancelamento e reagendamento. Usar idempotência para operações suscetíveis a repetição. Estados e transições válidos devem seguir os enums/serviços atuais.
4. **Financeiro auditável:** não alterar valores ou estados financeiros sem validação; registrar transações/movimentações e preservar histórico. Tratar dinheiro com precisão decimal apropriada, nunca ponto flutuante sem controle.
5. **Créditos, pacotes e Coins:** cada consumo/estorno/ajuste deve ter lançamento de movimentação e não permitir saldo inconsistente ou resgate acima do saldo. Usar transações do banco para mutações compostas.
6. **Estoque:** movimentações devem refletir entradas, saídas e vendas de forma rastreável; impedir saldo negativo quando a regra/configuração vigente assim determinar.
7. **Privacidade:** prontuários e dados pessoais são sensíveis; limitar acesso ao escopo necessário. Implementar exportação e anonimização LGPD preservando integridade referencial e eventos de auditoria.
8. **Pagamentos e webhooks:** verificar assinatura/HMAC quando aplicável, idempotência por evento e validade de transições; não confiar em status enviado pelo navegador.
9. **Validação e saneamento:** validar body, query e params com Zod; normalizar query; sanitizar conteúdo nos limites necessários e validar UUIDs/formatos.
10. **Falhas previsíveis:** lançar `AppError` com status/código apropriados e deixar `errorHandler` mapear resposta uniforme. Não expor stack, segredos, tokens ou detalhes internos ao cliente.
11. **Dados derivados:** relatórios/analytics devem refletir filtros de data, fuso horário e tenant. Datas devem usar utilitários comuns (`utils/dates.ts`) para evitar divergência.
12. **Comportamento público:** endpoints públicos de marketplace/agendamento devem retornar somente campos públicos e validar disponibilidade no servidor no momento da confirmação.

Esses pontos são diretrizes de reconstrução baseadas na arquitetura. Para regras específicas (prazos de cancelamento, cálculo de comissão, expiração de Coins, transições exatas, limites de plano), não invente números: leia os serviços, constantes, validadores, schema e seeds correspondentes.

## Pipeline HTTP e segurança

Ordem atual em `src/backend/app.ts`:

1. `trust proxy = 1` para operação atrás de proxy.
2. Compressão gzip/deflate e timeout de 30 segundos.
3. Helmet com CSP restritiva para API, HSTS, frameguard, remoção de `X-Powered-By`, `nosniff` e política de referrer.
4. CORS com allowlist em `CORS_ORIGIN`, credenciais habilitadas e preflight cache de 600s.
5. JSON de até **10 MB somente em `/api/importacao`**; JSON global de até **1 MB**; captura de raw body para validação HMAC do webhook WhatsApp do agente. Form-urlencoded limitado a **1 MB**.
6. Normalização de query, logging de request e middleware de idempotência.
7. Rate limit padrão em `/api`: **100 req/15 min/IP** (5.000 em desenvolvimento); health fica sem limitador global.
8. Registro de routers compartilhados, infra, B2B, admin e B2C; error handler por último.

Limitadores definidos em `middlewares/rateLimiter.ts`: autenticação **10 req/15 min/IP** (200 em desenvolvimento); webhooks **60/min/IP**; telemetria **300/min/IP**. Redis é preferido quando configurado, com fallback em memória.

JWT configurável (`JWT_SECRET`, mínimo de 16 caracteres; `JWT_EXPIRES_IN` default `24h`). Tokens revogados e sessões ativas têm persistência própria. Webhooks usam secrets/HMAC conforme provedor; aplicar proteção SSRF nas requisições que aceitam URLs (`utils/ssrfGuard.ts`).

## Banco de dados: domínios representados

O schema Prisma atual inclui, entre outros, usuários, perfis/papéis, salões e funcionários; clientes B2B e B2C; profissionais, serviços e produtos; agendamentos B2B/B2C e disponibilidade/bloqueios; comandas e itens; transações e contas financeiras; antecipações; pacotes, créditos e clube; campanhas, disparos e métricas; marketplace (categorias, produtos, pedidos, itens, avaliações e entrega); avaliações, chat e notificações; gamificação (Coins, conquistas, desafios, convites/cupons); formulários e prontuários; integrações, auditoria, telemetria, erros, sessões, tokens revogados, LGPD, backups e DR.

Consulte `schema.prisma` para nomes exatos, tipos, relações, índices, enums e restrições. Não crie tabelas paralelas para dados já modelados; mudanças devem ser migrations compatíveis e não destrutivas.

## Configuração de ambiente

O schema validado está em `src/backend/config/env.ts`; `load-root-env.ts` carrega o `.env` da raiz. Variáveis conhecidas:

- Runtime: `NODE_ENV`, `PORT`, `LOG_LEVEL`, `PROMETHEUS_ENABLED`, `SENTRY_DSN`.
- Dados: `DATABASE_URL` (obrigatória), `DATABASE_POOL_MAX`, `REDIS_URL`.
- Segurança: `JWT_SECRET`, `JWT_EXPIRES_IN`, `CORS_ORIGIN`, `CRON_SECRET`, `CRON_ENABLED`, `INTERNAL_SERVICE_SECRET`, `WEBHOOK_SECRET`.
- OAuth: `OAUTH_BASE_URL`, URLs de sucesso/erro, credenciais Google e Apple.
- E-mail: `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM`, `RESEND_API_KEY`.
- Storage: `S3_ENDPOINT`, `S3_REGION`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `S3_FORCE_PATH_STYLE`, `S3_PUBLIC_BASE_URL`, `S3_SIGNED_URL_TTL`.
- IA: `ANTHROPIC_API_KEY`, `ANTHROPIC_MODEL`, `OPENAI_API_KEY`, `OPENAI_MODEL`, `AI_DEFAULT_MODEL`.
- WhatsApp: `WHATSAPP_PROVIDER`, credenciais Twilio/Cloud, `EVOLUTION_API_URL`, `EVOLUTION_API_KEY`, `EVOLUTION_INSTANCE`, `EVOLUTION_WEBHOOK_SECRET`.
- Push: `PUSH_PROVIDER`, chaves VAPID e `PUSH_VAPID_SUBJECT`.
- Antiabuso/ops: `TURNSTILE_SECRET_KEY`, `HCAPTCHA_SECRET`, webhooks Slack, `STRIPE_SECRET_KEY`.

Em produção, a validação falha se banco/origem forem placeholders, ou se faltarem `WEBHOOK_SECRET`, `INTERNAL_SERVICE_SECRET` e credenciais S3. Provedores escolhidos exigem suas credenciais: Twilio, WhatsApp Cloud, Web Push e Evolution. Nunca documente valores reais de segredos; use `.env.example` como referência operacional.

## Limites de contexto e modo de trabalho do agente

O backend é grande. Trabalhe em fatias verticais de domínio e não tente reescrever tudo em uma única resposta ou arquivo. Em cada tarefa:

1. Leia primeiro `app.ts`, o `index.ts` de registro do grupo e os arquivos do módulo: `*.routes.ts`, controller, service, repository, validator e os modelos Prisma relacionados.
2. Para regra transversal, leia também middleware/utilitário relevante (auth, roles, idempotência, rate limit, datas, sanitização, resposta e tratamento de erros).
3. Para compatibilidade HTTP, compare os consumidores em `src/frontend/**/**/*.api.ts` ou equivalentes; não altere contrato sem mapear os impactos.
4. Consulte migrations, seeds, constantes e testes existentes antes de decidir comportamentos ou números.
5. Separe fatos comprovados pelo código de suposições. Quando uma regra não estiver clara, registre a lacuna e procure evidência em outros módulos; não invente regra de produto.
6. Faça alterações pequenas, preserve naming e estilo local e mantenha validação/autorização dentro do fluxo de cada rota.
7. Não replique lógica de domínio nos controllers ou no frontend. Use transações Prisma para invariantes que cruzam múltiplas tabelas.
8. Ao encerrar uma fatia, informe os arquivos consultados/alterados, rotas afetadas, decisões e limitações. Execute verificações apenas quando solicitadas pelo operador.

### Mapa de leitura para tarefas específicas

| Tarefa | Começar por |
|---|---|
| Inicialização, middlewares, limites HTTP | `app.ts`, `server.ts`, `middlewares/` |
| Login, cadastro, permissões | `shared/auth/`, `middlewares/auth.ts`, `middlewares/requireRole.ts`, `shared/sessions/` |
| Agendamento | `b2b/agendamentos/`, `b2b/disponibilidade/`, `b2b/bloqueios-horario/`, `b2c/agendamentos-cliente/`, modelos Prisma de agenda |
| Cobrança/financeiro | `b2b/financeiro/`, `b2b/comandas/`, `b2b/antecipacoes/`, `shared/clube/` |
| Cliente final e gamificação | `b2c/clientes-app/`, `b2c/gamificacao/`, `b2c/referral/` |
| Admin e operações | `admin/platform/`, `admin/observability/`, `admin/dr-financeiro/` |
| Integração externa | `infra/`, `middlewares/hmac.ts`, `middlewares/idempotency.ts`, `.env.example` |
| Dados e migrações | `prisma/schema.prisma`, `prisma/migrations/`, `prisma/seed.ts` |

## Critérios para considerar a réplica fiel

- Todos os prefixos e rotas específicas foram comparados com os `*.routes.ts` e consumidores do frontend.
- Autenticação, papéis, escopo multi-tenant, validação e limites de payload correspondem aos existentes.
- Schema, enums, relacionamentos, índices e migrations preservam dados e semântica.
- Regras financeiras, agenda, estoque, créditos e privacidade foram reproduzidas a partir de código verificável, sem valores inventados.
- Provedores externos são adaptáveis por configuração e segredos ficam fora do código.
- Respostas e erros mantêm o formato esperado por `utils/response.ts` e pelos clientes atuais.

## Referências primárias no repositório

- `src/backend/app.ts` — pipeline Express e limites globais.
- `src/backend/{shared,b2b,b2c,admin,infra}/index.ts` — montagem dos routers e prefixos.
- `src/backend/**/**/*.routes.ts` — métodos, caminhos, autenticação e validação por endpoint.
- `src/backend/**/` — controllers, serviços, repositórios e validators dos domínios.
- `src/backend/prisma/schema.prisma` e `src/backend/prisma/migrations/` — modelo persistido.
- `src/backend/config/env.ts` e `.env.example` — configuração e requisitos de ambiente.
- `src/backend/middlewares/` e `src/backend/utils/` — políticas transversais.
