# Tratamento-de-dados-utilizando-R

O presente texto tem como objetivo demonstrar o processo de tratamento de dados em R, utilizando a biblioteca padrão da linguagem.

A primeira coisa que devemos notar é que dados devem seguir alguns parâmetros para que possam ser submetidos à análise, sendo eles:

- Consistência
- Duplicidades
- Consistência
- Completude
- Conformidade
- Integridade

Todas esses possíveis problemas podem ser danosos caso a base de dados não sofra nenhum tratamento antes de ser exposta a um modelo ou a uma análise de dados, visto que todos esses fatores podem gerar distorções no resultado final.

## Explorando os dados e estabelecendo parâmetros

Primeiramente, vamos explorar o conjunto de dados e verificar se os dados estão em conformidade com os princípios listados anteriormente e com a regra de negócio da empresa. 
Existem algumas funções que podem nos auxiliar nesse processos, são elas: 

- Head()
- Summary()
- Colnames()

Utilizaremos a função Head() para verficarmos os primeiros dados:

```r
head(dados)
```
![Função Head()](https://github.com/user-attachments/assets/00e16b5b-9bab-4c6e-8901-47f2e7eb295d)

Utilizaremos a função Summary() para verificarmos o primeiro quartil, terceiro quartil, moda, mediana, média, valores máximos e mínimos (esses parâmetros variam conforme a categoria dos dados, sendo classificados em quantitativos e qualitativos)
```r
summary(dados)
```
![Função Sumarry](https://github.com/user-attachments/assets/29dd7782-0117-4fce-989e-6e3cfea5630a)

As informações foram resumidas e podemos ter uma noção melhor delas, entretanto, há um problema bem claro, as tabelas retornadas não são de facil entendimento, pois não sabemos os nomes das colunas, para resolver isso, utilizaremos a função "Colnames()" e, logo em seguida, verificaremos o resultado com a função "Summarry()"

```r
colnames(dados) = c("id", "score", "Estado", "Gênero", "Idade", "Patrimônio", "Saldo", "Produtos", "TemCartCrédito", "Ativo", "Salario", "Saiu") 
```
![Função Colnames()](https://github.com/user-attachments/assets/4b9c1b05-cbc0-49ce-bc2d-6ffe412ded3e)

Logo de cara percebemos alguns problemas: 

- A idade máxima está em 140 anos
- A idade mínima está negativa
- A coluna de "gêneros" está com problemas de consistência, usando diversas nomeclaturas para uma mesma coisa ("F", "Feminino", "M", "Masculino")
- Algumas colunas estão com valores "NA's"

Agora que notamos os erros presentes no conjunto de dados, devemos nos atentar a regra de negócio da empresa, vamos fazer a seguinte suposição: **a empresa trabalha apenas na região sudeste. Com isso em mente, surge um outro problema, se a empresa trabalha apenas na região sudeste, por quê existem dados de outras regiões? Também precisaremos corrigir os dados da coluna "Estados".** Levando em consideração todos esses fatores, podemos finalmente começar a executar mudanças no conjunto de dados, visando a mínima distorção nas informações, bem como a preservação das mesmas.

### Iniciando o processo de tratamento de dados - Coluna "gêneros"

Utilizando um gráfico, podemos visualizar o problema com mais facilidade

```r
x = table(dados$Gênero)
barplot(x)
```

![Gráfico](https://github.com/user-attachments/assets/c58bccdc-fc03-47cb-b90e-584ae32605b4)

Para resolvermos o problema acima, igualaremos todas os dados em um padrão, sendo: 

- "F" --> "Feminino"
- "Feminino" --> Se manterá "Feminino"
- "M" --> "Masculino"
- "Masculino" --> Se manterá "Masculino"

Ou seja, apenas tornaremos os dados consistentes, condensando todos em duas definições.

Removendo os dados: 

```r
dados[dados$Gênero == "M",] = "Masculino"
dados[dados$Gênero == "F",] = "Feminino"
dados$Gênero[is.na(dados$Gênero)] = "Masculino" #Alterando os valores NA's para a moda
dados$Gênero = factor(dados$Gênero) # Removendo os níveis sem dados que sobraram após o processo

```

Criando o gráfico

```r
x = table(dados$Gênero)
cores = c("pink", "blue")
barplot(x,  space = 1.8, width = 5, border = TRUE, col = cores)
abline(h = 0, col = "black")  
```
Resultado:

![Gráfico 2 exemplo](https://github.com/user-attachments/assets/630985dd-4ac4-4240-9282-4d557e5731d6)

Agora que tornamos os dados da coluna "$Gênero" condensados em apenas duas possibilidades, não há mais problemas nessa coluna.

## Continuidade do processo de tratamento de dados - Coluna "idade"

A coluna idade possui alguns problemas também, podemos visualizá-los claramente utilizando um gráfico:

Criando gráfico:

```r
mediana = median(dados$Idade, na.rm = T)
y = table(dados$Idade)
plot(y, type = "l", xlab = "idade", lty = 1)
abline(v = mediana, col = "red", lwd = 2, lty = 2)  #Linha vertical (mediana)
```

![Gráfico 3](https://github.com/user-attachments/assets/5b517011-36ed-4e27-aebe-99428a7f3ac7)

Em uma percepção inicial, o gráfico não está agradável aos olhos, mas corrigiremos isso em breve, isso ocorre devido a distorção dos dados, repare, a idade máxima é 140 e a mínima é negativa, condições que não condizem com a realidade. Podemos corrigir isso da seguinte forma:

```r
dados_na = dados$Idade > 110 | dados$Idade < 0
dados$Idade[dados_na] = median(dados$Idade, na.rm = T)
```

Com isso, alteramos as idades que são maiores que 110 ou menores que zero, atribuindo a mediana da coluna "$Idade" para os valores que atendem esse requisito citado. Sendo assim, ao chamarmos a função que cria o gráfico novamente, veremos algumas alterações interessantes:

![Screenshot_1](https://github.com/user-attachments/assets/109f5b51-54f1-4da2-831c-f9818a440385)

Conferindo os dados: 

```r
summary(dados$idade)
```

![image](https://github.com/user-attachments/assets/95887b8e-a5cc-4de2-b89a-30a013c4af4d)

Não há mais idades que ultrapassam com uma margem enorme a expectativa de vida e também não há idades negativas, ou seja, não existem mais "outliers". Porém, ainda existem dois valores nulos, para resolvermos isso, simplesmente vamos alterá-los para a mediana, como feito anteriormente com as idades consideradas outliers:

```r
dados[!complete.cases(dados$Idade),] = median(dados$Idade, na.rm = T)
```

Conferindo os dados: 

![image](https://github.com/user-attachments/assets/b86f0791-89f2-4f7b-b075-eb781ec5aadb)

Não há mais valores que possam gerer algum tipo de distorção, portanto, o tratamento de dados da coluna "idade" está finalizado.

## Tratamento de dados - manipulando os valores "NA's"

A forma de lidar com esses dados varia conforme a categoria dos dados, sendo:

- Dados categóricos: alteramos os valores "NA's" para a moda 
- Dados quantitativos: alteramos os valores "NA's" para a mediana

Essas medidas garantem a remoção dos dados "NA's", entretanto, não geram alterações na soma total da quantidade de dados dentro do conjunto de d
ados, permitindo a análise de dados sem grandes distorções nos resultados finais de modelos/constatações.

## Tratamento de dados - Coluna "Estados"

Como dito anteriormente, precisamos remover todos os Estados que não pertecem a região sudeste, portanto:

```r
summary(dados$Estado)

estados_permitidos = c("SP", "RS", "PR", "SC")

dados$Estado[!dados$Estado %in% estados_permitidos] = "RS"

summary(dados$Estado)

dados$Estado = factor(dados$Estado)
```

Conferindo:

```r
summary(dados$Estado)
```

![image](https://github.com/user-attachments/assets/032f71f0-5e98-4f20-bbc1-c826b1706f45)


## Tratamento de dados - Coluna "Salário"

Os salários estão com valores "NA's", para resolver esse problema, vamos alterar esses dados para a mediana da coluna. 

![Função Sumarry](https://github.com/user-attachments/assets/29dd7782-0117-4fce-989e-6e3cfea5630a)

Corrigindo:

```r
dados$Salario[is.na(dados$Salario)] <- median(dados$Salario, na.rm = TRUE)
```

## Tratamento de dados - Coluna "ID's"

Aplicando:

```r
z = dados[duplicated(dados$id),]
dados = dados[!dados$id %in% c(z$id), ]

z = dados[duplicated(dados$id),]
```



