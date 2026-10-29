# Observabilidade, Alta Disponibilidade e Recuperação de Desastres — Sistema Web de Biblioteca

**Disciplina:** Cloud Computing e DevOps Avançado  
**Professor:** Ranghetti  
**Equipe:** Isabely e Felipe do Nascimento  
**Trabalho:** T7 — Observabilidade, disponibilidade e recuperação

---

## 1. Introdução

Depois de definir como implantar e reverter o sistema web de biblioteca (T6), este trabalho define como **detectar** problemas (observabilidade), **resistir** a falhas (alta disponibilidade e resiliência) e **recuperar** o serviço em eventos graves (recuperação de desastres, DR).

## 2. Relação com o Trabalho 6

O T7 usa o **mesmo serviço do T6**. Foram reaproveitados:

- a operação crítica (empréstimo e devolução de livros);
- as métricas disponíveis (taxa de erros e latência) e seus limites (erros abaixo de 2%, latência p95 abaixo de 1 segundo);
- o limite de 5 minutos de indisponibilidade;
- o banco de dados compartilhado;
- os smoke tests ST01–ST07 e o plano de rollback.

## 3. Objetivo

Propor uma estratégia observável e resiliente para o sistema de biblioteca, com indicadores e metas de serviço, painel, alertas acionáveis, arquitetura de alta disponibilidade, RPO e RTO justificados, estratégia de DR e um runbook para uma falha.

## 4. Cenário do Sistema

| Item | Descrição |
|------|-----------|
| Sistema | Aplicação web de biblioteca |
| Período de maior utilização | Início de cada semestre |
| Indisponibilidade máxima tolerada | 5 minutos |
| Banco de dados | Compartilhado entre as versões da aplicação |
| Operação crítica | Empréstimo e devolução de livros |
| Métricas disponíveis | Taxa de erros e latência |
| Estratégia de deploy (T6) | Blue/Green |

## 5. Estrutura do Repositório

| Arquivo | Conteúdo |
|---------|----------|
| [README.md](README.md) | Apresentação do trabalho |
| [01-observabilidade.md](01-observabilidade.md) | Operação crítica, sinais, SLIs, SLOs, painel mínimo e alertas |
| [02-alta-disponibilidade-dr.md](02-alta-disponibilidade-dr.md) | Arquitetura, alta disponibilidade, escalabilidade, resiliência, RPO/RTO, DR e backup |
| [03-runbook-e-riscos.md](03-runbook-e-riscos.md) | Runbook de uma falha, riscos e justificativas |

Os documentos são complementares e devem ser analisados em conjunto.

## 6. Onde encontrar cada entregável do T7

| Entregável | Onde está |
|------------|-----------|
| Diagrama da arquitetura | [02](02-alta-disponibilidade-dr.md), seção 2 |
| Três ou mais SLIs | [01](01-observabilidade.md), seção 4 |
| Dois SLOs | [01](01-observabilidade.md), seção 5 |
| Painel mínimo proposto | [01](01-observabilidade.md), seção 6 |
| Três alertas com ação | [01](01-observabilidade.md), seção 7 |
| Estratégia de alta disponibilidade | [02](02-alta-disponibilidade-dr.md), seções 3 a 6 |
| RPO e RTO justificados | [02](02-alta-disponibilidade-dr.md), seção 7 |
| Estratégia de DR | [02](02-alta-disponibilidade-dr.md), seções 8 e 9 |
| Runbook resumido de uma falha | [03](03-runbook-e-riscos.md), seção 1 |

## 7. Observação sobre os valores numéricos

Os valores (SLOs, RPO, RTO) são **metas iniciais propostas** para o exercício. Eles partem dos limites do T6 e dos exemplos apresentados na aula, e devem ser confirmados com a linha de base real do sistema e com o impacto para os usuários.

## 8. Referências

- Aula 12 — *Observabilidade, Alta Disponibilidade e Recuperação de Desastres*. Prof. Ranghetti. Cloud Computing e DevOps Avançado.
- KOLBE JÚNIOR, Armando. *Computação em Nuvem*. Contentus, 2020.
- SOUSA NETO, Manoel Veras de. *Computação em Nuvem*. Brasport, 2015.
- GOOGLE. *Site Reliability Engineering*. <https://sre.google/books/>
- OPENTELEMETRY. *Documentation*. <https://opentelemetry.io/docs/>
- AWS. *Disaster Recovery of Workloads on AWS*. <https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html>
