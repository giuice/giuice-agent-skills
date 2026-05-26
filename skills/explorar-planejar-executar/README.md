# explorar-planejar-executar

Fluxo em pt-BR para transformar objetivos vagos em execução controlada por três fases explícitas: explorar, planejar e executar.

## Quando usar

Use esta skill quando o usuário tiver um objetivo aberto que precise de descoberta, planejamento ou execução passo a passo.

Bons gatilhos incluem:

- `/explorar`
- `/planejar`
- `/executar`
- "me ajuda a organizar essa ideia"
- "quero planejar antes de fazer"
- "vamos executar tarefa por tarefa"

## O que ela faz

A skill conduz três fases:

1. **Explorar** — entende objetivo, contexto, restrições e critérios de sucesso.
2. **Planejar** — propõe abordagens e salva um plano com tarefas atômicas.
3. **Executar** — executa uma tarefa por vez, marca progresso e pede confirmação antes de continuar.

As transições não são automáticas. O usuário decide quando avançar para a próxima fase.

## Instalação

```bash
npx skills add giuice/giuice-agent-skills --skill explorar-planejar-executar
```
