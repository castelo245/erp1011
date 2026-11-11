# ERP para pequenas empresas (Brasil)

Monólito modular: **Fastify + Prisma + PostgreSQL** (API) e **React + Vite** (web).
Multi-tenant por `tenantId`, RBAC (Administrador, Vendedor, Financeiro, Estoquista), auditoria e valores monetários em `Decimal`.

## Rodar (Node 20+ e Docker)

```bash
docker compose up -d          # PostgreSQL
npm run setup                 # instala, cria as tabelas e carrega dados de demonstração
npm run dev:api               # terminal 1 -> http://localhost:3333
npm run dev:web               # terminal 2 -> http://localhost:5173
```

Logins de demonstração: `admin@demo.com / admin123`, `vendedor@demo.com / vendedor123`,
`financeiro@demo.com / financeiro123`, `estoque@demo.com / estoque123`.

Antes de produção: troque `JWT_SECRET` em `apps/api/.env`, use HTTPS e aponte `VITE_API_URL` em `apps/web/.env`.

## O que está implementado

| Módulo | Entregue |
|---|---|
| Painel | Faturamento, despesas, saldo, a pagar/receber do dia, vencidos, estoque mínimo, gráfico de fluxo de caixa |
| Cadastros | Pessoas com busca de CEP (ViaCEP) e CNPJ (BrasilAPI), limite de crédito, notas; produtos com SKU/EAN/NCM, margem e preço automático |
| Vendas | Orçamento → pedido → faturamento, PDV com leitor de código de barras, pagamento dividido, troco, desconto com trava de alçada, comissão |
| Estoque e compras | Saída/entrada automáticas, custo médio, alerta de mínimo com sugestão de compra, inventário, perdas, pedidos/cotações, **importação de XML de NF-e de compra** |
| Financeiro | A pagar/receber, parcelas, recorrência, juros e multa, baixa parcial, contas/caixas, **OFX**, fluxo previsto × realizado, DRE |
| Fiscal | Fila de NF-e/NFC-e/NFS-e e adaptador Focus NFe (`apps/api/src/modules/fiscal/provider.ts`) |
| Relatórios | Curva ABC (produtos/clientes), vendas por vendedor, lucratividade, inadimplência; CSV e PDF (impressão) |

## Pontos de atenção

- **Fiscal:** o adaptador é um ponto de partida. A emissão real exige certificado, CSC, regime tributário e CST/CSOSN por produto, cadastrados no provedor. Teste em homologação.
- **Crédito:** limite 0 bloqueia vendas a prazo (boleto/crediário).
- **Conciliação:** use "Conciliar" para lançamentos ainda não baixados no sistema, para não duplicar o saldo.
- **LGPD:** falta criptografar `Person.document` em nível de coluna, endpoint de exportação/exclusão do titular e registro de consentimento.
- **Tenant:** todas as consultas filtram `tenantId` manualmente; considere Row Level Security no Postgres.
- **Login:** identifica o tenant pelo e-mail. Com e-mails repetidos entre empresas, use subdomínio.
- **Testes:** não há suíte automatizada ainda; a primeira prioridade são testes de `fulfillSale`, `settleTitle` e `receiveStock`.
