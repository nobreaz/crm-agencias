# CRM Agências

SaaS multi-tenant para agências de marketing gerenciarem clientes, contratos, cobranças e a produção de conteúdo num lugar só. É o meu TCC de Sistemas de Informação (FEMASS) e já está em produção com duas agências piloto.

> **Este repositório é uma vitrine.** O código-fonte é privado porque o sistema é um produto comercial em uso. Aqui estão o problema, a arquitetura, as decisões técnicas e os números. Posso fazer uma demonstração ao vivo ou mostrar o código numa conversa: [LinkedIn](https://www.linkedin.com/in/joao-pedro-nobre/).

## Em números

| | |
|---|---|
| Do repositório vazio à produção | **9 dias** (26/10 → 04/11/2026) |
| Agências usando no dia a dia | **2** |
| Testes automatizados na API | **230** |

## O problema

Agências pequenas cuidam de clientes, contratos, cobranças, calendário de posts e tarefas da equipe em ferramentas separadas. O CRM junta tudo isso, e cada agência é um *tenant* isolado, com assinatura própria.

## O que o sistema faz

- **Clientes e financeiro:** contratos, cobranças mensais geradas automaticamente, cobrança avulsa, pagamentos parciais, inadimplência e resumo de recebido, a receber e em atraso.
- **Posts:** calendário, quadro por etapas (produção → aprovação → programação → publicado) com arrastar e soltar, mídias com conversa e pedido de alteração.
- **Tarefas:** kanban com vários quadros.
- **Google Drive da agência:** as mídias vão sozinhas para a pasta do cliente.
- **Perfis e permissões** configuráveis por agência.
- **Administração da plataforma:** criar, suspender e excluir agências.
- **LGPD:** termos e política com aceite registrado, banner de cookies com opção de recusar, consentimento de marketing separado, retenção automática de dados, baixar e excluir a conta.
- **Acessibilidade:** pular para o conteúdo, contraste, menos movimento e kanban operável pelo teclado. Modo escuro.

## Arquitetura

```
Navegador ──► Caddy (HTTPS) ──┬──► web   Next.js + TypeScript
                              └──► api   ASP.NET Core (.NET 10) + EF Core ──► PostgreSQL
                                           ├──► Cloudflare R2 (mídias, API compatível com S3)
                                           └──► Google Drive API (OAuth)
VPS Linux
```

| Camada | Tecnologias |
|---|---|
| API | ASP.NET Core (.NET 10), EF Core, PostgreSQL, JWT |
| Web | Next.js, TypeScript, Tailwind CSS, dnd-kit, tipos gerados a partir do OpenAPI da API |
| Armazenamento | Cloudflare R2 em produção, SeaweedFS em desenvolvimento |
| Infra | VPS Linux, Caddy, migrations com EF Core |

## Decisões técnicas

- **Multi-tenant em banco compartilhado.** Toda tabela pertence a uma agência, e o filtro global de consulta do EF Core aplica esse filtro automaticamente na leitura e na gravação. Esquecer um `WHERE` não vaza dados de outra agência.
- **Por que .NET.** Comparei NestJS, ASP.NET Core e Spring Boot. O .NET ganhou pelo filtro global de consulta, pelas políticas de autorização nativas e pelo consumo de memória baixo numa VPS pequena.
- **Permissões, não perfis.** A API verifica permissões (por exemplo, `posts.aprovar`), nunca o nome do perfil. Cada agência monta seus perfis, e as permissões são lidas do banco a cada requisição, então uma mudança vale na hora.
- **Autenticação enxuta.** JWT com o `PasswordHasher` do ASP.NET Identity, sem o Identity completo.
- **Auditoria imutável.** Um trigger no PostgreSQL bloqueia UPDATE, DELETE e TRUNCATE na trilha de auditoria. A única exceção é a exclusão da própria agência, liberada só dentro daquela transação.
- **Concorrência otimista** com a coluna de sistema `xmin` do PostgreSQL no fluxo de aprovação, para que duas pessoas não aprovem ou alterem o mesmo post ao mesmo tempo.
- **Exclusão de agência com duas travas.** Ela precisa estar suspensa ou inativa, e o administrador digita o identificador da agência para confirmar.

## Como foi desenvolvido

Eu cuidei do produto e da engenharia de requisitos: requisitos funcionais, recorte do MVP, modelo de dados, regras de negócio e revisão de cada entrega. A implementação foi feita com o Claude Code a partir de um `CLAUDE.md` com as regras do projeto e de subagentes especializados (backend, QA e segurança). Cada mudança passa pelos testes automatizados e por uma revisão de segurança antes do commit. Com o sistema no ar, o uso real das agências virou a principal fonte de ajustes de UI e UX.

## Próximos passos

- App mobile consumindo a mesma API
- Assistente de conteúdo para Instagram com skills do Claude
- Cobrança da assinatura dentro do sistema
- Integrações com Meta Ads e Google Ads

---

João Pedro Azevedo Nobre · [LinkedIn](https://www.linkedin.com/in/joao-pedro-nobre/) · [GitHub](https://github.com/nobreaz)
