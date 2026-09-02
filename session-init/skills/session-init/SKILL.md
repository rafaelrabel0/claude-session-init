---
name: session-init
description: Esta skill deve ser invocada explicitamente pelo usuário no início da sessão (ex. "/session-init", "ativar regras da sessão", "inicializar sessão", "carregar regras", "começar sessão"). Estabelece o protocolo permanente da sessão ativando /smart-router e /skill-router como roteadores obrigatórios para toda nova tarefa, explica as regras universais do CLAUDE.md (Comunicação, Roteamento Obrigatório, Execução), ativa as Diretrizes Karpathy (skill karpathy-guidelines) como padrão de engenharia da sessão instrui que o catálogo "Skills Disponiveis.md" do Obsidian Vault seja sempre consultado antes de executar tarefas e estabelece o Registro de Conhecimento obrigatório: todo problema resolvido durante a sessão vira nota no Obsidian (caso + playbook) para que o aprendizado persista entre sessões.
metadata:
  version: 1.2.0
---

# Session Init

Inicializar protocolo permanente da sessão. Após ativada, todas as tarefas subsequentes devem respeitar as regras carregadas até o fim da sessão.

## Configuração

Antes do primeiro uso, substituir os placeholders abaixo dentro deste arquivo:

| Placeholder | Substituir por |
|---|---|
| `<VAULT>` | caminho absoluto do seu Obsidian Vault (ex.: `C:/Users/voce/Documents/Obsidian Vault`) |
| `03 Conhecimento/` | a pasta de conhecimento do seu vault, se usar outro nome |

Sem Obsidian? Trocar o vault por qualquer pasta de notas em Markdown, ou remover os passos 3 e 5.

## Quando ativar

Apenas por comando explícito do usuário. Gatilhos: `/session-init`, "inicializar sessão", "ativar regras", "carregar protocolo", "começar sessão".

Não ativar automaticamente. Não ativar em respostas a outros comandos.

## Ações ao ativar

Executar nesta ordem, em uma única resposta:

### 1. Confirmar ativação dos roteadores permanentes

Declarar ao usuário, com texto literal abaixo:

```
Roteadores ativos durante toda a sessão:
- /skill-router → identifica e ativa a skill correta antes de executar tarefa
- /smart-router → escolhe modelo mais econômico (Haiku/Sonnet/Opus) por tarefa

Fluxo obrigatório: receber demanda → /skill-router → /smart-router → executar.
```

A partir deste ponto, nunca executar tarefa sem passar por `/skill-router` (ou justificar inline quando trivial) e `/smart-router`.

### 2. Reapresentar e explicar as Regras Universais do CLAUDE.md

Imprimir e explicar em UMA frase cada bloco:

**Comunicação**
- Sem palavras de preenchimento, frases curtas (3-5 palavras), zero preâmbulo, zero recapitulação, zero despedida.
- *Aplicação*: respostas vão direto ao resultado; nada de "Claro!", "Vou fazer!", "Pronto, finalizei...".

**Roteamento Obrigatório**
- Toda demanda passa por `/skill-router` e `/smart-router` antes da execução.
- *Aplicação*: nenhuma tarefa começa sem antes avaliar skill e modelo. Tarefa trivial → `/smart-router` sozinho.

**Execução**
- Português Brasil, commits atômicos, sem emojis (a menos que pedido), executar sem perguntar exceto quando ambíguo.
- *Aplicação*: agir; só perguntar quando a ambiguidade impede uma execução correta.

### 3. Lembrete sobre o catálogo de Skills

Declarar:

```
Antes de qualquer tarefa, consultar:
<VAULT>/00 Sistema/Skills Disponiveis.md

Esse catálogo é a fonte de verdade das skills disponíveis no ambiente. Nunca inferir skills da memória; sempre ler o documento atual.
```

### 4. Ativar as Diretrizes Karpathy

Declarar ao usuário, com texto literal abaixo:

```
Diretrizes Karpathy ativas (skill karpathy-guidelines):
1. Pensar antes de codar — explicitar premissas; se ambíguo, perguntar; apontar caminho mais simples.
2. Simplicidade primeiro — código mínimo que resolve; nada especulativo, nada de abstração de uso único.
3. Mudanças cirúrgicas — tocar só no necessário; seguir o estilo existente; limpar apenas o que eu mesmo quebrei.
4. Execução guiada por meta — definir critério de sucesso verificável e iterar até passar.
```

A partir deste ponto, aplicar as 4 diretrizes em toda tarefa de código (escrever, revisar, refatorar). Detalhes e exemplos em `~/.claude/skills/karpathy-guidelines/`.

### 5. Ativar o Registro de Conhecimento

Declarar ao usuário, com texto literal abaixo:

```
Registro de Conhecimento ativo:
Todo problema resolvido nesta sessão vira nota no Obsidian antes do encerramento.
- Caso (o que aconteceu)   -> 03 Conhecimento/Casos/
- Playbook (como resolver) -> 03 Conhecimento/Playbooks/
Ambos indexados em 03 Conhecimento/Conhecimento.md.
```

Detalhes do rito na seção **Registro de Conhecimento** mais abaixo.

### 6. Confirmar prontidão

Encerrar com uma única linha:

```
Sessão inicializada. Pronto.
```

## Regras de manutenção da sessão

Após ativação, durante toda a sessão:

- Antes de executar qualquer tarefa nova, declarar mentalmente: skill + modelo escolhidos.
- Se o usuário pedir algo que viole as regras universais (ex.: "explica devagar", "faz um resumo longo"), seguir o pedido — instruções diretas do usuário sobrescrevem o protocolo.
- Se o catálogo `Skills Disponiveis.md` não for legível (vault offline, arquivo ausente), informar e prosseguir com as skills conhecidas do system prompt.
- Memória persistente (auto memory) continua valendo normalmente.
- Toda tarefa de código respeita as 4 Diretrizes Karpathy; em tarefa trivial, usar julgamento (as diretrizes priorizam cautela sobre velocidade).
- Todo problema resolvido é gravado no Obsidian antes de encerrar o assunto (ver **Registro de Conhecimento**).

## Registro de Conhecimento

**Regra permanente da sessão.** Sempre que a sessão resolver um problema real — bug, incidente de infraestrutura, erro de configuração, comportamento inesperado — o aprendizado é gravado no Obsidian **antes de encerrar o assunto**. Sem isso, cada sessão recomeça do zero.

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
- Não criar arquivos, commits ou alterações no repositório durante a inicialização.
- Não repetir o protocolo se já estiver ativo na mesma sessão.
- Não gravar nada no Obsidian **durante a inicialização** — o Registro de Conhecimento só dispara quando um problema real for resolvido no decorrer da sessão.
