# session-init

Skill de **protocolo permanente de sessão** para [Claude Code](https://claude.com/claude-code).

Invocada uma vez no começo da sessão, ela fixa regras que valem até o fim: como o agente escolhe skill e modelo, como se comunica, como escreve código e o que ele é obrigado a documentar antes de encerrar um assunto.

Sem ela, cada sessão recomeça do zero: o agente escolhe modelo caro para tarefa trivial, escreve resposta longa quando bastava uma linha e joga fora o diagnóstico que levou uma hora para achar.

---

## O que ela faz

Ao ser invocada, imprime e ativa cinco blocos:

| Bloco | Efeito no resto da sessão |
|---|---|
| **Roteadores** | Toda demanda passa por `/skill-router` (qual skill) e `/smart-router` (qual modelo) antes de executar |
| **Regras de comunicação** | Frases curtas, zero preâmbulo, zero recapitulação, direto ao resultado |
| **Catálogo de skills** | O agente consulta o catálogo do vault em vez de inferir skills de memória |
| **Diretrizes Karpathy** | Pensar antes de codar, simplicidade primeiro, mudanças cirúrgicas, meta verificável |
| **Registro de Conhecimento** | Todo problema com causa raiz identificada vira nota no Obsidian (Caso + Playbook) antes de encerrar |

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

Não usa Obsidian? Aponte para qualquer pasta de notas em Markdown, ou apague os passos **3** e **5** do `SKILL.md` — o resto do protocolo funciona sozinho.

---

## Uso

```
/session-init
```

Também responde a: "inicializar sessão", "ativar regras", "carregar protocolo", "começar sessão".

Ela **não** dispara sozinha — é sempre invocação explícita, para não gastar contexto em sessões curtas.

---

## Dependências opcionais

O protocolo cita três skills que **não** vêm neste repositório:

- `/skill-router` — escolhe a skill certa para a demanda
- `/smart-router` — escolhe o modelo mais econômico (Haiku / Sonnet / Opus)
- `karpathy-guidelines` — detalhes e exemplos das 4 diretrizes

Sem elas, a skill continua funcionando: o agente aplica os princípios de roteamento e engenharia por conta própria, apenas sem as heurísticas detalhadas de cada uma. Se quiser o comportamento completo, crie skills com esses nomes ou remova as menções do `SKILL.md`.

---

## Recomendado: espelhar as regras no CLAUDE.md

A skill só vale a partir do momento em que é invocada. Para que as regras valham desde a primeira mensagem, copie os blocos que te interessam para o seu `~/.claude/CLAUDE.md`. A skill passa então a ser o **lembrete explícito** do protocolo, não a única fonte dele.

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
