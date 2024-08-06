# Analise_Vendas
Análise exploratória das vendas de uma empresa fictícia no ramo produtos eletrônicos com foco em identificar os principais caracteristicas de vendas da empresa.

A empresa possui 160 funcionários em 8 lojas distribuidas pelo Brasil.

A partir dos dados, observamos que a marca mais vendida é a SONY (93,32%), porém é a marca com menor margem unitária.
Nota-se que a estratégia da empresa está concentrada em escalabilidade das vendas.
A margem da empresa depende muito da receita da marca SONY (87,79% margem total).

DELL e ALTURA são as marcars com as maiores margens.
SONY e LOGITECH são as menores.

Quanto as lojas:
Belo Horizonte possui a maior margem seguida de São Paulo.

O gerente que obeteve o maior volume de vendas foi Rogéro Lopes, com 58 pedidos da loja de São Paulo.

Quanto aos funcionários:
Loja de Belo Horizonte possui a melhor média por funcionário e São Paulo a pior.

## Recomendação a partir dos dados

Atualmente, a empresa depende muito da SONY, que embora tenha alto volume de vendas, oferece uma margem unitária baixa. Diversificar e promover marcas como DELL e ALTURA, que possuem maiores margens, pode equilibrar a dependência e aumentar a lucratividade.

Criar pacotes promocionais que combinem produtos da SONY com marcas de maior margem para incentivar os clientes a comprar itens adicionais com melhores margens.

Analisar as práticas das lojas de Belo Horizonte (com melhor desempenho) e implementar estratégias similares em São Paulo e outras localidades com margens menores.

Essas são algumas ações que empresa pode fazer para aumentar tanto as vendas quanto a margem de lucro, ao mesmo tempo em que reduz a dependência de uma única marca e melhora o desempenho geral.

Destaco que são apenas algumas recomendações e que não descarta a necessidade de fazer outras melhorias que não estão em pautas como automatizações, controle de estoque, análise dos custos etc.

Os códigos em linguagem SQL desenvolvido em MYSQL usados para responder as perguntas estão abaixo:

## 1.	Quais estados e cidades a empresa possui lojas?
select * from banco.locais;

## 2. Quantidade de Lojas
SELECT count(ID_Loja) AS 'Qtd_Lojas' FROM banco.lojas;

## 3. QUAIS ESTADOS E CIDADES ESTÃO AS LOJAS?
SELECT B.região, A.Loja AS 'Cidade_Loja', B.Estado
FROM banco.lojas A
INNER JOIN banco.locais B ON A.Loja = B.Cidade
ORDER BY B.região, B.Estado ASC;

## 4. Qual o total de funcionários? 
SELECT SUM(A.Num_Funcionarios) AS 'Qtd_Funcionarios'
FROM banco.lojas A;
## 5. Quantidade Funcionários por estado
SELECT B.Estado, SUM(A.Num_Funcionarios) AS 'Qtd_Funcionarios'
FROM banco.lojas A
INNER JOIN banco.locais B ON A.Loja = B.Cidade
GROUP BY B.Estado
ORDER BY SUM(A.Num_Funcionarios) DESC;


## 6. Qual a região com mais funcionários?
SELECT 
    B.Região, 
    SUM(A.Num_Funcionarios) AS 'Qtd_Funcionarios',
    ROUND((SUM(A.Num_Funcionarios) / (SELECT SUM(Num_Funcionarios) FROM banco.lojas)) * 100, 2) AS 'Percentual_Total'
FROM 
    banco.lojas A
INNER JOIN 
    banco.locais B ON A.Loja = B.Cidade
GROUP BY 
    B.Região
ORDER BY 
    SUM(A.Num_Funcionarios) DESC;

    
## 7. Qual a média de venda por funcionário?
SELECT 
    A.ID_Loja,
    A.Loja,
    A.Num_Funcionarios AS 'Qtd_Funcionarios', 
    COUNT(B.ID_PEDIDO) AS 'Qtd_Pedidos',
    ROUND((COUNT(B.ID_PEDIDO) / A.Num_Funcionarios), 2) AS 'Media_Pedidos',
    ROUND(AVG(COUNT(B.ID_PEDIDO) / A.Num_Funcionarios) OVER (), 2) AS 'Media_Geral',
    CASE 
        WHEN (COUNT(B.ID_PEDIDO) / A.Num_Funcionarios) >= AVG(COUNT(B.ID_PEDIDO) / A.Num_Funcionarios) OVER () 
        THEN 'SIM' 
        ELSE 'NÃO' 
    END AS 'Acima_Media'
FROM 
    banco.lojas A
INNER JOIN 
    banco.pedidos B ON A.ID_LOJA = B.ID_LOJA
GROUP BY 
    A.ID_Loja, A.Loja, A.Num_Funcionarios
ORDER BY Acima_Media DESC;

## 8. Qual a loja que mais vendeu? Qual o nome do gerente?
SELECT 
    A.Loja, A.Gerente, COUNT(B.ID_PEDIDO) AS TotalPedidos
FROM 
    banco.lojas A
INNER JOIN
    banco.pedidos B ON A.ID_LOJA = B.ID_LOJA
GROUP BY
    A.Loja, A.Gerente
ORDER BY     
	COUNT(B.ID_PEDIDO) DESC;


## 9. Qual a margem cada loja entregou em produtos vendidos?
SELECT 
    A.Loja, 
    A.Gerente, 
    COUNT(B.ID_PEDIDO) AS TotalPedidos, 
    SUM(B.Receita_Venda) AS Receita, 
    SUM(B.Custo_Unit) AS Custo_Unitario, 
    SUM(B.Receita_Venda - B.Custo_Unit) AS Margem
FROM
    banco.lojas A
INNER JOIN
    banco.pedidos B ON A.ID_LOJA = B.ID_LOJA
GROUP BY
    A.Loja, A.Gerente
ORDER BY     
    Margem DESC; 
    
## 10. Quais produtos temos na empresa? 
SELECT 
    *
FROM
    banco.produtos;
    
## 11. Quais as marcas possuem as melhores margens?
SELECT 
    A.Marca_Produto,
    ROUND(AVG(Preco_Unit), 2) as 'Preço',
    ROUND(AVG(Custo_Unit), 2) as 'Custo',
    ROUND((AVG(Preco_Unit) - AVG(Custo_Unit)), 2) AS 'Margem'
FROM
    banco.produtos A
GROUP BY 
    A.Marca_Produto
ORDER BY
    Margem DESC;     

## 12. Qual marca de produto a empresa mais vende e qual margem?
SELECT 
    A.Marca_Produto,
    COUNT(B.ID_Produto) AS 'Pedidos',
    ROUND((COUNT(B.ID_Produto) / (SELECT COUNT(*) FROM banco.pedidos) * 100), 2) AS 'Percentual_Pedidos',
    ROUND((AVG(A.Preco_Unit) - AVG(A.Custo_Unit)), 2) AS 'Margem_Unit',
    ROUND((COUNT(B.ID_Produto) * ROUND((AVG(A.Preco_Unit) - AVG(A.Custo_Unit)), 2)), 2) AS 'Margem_Total',
    ROUND((ROUND((COUNT(B.ID_Produto) * ROUND((AVG(A.Preco_Unit) - AVG(A.Custo_Unit)), 2)), 2) / SUM(ROUND((COUNT(B.ID_Produto) * ROUND((AVG(A.Preco_Unit) - AVG(A.Custo_Unit)), 2)), 2)) OVER ()*100), 2) AS 'Percentual_Margem_Total'
FROM
    banco.produtos A
LEFT JOIN
    banco.pedidos B ON A.ID_Produto = B.ID_Produto
GROUP BY 
    A.Marca_Produto
ORDER BY
    Margem_Total DESC;



