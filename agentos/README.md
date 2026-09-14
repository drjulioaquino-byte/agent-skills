# AgentOS v1

Arquitetura-base reutilizável para projetos de agentes.

## Princípio central

Separe quatro coisas:

- **Agent**: decide, julga, planeja ou valida.
- **Skill**: competência reutilizável carregada sob demanda.
- **Tool/MCP**: executa ação determinística em sistema externo.
- **Policy**: limita o que agentes e ferramentas podem fazer.

O objetivo não é manter muitos agentes ativos. O núcleo permanente deve ser pequeno e os especialistas devem ser efêmeros.

## Núcleo

1. **Guardian** — autoridade constitucional, classificação de risco, veto e aprovação.
2. **Orchestrator** — decompõe a missão, seleciona agentes/skills, coordena execução e encerra o fluxo.
3. **Observer** — registra traces, decisões, custo, chamadas de ferramentas, falhas e evidências.
4. **Librarian** — governa memória e aprendizado persistente.

## Workers efêmeros

- Architect / Planner
- Builder / Engineer
- Verifier / Reviewer
- QA / Evaluator
- Red Team
- Operator / DevOps
- Researcher
- **Skill Scout** — pesquisa e propõe fontes/candidatos; nunca instala ou adapta.
- **Skill Curator** — transforma somente uma proposta aprovada em pacote AgentOS normalizado; nunca autoaprova ou registra.
- Domain Experts
- Recovery Agent

## Cadeia segura de aquisição de skills

```text
CAPABILITY GAP
      ↓
SKILL SCOUT
  pesquisa / compara / propõe
      ↓
SKILL_PROPOSAL
      ↓
GUARDIAN — SOURCE GATE
  APPROVED_FOR_CURATION
      ↓
SKILL CURATOR
  normaliza / minimiza / empacota / cria testes
      ↓
SKILL_PACKAGE_CANDIDATE
      ↓
VERIFIER — PACKAGE GATE
  PASS
      ↓
GUARDIAN — REGISTRATION GATE
  APPROVED
      ↓
SKILL REGISTRY
      ↓
AGENT FACTORY / WORKERS
```

### Separação de autoridade

- **Scout não cura**: não executa, adapta, empacota, instala ou registra.
- **Curator não escolhe outra fonte**: trabalha somente com repositório, revisão, escopo e teto de permissões aprovados.
- **Curator não autoaprova**: seu pacote obrigatoriamente vai para Verifier e depois Guardian.
- **Verifier não ativa**: PASS é evidência técnica, não concessão de autoridade.
- **Guardian possui dois gates**: aprova a fonte antes da curadoria e o pacote final antes da ativação.

Se a curadoria descobrir que precisa ampliar capacidade, trocar revisão ou pedir permissões adicionais, o fluxo volta ao Scout e um novo `SKILL_PROPOSAL` é obrigatório.

## Ciclo padrão

```text
USER
  ↓
GUARDIAN PRE-FLIGHT
  ↓
ORCHESTRATOR
  ↓
PLAN / TASK GRAPH
  ↓
CAPABILITY GAP CHECK
  ├─ nenhuma lacuna → segue
  └─ lacuna
       ↓
     SKILL SCOUT
       ↓
     GUARDIAN SOURCE REVIEW
       ↓
     SKILL CURATOR
       ↓
     VERIFIER (skill_package)
       ↓
     GUARDIAN REGISTRATION REVIEW
       ↓
     SKILL REGISTRY
  ↓
WORKER(S)
  ↓
VERIFIER
  ├─ PASS
  ├─ REVISE → volta ao worker
  └─ REJECT → volta ao Orchestrator
  ↓
GUARDIAN POST-FLIGHT
  ↓
LIBRARIAN + OBSERVER
  ↓
USER
```

## Diretório replicável

Este diretório `agentos/` foi desenhado para ser copiado integralmente para outros projetos. O projeto novo deve alterar apenas os overlays locais, políticas de domínio, skills permitidas e integrações.

Estrutura:

```text
agentos/
├── README.md
├── agentos.yaml
├── agents/
│   ├── orchestrator.agent.yaml
│   ├── skill-scout.agent.yaml
│   ├── skill-curator.agent.yaml
│   └── verifier.agent.yaml
├── constitution/
│   └── guardian.policy.yaml
├── contracts/
│   ├── message.contract.yaml
│   ├── skill-proposal.contract.yaml
│   └── skill.contract.yaml
├── templates/
│   └── skill.package.yaml
└── workflows/
    └── default.workflow.yaml
```

## Pacote canônico de skill

O Curator deve produzir algo equivalente a:

```text
skills/<skill-id>/
├── SKILL.md
├── manifest.yaml
├── SOURCE.lock.yaml
├── tests/
└── references/
```

`SOURCE.lock.yaml` mantém a procedência imutável. `manifest.yaml` declara capacidades, ferramentas e permissões. Os testes demonstram que a skill funciona dentro do limite constitucional aprovado.

## Regra de ouro do Skill Scout

O Skill Scout **não instala** skills. Ele descobre, classifica, compara, audita e produz uma proposta. Toda incorporação passa por Guardian, Curator, Verifier e uma segunda decisão do Guardian.

## Regra de ouro do Skill Curator

O Curator pode **reduzir** escopo e permissões para tornar uma skill mais segura, mas nunca ampliá-los. Expansão exige nova descoberta e nova aprovação.

## Portabilidade

O AgentOS deve evitar dependência de um único framework. Adapters podem integrar OpenAI Agents SDK, LangGraph, MCP, A2A, GitHub, Firebase ou outros stacks, mantendo os contratos internos estáveis.
