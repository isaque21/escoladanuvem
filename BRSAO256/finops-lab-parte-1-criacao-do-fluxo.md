# FinOps Lab — Guia de implementação

## 1. Sobre o projeto

O **FinOps Lab** é um laboratório prático para aprender a integrar **Tally**, **Airtable** e **Activepieces**, construindo uma solução simples para registrar e acompanhar oportunidades de otimização de custos na AWS.

O projeto também poderá ser utilizado como atividade prática para alunos da Escola da Nuvem. Nesta primeira versão, são utilizados **dados fictícios**: nenhuma conta AWS é conectada e nenhuma alteração é realizada em recursos reais.

### Objetivo da primeira versão

Quando alguém envia uma oportunidade pelo formulário:

1. O Tally recebe as informações.
2. O Activepieces inicia o fluxo quando detecta uma nova submissão.
3. O Activepieces cria um registro na base do Airtable.
4. O Activepieces envia um e-mail de confirmação pelo Gmail.

### Ferramentas utilizadas

- **Tally:** formulário de entrada de oportunidades.
- **Airtable:** base de dados e acompanhamento.
- **Activepieces:** orquestração das etapas e integração entre serviços.
- **Gmail:** envio do e-mail de confirmação.

## 2. Arquitetura

```text
Pessoa preenche o formulário
            |
            v
          Tally
            |
            v
      Activepieces
       /         \\
      v           v
  Airtable      Gmail
  Registra      Envia confirmação
  oportunidade
```

O fluxo lógico é **Tally → Activepieces → Airtable → Gmail**. No Activepieces, as ações do Airtable e do Gmail são executadas como etapas do mesmo fluxo.

## 3. Criar a base no Airtable

### 3.1. Criar a base e a tabela

1. Acesse [Airtable](https://airtable.com/).
2. Entre na conta ou crie uma.
3. Crie uma base em branco (*Start from scratch*).
4. Nomeie a base como `FinOps Lab`.
5. Nomeie a primeira tabela como `Oportunidades`.

### 3.2. Criar os campos

Configure os campos da tabela:

| Campo | Tipo no Airtable | Observação |
|---|---|---|
| Título | Single line text | Campo principal da tabela |
| Responsável | Single line text | Pessoa que registrou a oportunidade |
| E-mail para retorno | Email | Endereço para a confirmação |
| Serviço AWS | Single select | Serviço associado à oportunidade |
| Recurso fictício | Single line text | Identificador de exemplo |
| Descrição da oportunidade | Long text | Contexto e recomendação |
| Custo mensal atual (R$) | Currency | Custo mensal de referência |
| Custo mensal estimado após otimização (R$) | Currency | Custo mensal hipotético após a otimização |
| Status | Single select | Etapa de acompanhamento |
| Data de registro | Created time | Preenchimento automático pelo Airtable |

Opções de `Serviço AWS`:

- `EC2`
- `EBS`
- `S3`
- `NAT Gateway`
- `RDS`
- `Lambda`
- `DynamoDB`
- `CloudWatch`
- `Elastic IP`
- `ELB`
- `EKS`
- `Outros`

> Atualização: a lista foi ampliada de 5 para 12 opções. Use exatamente estes nomes, pois eles devem coincidir com as opções do mesmo campo no formulário Tally e com os valores da coluna `Serviço AWS` do arquivo de importação de dados.

Opções de `Status`:

- `Nova`
- `Em análise`
- `Aprovada`
- `Em implementação`
- `Validada`
- `Rejeitada`

Configure `Nova` como opção inicial do status, se a configuração do campo permitir.

### 3.3. Inserir um registro de exemplo

Foi utilizado um registro fictício para conferir a estrutura da tabela:

| Campo | Valor de exemplo |
|---|---|
| Título | `LAB-001 — Otimização EBS` |
| Responsável | `Aluno Teste` |
| E-mail para retorno | Um e-mail de teste |
| Serviço AWS | `EBS` |
| Recurso fictício | `vol-lab-001` |
| Descrição da oportunidade | Avaliar a migração de um volume gp2 para gp3 |
| Custo mensal atual | R$ 100,00 |
| Custo mensal estimado após otimização | R$ 75,00 |
| Status | `Nova` |

Os valores são apenas ilustrativos. A economia sugerida não é uma estimativa validada de preços AWS.

## 4. Criar o formulário no Tally

### 4.1. Criar o formulário

1. Acesse [Tally](https://tally.so/).
2. Entre na conta.
3. Crie um formulário em branco (*Create new form*).
4. Use o título `FinOps Lab — Nova oportunidade`.
5. Adicione uma breve descrição, explicando que o formulário registra oportunidades fictícias de otimização de custos AWS para fins de estudo.

### 4.2. Adicionar os campos

Crie os oito campos a seguir. Os nomes devem ser claros e compatíveis com os campos da tabela do Airtable.

| Ordem | Campo no formulário | Tipo no Tally | Obrigatório |
|---:|---|---|---|
| 1 | Título da oportunidade | Short answer | Sim |
| 2 | Responsável | Short answer | Sim |
| 3 | E-mail para retorno | Email | Sim |
| 4 | Serviço AWS | Dropdown | Sim |
| 5 | Recurso fictício | Short answer | Sim |
| 6 | Descrição da oportunidade | Long answer | Sim |
| 7 | Custo mensal atual (R$) | Number | Sim |
| 8 | Custo mensal estimado após otimização (R$) | Number | Sim |

Opções do campo `Serviço AWS`:

- `EC2`
- `EBS`
- `S3`
- `NAT Gateway`
- `RDS`
- `Lambda`
- `DynamoDB`
- `CloudWatch`
- `Elastic IP`
- `ELB`
- `EKS`
- `Outros`

> Atualização: a lista de serviços foi ampliada de 5 para 12 opções para cobrir mais cenários de otimização de custos. As opções `RDS`, `Lambda`, `DynamoDB`, `CloudWatch`, `Elastic IP`, `ELB` e `EKS` foram acrescentadas. Mantenha exatamente estes nomes, pois eles precisam coincidir com as opções do campo `Serviço AWS` no Airtable para que a integração e a importação de dados funcionem.

Para os campos de custo, digite somente o valor numérico, sem incluir o símbolo `R$`. A formatação monetária é feita no Airtable.

Não crie campos para `Status` ou `Data de registro` no Tally. Essas informações são gerenciadas no Airtable.

### 4.2.1. Atualizar o dropdown `Serviço AWS` (ampliação dos serviços)

A versão inicial do formulário tinha apenas cinco opções no dropdown `Serviço AWS` (`EC2`, `EBS`, `S3`, `NAT Gateway`, `Outros`). A lista foi ampliada para doze opções, cobrindo mais cenários de otimização de custos.

Se o seu formulário já existe com as cinco opções originais, atualize-o:

1. Abra o formulário `FinOps Lab — Nova oportunidade` no Tally.
2. Clique no campo `Serviço AWS` para editá-lo.
3. Adicione as sete novas opções, uma a uma, mantendo os nomes exatamente como abaixo:
   - `RDS`
   - `Lambda`
   - `DynamoDB`
   - `CloudWatch`
   - `Elastic IP`
   - `ELB`
   - `EKS`
4. Confira se a lista final tem as doze opções: `EC2`, `EBS`, `S3`, `NAT Gateway`, `RDS`, `Lambda`, `DynamoDB`, `CloudWatch`, `Elastic IP`, `ELB`, `EKS` e `Outros`.
5. Clique em `Publish` para publicar a versão atualizada do formulário.

> Importante: os nomes das opções devem ser idênticos aos do campo `Serviço AWS` no Airtable (seção 3.2). Qualquer diferença de grafia (acento, maiúscula/minúscula, espaço) faz o Airtable tratar como uma opção diferente, o que quebra o mapeamento no Activepieces e a importação de dados.

### 4.3. Configurar a mensagem de conclusão

Na página de agradecimento (*Thank you page*), use uma mensagem semelhante a:

> **Oportunidade registrada com sucesso!**  
> Sua submissão foi recebida pelo FinOps Lab. O registro será processado para avaliação.

A mensagem confirma o envio do formulário, não a conclusão de todas as etapas da automação.

### 4.4. Publicar e testar

1. Clique em `Publish`.
2. Abra o link público do formulário.
3. Preencha os campos com dados fictícios.
4. Envie a submissão.
5. Confirme que ela aparece na área de respostas (*Submissions*) do Tally.

**Resultado confirmado:** o formulário foi criado, publicado e testado; a submissão fictícia apareceu nas respostas do Tally.

## 5. Criar o fluxo no Activepieces

### 5.1. Criar o Flow

1. Acesse [Activepieces](https://cloud.activepieces.com/).
2. Entre na conta.
3. Crie um novo Flow.
4. Nomeie-o `FinOps Lab - Registrar oportunidade`.

### 5.2. Configurar o gatilho do Tally

1. Adicione o passo inicial e pesquise por `Tally`.
2. Selecione o gatilho `New Submission`.
3. Crie a conexão solicitada. Se for necessário usar uma chave de API, gere-a nas configurações da sua conta Tally e siga as instruções do serviço.
4. Selecione o formulário `FinOps Lab — Nova oportunidade`.
5. Execute o teste do gatilho para carregar uma submissão de exemplo.

**Resultado confirmado:** o teste do gatilho funcionou e o Activepieces recebeu os dados enviados pelo Tally.

## 6. Conectar o Activepieces ao Airtable

### 6.1. Adicionar a ação de criação de registro

1. No mesmo Flow, clique no `+` abaixo do gatilho Tally.
2. Pesquise por `Airtable`.
3. Selecione a ação de criação de registro, identificada como `Create Record` ou `Create Airtable Record`, conforme a versão da interface.
4. Crie a conexão com a conta Airtable.
5. Selecione a base `FinOps Lab`.
6. Selecione a tabela `Oportunidades`.

### 6.2. Mapear os campos

Associe cada campo do Airtable ao dado correspondente vindo do gatilho Tally:

| Campo de destino no Airtable | Origem no Tally |
|---|---|
| Título | Título da oportunidade |
| Responsável | Responsável |
| E-mail para retorno | E-mail para retorno |
| Serviço AWS | Serviço AWS |
| Recurso fictício | Recurso fictício |
| Descrição da oportunidade | Descrição da oportunidade |
| Custo mensal atual (R$) | Custo mensal atual (R$) |
| Custo mensal estimado após otimização (R$) | Custo mensal estimado após otimização (R$) |

Para `Status`, defina o valor fixo `Nova`, caso a ação permita preencher esse campo. Não mapeie `Data de registro`; deixe o Airtable preenchê-la automaticamente.

Confira se os custos chegam como números, por exemplo `100` e `75`, e não como textos com `R$`.

### 6.3. Testar a ação

1. Execute `Test Step` na ação do Airtable.
2. Abra a base `FinOps Lab`.
3. Confira se um novo registro foi criado e se os valores estão corretos.
4. Confira o status inicial e a data de registro.

**Resultado confirmado:** o teste criou um registro no Airtable com sucesso.

## 7. Conectar o Activepieces ao Gmail

### 7.1. Adicionar a ação de e-mail

1. No mesmo Flow, adicione uma etapa abaixo da ação do Airtable.
2. Pesquise por `Gmail`.
3. Selecione a ação de envio de e-mail, como `Send Email`.
4. Conecte a conta Gmail seguindo o processo de autenticação exibido.

### 7.2. Configurar a mensagem

**Destinatário:** selecione o campo `E-mail para retorno` vindo do gatilho Tally.

**Assunto:**

`FinOps Lab — Oportunidade registrada com sucesso`

**Corpo sugerido:**

```text
Olá, [Responsável]!

Sua oportunidade foi registrada com sucesso no FinOps Lab.

Detalhes da oportunidade

- Título: [Título da oportunidade]
- Serviço AWS: [Serviço AWS]
- Recurso fictício: [Recurso fictício]
- Custo mensal atual: R$ [Custo mensal atual]
- Custo mensal estimado após otimização: R$ [Custo mensal estimado]
- Status inicial: Nova

A oportunidade está registrada para análise. Os valores são estimativas didáticas e não representam economia efetivamente realizada.

Obrigado por contribuir com o FinOps Lab!

Equipe FinOps Lab
```

Os textos entre colchetes são marcadores explicativos. No Activepieces, substitua cada um por seu respectivo dado dinâmico, usando o seletor de dados da etapa do Tally. Não digite os marcadores literalmente.

### 7.3. Testar o e-mail

1. Execute o teste da etapa Gmail.
2. Confira a caixa de entrada do destinatário e, se necessário, a pasta de spam.
3. Valide o assunto, o conteúdo e os valores.
4. Verifique o resultado da execução no Activepieces.

**Resultado confirmado:** o teste do Gmail funcionou e o e-mail chegou à caixa de entrada.

## 8. Estado atual e validação final

Até aqui, foram confirmados os seguintes testes:

- [x] Base `FinOps Lab` criada no Airtable.
- [x] Tabela `Oportunidades` criada com os campos definidos.
- [x] Registro fictício inicial inserido manualmente.
- [x] Formulário Tally criado, publicado e testado.
- [x] Gatilho `New Submission` do Tally testado no Activepieces.
- [x] Ação de criação de registro no Airtable testada com sucesso.
- [x] Ação de envio pelo Gmail testada com sucesso.

### Teste integrado pendente de confirmação

Os testes individuais foram bem-sucedidos. Para validar o fluxo completo como automação:

1. Salve o Flow no Activepieces.
2. Ative ou publique o Flow, usando a opção disponível na interface.
3. Abra o formulário público do Tally e envie uma **nova** submissão fictícia.
4. Não execute manualmente os passos de teste; deixe o gatilho iniciar o fluxo.
5. Confira se o novo registro apareceu no Airtable.
6. Confirme se `Status` ficou como `Nova` e se `Data de registro` foi preenchida.
7. Confirme se o Gmail enviou o e-mail com os dados daquela submissão.
8. Consulte o histórico de execuções do Activepieces para verificar se as etapas foram concluídas.

Marque o teste integrado como concluído somente depois de observar o registro e o e-mail gerados pela mesma nova submissão.

## 9. Próximas evoluções planejadas

Depois da validação integrada, o projeto pode crescer gradualmente:

1. **Fórmulas de economia:** calcular economia mensal e anual estimada no Airtable.
2. **Classificação de oportunidades:** organizar recomendações por serviço, impacto ou prioridade.
3. **Fluxo de aprovação:** registrar decisões e responsáveis antes da implementação.
4. **Acompanhamento de resultados:** comparar economia estimada com economia validada.
5. **Laboratório didático:** criar desafios para alunos analisarem recomendações, riscos e resultados.

A integração com dados reais da AWS pode ser avaliada futuramente, mas não faz parte da versão inicial. O laboratório atual utiliza dados fictícios e não modifica recursos AWS.

## 10. Princípios do laboratório

- Utilizar dados fictícios nesta fase.
- Tratar economia prevista como estimativa, não como resultado garantido.
- Não executar alterações em recursos AWS como parte deste fluxo inicial.
- Manter aprovação humana e controles operacionais para qualquer futura implementação.
- Evitar incluir credenciais, chaves de API ou outros segredos nos formulários e registros.

---

**Versão do documento:** primeira versão do FinOps Lab, cobrindo a configuração inicial do Airtable, Tally, Activepieces e Gmail.
