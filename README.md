## Um pouco de contexto

O plano gratuito da [Leads2b](https://leads2b.com/) foi lançado publicamente em Fevereiro de 2020 e 7 meses depois, no início de Agosto, cerca de 15 mil pessoas já haviam criado uma conta.

Como pesquisador e designer no time que projetou esse novo produto, precisávamos entender como as pessoas estavam engajando com ele, e quais eram os principais perfils.

Em Maio fizemos uma pesquisa de profundidade mais qualitativa, cruzando as métricas de uso do produto com um questionário disparado para toda a base e entrevistas de profundidade com diferentes perfils de usuários ativos.

Escolhemos o Mixpanel para ser nosso centralizador de métricas de produto, mas no início sofremos para aprender como usar ele corretamente e só começamos a ter uma série histórica limpa e consistente no início de Maio.

É essa série histórica entre o início de maio e o fim de agosto que vamos analisar aqui.

## O que queremos descobrir?

Quais são os principais padrões de engajamento? E quanto da base cada grupo representa?

Desde o início tivemos dificuldade de entender o engajamento dos usuários com o produto. Por que algumas pessoas usam tanto e outra tão pouco? Por que algumas entram todos os dias e depois somem? Por que algumas entram uma única vez e desaparecem?

Temos algumas hipóteses de perfils a partir dos dados que vemos no dia-a-dia, por exemplo:

- Quem se registra, usa muito, e não volta nunca mais.
- Quem usa apenas alguns dias por mês, mas volta todo mês.
- Quem usa bastante, tem um hiato de meses, e depois volta a usar.
- Quem começa muito engajado e aos poucos vai parando.

Mas esses são mesmo os principais perfils comportamentais? E quantos usuários temos em cada grupo? E quais são os números específicos? Quantos dias por mês voltam? Quandos eventos fazem?

Essas informações são bastante difíceis de extrair com as ferramentas que temos atualmente no Mixpanel.

## Então o que vamos fazer?

Vimos que a metodologia RFM, que calcula Recência, Frequência e Monetização é um tanto próxima do que queremos saber sobre os nossos usuários, então bolamos uma versão alterada dele focado não em compras mas em uso, o RFDV:

- Recência, a última vez que usou o produto
- Frequência, em quantos dias distintos usou o produto
- Duração, por quanto tempo usou o produto, ou o tempo entre o primeiro e último acesso
- Volume, o número de ações de engajamento que fez durante todo o uso

E nesses dados vamos aplicar um algorítmo que usa álgebra linear para agrupar usuários que tenham comportamentos similares.

## Fase 1: Importando os dados

*Essa etapa não tem link para o código por que ele tem dados sensíveis e eu gosto do meu emprego.*

Primeiro, fomos coletar os dados brutos. Para a minha sorte o trabalho mais difícil já foi feito a base de dor e sofrimento na própria estratégia de metrificação do produto e implementação do Mixpanel, então só precisou baixar e limpar.

Tive que baixar e instalar a terrivelmente defasada API do Mixpanel em Python 2.0 (!), e fazer duas chamadas.

Primeiro baixando todos os perfils de usuários do Plano Free, e na sequência o registro de doze tipos de eventos feitos por esses usuários entre 3 de Maio e 8 de Agosto, sendo eles:

1. Busca por uma empresa pelo CNPJ
2. Busca por uma empresa pelo nome
3. Click para ver uma empresa similar a buscada
4. Adicionar uma empresa ao funil de vendas
5. Visita à página de funil de vendas
6. Visita a página de Planos

e etc...

Por que esses eventos? São os que demonstram uma interação humana significativa com produto, diferente de um evento como "logout" ou "fechar banner", além de que somadas são quase 98% de todos os eventos coletados.

**Melhoria futura**: aqui baixamos todos os perfils de usuários no plano Free, porém isso significa que usuários que em algum momento da série histórica migraram para um plano pago não vai ser contabilizados. Isso é ruim pois seria bom saber qual foi o comportamento dos usuários antes de virarem pagantes.

## Fase 2: Preparando os dados

Dizem que a parte mais trabalhosa na ciência de dados é preparar os dados, e estão corretíssimos pois essa foi mesmo a parte mais longa e sofrida do processo. 

**Limpeza dos Dados**

Logo após a exportação de dados via API do Mixpanel, fiz uma limpeza:

- removi todos os usuários que se registraram antes ou depois da série histórica (5 de Maio até 8 de Agosto)
- removi as colunas de perfil com dados sensíveis (nome, telefone, email, empresa), ou irrelevantes (respostas dadas em campos de signup que não existem mais)
- removi usuários que tivessem emails da @leads2b, para não incluir na análise nossas próprias contas de teste
- na tabela de eventos consolidados deixei apenas as colunas com o ID do usuário, o nome e a data do evento, e descartei todas as outras 20 e tantas que continham em qual navegador ele estava quando fez aquele evento, quantos créditos tinha, etc.

Teoricamente tudo isso poderia ter sido feito direto na query da API, e ela já ter devolvido os dados bem limpos, mas como ela é bastante lenta e confusa, achei melhor baixar bruto e limpar na mão. Ao fim do processo, ficamos com cerca de 8200 perfils prontos para receberem o RFDV.

**Feature Engineering: a criação do RFDV (Aqui começa o código desse GitHub!)**

[Link para o 01.ipynb]

Começamos com as duas mais simples, Recência e Duração. Primeiramente transformando as datas que vieram do Mixpanel em UnixTime no formato de DateTime. 

**Recência**

Para a Recência, tínhamos uma coluna já vinda do Mixpanel com a última data de acesso daquele usuário, então foi só comparar a distancia em dias dela com o fim da série histórica, no caso, dia 8 de Agosto.

M**elhoria futura**: aqui fizemos a Recência considerando a última vez que o usuário foi visto pelo Mixpanel. Mas as vezes ele considera coisas simples como um descadastramento de recebimento de emails como "uso" do produto. Uma forma mais sofisticada de calcular isso seria analisar na tabela de eventos consolidados a última vez que o usuário fez um dos 12 eventos de engajamento. O mesmo se aplica a Duração.

**Duração**

Na duração o processo foi similar, comparamos a coluna já vinda do Mixpanel com a data de criação daquele usuário com a de última vez que ele foi visto, medindo a distância em dias.

**Volume**

Depois fomos para o Volume, contando quantas vezes o Unique ID daquele usuário aparecia na nossa tabela consolidada de eventos.

**Frequência**

A última e mais complexa foi a Frequência, calcular em quantos dias distintos os usuários utilizaram a plataforma. Fizemos isso também agrupando por Unique ID a tabela consolidada de eventos e utilizando funções próprias do Pandas para contar os dias únicos.

## Fase 3: Análise exploratória

Tendo nossos perfils e eventos limpinhos, hora de ouvir o que esses dados brutos podem nos dizer.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ba7acdaa-66d0-412c-8a3d-b13c24482698/describe-do-RFDV.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ba7acdaa-66d0-412c-8a3d-b13c24482698/describe-do-RFDV.png)

Alguns insights:

- a mediana de Volume de eventos é 14 enquanto a média é 67, quase cinco vezes maior, o que mostra que temos uma distribuição desigual, alguns usuários estão fazendo muito mais eventos que a média e puxando ela para cima
- a mesma coisa acontece com a Duração, a mediana é 0 dias (ou seja, só entraram uma vez), mas a média é 10 dias.
- a Duração tem uma forte relação com a Frequência, o que é natural, já que quanto mais dias distintos você acessar, maior vai ser a distancia entre o seu primeiro e último acesso, ou seja Duração.

Mas sinceramente, nada muito interessante aqui, nada que já não tivéssemos intuído olhando para esses usuários no dia-a-dia.

## Fase 4: O número de grupos e o algoritmo

Vamos partir para o que interessa, ciência de dados real oficial, machine learning, artifical inteligence ™️, etc. 

Vamos usar o Kmeans porque ele é um clássico para a clusterização e bastante simples de ser implementado, perfeito para um primeiro trabalho.

### **Normalização**

Como o Kmeans é um algorítimo de álgebra linear, ele é muito sensível ao tipo de dado que recebe e naturalmente só aceita dados numéricos. Então primeiro vamos separar nossos dados nos numéricos e nos categóricos e daí ajustar eles.

Nosso dados numéricos são apenas os do RFDV, e neles testamos algumas normalizações, como Log, MinMax e por fim o Robust Scaler, que usa alguns truques bem inteligentes de estatística para equilibrar os dados e é especialmente útil em casos com outliers pesados, que é a nossa situação.

Tendo isso resolvido, vamos aplicar um "One-Hot" nos dados categóricos, isso transforma todas as opções em colunas, e se o usuário tem aquela propriedade ele recebe um "1" naquela coluna, se não tem, fica com um "0".

Ou seja, ao invés de ser "Cidade: São Paulo", vai ser "São Paulo: 1, Manaus: 0, Florianópolis: 0, ..." e assim por diante. O problema desse método é que ele gera muitas colunas na nossa tabela. Nesse caso, de 7 para 12.229, oof.

### **Descobrindo o número de clusters**

O problema agora é descobrir quantos grupos vamos ter, o K do Kmeans é o número de clusters e isso é a gente que tem que informar para ele. Podemos chutar isso na intuição ou fazer experimentos manuais, mas há formas mais ciêntíficas de descobrir um número ideal.

No nosso caso fizemos o famoso "teste do cotovelo" usando a "inércia", basicamente ele roda o algorítmo várias vezes, começando com dois grupos e indo até o limite que você estabeleceu, e faz uma análise estatística de quanta variação tem dentro dos clusters em cada um desses testes. No geral, depois de uma certa quantidade de clusters, se você adicionar mais eles vão ficar todos muito parecidos, então o ponto ideal é logo depois desse "cotovelo", onde fatiar em mais grupos não vai mais ajudar a muito a entender as diferenças nos dados.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8768be8c-dbc4-4e1c-bf19-2be6e4f90e25/teste-do-cotovelo.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8768be8c-dbc4-4e1c-bf19-2be6e4f90e25/teste-do-cotovelo.png)

No nosso caso o teste do cotovelo deu como resultado por volta de 5 grupos, e é com isso que vamos ir.

### Rodando **o K-means**

Tendo todos os dados coletados, limpos, normalizados e o número de clusters já descoberto, rodar o algoritmo em si é terrivelmente simples, apenas algumas linhas.

Aqui ele já devolve para a nossa tabela de usuários o cluster a qual ele pertence, e podemos começar a analisar.

## Fase 5: Enfim, a tão esperada clusterização

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cc185d25-2355-4bc0-9192-1783985aa835/usuarios-por-cluster.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cc185d25-2355-4bc0-9192-1783985aa835/usuarios-por-cluster.png)

A clusterização com 5 grupos pelo K-means deu um resultado bastante interessante: um grande grupo com a maioria dos nossos usuários, dois menores quão quase todos os outros, e dois últimos com os "outliers".

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4affefaa-8bbf-400f-9b26-54b3199cf93f/volume-por-cluster.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4affefaa-8bbf-400f-9b26-54b3199cf93f/volume-por-cluster.png)

A primeira coisa que salta aos olhos é que existe uma relação inversa entre quantos usuários tem no cluster e a média de volume de eventos. Ou seja, valida nossa hipótese que temos a maior parte dos usuários fazendo bem poucos eventos contrastando com pequenos grupos muito engajados.

A partir dessa ótica podemos criar dois macro-grupos, nomear nossos cluster e já tirar alguns insights.

### Macro-Grupo 1: Usuários Comuns (99% dos usuários)

### 1.1: Bounces (6568 usuários, 81%)

- 6612 usuários na amostragem
- 25% usam a Leads em um dispositivo móvel
- Fizeram apenas 10 eventos
- Praticamente todos entram em apenas 1 dia, e não voltam mais

**Hipóteses:**

- São pessoas que só querem pegar um dado específico.
- Não sabem que o site da Receita Federal ou similares entrega esses dados sem necessidade de cadastro.
- Fazem mais eventos de coleta de dado, como copiar email e telefone.

### 1.2: Ocasionais (1230 usuários, 15%)

- 4% usam a Leads em um dispositivo móvel
- Fazem 100 eventos, cerca de 7x mais que a mediana
- Passaram quase 40 dias em contato com a plataforma
- Nesse período, entraram em 5 dias distintos
- Ou uma vez a cada 5 dias

### 1.3: Frequentes (210 usuários, 2.6%)

- 10% usam a Leads em um dispositivo móvel
- Fizeram 361 eventos, ou cerca de 26x mais que a mediana
- Passaram quase 77 dias em contato com a plataforma, quase a série histórica completa
- Nesse período, entraram em 23 dias distintos
- Ou uma vez a cada 3 dias

**Hipóteses:**

- São pessoas que precisam recorrentemente validar dados.
- Alguns estão enriquecendo listas externas.
- Outros estão validando dados que não são do processo comercial, como contadores e advogados.

### Macro-Grupo 2: Outliers (>1% dos usuários)

### 2.1: Mini-Baleias (79 usuários, 0.9%)

- São muito parecidos com as Baleias, mas fazem quatro vezes menos eventos.
- 2,5% usam a Leads em um dispositivo móvel
- Fizeram 1107 eventos, ou cerca de 79x mais que a mediana
- Passaram quase 50 dias em contato com a plataforma, quase a série histórica completa
- Nesse período entraram 12 dias distintos
- Ou uma vez a cada 4 dias

### 2.2: Baleias (5 usuários, 0.06%)

- Nenhum usou via dispositivo móvel
- Fizeram 4296 eventos (!), ou cerca de 306x mais que a mediana
- De forma similar aos Frequentes, passaram quase 77 dias em contato com a plataforma
- Nesse período entraram 12 dias distintos
- Ou uma vez a cada 6 dias

## **Eaí, quais os próximos passos?**

### **Refazer com dados mais frescos**

Entre a coleta do dado e o fim desse projeto se passaram dois meses, nosso produto anda rápido então seria bom refazer com a janela temporal entre **6 e Julho** e **26 de Setembro**, o terceiro "Quarter" do ano e então comparar os resultados. Será que os grupos vão continuar parecidos?

### Colocar o RFDV em produção

Com ou sem os clusters, o RFDV por si só nos deu uma ótima visibilidade sobre comportamento e seria uma excelente adição as nossas métricas de produto atuais, podemos encontrar uma forma de usar a própria API do Mixpanel para criar essas features e manter toda nossa base atualizada com ela.

### **Cruzar com dados do formulário de signup**

Ao se registrarem no produto as pessoas respondem algumas questões simples sobre a sua atuação profissional e a empresa que trabalham, não é um dado muito consistente nem preciso, mas seria interessante cruzar com os clusters e vê se aparecem padrões claros.

Sabendo os principais grupos comportamentais, será que as pessoas desses grupos tem características em comum, como supomos?

### Partir para a pesquisa qualitativa

Como UX Researcher gostaria muito de entrevistar algumas dessas pessoas, especialmente os mais engajados, e entender se dentro desses grupos existem diferenças de *tarefas.* Por que algumas dessas pessoas usam tanto mais? O que o nosso produto ajuda elas a fazerem?

Por exemplo, será que os Mini-Baleias costumam ser auxiliares administrativos? Talvez fazendo um projeto específico?

Será que os Ocasionais são pequenos empresários ou autônomos? Talvez apenas coletando dados para criar notas fiscais?

### **Entender a questão do mobile**

Um a cada cinco usuários acessam via dispositivos móveis, e quase todos eles usam apenas uma vez e não voltam. As pessoas que chegam até o nosso produto via mobile são de um perfil diferente ou o nosso produto não está adequado para essa plataforma, ou os dois?

### Brincar com mais features

Seria interessante rodar novamente o K-means com mais features para ver se ele devolve algo ainda mais instrutivo, podemos criar:

- Se registrou pré-trava de créditos
- Se tem email comercial ou não
- É se nacional ou internacional
- Ratio: volume dividido por frequencia
- Recorrencia: duração dividido pela frequencia
- Se vende para B2B, B2C ou os dois