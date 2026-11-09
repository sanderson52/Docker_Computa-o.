# Evidências de execução: Avaliação P2 (Docker e Docker Compose)

**Aluno:** Nome do Aluno
**Data da execução:** 05/11/2026
**Ambiente:** Windows 11, Docker Desktop 4.86 (WSL2), Docker Engine 29.7.2, Compose v5.3.1, PowerShell 5.1

Os 14 labs foram executados nesta máquina em duas rodadas:

1. A primeira validou a base com boas práticas aplicadas.
2. A segunda foi repetida **depois da personalização**: scripts, sites, APIs, modelo de dados e documentação próprios.

Os labs 01 e 05 não têm arquivos de código próprios. Só o README mudou, e os comandos
são os mesmos da 1ª rodada.

---

## Personalização sobre o repositório base

Dos 90 arquivos do repositório base, só **3** continuam idênticos, todos de propósito:

| Arquivo | Por que não mudou |
|---|---|
| `docs/Roteiro_Docker_Aluno.docx` | É o roteiro do professor (material de referência) |
| `labs/04-volumes-bind-mounts/dados/.gitkeep` | Arquivo vazio que só mantém a pasta no git |
| `labs/13-registry-seguranca/secrets/db_password.txt` | Segredo **de exemplo** didático, explicado no README do Lab 13 |

### Mudanças transversais

| Área | Mudança | Motivo |
|---|---|---|
| `.gitignore` | Ignora `.env` (mantém `.env.example`), `dados/*` do Lab 04, `backup.sql` e `dist/` | O original ignorava só a pasta `.env/`, então um `.env` com senha real seria commitado |
| `.gitattributes` | `eol=lf` em `.sh`, `.sql`, `.conf`, `Dockerfile*`, `.env*` etc. | O CRLF de um clone no Windows quebra scripts e SQL dentro de containers Linux |
| Dockerfiles próprios | Labels OCI, usuário não-root, `HEALTHCHECK` com `wget` em `127.0.0.1`, `COPY --chown`, `.dockerignore` | Segurança e imagens menores |
| Documentação | README da raiz e os 14 READMEs reescritos: objetivo, decisões, comandos PowerShell e bash, resultado, troubleshooting e limpeza | Autoria própria |

**Mantido de propósito**, pelo objetivo didático: o `Dockerfile.single-stage` (Lab 11) continua
ingênuo (só ganhou `ENV IMAGEM_VARIANTE`) e o Lab 02 continua rodando como root.

---

## Resultados por lab (2ª rodada, após a personalização)

| Lab | Resultado observado | Status |
|---|---|---|
| **01** Fundamentos | `hello-world` OK; run → pause (`Up (Paused)`) → unpause → logs → inspect → rm (1ª rodada) | ✅ |
| **02** Ubuntu | Novo `explorar.sh`: kernel `6.18.54.1-microsoft-standard-WSL2` (do host); `curl [ ] ausente` → `[x] presente` depois do `apt install`; container novo volta sem curl | ✅ |
| **03** Dockerfile | `healthy`, `autor: "Nome do Aluno"`, `usuarioLinux: "node"`, **193 MB** | ✅ |
| **04** Volumes | Bind: 1 → 2, e o log mostra **hostnames diferentes** a cada `--rm`; tmpfs: 1 → 1; sem volume: mensagem de erro clara e exit 1 | ✅ |
| **05** Redes | DNS por nome na rede customizada OK; bridge padrão: `bad address` (**esperado**) (1ª rodada) | ✅ |
| **06** Apache | HTTP 200, `X-Servido-Por` + `X-Autor`, novo título; `estilo.css` 200; `httpd -t` → `Syntax OK` | ✅ |
| **07** Nginx | `/api/status` OK; `/api/info` mostra `xRealIp`/`xForwardedFor` (prova do proxy); rota inexistente → 404 em JSON; backend `healthy` como `node` | ✅ |
| **08** Compose | 3 serviços `healthy`; o Nginx sobe ~5 s depois, quando o backend fica `healthy`; títulos e `backend-lab08` próprios | ✅ |
| **09** PostgreSQL | 5 alunos do novo seed (`BS2026-001…005`, 3 cursos); view `vw_resumo_turmas` com 4 turmas; `CHECK` rejeitou período 12; 6 alunos depois de recriar o container | ✅ |
| **10** pgAdmin | `db` e `pgadmin` `healthy`; importou o servidor "Lab 10 · Postgres (Nome do Aluno)" no grupo "Unifeg 2026B — Nome do Aluno"; o psql do pgAdmin autenticou no `db` | ✅ (login na UI: manual) |
| **11** Multi-stage | **single = 1,65 GB** (root, com tsc) vs **multi = 193 MB** (node, sem tsc), **~88% menor**; a API informa `"imagem":"single-stage"` ou `"multi-stage"` | ✅ |
| **12** React | Dev `healthy` e `[vite] hmr update /src/App.jsx`. Prod: assets com `Cache-Control: public, immutable` + gzip; `index.html` com `no-cache` e cabeçalhos de segurança; rota SPA 200 | ✅ |
| **13** Registry/Secrets | `appuser`, `rodandoComoRoot: false`; segredo fora do `env`; `ReadonlyRootfs=true` (`touch /app/x` → read-only), `/tmp` gravável; `CapEff: 0000000000000000`; `no-new-privileges`; 0.5 CPU / 128 MiB; push/pull no registry local | ✅ |
| **14** Integrador | Detalhes abaixo | ✅ |

---

## Lab 14: projeto integrador

```
 Host :8084 ──► frontend (Nginx + React) ──rede-frontend──► backend (Node)
                                                              │
 Host :8083 ──► pgadmin ──────rede-backend (internal)─────────┴──► db (Postgres, sem porta no host)
```

| Teste | Resultado |
|---|---|
| `down -v` + `up -d --build --wait` (ambiente limpo) | 4/4 `healthy` |
| `GET /api/alunos` | 5 alunos do seed |
| `GET /api/resumo` | totais por curso e período, vindos da view |
| `POST` válido | **201** |
| `POST` com RA duplicado | **409** |
| `POST` sem RA, período 12 ou JSON inválido | **400** |
| `POST` de 20 KB | **413**, e a API continua respondendo |
| `POST` com BOM (pipe do PowerShell) | **201** |
| `POST` com corpo em ISO-8859-1 (PS 5.1 sem charset) | **400**, sem gravar texto corrompido |
| `POST` do comando do README (`charset=utf-8` + bytes UTF-8) | **201**; `Informação` gravado com 24 bytes (UTF-8 correto) |
| Rota inexistente | 404 em JSON |
| Cadastro pela interface (curso escolhido no select) | Gravado; tabela e resumo atualizados |
| frontend → `db:5432` | **bloqueado** (`bad address 'db'`) |
| backend → `db:5432` | alcança |
| db → internet | **sem saída** (`internal: true`) |
| psql do pgAdmin → db (senha do `.env`) | autenticou e leu a tabela |
| `down` + `up` (sem `-v`) | dados persistiram |
| `stop backend` | `SIGTERM recebido, encerrando backend-lab14...` |
| Senha no `compose.yaml` | nenhuma |
| Imagens | `frontend-lab14` = 93,8 MB, `backend-lab14` = 194 MB |

**Login na UI do pgAdmin** (`localhost:8083`): verificar manualmente com as credenciais do `.env`.

---

## Bugs encontrados e corrigidos durante a validação

| Bug | Como apareceu | Correção |
|---|---|---|
| `POST` grande devolvia **502** em vez de 413 | `req.destroy()` fechava a conexão antes da resposta, e o Nginx via um reset | Drenar o corpo, descartar os dados e responder 413 no `end` |
| Acentos corrompidos (`Informa��o`) | `Invoke-RestMethod` com `-Body` em string envia ISO-8859-1 no PS 5.1 | A API rejeita corpo não-UTF-8 (400) e o README usa `charset=utf-8` + `UTF8.GetBytes` |
| Cabeçalhos de segurança ausentes em `/` (Lab 12) | O `add_header` dentro de um `location` anula os do `server` | O conjunto completo foi repetido em cada `location` |
| `procps` marcado como "instalado nesta sessão" | A imagem `ubuntu:24.04` já traz procps | O script e o README dizem "presente"; só o curl prova a camada gravável |

---

## Troubleshooting encontrado na prática (Windows)

| Sintoma | Causa | Solução |
|---|---|---|
| Docker Desktop: `initializing Inference manager ... remove ...\Docker\run\dockerInference: Não é possível o acesso ao arquivo` | Sockets Unix (AF_UNIX) órfãos depois de um encerramento abrupto do Docker Desktop; o Windows não deixa removê-los | Fechar o Docker Desktop, **renomear** `%LOCALAPPDATA%\Docker\run` e `%LOCALAPPDATA%\docker-secrets-engine`, rodar `wsl --shutdown` e reabrir. **Não** usar "Reset to factory defaults", que apaga imagens e volumes |
| `docker` trava sem resposta | O engine (distro WSL `docker-desktop`) está `Stopped` | `wsl -l -v`; ver os logs em `%LOCALAPPDATA%\Docker\log\host\` |
| `map has no entry for key "IPAddress"` | O Docker 29 removeu `.NetworkSettings.IPAddress` | `{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}` |
| `NativeCommandError` em vermelho | O PowerShell 5.1 trata o stderr de executáveis como erro | Leia a mensagem: `Syntax OK` e `test is successful` não são falhas |
| `400 corpo não está em UTF-8 válido` | `-Body` em string no PS 5.1 é enviado como ISO-8859-1 | `-ContentType "application/json; charset=utf-8" -Body ([Text.Encoding]::UTF8.GetBytes($json))` |
| `sh -c '... "$VAR" ...'` falha só no PowerShell | O PS 5.1 remove as aspas internas ao chamar executáveis nativos | Rode pelo Git Bash ou WSL, ou escape com `\"` |
| `< backup.sql` dá erro / `> backup.sql` fica ilegível | O PS não tem `<`, e o `>` grava em UTF-16 | `pg_dump -f /tmp/backup.sql` + `docker cp` |
| `password authentication failed` depois de editar o `.env` | O Postgres grava a senha no volume na 1ª subida | `docker compose down -v` (apaga os dados) ou volte a senha antiga |
| Seed ou `servers.json` alterado e nada muda | Os scripts de init só rodam com o volume vazio | `docker compose down -v` |
| `localhost:6000` não abre no navegador | A porta 6000 é bloqueada pelos navegadores (X11) | `curl.exe http://localhost:6000` |
| Healthcheck do Nginx `unhealthy` | `localhost` resolve para `::1`, e o config só escuta IPv4 | Use `http://127.0.0.1/` |
| Edit/Write grava `﻿`/`�` como caractere invisível | Ferramenta de edição converteu o escape | Confira com `grep -c $'\xEF\xBB\xBF'` e mantenha o escape explícito |

---

## Como reproduzir

```powershell
cd labs\14-projeto-integrador
if (-not (Test-Path .env)) { Copy-Item .env.example .env }   # só cria se não existir; edite as senhas
docker compose up -d --build --wait
```

Para limpar, use `docker compose down` (mantém os dados) ou `docker compose down -v` (remove os volumes).
