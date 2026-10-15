# FinOps Lab — Melhorias no Airtable

## 1. Sobre este documento

Este guia dá continuidade ao **FinOps Lab** (ver `finops-lab-parte-1-criacao-do-fluxo.md`). O fluxo base **Tally → Activepieces → Airtable → Gmail** já está funcionando com dados fictícios.

Aqui o objetivo é evoluir a base do Airtable para que ela deixe de ser apenas um registro de oportunidades e passe a mostrar **impacto financeiro, prioridade e acompanhamento visual**.

Serão implementadas quatro melhorias:

1. **Fórmulas de economia** — calcular economia mensal, anual e percentual.
2. **Prioridade automática** — classificar cada oportunidade por impacto.
3. **Views de acompanhamento** — organizar os dados por status, serviço e impacto.
4. **Dashboard (Interface)** — visualizar os números em gráficos.

Pré-requisito: a base `FinOps Lab` e a tabela `Oportunidades` já criadas, com os campos descritos no guia principal, incluindo `Custo mensal atual (R$)` e `Custo mensal estimado após otimização (R$)`.

> Observação sobre nomes de campos nas fórmulas: o Airtable referencia campos pelo nome exato, entre chaves. Se o nome do seu campo for diferente do mostrado aqui, ajuste dentro das `{ }`. Os valores continuam sendo estimativas didáticas, não economia garantida.

---

## 2. Melhoria 1 — Fórmulas de economia

O objetivo é calcular, de forma automática, quanto cada oportunidade economiza por mês, por ano e em percentual.

### 2.1. Campo `Economia mensal (R$)`

1. Abra a base `FinOps Lab` e a tabela `Oportunidades`.
2. Clique no `+` à direita do último campo para adicionar um novo campo.
3. Nomeie o campo como `Economia mensal (R$)`.
4. Em tipo (*field type*), escolha `Formula`.
5. Cole a fórmula:

   ```
   {Custo mensal atual (R$)} - {Custo mensal estimado após otimização (R$)}
   ```

6. Em formatação do resultado, escolha `Currency`, símbolo `R$` e 2 casas decimais, para manter o padrão da base.
7. Salve o campo.

### 2.2. Campo `Economia anual (R$)`

1. Adicione um novo campo chamado `Economia anual (R$)`.
2. Tipo `Formula`.
3. Cole a fórmula:

   ```
   ({Custo mensal atual (R$)} - {Custo mensal estimado após otimização (R$)}) * 12
   ```

4. Formate o resultado como `Currency`, `R$`, 2 casas decimais.
5. Salve o campo.

### 2.3. Campo `% de economia`

1. Adicione um novo campo chamado `% de economia`.
2. Tipo `Formula`.
3. Cole a fórmula abaixo. O `IF` evita erro de divisão por zero quando o custo atual for 0:

   ```
   IF(
     {Custo mensal atual (R$)} = 0,
     0,
     ({Custo mensal atual (R$)} - {Custo mensal estimado após otimização (R$)}) / {Custo mensal atual (R$)}
   )
   ```

4. Em formatação, escolha `Percent` com 1 casa decimal. O Airtable multiplica por 100 automaticamente ao usar o formato de percentual, então **não** multiplique por 100 na fórmula.
5. Salve o campo.

### 2.4. Conferência

Use o registro fictício inicial (`LAB-001 — Otimização EBS`, custo atual R$ 100,00, estimado R$ 75,00):

- `Economia mensal (R$)` deve mostrar `R$ 25,00`.
- `Economia anual (R$)` deve mostrar `R$ 300,00`.
- `% de economia` deve mostrar `25,0%`.

Se os valores baterem, as fórmulas estão corretas.

---

## 3. Melhoria 2 — Prioridade automática

O objetivo é classificar cada oportunidade em `Alta`, `Média` ou `Baixa` com base na economia anual estimada, sem preenchimento manual.

### 3.1. Definir as faixas

Faixas sugeridas para o laboratório (ajuste conforme seu cenário didático):

| Prioridade | Critério (economia anual estimada) |
|---|---|
| Alta | Acima de R$ 1.000,00 |
| Média | De R$ 300,00 a R$ 1.000,00 |
| Baixa | Abaixo de R$ 300,00 |

### 3.2. Criar o campo `Prioridade`

1. Adicione um novo campo chamado `Prioridade`.
2. Tipo `Formula`.
3. Cole a fórmula:

   ```
   IF(
     (({Custo mensal atual (R$)} - {Custo mensal estimado após otimização (R$)}) * 12) > 1000,
     "Alta",
     IF(
       (({Custo mensal atual (R$)} - {Custo mensal estimado após otimização (R$)}) * 12) >= 300,
       "Média",
       "Baixa"
     )
   )
   ```

4. O resultado é texto, não precisa de formatação especial.
5. Salve o campo.

> Dica: se preferir reaproveitar o campo já criado, você pode trocar o cálculo da economia anual pela referência direta `{Economia anual (R$)}` dentro da fórmula. Fica mais legível:
>
> ```
> IF({Economia anual (R$)} > 1000, "Alta", IF({Economia anual (R$)} >= 300, "Média", "Baixa"))
> ```

### 3.3. Colorir a prioridade (opcional)

Para destacar visualmente, use a cor condicional na view (sem alterar os dados):

1. Com a tabela aberta, abra o menu `Color` (ícone de gota/paleta) na barra superior da view.
2. Escolha `Add condition`.
3. Crie três condições baseadas no campo `Prioridade`:
   - `Prioridade` é `Alta` → vermelho.
   - `Prioridade` é `Média` → amarelo.
   - `Prioridade` é `Baixa` → verde.
4. Salve.

### 3.4. Conferência

Com o registro `LAB-001` (economia anual R$ 300,00), a `Prioridade` deve ficar como `Média`, pois 300 se enquadra na faixa `>= 300`.

---

## 4. Melhoria 3 — Views de acompanhamento

As views reorganizam a mesma tabela de formas diferentes, sem duplicar dados. Vamos criar três.

### 4.1. View Kanban por `Status`

Mostra o fluxo das oportunidades como quadro de cartões.

1. No painel de views (à esquerda), clique em `+` para adicionar uma view.
2. Escolha o tipo `Kanban`.
3. Nomeie a view como `Fluxo por status`.
4. Quando solicitado o campo de agrupamento (*stack by*), escolha `Status`.
5. Cada coluna representará um status (`Nova`, `Em análise`, `Aprovada`, etc.). Arrastar um cartão entre colunas altera o status do registro.

### 4.2. View agrupada por `Serviço AWS`

Mostra onde estão concentradas as oportunidades.

1. Adicione uma nova view do tipo `Grid`.
2. Nomeie como `Por serviço AWS`.
3. Abra o menu `Group` na barra superior.
4. Agrupe por `Serviço AWS`.
5. Opcional: adicione um segundo agrupamento por `Prioridade` para ver, dentro de cada serviço, as oportunidades mais relevantes.
6. Opcional: no rodapé de cada grupo, configure um resumo (*summary*) do campo `Economia anual (R$)` como `Sum`, para somar a economia por serviço.

### 4.3. View ordenada por impacto

Ranking das oportunidades pela economia anual estimada.

1. Adicione uma nova view do tipo `Grid`.
2. Nomeie como `Ranking de impacto`.
3. Abra o menu `Sort`.
4. Ordene por `Economia anual (R$)`, de forma decrescente (*descending*, 9 → 1).
5. Opcional: use `Filter` para exibir só o que está em aberto, por exemplo `Status` diferente de `Rejeitada` e diferente de `Validada`.

### 4.4. Conferência

- A view `Fluxo por status` deve mostrar o registro `LAB-001` na coluna `Nova`.
- A view `Por serviço AWS` deve agrupar o registro em `EBS`.
- A view `Ranking de impacto` deve listar as oportunidades da maior para a menor economia anual.

---

## 5. Melhoria 4 — Dashboard (Interface)

As Interfaces do Airtable permitem montar uma página visual com números e gráficos a partir da tabela `Oportunidades`.

### 5.1. Criar a Interface

1. No topo da base, clique em `Interfaces`.
2. Clique em `Start building` / `Create interface`.
3. Se for oferecido um ponto de partida, escolha um layout de dashboard (ex.: `Dashboard` ou `Blank`).
4. Associe a interface à tabela `Oportunidades`.
5. Nomeie a interface como `FinOps Lab — Painel`.

### 5.2. Adicionar os números-chave (elementos de número)

Adicione elementos do tipo `Number` para indicadores de topo:

1. **Total de oportunidades**
   - Elemento `Number`.
   - Fonte: tabela `Oportunidades`.
   - Cálculo: `Count` de registros.
2. **Economia anual estimada total**
   - Elemento `Number`.
   - Campo: `Economia anual (R$)`.
   - Cálculo: `Sum`.
   - Formate como moeda, se a opção estiver disponível.
3. **Economia mensal estimada total**
   - Elemento `Number`.
   - Campo: `Economia mensal (R$)`.
   - Cálculo: `Sum`.

### 5.3. Adicionar os gráficos

1. **Oportunidades por status**
   - Elemento `Chart`, tipo barra ou pizza.
   - Agrupe (*X axis / group by*) por `Status`.
   - Métrica: contagem de registros.
2. **Economia anual por serviço AWS**
   - Elemento `Chart`, tipo barra.
   - Agrupe por `Serviço AWS`.
   - Métrica: `Sum` de `Economia anual (R$)`.
3. **Oportunidades por prioridade**
   - Elemento `Chart`, tipo pizza ou barra.
   - Agrupe por `Prioridade`.
   - Métrica: contagem de registros.

### 5.4. Adicionar a lista de oportunidades (opcional)

1. Adicione um elemento `Grid` ou `List`.
2. Fonte: tabela `Oportunidades`.
3. Exiba os campos principais: `Título`, `Serviço AWS`, `Prioridade`, `Economia anual (R$)`, `Status`.
4. Opcional: ordene por `Economia anual (R$)` decrescente para repetir o ranking de impacto no painel.

### 5.5. Publicar e conferir

1. Clique em `Publish` para disponibilizar a interface.
2. Confira se os números batem com os dados da tabela.
3. Com poucos registros fictícios, os gráficos ficarão simples. Para enriquecer o painel sem preencher o formulário dezenas de vezes, importe o conjunto de dados fictícios descrito na próxima seção.

---

## 6. Popular a tabela importando o CSV

Para que as fórmulas, as views e o dashboard fiquem visualmente úteis, a base precisa de volume de dados. O projeto inclui o arquivo `finops-lab-oportunidades.csv`, com 120 oportunidades fictícias prontas para importar.

### 6.1. Entender a estrutura do arquivo

O CSV contém apenas as nove colunas que são preenchidas na entrada, exatamente com os mesmos nomes dos campos da tabela `Oportunidades`:

- `Título`
- `Responsável`
- `E-mail para retorno`
- `Serviço AWS`
- `Recurso fictício`
- `Descrição da oportunidade`
- `Custo mensal atual (R$)`
- `Custo mensal estimado após otimização (R$)`
- `Status`

Os campos de fórmula (`Economia mensal`, `Economia anual`, `% de economia`, `Prioridade`) e o campo `Data de registro` (created time) **não** estão no arquivo de propósito: o Airtable os calcula sozinho. Se forem incluídos na importação, o Airtable os ignora ou gera erro.

> Pré-requisito: antes de importar, confirme que o campo `Serviço AWS` já tem as doze opções cadastradas (`EC2`, `EBS`, `S3`, `NAT Gateway`, `RDS`, `Lambda`, `DynamoDB`, `CloudWatch`, `Elastic IP`, `ELB`, `EKS`, `Outros`). O CSV usa todos esses valores. O mesmo vale para as opções de `Status`.

### 6.2. Importar como novos registros na tabela existente

O objetivo é adicionar os registros à tabela `Oportunidades` já existente, preservando os campos de fórmula.

1. Abra a base `FinOps Lab` e a tabela `Oportunidades`.
2. Na barra de abas das tabelas, clique no `+` (ou na seta ao lado do nome da tabela) e escolha a opção de importar dados.
3. Selecione a origem `CSV file`.
4. Envie o arquivo `finops-lab-oportunidades.csv`.
5. Quando o Airtable perguntar o destino, escolha inserir os dados na tabela `Oportunidades` existente (append), e não criar uma tabela nova.
6. Marque a opção que indica que a primeira linha do arquivo é o cabeçalho (*first row is header* / *use first row as field names*).

> Alternativa: você também pode usar `Add or import` dentro da própria tabela, quando a interface oferecer essa entrada. O importante é que o destino seja a tabela `Oportunidades`, em modo de adicionar registros.

### 6.3. Mapear as colunas

O Airtable tenta casar cada coluna do CSV com um campo da tabela pelo nome. Confira o mapeamento:

| Coluna do CSV | Campo da tabela | Tipo esperado |
|---|---|---|
| Título | Título | Single line text |
| Responsável | Responsável | Single line text |
| E-mail para retorno | E-mail para retorno | Email |
| Serviço AWS | Serviço AWS | Single select |
| Recurso fictício | Recurso fictício | Single line text |
| Descrição da oportunidade | Descrição da oportunidade | Long text |
| Custo mensal atual (R$) | Custo mensal atual (R$) | Currency / Number |
| Custo mensal estimado após otimização (R$) | Custo mensal estimado após otimização (R$) | Currency / Number |
| Status | Status | Single select |

Pontos de atenção no mapeamento:

- **Não mapeie** nada para `Economia mensal`, `Economia anual`, `% de economia`, `Prioridade` ou `Data de registro`. Como são campos calculados, eles não aparecem como destino de importação.
- **Single select (`Serviço AWS` e `Status`):** se o Airtable avisar que vai criar opções novas, verifique se não é por diferença de grafia. O ideal é que ele reaproveite as opções já cadastradas. Se surgir uma opção duplicada (por acento ou espaço extra), cancele, ajuste o campo e importe de novo.
- **Custos:** confirme que chegam como números (`1200.00`, `650.00`) e não como texto. O arquivo usa ponto como separador decimal.

### 6.4. Confirmar a importação

1. Conclua a importação.
2. Confira que a tabela ganhou 120 novos registros (além do `LAB-001` inserido manualmente, se ainda existir).
3. Abra alguns registros e verifique se os campos de fórmula foram calculados automaticamente: `Economia mensal`, `Economia anual`, `% de economia` e `Prioridade`.
4. Abra as views criadas (`Fluxo por status`, `Por serviço AWS`, `Ranking de impacto`) e confirme que os dados aparecem distribuídos.
5. Abra a Interface `FinOps Lab — Painel` e confira se os números e gráficos agora refletem o volume maior de dados.

### 6.5. Sobre o separador decimal

O CSV usa ponto como separador decimal (padrão internacional). Se a sua conta Airtable estiver configurada para interpretar vírgula como decimal e os custos chegarem errados (por exemplo, `1200.00` virando `120000`), ajuste a configuração regional ou gere uma versão do arquivo com vírgula. O script `gerar_oportunidades.py`, incluído no projeto, permite regenerar o CSV com ajustes.

---

## 7. Checklist de validação

- [ ] Campo `Economia mensal (R$)` criado e exibindo valor correto.
- [ ] Campo `Economia anual (R$)` criado e exibindo valor correto.
- [ ] Campo `% de economia` criado e formatado como percentual.
- [ ] Campo `Prioridade` criado e classificando corretamente.
- [ ] Cores condicionais aplicadas à prioridade (opcional).
- [ ] View `Fluxo por status` (Kanban) criada.
- [ ] View `Por serviço AWS` (agrupada) criada.
- [ ] View `Ranking de impacto` (ordenada) criada.
- [ ] Interface `FinOps Lab — Painel` criada com números e gráficos.
- [ ] CSV `finops-lab-oportunidades.csv` importado na tabela `Oportunidades`.
- [ ] Campos de fórmula recalculados automaticamente após a importação.
- [ ] Painel publicado e conferido com o volume maior de dados.

---

## 8. Observações e boas práticas

- Os campos de fórmula são somente leitura: o Airtable os recalcula sozinho e eles não devem ser preenchidos pelo Activepieces nem pelo formulário Tally.
- Como esses campos são derivados dos custos, o fluxo **Tally → Activepieces → Airtable** não precisa de alteração. Basta que `Custo mensal atual` e `Custo mensal estimado` cheguem como números.
- Mantenha o princípio do laboratório: todos os valores são estimativas didáticas, não economia efetivamente realizada.
- As faixas de prioridade e os formatos de moeda podem ser ajustados conforme o cenário de ensino.

---

**Versão do documento:** melhorias no Airtable (fórmulas de economia, prioridade automática, views, dashboard e importação do CSV de oportunidades), complementando o guia principal do FinOps Lab.
