# session-init

Skill de **protocolo permanente de sessão** para [Claude Code](https://claude.com/claude-code).

Invocada uma vez no começo da sessão, ela fixa regras que valem até o fim: como o agente **procura no que já foi resolvido antes de investigar**, como ele monta a **pilha de skills** de cada demanda, qual modelo usa, como se comunica, como escreve código e o que é obrigado a documentar antes de encerrar um assunto.

Sem ela, cada sessão recomeça do zero: o agente reabre um diagnóstico que já custou uma hora na semana passada, usa uma skill genérica onde cabiam quatro especializadas, escolhe modelo caro para tarefa trivial e joga fora o aprendizado no fim.

> **Novo na v2.0.0** — dois blocos que mudam o comportamento na raiz:
> **Consulta à Base de Conhecimento** (procurar antes de diagnosticar) e
> **Roteamento em pilha** (várias skills complementares por demanda, nunca uma só).

---

## O que ela faz

Ao ser invocada, imprime e ativa sete blocos:

| Bloco | Efeito no resto da sessão |
|---|---|
| **Roteadores** | Toda demanda passa por `/skill-router` (quais skills) e `/smart-router` (qual modelo) antes de executar |
| **Regras de comunicação** | Frases curtas, zero preâmbulo, zero recapitulação, direto ao resultado |
| **Consulta à Base de Conhecimento** | Antes de investigar qualquer problema, procurar no vault se ele já foi resolvido — e aplicar a nota em vez de reinvestigar |
| **Pilha de skills + catálogo** | O agente monta 2 a 5 skills complementares por demanda (base → refino → verificação → registro) e consulta o catálogo do vault em vez de inferir skills de memória |
| **Diretrizes Karpathy** | Pensar antes de codar, simplicidade primeiro, mudanças cirúrgicas, meta verificável |
| **Registro de Conhecimento** | Todo problema com causa raiz identificada vira nota no Obsidian (Caso + Playbook) antes de encerrar |
| **Processo Mestre** | Projeto com usuário externo, autenticação ou produção segue 9 fases com gate + 14 regras fixas de banco, aplicação e segurança |

### O ciclo que fecha

```
problema → busca no vault → achou? aplica a nota
                          → não achou? diagnostica → grava Caso + Playbook
                                                     ↑
                                        vira o "achou" da próxima vez
```

### Pilha, não skill única

Pedir "desenha essa página" e receber só `frontend-design` é entrega pela metade. A v2.0.0 obriga o roteador a montar a pilha inteira:

```
[skill-router] pilha: frontend-design → design-taste-frontend + high-end-visual-design
               → impeccable → responsiveness-check → ux-audit
```

Cada camada tem função: **base** constrói, **refino** lapida o que a base fez, **verificação** prova que presta, **registro** documenta.

---

## Instalação

### Opção A — plugin marketplace (recomendado)

Dentro do Claude Code:

```
/plugin marketplace add rafaelrabel0/claude-session-init
/plugin install session-init@rabelo-skills
```

Atualizar depois:

```
/plugin marketplace update rabelo-skills
```

### Opção B — cópia manual da skill

**macOS / Linux**

```bash
git clone https://github.com/rafaelrabel0/claude-session-init.git /tmp/csi
mkdir -p ~/.claude/skills
cp -r /tmp/csi/session-init/skills/session-init ~/.claude/skills/
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/rafaelrabel0/claude-session-init.git $env:TEMP\csi
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse "$env:TEMP\csi\session-init\skills\session-init" "$env:USERPROFILE\.claude\skills\"
```

Para deixar a skill válida só em um projeto, use `.claude/skills/` na raiz do projeto em vez de `~/.claude/skills/`.

Reinicie o Claude Code depois de copiar.

---

## Configuração (obrigatória)

A skill referencia um vault Obsidian por meio de placeholders. Abra
`~/.claude/skills/session-init/SKILL.md` e substitua:

| Placeholder | Substituir por |
|---|---|
| `<VAULT>` | caminho absoluto do seu vault (ex.: `C:/Users/voce/Documents/Obsidian Vault`) |
| `03 Conhecimento/` | o nome real da sua pasta de conhecimento, se for diferente |

A estrutura de pastas que a skill espera dentro do vault:

```
<VAULT>/
  00 Sistema/Skills Disponiveis.md          # catálogo das suas skills instaladas
  03 Conhecimento/
    Conhecimento.md                         # índice — nota não indexada é nota perdida
    Casos/                                  # o que aconteceu daquela vez
    Playbooks/                              # o método para a classe do problema
    Processos/                              # processos que regem a entrega
```

Não usa Obsidian? Aponte para qualquer pasta de notas em Markdown, ou apague os passos **3**, **4** e **6** do `SKILL.md` — o resto do protocolo funciona sozinho.

---

## Uso

```
/session-init
```

Também responde a: "inicializar sessão", "ativar regras", "carregar protocolo", "começar sessão".

Ela **não** dispara sozinha — é sempre invocação explícita, para não gastar contexto em sessões curtas.

---

## Skills que o protocolo usa

A `session-init` é um **protocolo**, não um pacote: ela decide *quando* e *em que ordem* usar skills que vivem em outros repositórios. Nada aqui é instalado junto.

**Ela funciona sem nenhuma delas** — o agente aplica os princípios por conta própria, só sem as heurísticas detalhadas de cada skill. Quanto mais da lista você tiver, mais fundo a pilha vai.

### Núcleo — instale estas primeiro

Sem estas três, as pilhas ficam rasas e o roteamento vira improviso.

| Skill | Tipo | O que faz | Onde conseguir |
|---|---|---|---|
| `/smart-router` | slash command | escolhe o modelo mais econômico (Haiku / Sonnet / Opus) por tarefa | [rafaelrabel0/smart-router](https://github.com/rafaelrabel0/smart-router) |
| `/skill-router` | slash command | monta a pilha de skills da demanda | não publicado — veja **Montar seu próprio skill-router** abaixo |
| `karpathy-guidelines` | skill | detalhes e exemplos das 4 diretrizes de engenharia | [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) |

Instalar o `smart-router`: copie `skill/smart-router.md` do repositório para
`~/.claude/commands/smart-router.md` (ou `.claude/commands/` na raiz do projeto).

### Camada de design — a pilha visual

A pilha de UI é o caso onde usar uma skill só mais dói. Estas são as que o `SKILL.md` cita nominalmente:

| Skill | Camada | Papel na pilha |
|---|---|---|
| `frontend-design` | base | constrói a interface production-grade |
| `design-taste-frontend` | refino | aplica gosto refinado ao código já escrito |
| `high-end-visual-design` | refino | eleva ao padrão de agência (tipografia, sombra, ritmo) |
| `emil-design-eng` | refino | polish, micro-interações, animação |
| `impeccable` | refino | audit e melhoria ampla de UI — a lapidação final |
| `minimalist-ui` / `industrial-brutalist-ui` | refino | direção estética específica, quando houver |
| `image-to-code` | base | reproduz mockup ou screenshot em código |
| `redesign-existing-projects` | base | moderniza UI legada |
| `web-design-guidelines` | verificação | confere padrões de web |
| `responsiveness-check` | verificação | valida breakpoints |
| `ux-audit` | verificação | navega como usuário e acha fricção |

Essas skills de "taste" vêm de pacotes da comunidade, instaláveis com o CLI do [skills.sh](https://skills.sh):

```bash
npx skills add <nome-da-skill>
```

Busque cada nome em [skills.sh](https://skills.sh) para pegar o pacote correto — os slugs mudam conforme o autor republica.

### Camada de código e entrega

| Skill | Camada | Papel na pilha |
|---|---|---|
| `senior-architect` | base | decide arquitetura e trade-offs antes de codar |
| `senior-frontend` / `senior-backend` / `senior-fullstack` | base | implementação |
| `senior-devops` | base | CI/CD, container, deploy |
| `senior-security` | verificação | auditoria, threat model — camada obrigatória no Processo Mestre |
| `generate-rest-api`, `build-auth-system`, `implement-caching`, `implement-error-handling` | base | blocos prontos de backend |
| `supabase-postgres` | base | schema, RLS, performance de Postgres |
| `code-reviewer` | verificação | review antes de fechar |
| `simplify` | refino | tira o excesso do que acabou de ser escrito |
| `vitest` / `webapp-testing` | verificação | testes unitários e de browser |
| `project-health` | verificação | auditoria de projeto inteiro |
| `release` / `changelog-generator` | registro | publicação e release notes |
| `obsidian` | registro | grava Caso, Playbook e status do projeto no vault |

A maioria destas é distribuída como slash command em `~/.claude/commands/`. Use `/find-skills` dentro do Claude Code para localizá-las, ou escreva as suas com `/skill-creator`.

### Complementares citadas nas pilhas

`copywriting`, `landing-page`, `seo-local-business`, `brandkit`, `color-palette`, `icon-set-generator`, `favicon-gen`, `ui-design-system`, `shadcn-ui`, `ai-agent-create`, `claude-api`, `ai-agents-test`, `n8n-builder`, `compliance-lgpd`, `project-docs`, `generate-api-docs`.

Nenhuma é obrigatória. Cada uma que faltar apenas encurta a pilha do seu tema.

---

## Como instalar skills no Claude Code

Quatro caminhos, do mais simples ao mais manual:

**1. Plugin marketplace** — para pacotes com vários commands/skills:

```
/plugin marketplace add <usuario>/<repo>
/plugin install <plugin>@<marketplace>
/plugin marketplace update <marketplace>
```

**2. CLI do skills.sh** — para skills soltas da comunidade:

```bash
npx skills add <nome-da-skill>
```

**3. Descoberta assistida** — quando você não sabe o nome:

```
/find-skills preciso de algo que audite acessibilidade de UI
```

O Claude procura, lista candidatas e instala a que você escolher.

**4. Cópia manual** — sempre funciona:

- **Skill** → pasta com `SKILL.md` em `~/.claude/skills/<nome>/`
- **Slash command** → arquivo único em `~/.claude/commands/<nome>.md`
- Escopo de projeto: as mesmas pastas sob `.claude/` na raiz do repositório

Reinicie o Claude Code e confirme com `/` na linha de comando.

### Depois de instalar: registre no catálogo

O passo 4 do protocolo manda o agente ler `<VAULT>/00 Sistema/Skills Disponiveis.md` antes de rotear. **Skill instalada e não catalogada é skill que nunca entra na pilha.** Ao instalar qualquer coisa, adicione uma linha lá: nome, o que faz, quando usar, de onde veio e a data.

### Montar seu próprio skill-router

O `/skill-router` é um slash command simples e local — um `~/.claude/commands/skill-router.md` contendo:

1. Uma tabela com as suas skills por categoria e o gatilho de cada uma
2. As quatro camadas da pilha (base, refino, verificação, registro)
3. A regra central: **nunca parar na primeira skill que deu match** — depois da base, perguntar quem refina e quem verifica
4. O formato de saída em uma linha: `[skill-router] pilha: a → b → c — motivo`

As pilhas de referência da seção "Roteamento em pilha de skills" do `SKILL.md` servem de ponto de partida.

---

## Recomendado: espelhar as regras no CLAUDE.md

A skill só vale a partir do momento em que é invocada. Para que as regras valham desde a primeira mensagem, copie os blocos que te interessam para o seu `~/.claude/CLAUDE.md`. A skill passa então a ser o **lembrete explícito** do protocolo, não a única fonte dele.

Os dois blocos que mais compensam espelhar são a **Consulta à Base de Conhecimento** e o **Registro de Conhecimento** — juntos, eles são o ciclo que faz a sessão de amanhã começar onde a de hoje parou.

---

## Estrutura do repositório

```
.claude-plugin/marketplace.json          # catálogo do marketplace
session-init/
  .claude-plugin/plugin.json             # manifesto do plugin
  skills/session-init/SKILL.md           # a skill
```

## Licença

MIT — veja [LICENSE](LICENSE).
