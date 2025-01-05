# Dashboard de análise de vendas e estoque de uma empresa de varejo

***(OBS: No fim desse readme, você verá as imagem dos dashboard e do Schema das tabelas)***

## Para esse projeto foram usadas as seguintes fórmulas dax:

### 1. Fórmula para calcular a cobertura do estoque:

Calcula a relação entre a quantidade de produtos em estoque e a média de vendas por mês de um produto. Utiliza `COALESCE` para garantir que, se o valor for `nulo`, seja retornado `0`.

```
CoberturaEstoque = 
COALESCE(
    CALCULATE(
        DIVIDE(
            [QuantidadeEmEstoque], 
            [MediaVendasPorMêsProduto]
        )
    ),
    0
)
```
### 2. Fórmulas de cores condicionais:

Verde (#00FF00) se o valor for positivo.
Vermelho (#FF0000) se o valor for negativo.
Branco (#FFFFFF) se o valor for zero.

```
CorCondicionalCustoAnoAnterior = 
SWITCH(
    TRUE(),
    [CustoAnoAnterior] > 0, "#00FF00",
    [CustoAnoAnterior] < 0, "#FF0000",
    "#FFFFFF"
)
```
```
CorCondicionalMargemLucroAnoAnterior = 
SWITCH(
    TRUE(),
    [MargemLucroAnoAnterior] > 0, "#00FF00",
    [MargemLucroAnoAnterior] < 0, "#FF0000",
    "#FFFFFF"
)
```
```
CorCondicionalReceitaAnoAnterior = 
SWITCH(
    TRUE(),
    [ReceitaAnoAnterior] > 0, "#00FF00",
    [ReceitaAnoAnterior] < 0, "#FF0000",
    "#FFFFFF"
)
```
### 3. Fórmula do custo do ano anterior:

Calcula a variação do CustoTotal comparando com o CustoTotal do mesmo período do ano anterior de acordo com o ano do filtro. Retorna 0 se não houver dados para comparação (pois pode ser o primeiro ano que possuímos dados).

```
CustoAnoAnterior = 
VAR VariavelCustoAnoAnterior =
    IF(
        HASONEVALUE(dData[DATA DE VENDA].[Ano]),
        CALCULATE(
            [CustoTotal],
            SAMEPERIODLASTYEAR(dData[DATA DE VENDA])
        ),
        BLANK()
    )
RETURN
    IF(
        NOT ISBLANK(VariavelCustoAnoAnterior) && VariavelCustoAnoAnterior <> 0,
        ([CustoTotal] / VariavelCustoAnoAnterior) - 1,
        0
    )
```
### 4. Fórmula do custo total:

Calcula o custo total de vendas multiplicando a quantidade de itens vendidos pelo custo unitário. Utiliza COALESCE para garantir que, se o valor for nulo, seja retornado 0.

```
CustoTotal = 
COALESCE(
    CALCULATE(
        SUMX(
            fVendas, 
            fVendas[Quantidade] * fVendas[Custo Unitario]
        )
    ),
    0
)
```

### 5. Fórmulas dos ícones mostrando aumento ou queda:

Define um ícone baseado no valor do mesmo periodo do ano anterior como:

▲ para valores positivos.
▼ para valores negativos.

```
IconeCustoAnoAnterior = 
SWITCH(
    TRUE(),
    [CustoAnoAnterior] > 0, "▲",
    [CustoAnoAnterior] < 0, "▼",
    ""
)
```

```
IconeMargemLucroAnoAnterior = 
SWITCH(
    TRUE(),
    [MargemLucroAnoAnterior] > 0, "▲",
    [MargemLucroAnoAnterior] < 0, "▼",
    ""
)
```
```
IconeReceitaAnoAnterior = 
SWITCH(
    TRUE(),
    [ReceitaAnoAnterior] > 0, "▲",
    [ReceitaAnoAnterior] < 0, "▼",
    ""
)
```

### 6. Fórmula para margem de lucro:

Calcula a  margem de lucro subtraindo o custo total da receita total.

```
MargemLucro = 
COALESCE(
    CALCULATE(
        [ReceitaTotal] - SUMX(
                            fVendas, 
                            fVendas[Quantidade] * fVendas[Custo Unitario]
                        )
    ),
    0
)
```

### 7. Fórmula para margem de lucro do mesmo periodo do ano anterior:

```
MargemLucroAnoAnterior = 
VAR VariavelMargemLucroAnoAnterior =
    IF(
        HASONEVALUE(dData[DATA DE VENDA].[Ano]),
        CALCULATE(
            [MargemLucro],
            SAMEPERIODLASTYEAR(dData[DATA DE VENDA])
        ),
        BLANK()
    )
RETURN 
    IF(
        NOT ISBLANK(VariavelMargemLucroAnoAnterior) && VariavelMargemLucroAnoAnterior <> 0,
        ([MargemLucro] / VariavelMargemLucroAnoAnterior) - 1,
        0
    )
```

### 8. Fórmula para calcular a média de vendas mensais de um produto e com isso econtrarmos a cobertura de estoque:

```
MediaVendasPorMêsProduto = 
COALESCE(
    AVERAGEX(
        VALUES(dData[MES ANO DATA DE VENDA]),
        [QuantidadeVendida]
    ),
    0
)
```

### 9. Fórmula para calcular o percentual da margem de lucro, dividindo o valor da margem de lucro pela Receita total:

```
PercentMargemLucro = 
COALESCE(
    CALCULATE(
        DIVIDE(
            [MargemLucro],
            [ReceitaTotal],
            0
        )
    ),
    0
)
```

### 10. Fórmula para calcular a quantidade de estoque:

```
QuantidadeEmEstoque = 
COALESCE(
    CALCULATE(
        SUMX(
            dEstoque,
            dEstoque[Qtd Estoque]
        )
    ),
    0
)
```

### 11. Fórmula para calcular quantidade vendida:

```
QuantidadeVendida = 
COALESCE(
    CALCULATE(
        SUMX(
            fVendas, 
            fVendas[Quantidade]
        )
    ),
    0
)
```

### 12. Fórmula para calcular a receita do mesmo período no ano anterior:

```
ReceitaAnoAnterior = 
VAR VariavelReceitaAnoAnterior =
    IF(
        HASONEVALUE(dData[DATA DE VENDA].[Ano]),
        CALCULATE(
            [ReceitaTotal],
            SAMEPERIODLASTYEAR(dData[DATA DE VENDA])
        ),
        BLANK()
    )
RETURN 
    IF(
        NOT ISBLANK(VariavelReceitaAnoAnterior) && VariavelReceitaAnoAnterior <> 0,
        ([ReceitaTotal] / VariavelReceitaAnoAnterior) - 1,
        0
    )
```


### 13. Fórmula para calcular a receita total de venda:
```
ReceitaTotal = 
COALESCE(
    CALCULATE(
        SUMX(
            fVendas,
            fVendas[Quantidade] * fVendas[Valor Unitario]
        )
    ),
    0
)
```








