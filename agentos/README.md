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
- Skill Scout
- Domain Experts
- Recovery Agent

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
  └─ lacuna → SKILL SCOUT → GUARDIAN REVIEW → registro da skill
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
│   └── verifier.agent.yaml
├── constitution/
│   └── guardian.policy.yaml
├── contracts/
│   ├── message.contract.yaml
│   └── skill.contract.yaml
└── workflows/
    └── default.workflow.yaml
```

## Regra de ouro do Skill Scout

O Skill Scout **não instala** skills. Ele descobre, classifica, compara, audita e produz uma proposta. Toda incorporação passa por Guardian e por verificação de origem, licença, permissões, integridade e testes.

## Portabilidade

O AgentOS deve evitar dependência de um único framework. Adapters podem integrar OpenAI Agents SDK, LangGraph, MCP, A2A, GitHub, Firebase ou outros stacks, mantendo os contratos internos estáveis.
