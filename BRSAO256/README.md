# FinOps Lab — BRSAO256

Laboratório prático de **FinOps** usando ferramentas no-code. O lab monta um fluxo para registrar e acompanhar oportunidades de otimização de custos AWS, integrando **Tally**, **Activepieces**, **Airtable** e **Gmail**.

Projeto desenvolvido como atividade prática da **Escola da Nuvem** (turma BRSAO256). Nesta fase, todos os dados são **fictícios**: nenhuma conta AWS é conectada e nenhum recurso real é alterado.

## Visão geral do fluxo

```text
Pessoa preenche o formulário
            |
            v
          Tally
            |
            v
      Activepieces
       /         \
      v           v
  Airtable      Gmail
  Registra      Envia confirmação
  oportunidade
```

Fluxo lógico: **Tally → Activepieces → Airtable → Gmail**.

## Conteúdo deste diretório

| Arquivo | Descrição |
|---|---|
| `finops-lab-parte-1-criacao-do-fluxo.md` | Parte 1: criação do fluxo completo (Airtable, Tally, Activepieces e Gmail), com testes passo a passo. |
| `finops-lab-parte-2-melhorias-airtable.md` | Parte 2: melhorias no Airtable (fórmulas de economia, prioridade automática, views, dashboard) e importação do CSV. |
| `finops-lab-oportunidades.csv` | 120 oportunidades fictícias prontas para importar na tabela `Oportunidades` do Airtable. |

## Ordem de leitura

1. Comece pela **Parte 1** para montar e validar o fluxo base.
2. Siga para a **Parte 2** para enriquecer a base do Airtable e popular os dados com o CSV.

## Ferramentas utilizadas

- **Tally** — formulário de entrada de oportunidades.
- **Airtable** — base de dados, cálculos e acompanhamento visual.
- **Activepieces** — orquestração e integração entre os serviços.
- **Gmail** — e-mail de confirmação.

## Princípios do laboratório

- Utilizar dados fictícios nesta fase.
- Tratar a economia prevista como estimativa, não como resultado garantido.
- Não executar alterações em recursos AWS como parte deste fluxo.
- Manter aprovação humana para qualquer implementação futura.
- Não incluir credenciais, chaves de API ou outros segredos nos formulários e registros.
