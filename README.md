# Docker e Docker Compose: 14 práticas resolvidas

**Aluno:** Nome do Aluno
**Disciplina:** Sistemas Distribuídos, Ciência da Computação, 8º período, Unifeg
**Avaliação:** P2, 2026B
**Base:** roteiro de laboratório do Prof. Paulo Dias (`docs/Roteiro_Docker_Aluno.docx`)

Este repositório traz a minha execução das 14 práticas do roteiro. Cada lab foi
rodado e validado no meu ambiente. Também apliquei boas práticas de produção onde
o objetivo do lab permitia, e documentei o que mudei e por quê.

- **Documentação acadêmica (Word):** [`docs/Documentacao_Projeto_Docker.docx`](docs/Documentacao_Projeto_Docker.docx)
- **Evidências e resultados medidos:** [`EVIDENCIAS.md`](EVIDENCIAS.md)
- **Documentação de cada prática:** `labs/<NN>-*/README.md`

## Ambiente usado

| Item | Versão |
|---|---|
| Sistema | Windows 11 + WSL2 |
| Docker Engine | 29.7.2 (Docker Desktop) |
| Docker Compose | v5.3.1 |
| Shell | PowerShell 5.1 (os comandos também vêm em bash) |

O roteiro exige Docker Engine 24 ou mais recente e Compose 2 ou mais recente.

## Mapa das práticas

| # | Prática | O que é demonstrado | Porta(s) |
|---|---|---|---|
| 01 | [Fundamentos](labs/01-fundamentos-containers/) | ciclo de vida: run, pause, logs, inspect, rm | — |
| 02 | [Ubuntu em container](labs/02-ubuntu-container/) | estado na camada gravável e isolamento de processos | — |
| 03 | [Dockerfile](labs/03-dockerfile-imagens/) | imagem própria não-root, com cache de camadas | 3000 |
| 04 | [Volumes](labs/04-volumes-bind-mounts/) | bind mount × named volume × tmpfs | — |
| 05 | [Redes](labs/05-redes-portas/) | DNS em rede customizada e publicação de portas | 8080 |
| 06 | [Apache](labs/06-apache/) | `httpd.conf` próprio montado por bind mount | 8080 |
| 07 | [Nginx](labs/07-nginx/) | proxy reverso para API interna | 8081 |
| 08 | [Compose](labs/08-compose-fundamentos/) | labs 06 e 07 num só `compose.yaml` com healthchecks | 8080, 8081 |
| 09 | [PostgreSQL](labs/09-postgresql/) | scripts de init, backup e persistência | 5432 |
| 10 | [pgAdmin](labs/10-pgadmin-compose/) | servidor pré-cadastrado via `servers.json` | 8082 |
| 11 | [Multi-stage](labs/11-multistage-build/) | 1,65 GB → 193 MB | 5000, 5001 |
| 12 | [React](labs/12-react-stack/) | hot reload em dev e Nginx em produção | 5173, 8090 |
| 13 | [Registry e segurança](labs/13-registry-seguranca/) | registry local, secrets e hardening | 5000, 6000 |
| 14 | [Projeto integrador](labs/14-projeto-integrador/) | React → API → Postgres + pgAdmin | 8084, 8083 |

As portas 5000 (labs 11 e 13) e 8080 (labs 05, 06 e 08) se repetem. Derrube um
lab antes de subir o próximo.

## Decisões que valem para todos os labs

1. **Nenhum segredo no git.** Os arquivos `.env` ficam fora do repositório (`.gitignore`)
   e só o `.env.example` é versionado. No Lab 14, o `compose.yaml` não contém nenhuma senha.
2. **LF forçado** (`.gitattributes`). Num clone no Windows, o CRLF quebra scripts `.sh`,
   SQLs de init e arquivos `.env` dentro dos containers Linux.
3. **Imagens próprias rodam como usuário sem privilégio.** Todas têm labels OCI, um
   `HEALTHCHECK` com ferramenta que existe na imagem (`wget`, nunca `curl` em
   alpine) e um `.dockerignore` que barra `node_modules`, `.git` e `.env*`.
4. **Ordem de subida garantida.** Uso `depends_on` com `condition: service_healthy`
   em vez de `depends_on` simples, que só espera o container iniciar.
5. **Os objetivos didáticos foram preservados.** O `Dockerfile.single-stage` (Lab 11)
   continua ingênuo, o Lab 02 roda como root e o segredo de exemplo do Lab 13
   continua versionado. Cada caso está justificado no README do respectivo lab.

## Como rodar qualquer prática

```powershell
cd labs\<NN>-<nome>
if (Test-Path .env.example) { if (-not (Test-Path .env)) { Copy-Item .env.example .env } }
docker compose up -d --build --wait      # labs com compose.yaml
```

Os labs sem `compose.yaml` usam `docker run` e `docker build`; os comandos estão no
README de cada um.

## Limpeza

```bash
docker compose down        # remove containers e rede; mantém volumes (dados)
docker compose down -v     # remove também os volumes nomeados (apaga dados)
docker image prune         # imagens sem tag que sobraram dos rebuilds
```
