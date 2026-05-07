# Triforce Skills

Duas skills para Claude Code criadas pela Triforce Auto.

## mcp-creator (Lucas)

Criador de MCP Servers Senior. Cria, mantém e documenta MCP Servers em TypeScript usando SDK oficial e FastMCP.

**Instalar:**
```bash
cd ~/.claude/skills
cp -r mcp-creator/ mcp-creator/
```

**Contém:**
- `SKILL.md` — instrução principal (249 linhas)
- `references/constraints-plataforma.md` — limites de MCP Spec, SDKs, plataformas de deploy
- `references/operacional-ferramentas.md` — code patterns, deploy commands, integrações
- `references/estrategico-frameworks.md` — design agent-centric, segurança, monitoring
- `references/estrategico-sops.md` — SOPs para criar, atualizar, deprecar, debugar MCPs

**Stack:** MCP Spec 2025-11-25 | @modelcontextprotocol/sdk 1.29.0 | FastMCP 4.x (TS) | mcp-handler 1.1.0

## skill-creator (Thiago)

Criador de Skills Senior. Cria, mantém, versiona e audita skills (.md) para agentes de IA.

**Instalar:**
```bash
cd ~/.claude/skills
cp -r skill-creator/ skill-creator/
```

**Contém:**
- `SKILL.md` — instrução principal (216 linhas)
- `references/constraints-plataforma.md` — SkillsBench, progressive disclosure, segurança
- `references/operacional-ferramentas.md` — templates, checkpoint questions, security gate
- `references/estrategico-frameworks.md` — prompt engineering, eval-driven dev, lifecycle
- `references/estrategico-sops.md` — SOPs para criar, atualizar, auditar, deprecar skills

**Fontes:** SkillsBench (arXiv:2602.12670) | Anthropic Best Practices 2026 | agentskills.io spec
