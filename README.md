# Jobora

Sistema pessoal de busca de vagas: captura e triagem de vagas, análise de mercado, adequação de currículo e acompanhamento de candidaturas. Laravel 13 + Filament 5, com servidor MCP local (`jobora`) para agentes.

## Requisitos

- PHP 8.4+ com `intl`, `pdo_sqlite`, `mbstring`, `bcmath`, `gd`, `zip`
- Composer 2
- Bun (ou Node 22 com npm)

## Subir o projeto

```bash
composer install
cp .env.example .env
php artisan key:generate
touch database/database.sqlite
php artisan migrate
bun install
bun run build
```

O painel Filament fica na raiz (`/`). Para desenvolvimento:

```bash
php artisan serve
```

Com `lerd`:

```bash
lerd link
lerd setup
lerd open
```

O site sobe em `https://jobora-v2.test` e os workers de `queue`, `schedule` e `reverb` em `lerd worker start queue` (idem para os demais).

## Banco de dados

SQLite por padrão: `DB_CONNECTION=sqlite` e `DB_DATABASE=database/database.sqlite`. Migrações com `php artisan migrate`. A suíte usa `database/database.testing.sqlite`, criado no primeiro run (ou com `touch`).

## Fila

`QUEUE_CONNECTION=database`. As operações de IA (extração, triagem, adequação, ATS e competências) rodam em jobs; sem worker elas ficam pendentes.

```bash
php artisan queue:work
```

## Agendador

```bash
php artisan schedule:work
```

## Broadcast (Reverb)

`BROADCAST_CONNECTION=reverb`. Preencha as chaves `REVERB_*` no `.env` e rode:

```bash
php artisan reverb:start
```

## Servidor MCP

A identidade é fixa por instância: `JOBORA_MCP_USER` aceita o e-mail ou o id numérico do usuário dono dos dados. Sem esse valor, as tools falham com o erro `configuration`.

```bash
JOBORA_MCP_USER=voce@exemplo.com php artisan mcp:start jobora
```

Exemplo de cliente stdio:

```json
{
  "mcpServers": {
    "jobora": {
      "command": "php",
      "args": ["artisan", "mcp:start", "jobora"],
      "cwd": "/caminho/para/jobora-v2"
    }
  }
}
```

## IA

O provedor padrão é o OpenCode Zen (`AI_PROVIDER=opencode_zen`), configurado em `config/ai.php`. Preencha `OPENCODE_ZEN_API_KEY` no `.env`. O header `x-opencode-session` usa um id estável por padrão; defina `OPENCODE_ZEN_SESSION_ID` só se precisar de um id próprio.

## Extensão de captura

A extensão de navegador (WXT) fica em `extension/`:

```bash
cd extension
bun install
bun run build   # ou bun run dev
bun run test
```

## Testes

```bash
php artisan test
```

O workflow `.github/workflows/tests.yml` roda a suíte no GitHub Actions e um smoke test do MCP (`tools/list` em `php artisan mcp:start jobora`).

## Documentação

- `GLOSSARY.md` — vocabulário de domínio
- `docs/adr/` — decisões de arquitetura
- `PRD.md`, `TECHNICAL_PLAN.md`, `IMPLEMENTATION_PLAN.md`, `MCP_SPEC.md` — produto, plano técnico e contrato MCP
