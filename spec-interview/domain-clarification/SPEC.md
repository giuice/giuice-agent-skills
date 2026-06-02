# SPEC: domain-clarification (Skill de Clarificação de Domínios)

> Idioma da SPEC: pt-BR (idioma da conversa). A skill especificada **opera em inglês** (FR-010).

## 1. Summary

Skill portátil que transforma um pedido vago em um **Mapa de Domínios** confirmado: ela infere quais domínios de conhecimento o pedido envolve, conduz uma entrevista exaustiva em rodadas para resolver ambiguidades por domínio e encerra somente quando o usuário confirma o mapa. É um pré-passo autônomo cuja saída pode alimentar qualquer fluxo de spec/planejamento — mas ela **não** escreve a SPEC nem o plano.

## 2. Problem statement

Pedidos iniciais costumam misturar vários domínios mal delimitados e termos ambíguos. Pular essa clareza joga as suposições para dentro da SPEC, onde viram retrabalho. Quem escopa uma ideia — técnico ou não — precisa primeiro saber *quais áreas estão em jogo* e *o que cada termo significa* antes de especificar qualquer coisa.

## 3. Goals and success metrics

### Goals
- Inferir e confirmar com o usuário os domínios de conhecimento de um pedido vago.
- Resolver ambiguidades por domínio através de perguntas direcionadas, em rodadas.
- Entregar um Mapa de Domínios reutilizável por fluxos downstream.

### Success metrics
- Redução de suposições/ambiguidades não resolvidas antes da spec (métrica central).
- Usuário confirma o Mapa em poucas rodadas (eficiência).
- Cobertura: todo domínio relevante ao pedido aparece no Mapa.

## 4. Users and stakeholders

### Primary users
- Qualquer pessoa escopando uma ideia, incluindo não-técnicos.

### Secondary users
- Quem vai escrever a SPEC/plano depois (consome o Mapa de Domínios).

### Stakeholders
- Autores de fluxos downstream (ex.: spec-from-scratch) que recebem o Mapa como insumo.

## 5. Scope

### In scope (v1 — must-have)
- Inferência de domínios candidatos a partir do pedido.
- Confirmação ativa: usuário marca quais valem, remove e adiciona.
- Perguntas de clarificação por domínio, entregues em rodadas exaustivas.
- Entrega por UI nativa quando disponível, HTML como fallback.
- Mapa de Domínios salvo em arquivo.

### Out of scope
- Escrever a SPEC, requisitos, plano ou desenho de solução (a skill para na clareza de domínios).
- Persistência de estado entre rodadas (**should**, não v1).
- Golden set de evals empacotado (**should**, não v1 — forma já definida em FR-013).

## 6. User journeys

1. Usuário traz um pedido vago e ativa a skill.
2. A skill infere domínios candidatos e os apresenta para confirmação (marcar/remover/adicionar).
3. Para cada domínio confirmado, gera uma rodada de perguntas (UI nativa ou HTML); usuário responde clicando + notas livres.
4. A skill atualiza o Mapa, identifica ambiguidades remanescentes e gera novas rodadas só onde necessário.
5. Quando nenhum domínio tem ambiguidade bloqueante, apresenta o Mapa para confirmação final.
6. Usuário confirma → Mapa salvo. Fim (sem avançar para spec).

## 7. Functional requirements

| ID | Requirement | Priority | Acceptance signal |
|---|---|---|---|
| FR-001 | Inferir domínios candidatos a partir do pedido bruto | Must | Lista de domínios inferidos é apresentada |
| FR-002 | Apresentar domínios inferidos para confirmação: marcar válidos, remover, adicionar | Must | Usuário consegue editar a lista antes de prosseguir |
| FR-003 | Conduzir entrevista exaustiva em rodadas, com perguntas direcionadas por domínio | Must | Round 1 cobre domínios independentes; rounds seguintes só lacunas/dependências |
| FR-004 | Entregar perguntas via UI nativa quando disponível; senão gerar arquivo HTML self-contained por rodada | Must | HTML abre offline, com radio/checkbox e botão "Copiar respostas" |
| FR-005 | Toda pergunta permite resposta/nota livre, além das opções | Must | Cada pergunta tem campo de texto livre |
| FR-006 | Produzir um Mapa de Domínios salvo em arquivo, com nome+descrição, ambiguidades/perguntas em aberto, definições resolvidas e status/confiança por domínio | Must | Arquivo do Mapa contém os 4 campos por domínio |
| FR-007 | Quando há domínios demais, priorizar os que bloqueiam a spec; registrar o resto como "aberto" | Must | Domínios excedentes aparecem com status "aberto", não são descartados |
| FR-008 | Encerrar somente quando nenhum domínio tem ambiguidade bloqueante e o usuário confirma o Mapa | Must | Skill pede confirmação explícita antes de finalizar |
| FR-009 | Não escrever SPEC, requisitos, plano ou solução | Must | Saída limita-se ao Mapa de Domínios |
| FR-010 | Operar em inglês e ser portátil entre Claude Code, Copilot CLI e Codex | Must | Instruções usam verbos genéricos (ler/salvar/editar), texto em inglês |
| FR-011 | Tratar casos extremos: pedido vago sem domínio claro; muitos domínios; domínios/termos conflitantes; usuário não sabe responder; pedido já claro | Must | Cada caso tem comportamento definido (ver seção 11) |
| FR-012 | Persistir estado das rodadas entre execuções | Should | Estado recuperável após chat interrompido |
| FR-013 | Disponibilizar golden set de evals (pedidos-exemplo → Mapa esperado) | Should | Conjunto de casos comparáveis e repetíveis |

## 8. Business rules

| ID | Rule | Rationale |
|---|---|---|
| BR-001 | A skill não produz SPEC, plano ou desenho de solução — a saída é só o Mapa de Domínios | Mantém fronteira clara com fluxos downstream (FR-009) |
| BR-002 | Com domínios em excesso, priorizar os bloqueantes; os demais ficam como "aberto", nunca descartados | Evita perda de cobertura sob sobrecarga (FR-007) |
| BR-003 | Um domínio só é marcado "claro" quando não tem ambiguidade bloqueante | Garante profundidade real, não cobertura superficial |
| BR-004 | Domínios são inferidos pela IA, mas só finalizados após confirmação explícita do usuário | Sem suposições silenciosas (FR-002) |
| BR-005 | A conclusão exige confirmação explícita do Mapa pelo usuário | Define "pronto" de forma observável (FR-008) |

## 9. Data and integrations

### Data inputs
- Pedido/ideia em texto livre do usuário.
- Respostas das rodadas (opções marcadas + notas livres).

### Data outputs
- Mapa de Domínios (arquivo): por domínio → nome+descrição, ambiguidades em aberto, definições resolvidas, status/confiança.

### Integrations
- UI nativa de perguntas do agente hospedeiro quando existir; caso contrário, arquivo HTML local.
- Saída consumível por fluxos de spec/planejamento (ex.: spec-from-scratch), sem acoplamento obrigatório.

## 10. Constraints and assumptions

### Constraints
- Idioma de operação: inglês (melhor performance para LLMs — nota do usuário).
- Portável entre Claude Code, Copilot CLI e Codex (sem depender de nomes de ferramentas específicos).
- HTML de rodada deve ser self-contained e funcionar offline.

### Assumptions
- O agente hospedeiro consegue ler um bloco colado pelo usuário ou um arquivo local de respostas.

## 11. Edge cases and failure modes

| Case | Expected behavior |
|---|---|
| Pedido tão vago que nenhum domínio é claro | Fazer perguntas amplas de enquadramento antes de inferir domínios |
| Muitos domínios de uma vez | Priorizar os bloqueantes; agrupar/registrar o resto como "aberto" (BR-002) |
| Domínios conflitantes ou termos com sentidos opostos | Sinalizar o conflito e pedir desambiguação explícita ao usuário |
| Usuário não sabe responder uma clarificação | Marcar o item como "aberto/deferido" e seguir, sem travar |
| Pedido já claro, sem ambiguidade | Encerrar rápido, sem perguntas desnecessárias |

## 12. Security, privacy, compliance, and abuse considerations

- A skill manipula apenas o texto fornecido pelo usuário; sem coleta de dados externos.
- HTML sem acesso à rede elimina vazamento de respostas para terceiros.

## 13. Accessibility, localization, and usability considerations

- HTML com `<label>` associadas e foco visível (acessibilidade de teclado).
- Toda pergunta com campo livre, para quem prefere escrever em vez de marcar.
- Operação em inglês (FR-010); notas do usuário aceitas em qualquer idioma.

## 14. Acceptance criteria

Cada critério referencia o(s) FR que valida (TDD-ready):

- AC-001 (FR-001, FR-002): Dado um pedido vago, quando a skill roda, então apresenta domínios inferidos que o usuário pode marcar, remover e adicionar.
- AC-002 (FR-003, FR-005): Dado domínios confirmados, quando uma rodada é gerada, então cada pergunta tem opções e um campo de resposta/nota livre.
- AC-003 (FR-004): Dado um agente sem UI nativa de perguntas, quando a rodada é entregue, então é gerado um HTML self-contained que abre offline e copia as respostas em bloco estruturado.
- AC-004 (FR-006): Dado o fim da entrevista, quando o Mapa é salvo, então cada domínio contém nome+descrição, ambiguidades, definições resolvidas e status/confiança.
- AC-005 (FR-007, BR-002): Dado excesso de domínios, quando a skill prioriza, então os bloqueantes vêm primeiro e os demais ficam como "aberto".
- AC-006 (FR-008, BR-005): Dado que nenhum domínio tem ambiguidade bloqueante, quando a skill encerra, então só finaliza após confirmação explícita do Mapa pelo usuário.
- AC-007 (FR-009, BR-001): Dado qualquer estado, quando o usuário pede a SPEC/plano, então a skill recusa e entrega apenas o Mapa de Domínios.
- AC-008 (FR-011): Dado cada caso extremo da seção 11, quando ocorre, então a skill segue o comportamento esperado correspondente.

## 15. Test strategy

*(Projeto com eval — q11/FR-013.)*

- **Forma:** golden set — conjunto de pedidos-exemplo (vago, multi-domínio, conflitante, já-claro) com o Mapa de Domínios esperado para cada.
- **Trace:** cada caso do golden set valida um ou mais AC acima (ex.: caso "multi-domínio" → AC-005; caso "já-claro" → AC-008/early exit).
- **Níveis:** sem unit tests de código (artefato é markdown); validação é comportamental via execução da skill contra o golden set + checklist de auto-revisão por execução.
- **Casos obrigatórios:** os cinco casos extremos da seção 11 devem ter ao menos um exemplo no golden set.
- **Maturidade:** golden set é **Should** para o v1 (FR-013); no v1 mínimo, vale o checklist qualitativo.

## 16. Validation and launch checklist

- [ ] Round 1 cobre todos os domínios inferidos e independentes.
- [ ] HTML de rodada abre offline e o botão "Copiar respostas" gera bloco válido.
- [ ] Mapa de Domínios salvo com os 4 campos por domínio.
- [ ] Skill recusa escrever SPEC/plano (BR-001).
- [ ] Confirmação explícita do usuário antes de finalizar (BR-005).
- [ ] Comportamento verificado para os 5 casos extremos.

## 17. Open questions

*(Não-bloqueantes — diferidos para a fase de planejamento da skill.)*

- Onde salvar o Mapa de Domínios por padrão (caminho/convenção de pasta)?
- Formato do Mapa: markdown, JSON, ou ambos?
- Persistência de estado entre rodadas (FR-012): formato e local do arquivo de estado.
