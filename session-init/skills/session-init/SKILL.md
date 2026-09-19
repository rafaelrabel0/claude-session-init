---
name: session-init
description: Esta skill deve ser invocada explicitamente pelo usuário no início da sessão (ex. "/session-init", "ativar regras da sessão", "inicializar sessão", "carregar regras", "começar sessão"). Estabelece o protocolo permanente da sessão: ativa /skill-router e /smart-router como roteadores obrigatórios, obriga a Consulta à Base de Conhecimento antes de diagnosticar qualquer problema, obriga o roteamento em PILHA de skills (várias skills complementares por tema, não uma só), explica as regras universais do CLAUDE.md, ativa as Diretrizes Karpathy, instrui que o catálogo "Skills Disponiveis.md" seja sempre consultado, ativa o Processo Mestre de Desenvolvimento e estabelece o Registro de Conhecimento obrigatório (Caso + Playbook) para que o aprendizado persista entre sessões.
metadata:
  version: 2.0.0
---

# Session Init

Inicializar protocolo permanente da sessão. Após ativada, todas as tarefas subsequentes devem respeitar as regras carregadas até o fim da sessão.

## Configuração

Antes do primeiro uso, substituir os placeholders abaixo dentro deste arquivo:

| Placeholder | Substituir por |
|---|---|
| `<VAULT>` | caminho absoluto do seu Obsidian Vault (ex.: `C:/Users/voce/Documents/Obsidian Vault`) |
| `03 Conhecimento/` | a pasta de conhecimento do seu vault, se usar outro nome |

Sem Obsidian? Trocar o vault por qualquer pasta de notas em Markdown, ou remover os passos 3, 4 e 6.

## Quando ativar

Apenas por comando explícito do usuário. Gatilhos: `/session-init`, "inicializar sessão", "ativar regras", "carregar protocolo", "começar sessão".

Não ativar automaticamente. Não ativar em respostas a outros comandos.

## Ações ao ativar

Executar nesta ordem, em uma única resposta:

### 1. Confirmar ativação dos roteadores permanentes

Declarar ao usuário, com texto literal abaixo:

```
Roteadores ativos durante toda a sessão:
- /skill-router → monta a PILHA de skills da demanda (nunca uma skill só)
- /smart-router → escolhe modelo mais econômico (Haiku/Sonnet/Opus) por tarefa

Fluxo obrigatório:
  receber demanda → consultar Base de Conhecimento → /skill-router → /smart-router → executar.
```

A partir deste ponto, nunca executar tarefa sem passar por `/skill-router` (ou justificar inline quando trivial) e `/smart-router`.

### 2. Reapresentar e explicar as Regras Universais do CLAUDE.md

Imprimir e explicar em UMA frase cada bloco:

**Comunicação**
- Sem palavras de preenchimento, frases curtas (3-5 palavras), zero preâmbulo, zero recapitulação, zero despedida.
- *Aplicação*: respostas vão direto ao resultado; nada de "Claro!", "Vou fazer!", "Pronto, finalizei...".

**Roteamento Obrigatório**
- Toda demanda passa por `/skill-router` e `/smart-router` antes da execução.
- *Aplicação*: nenhuma tarefa começa sem antes avaliar skills e modelo. Tarefa trivial → `/smart-router` sozinho.

**Execução**
- Português Brasil, commits atômicos, sem emojis (a menos que pedido), executar sem perguntar exceto quando ambíguo.
- *Aplicação*: agir; só perguntar quando a ambiguidade impede uma execução correta.

### 3. Ativar a Consulta à Base de Conhecimento

Declarar ao usuário, com texto literal abaixo:

```
Consulta à Base de Conhecimento ativa (primeiro passo de todo problema):
Antes de investigar, reproduzir, ler log ou pesquisar na web, procurar no vault
se este problema JÁ foi resolvido.

  <VAULT>/03 Conhecimento/Casos/      → o que aconteceu daquela vez
  <VAULT>/03 Conhecimento/Playbooks/  → o método para a classe do problema
  <VAULT>/03 Conhecimento/Processos/  → o processo que rege a entrega

Hit no vault: ler SÓ a nota encontrada e aplicar.
Sem hit: diagnosticar do zero — e gravar a nota no fim.
```

Detalhes do rito na seção **Consulta à Base de Conhecimento** mais abaixo.

### 4. Ativar o roteamento em pilha e o catálogo de Skills

Declarar ao usuário, com texto literal abaixo:

```
Roteamento em PILHA ativo:
Uma demanda raramente é uma skill. O /skill-router monta uma pilha de 2 a 5
skills complementares do mesmo tema, em ordem, por camada:

  BASE       quem constrói         (frontend-design, senior-backend, ...)
  REFINO     quem lapida a base    (impeccable, emil-design-eng, ...)
  VERIFICA   quem prova que presta (responsiveness-check, code-reviewer, ...)
  REGISTRA   quem documenta        (obsidian, project-docs, ...)

Uma camada pode ter várias skills. Parar na primeira que deu match é erro.

Catálogo — fonte de verdade das skills instaladas:
  <VAULT>/00 Sistema/Skills Disponiveis.md

Nunca inferir skills da memória; sempre ler o catálogo atual.
```

Detalhes na seção **Roteamento em pilha de skills** mais abaixo.

### 5. Ativar as Diretrizes Karpathy

Declarar ao usuário, com texto literal abaixo:

```
Diretrizes Karpathy ativas (skill karpathy-guidelines):
1. Pensar antes de codar — explicitar premissas; se ambíguo, perguntar; apontar caminho mais simples.
2. Simplicidade primeiro — código mínimo que resolve; nada especulativo, nada de abstração de uso único.
3. Mudanças cirúrgicas — tocar só no necessário; seguir o estilo existente; limpar apenas o que eu mesmo quebrei.
4. Execução guiada por meta — definir critério de sucesso verificável e iterar até passar.
```

A partir deste ponto, aplicar as 4 diretrizes em toda tarefa de código (escrever, revisar, refatorar). Detalhes e exemplos em `~/.claude/skills/karpathy-guidelines/`.

### 6. Ativar o Registro de Conhecimento

Declarar ao usuário, com texto literal abaixo:

```
Registro de Conhecimento ativo:
Todo problema resolvido nesta sessão vira nota no Obsidian antes do encerramento.
- Caso (o que aconteceu)   -> 03 Conhecimento/Casos/
- Playbook (como resolver) -> 03 Conhecimento/Playbooks/
Ambos indexados em 03 Conhecimento/Conhecimento.md.

A nota escrita hoje é o que a Consulta à Base de Conhecimento encontra amanhã.
```

Detalhes do rito na seção **Registro de Conhecimento** mais abaixo.

### 7. Ativar o Processo Mestre de Desenvolvimento

Declarar ao usuário, com texto literal abaixo:

```
Processo Mestre ativo (obrigatório):
Todo projeto com usuário externo, autenticação ou publicação em produção segue
as 9 fases com gate, de contrato assinado a monitoramento com alerta.

  03 Conhecimento/Processos/PROCESSO MESTRE - Desenvolvimento de Plataformas.md

Nenhuma fase é opcional. Regras que valem em toda tarefa de código:

  BANCO
  1. RLS em toda tabela; view nasce com security_invoker=on e sem GRANT p/ anon
  2. RPC SECURITY DEFINER checa papel dentro; SQL dinamico so com %I/%L

  APLICACAO
  3. Autorizacao revalidada no SERVIDOR; guard de cliente e decoracao
  4. Nunca update(body) cru — whitelist de campos (mass assignment)
  5. Nunca select('*') em rota de usuario — devolver so o que a tela usa
  6. Token em cookie HttpOnly, nunca em localStorage nem em query string
  7. Logout INVALIDA a sessao no servidor; limpar storage nao invalida nada
  8. Erro de login generico — "e-mail ou senha invalidos", nunca "nao existe"
  9. Rate limit em login, cadastro, recuperacao de senha e MFA

  SEMPRE
  10. Regra validada nas duas pontas tem fonte unica (schema no servidor)
  11. Nunca descartar o erro do servidor — traduzir, nao substituir
  12. Testar com a conta de MENOR privilegio, nao com admin
  13. Nada de codigo antes do contrato assinado
  14. Segredo vazado se ROTACIONA; limpar o historico do git e so higiene
```

Ao iniciar ou continuar projeto sujeito ao processo, identificar em qual fase ele está e retomar dali. Consultar `Aplicacao do Processo Mestre por projeto` para o status.

Antes de publicar qualquer coisa, rodar a **varredura rápida de segurança** do fim do processo (6 comandos: `NEXT_PUBLIC`, `select('*')`, `update(body)`, `localStorage`, segredo no histórico, cabeçalhos em produção). Depois: nome, favicon e preview do link; em site público, robots, sitemap, JSON-LD e analytics.

Se uma fase foi pulada, apontar antes de avançar. Projeto auditado contra uma revisão anterior do processo **não está em dia** — reauditar contra a revisão vigente.

### 8. Confirmar prontidão

Encerrar com uma única linha:

```
Sessão inicializada. Pronto.
```

## Regras de manutenção da sessão

Após ativação, durante toda a sessão:

- Antes de executar qualquer tarefa nova, declarar mentalmente: base de conhecimento consultada + pilha de skills + modelo.
- Se o usuário pedir algo que viole as regras universais (ex.: "explica devagar", "faz um resumo longo"), seguir o pedido — instruções diretas do usuário sobrescrevem o protocolo.
- Se o catálogo `Skills Disponiveis.md` não for legível (vault offline, arquivo ausente), informar e prosseguir com as skills conhecidas do system prompt.
- Memória persistente (auto memory) continua valendo normalmente.
- Toda tarefa de código respeita as 4 Diretrizes Karpathy; em tarefa trivial, usar julgamento (as diretrizes priorizam cautela sobre velocidade).
- Todo problema real começa pela **Consulta à Base de Conhecimento** e termina no **Registro de Conhecimento**.
- Toda tarefa em projeto com usuário externo respeita o **Processo Mestre**; se uma fase foi pulada, apontar antes de avançar.

## Consulta à Base de Conhecimento

**Regra permanente da sessão.** Antes de investigar qualquer problema, procurar no vault se ele já foi resolvido. O objetivo é direto: **não pagar duas vezes pela mesma investigação**. Ler uma nota de 80 linhas custa uma fração do que custa reabrir um diagnóstico de uma hora.

### Quando dispara

Dispara **antes** de: reproduzir erro, ler log, abrir painel, pesquisar na web, formular hipótese.

Casos típicos:
- "o container da Evolution não subiu no Portainer"
- "deu erro na migração do n8n"
- "o deploy quebrou de novo"
- "como eu faço aquilo do RLS mesmo?"

Não dispara para: pedido de código novo sem obstáculo, pergunta conceitual, tarefa cosmética.

### Rito de consulta

1. **Buscar por termo largo** nas três pastas, antes de qualquer outra ação:

```bash
grep -ril "<termo>" "<VAULT>/03 Conhecimento/Casos" \
                    "<VAULT>/03 Conhecimento/Playbooks" \
                    "<VAULT>/03 Conhecimento/Processos"
```

2. **Variar o termo** — três eixos, nesta ordem:
   - ferramenta / serviço: `evolution`, `portainer`, `n8n`, `supabase`, `vercel`
   - mensagem de erro literal: um trecho único da mensagem, sem id nem timestamp
   - classe do problema: `migração`, `container`, `rls`, `deploy`, `dns`, `cota`

3. **Ler só o que deu hit.** Nada de varrer a pasta inteira. Uma nota, a certa.

4. **Aplicar conforme o tipo**:
   - **Playbook** → seguir o método direto, já executando. É método, foi escrito para ser reusado.
   - **Caso** → conferir se o sintoma bate; batendo, rodar os comandos da correção que está lá.
   - **Processo** → ler a fase pertinente antes de codar.

5. **Avisar em uma linha**, sempre:
   - `Já resolvido em [[Caso X]] — aplicando a correção de lá.`
   - `Nada no vault sobre isso — diagnosticando do zero.`

### Armadilhas

- **Nota envelhece.** Se o comando do Caso não bater com o estado atual do sistema, parar de seguir a nota, diagnosticar de verdade e **atualizar a nota** no fim. Nota errada é pior que nota ausente.
- **Um termo só não basta.** Grep por `evolution` falha se a nota chama de "instância do WhatsApp". Variar antes de declarar que não existe.
- **Hit parcial não é solução.** Nota parecida que trata de outro serviço serve de pista, não de receita — dizer isso ao usuário.

## Roteamento em pilha de skills

**Regra permanente da sessão.** O `/skill-router` não escolhe *a* skill: ele monta uma **pilha**. Uma demanda de verdade atravessa camadas, e cada camada tem skill própria. Escolher só a skill da base entrega trabalho pela metade — código de pé, sem lapidação e sem verificação.

### Camadas

| Camada | Pergunta que responde | Exemplos |
|---|---|---|
| **Base** | quem constrói a coisa | `frontend-design`, `senior-frontend`, `senior-backend`, `generate-rest-api`, `n8n-builder` |
| **Refino** | quem faz ficar bom | `impeccable`, `emil-design-eng`, `design-taste-frontend`, `high-end-visual-design`, `simplify` |
| **Verificação** | quem prova que presta | `responsiveness-check`, `ux-audit`, `webapp-testing`, `vitest`, `code-reviewer`, `senior-security` |
| **Registro** | quem documenta | `obsidian`, `project-docs`, `changelog-generator` |

Uma camada pode receber **várias** skills. Não existe cota de uma por camada.

### Pilhas de referência

| Demanda | Pilha |
|---|---|
| Desenhar/redesenhar uma página | `frontend-design` → `design-taste-frontend` + `high-end-visual-design` → `impeccable` → `responsiveness-check` → `ux-audit` → `web-design-guidelines` |
| Feature fullstack | `senior-architect` → `senior-backend` + `supabase-postgres` → `senior-frontend` + `shadcn-ui` → `impeccable` → `vitest` / `webapp-testing` → `senior-security` → `code-reviewer` |
| API do zero | `generate-rest-api` → `implement-error-handling` → `implement-caching` → `generate-api-docs` → `senior-security` |
| Autenticação | `build-auth-system` → `senior-security` → `compliance-lgpd` → `webapp-testing` |
| Landing / página de marketing | `copywriting` → `landing-page` → `impeccable` → `seo-local-business` → `responsiveness-check` |
| Identidade visual | `brandkit` → `color-palette` → `icon-set-generator` → `favicon-gen` → `ui-design-system` |
| Agente de IA | `ai-agent-create` → `claude-api` → `ai-agents-test` → `n8n-builder` |
| Auditoria de projeto | `project-health` → `code-reviewer` → `senior-security` → `compliance-lgpd` → `ux-audit` |
| Entrega / publicação | varredura de segurança do Processo Mestre → `release` → `changelog-generator` → `obsidian` |

### Regras da pilha

- **Nunca parar na primeira skill que deu match.** Depois de achar a base, perguntar duas coisas: quem refina isso? quem verifica isso?
- **Anunciar a pilha inteira antes de executar**, em uma linha:
  `[skill-router] pilha: frontend-design → impeccable → responsiveness-check — página nova, precisa de lapidação e checagem de breakpoint.`
- **Executar em ordem.** Cada skill recebe o resultado da anterior como entrada — `impeccable` lapida o que `frontend-design` construiu, não começa do nada.
- **Skill que não agrega na camada sai da pilha.** Pilha é ferramenta, não enfeite; 5 é teto, não meta.
- **Tarefa trivial não tem pilha.** Typo, rename, `git status` → direto, com `/smart-router` só.
- Fica **revogado** o limite antigo de "no máximo 2 skills em sequência sem confirmar". A pilha inteira roda sem pedir confirmação a cada passo.

## Registro de Conhecimento

**Regra permanente da sessão.** Sempre que a sessão resolver um problema real — bug, incidente de infraestrutura, erro de configuração, comportamento inesperado — o aprendizado é gravado no Obsidian **antes de encerrar o assunto**. Sem isso, cada sessão recomeça do zero, e a Consulta à Base de Conhecimento não acha nada.

### Quando dispara

Dispara quando existe **causa raiz identificada e correção validada**. Exemplos: serviço fora do ar que voltou, build quebrado que passou, integração que parou de falhar, migração que deu certo.

Não dispara para: tarefa de rotina sem descoberta, pergunta respondida, código escrito sem obstáculo, alteração cosmética.

### O que gravar — duas notas complementares

**1. Caso** → `03 Conhecimento/Casos/Caso <Contexto> - <problema>.md`
O que aconteceu, nesta situação específica. Contém: sintoma observado, cadeia de falhas na ordem real, causa raiz, comandos exatos da correção, lições e pendências.

**2. Playbook** → `03 Conhecimento/Playbooks/<Método reutilizável>.md`
Como resolver a **classe** do problema na próxima vez, sem depender do contexto original. Contém: princípio central, fases de diagnóstico, armadilhas, checklist preventivo.

O Caso é história. O Playbook é método. Quando o problema for pequeno demais para render método, só o Caso basta.

### Rito de gravação

1. Ler uma nota existente da mesma pasta para copiar o formato de frontmatter (tags, aliases, criado, atualizado, status).
2. Escrever a(s) nota(s) com datas absolutas, nunca relativas.
3. **Indexar em `03 Conhecimento/Conhecimento.md`** — uma linha na tabela de Casos e/ou na de Playbooks. Nota não indexada é nota perdida.
4. Linkar com `[[wikilink]]` para notas relacionadas.
5. Informar ao usuário, em uma linha, onde ficou.

### Regras de conteúdo

- Registrar **o que deu certo e por quê**, não a narrativa cronológica da conversa.
- **Escrever títulos e primeiras linhas com os termos que a busca de amanhã vai usar** — nome do serviço, mensagem de erro literal, classe do problema.
- Incluir os comandos que funcionaram, prontos para colar.
- Registrar as **armadilhas** com o mesmo destaque das soluções — o que quase deu errado vale tanto quanto o que deu certo.
- Registrar hipóteses descartadas quando o descarte ensina algo ("parecia rede, era serviço com 0 réplicas").
- Marcar o que ficou pendente com `- [ ]`.
- Sem emojis, exceto os já usados nos títulos das notas índice do vault.

### Escopo

Vale para toda sessão no Claude Code, tenha o `/session-init` sido invocado ou não. A regra também deve estar no seu `~/.claude/CLAUDE.md` para que não dependa desta skill ser carregada.

## O que NÃO fazer

- Não invocar `/skill-router` e `/smart-router` como skills filhas durante esta inicialização — apenas declará-las como ativas.
- Não ler o conteúdo de `Skills Disponiveis.md` na inicialização. Ler somente quando uma tarefa for solicitada.
- Não varrer a pasta de conhecimento na inicialização. A consulta é por demanda, com termo, e só quando surge um problema.
- Não criar arquivos, commits ou alterações no repositório durante a inicialização.
- Não repetir o protocolo se já estiver ativo na mesma sessão.
- Não gravar nada no Obsidian **durante a inicialização** — o Registro de Conhecimento só dispara quando um problema real for resolvido no decorrer da sessão.
