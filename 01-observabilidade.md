# 01 — Observabilidade: SLIs, SLOs, Painel e Alertas

## 1. Objetivo

Definir como a equipe vai perceber, medir e reagir ao estado do sistema de biblioteca, usando sinais ligados à experiência do usuário e não apenas ao estado interno dos servidores.

## 2. Operação Crítica e Experiência Esperada

| Item | Definição |
|------|-----------|
| Operação crítica | Empréstimo e devolução de livros |
| Experiência esperada | O usuário consegue registrar um empréstimo ou uma devolução, de forma correta e sem espera perceptível |
| Quando falha | Erro ao registrar, lentidão, sistema fora do ar ou registro inconsistente |
| Pior momento | Início do semestre, com maior volume de acessos |

A escolha é coerente com o T6, em que o empréstimo foi a funcionalidade alterada e a mais protegida pelos smoke tests.

## 3. Sinais de Telemetria

| Sinal | Pergunta que responde | O que coletar no sistema |
|-------|----------------------|--------------------------|
| Métricas | Quanto e com que frequência? | Taxa de erros, latência p95, requisições por minuto, CPU, memória, conexões com o banco, reinicializações |
| Logs | O que aconteceu neste ponto? | Eventos com momento, serviço, severidade, evento e identificador de correlação, sem dados pessoais nem segredos |
| Traces | Por onde a requisição passou? | Caminho da requisição entre aplicação e banco, com tempo por etapa; passam a ser essenciais se o sistema for dividido em mais serviços |

**Correlação:** a métrica detecta o aumento de erros, o trace localiza a etapa lenta e o log explica a falha. Para isso funcionar, as requisições carregam um identificador comum e os horários seguem o mesmo padrão em todos os componentes.

**Cuidado com custo e ruído:** coletar tudo sem objetivo gera custo e ruído. Só entram os sinais que alimentam os SLIs, o painel ou os alertas abaixo.

## 4. SLIs (Indicadores de Nível de Serviço)

| ID | SLI | Como medir | Por que importa |
|----|-----|-----------|-----------------|
| SLI-1 | Sucesso das operações de empréstimo e devolução | Requisições válidas concluídas sem erro ÷ requisições válidas | Mede diretamente a operação crítica |
| SLI-2 | Latência das requisições de empréstimo e devolução | Proporção de requisições respondidas em menos de 1 segundo | Mede a velocidade percebida |
| SLI-3 | Disponibilidade observada | Proporção do tempo em que o readiness está aprovado | Mede se o serviço está acessível |
| SLI-4 | Taxa de erros HTTP do serviço | Respostas de erro do servidor ÷ total de respostas | Sinal geral de saúde, base dos alertas |

**Requisição válida:** exclui tráfego inválido, como erros causados pelo cliente (por exemplo, dados digitados incorretamente). Essa exclusão evita que erros do usuário consumam a meta do serviço.

## 5. SLOs (Objetivos de Nível de Serviço)

| SLO | SLI | Meta | Janela |
|-----|-----|------|--------|
| SLO-1 | SLI-1 | 99,9% das requisições válidas de empréstimo e devolução concluídas com sucesso | 30 dias |
| SLO-2 | SLI-2 | 95% das requisições de empréstimo e devolução respondidas em menos de 1 segundo | 30 dias |

**Justificativa**
- **SLO-1:** 99,9% é o valor usado como exemplo na aula e é compatível com o limite do T6. Em 30 dias, os 0,1% tolerados equivalem a cerca de **43 minutos** de falha total (30 × 24 × 60 × 0,001 = 43,2). Como cada incidente pode durar no máximo 5 minutos, o orçamento comporta aproximadamente 8 incidentes no mês.
- **SLO-2:** "95% abaixo de 1 segundo" é a mesma exigência do T6 (p95 inferior a 1 segundo), agora medida ao longo de uma janela de 30 dias.
- **100% não é a meta:** disponibilidade total tem custo impraticável e não é necessária para uma biblioteca.

**Relação com os limites do T6:** o limite de 2% de erros do T6 é um gatilho de **incidente em janela curta** (5 minutos). O SLO é a meta **de longo prazo** (30 dias). Os dois se complementam: o alerta reage rápido, o SLO mostra se a confiabilidade do mês está saudável.

### Error budget (orçamento de erro)

O error budget é a falha tolerada pelo SLO (os 0,1% do SLO-1). Se o consumo do orçamento acelerar, a equipe deve:
- reduzir o risco de mudanças (adiar deploys não essenciais);
- priorizar confiabilidade sobre novas funcionalidades;
- investigar as causas.

O SLO é uma meta interna de engenharia, não um SLA (compromisso formal com consequências).

## 6. Painel Mínimo

O painel proposto é **independente de ferramenta**: define o que mostrar, de onde vem o dado e quando o bloco fica em alerta. Os valores de limite são os do T6.

| Bloco | O que mostra | Fonte | Limite de alerta | Ferramenta possível |
|-------|--------------|-------|------------------|---------------------|
| Estado do serviço | No ar / degradado / fora | Health check (readiness) | Qualquer falha | Grafana ou CloudWatch |
| Volume | Requisições por minuto | Métrica | Comparar com o mesmo horário de semanas anteriores | Grafana ou CloudWatch |
| Erros | % de erros HTTP (SLI-4) | Métrica | 2% | Grafana, CloudWatch ou Sentry |
| Latência | p95 (SLI-2) | Métrica | 1 s | Grafana ou CloudWatch |
| Saturação | CPU, memória, conexões com o banco | Métrica | Recurso próximo do limite | Grafana ou CloudWatch |
| Dependências críticas | Banco (primário e réplica), atraso de replicação | Métrica e health check | Banco inacessível | Grafana ou CloudWatch |
| Operação crítica | Empréstimos e devoluções concluídos (SLI-1) | Métrica de negócio | Queda frente à linha de base | Grafana ou Datadog |
| Orçamento de erro | Quanto do orçamento de 30 dias já foi consumido | Cálculo sobre SLI-1 | Consumo acelerado | Grafana ou Datadog |
| Eventos de deploy | Marcador de cada deploy no gráfico | Pipeline (T6) | n/a | Qualquer uma |
| Comparação temporal | Hoje × semana anterior | Métrica | n/a | Qualquer uma |
| Investigação | Link para logs e traces do período | Logs e traces | n/a | Qualquer uma |

Sugestão de organização da tela:

```text
+--------------------------------------------------------------+
| ESTADO: No ar          Orçamento de erro (30d): ████░░░░    |
+------------------+-------------------+-----------------------+
| Volume (req/min) | Erros (%)         | Latência p95          |
|  [gráfico]       |  [gráfico + 2%]   |  [gráfico + 1 s]      |
+------------------+-------------------+-----------------------+
| Saturação        | Banco e réplica   | Empréstimos/devolução |
|  [gráfico]       |  [estado + atraso]|  [sucesso vs. base]   |
+------------------+-------------------+-----------------------+
| Linha do tempo: deploys (marcadores) | Links: logs / traces  |
+--------------------------------------------------------------+
```

As ferramentas listadas são opções. A ferramenta não substitui bons sinais e objetivos: o que importa é que cada bloco responda a uma pergunta de operação.

## 7. Alertas com Ação

Um alerta só existe se exigir uma ação. Todos são baseados em **sintoma** (impacto ao usuário), e CPU, disco e conexões ficam no painel para ajudar a explicar a causa.

| Alerta | Sintoma e limite | Janela | Severidade | Responsável | Ação |
|--------|------------------|--------|------------|-------------|------|
| A1 — Erros elevados | Taxa de erros ≥ 2% | 5 min consecutivos | Crítica | Observabilidade avisa; Rollback decide | Se ocorreu logo após um deploy, executar o [plano de rollback do T6](https://github.com/bel1020304050/Atividade-CD-Deploy-Rollback-); caso contrário, abrir o runbook em [03](03-runbook-e-riscos.md) |
| A2 — Latência degradada | p95 acima de 1 s, em degradação frente à linha de base | 5 min consecutivos | Alta | Observabilidade | Verificar saturação e atraso do banco; se saturado, aumentar instâncias (ver [02](02-alta-disponibilidade-dr.md), seção 5); se após deploy, avaliar rollback |
| A3 — Serviço ou banco indisponível | Readiness reprovado (aplicação ou acesso ao banco) | Falha em verificações consecutivas, sem esperar a janela de 5 min (alerta crítico) | Crítica | Banco de dados e Deploy | Seguir o runbook de falha do banco em [03](03-runbook-e-riscos.md) |

Para evitar fadiga de alertas, cada alerta tem ação definida e responsável; avisos sem ação ficam apenas no painel.

Os limites são metas iniciais propostas e devem ser confirmados com a linha de base do sistema.
