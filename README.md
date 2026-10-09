# Dashboard de Vendas, Logística e Satisfação (Olist) | Power BI

Painel de indicadores em 3 páginas sobre ~100 mil pedidos do e-commerce brasileiro **Olist** (2016–2018), construído em Power BI com ETL no Power Query, modelagem relacional e medidas em DAX.

O foco do projeto é responder perguntas de negócio, não só mostrar números:

1. **Vendas:** como a receita evolui e o que mais vende?
2. **Logística:** as entregas estão cumprindo o prazo prometido?
3. **Satisfação:** o atraso na entrega afeta a nota do cliente?

---

## Prévia

### Vendas
![Dashboard de Vendas](docs/dashboard-vendas.png)

### Logística
![Dashboard de Logística](docs/dashboard-logistica.png)

### Cliente (Satisfação)
![Dashboard de Cliente](docs/dashboard-cliente.png)

---

## Principais achados

- **Crescimento e concentração:** a receita total foi de cerca de R$ 13,6 milhões, com ticket médio de ~R$ 138. As vendas crescem ao longo de 2017 e se estabilizam em torno de R$ 1 milhão por mês a partir de nov/2017. São Paulo concentra aproximadamente 38% da receita.
- **Pedidos:** 97% dos pedidos aparecem como entregues; o restante se divide entre enviado, cancelado, indisponível e outros status.
- **Atraso é regional:** 8,11% dos pedidos entregues chegaram depois da data estimada, mas o problema se concentra no Nordeste (AL com ~24% e MA com ~20%).
- **Prazo prometido x tempo real:** estados do Norte (AP, AM, RR) têm os maiores tempos médios de entrega (26 a 29 dias), mas os prazos prometidos lá já são longos, então o atraso em si é baixo em AP e AM.
- **Atraso derruba a nota:** no gráfico de dispersão por estado, quanto maior o % de atraso, menor a nota média (de ~4,2 nos estados com menos atraso para ~3,75 em AL e MA).
- **Satisfação geral:** nota média de 4,09, com 57,8% de avaliações nota 5.

---

## Modelo de dados

Modelo relacional com a tabela de **pedidos** e a de **itens do pedido** no centro, ligadas a clientes, vendedores, produtos, avaliações, pagamentos e a uma **tabela calendário** (criada em DAX e marcada como tabela de datas).

![Modelo de dados](docs/modelo-de-dados.png)

Decisões de modelagem:

- A **receita** é calculada pela tabela de itens (`price`) e não pela de pagamentos, porque um pedido pode ter vários pagamentos (parcelas, vouchers).
- Clientes são contados por `customer_unique_id`, já que o `customer_id` muda a cada pedido.
- A tabela de **geolocalização foi removida do modelo**: ela tem várias linhas por CEP e criava caminhos ambíguos entre tabelas. Estado e cidade já existem nas tabelas de clientes e vendedores.
- Pedidos com datas de entrega nulas (em processamento, enviados, cancelados) foram **mantidos**, pois são informação. As medidas de logística filtram apenas pedidos entregues.

---

## Tratamentos no Power Query (ETL)

- Importação dos CSVs e ajuste de tipos de dados (datas e números).
- Correção do separador decimal em `price`, `freight_value` e `payment_value` (conversão com localidade Inglês/EUA, pois o CSV usa ponto como decimal).
- Remoção de colunas desnecessárias (como CEPs e campos usados apenas para geolocalização).
- Criação de colunas de prazo: `days_to_delivery` (dias entre a compra e a entrega) e `is_late` (entrega posterior à data estimada).
- Tradução/padronização de status dos pedidos.

---

## Medidas em DAX

Todas as medidas ficam em uma tabela dedicada (`Medidas`). Exemplos:

```dax
Receita = SUM(olist_order_items_dataset[price])

Ticket Médio =
DIVIDE([Receita], DISTINCTCOUNT(olist_order_items_dataset[order_id]))

Pedidos Atrasados =
CALCULATE(SUM(olist_orders_dataset[is_late]), olist_orders_dataset[order_status] = "delivered")

% Atrasados = DIVIDE([Pedidos Atrasados], [Pedidos Entregues])

Nota Média = AVERAGE(olist_order_reviews_dataset[review_score])

Receita Mês Anterior =
CALCULATE([Receita], DATEADD(Calendario[Date], -1, MONTH))

Crescimento MoM =
DIVIDE([Receita] - [Receita Mês Anterior], [Receita Mês Anterior])
```

> As medidas de nota por categoria de produto usam `TREATAS` para propagar o filtro de itens até as avaliações, já que o caminho produtos → itens → pedidos → avaliações não flui naturalmente pela direção dos relacionamentos.

---

## Limitações conhecidas

- Estados com poucos pedidos (RR, AP, AC) geram pontos extremos no gráfico de dispersão, sem significado estatístico.
- Categorias com poucas avaliações aparecem com nota 5,0 na página de satisfação.
- Uma avaliação é associada a todas as categorias dos itens do pedido, o que pode repetir a mesma nota em mais de uma categoria (caso raro no Olist).
- Os dados de 2016 e do fim de 2018 são muito esparsos, o que afeta os extremos dos gráficos temporais.

---

## Próximos passos

- [ ] Segmentações (ano, estado, categoria) sincronizadas entre as páginas e navegação entre páginas.
- [ ] Corrigir/ajustar o gráfico de crescimento mensal e a ordenação cronológica dos gráficos temporais.
- [ ] Filtrar categorias e estados com pouco volume nos rankings.
- [ ] **Modelo de machine learning (scikit-learn)** para prever atraso na entrega, com integração das previsões ao dashboard.

---

## Como abrir

1. Instale o [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (gratuito, apenas Windows).
2. Baixe o dataset no Kaggle: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce).
3. Abra o arquivo `dashboard-olist.pbix` e, se necessário, atualize os caminhos das fontes de dados em **Transformar dados > Configurações da fonte de dados**.

Se não quiser instalar nada, as imagens estão na pasta `docs/`.

---

## Estrutura do repositório

```
.
├── README.md
├── dashboard-olist.pbix
└── docs/
    ├── dashboard-vendas.png
    ├── dashboard-logistica.png
    ├── dashboard-cliente.png
    └── modelo-de-dados.png
```

---

## Tecnologias

Power BI · Power Query (M) · DAX

## Dados

Dataset público da Olist, disponível no Kaggle sob licença CC BY-NC-SA 4.0. Os dados originais **não** estão incluídos neste repositório.

## Autor

**Lucas da Costa Paula**
[LinkedIn](https://linkedin.com/in/lucascostapaula/) · [GitHub](https://github.com/Jialke)
