# ArangoDB Tutorial

#### Neste tutorial iremos cobrir alguns conceitos básicos sobre ArangoDB e algumas de suas arquiteturas, além de mostrar o passo a passo necessário para a criação de um sistema com um cluster e 3 nós na arquitetura Master/Slave

## Visão Geral
  ArangoDB é um NoSQL, native multi-model, SGBD (Sistema de Gerenciamento de Banco de Dados), o que significa que ele une 3 tipos de modelos de dados dentro de seu núcleo, sendo eles: chave-valor, documentos e grafos. Isso permite que usuários possam armazenar seus dados em qualquer um destes modelos e utilizar apenas uma linguagem declarativa para consultas, inserções, remoções e alterações na base de dados.
        
  Como ArangoDB é native multi-model é possível realizar consultas combinando padrões de acesso a diferentes tipos de dados em uma única consulta, assim possibilitando consultas que relacionem os dados armazenados nos diferentes modelos.
  
  ## Instalação
  Devido a escolha do sistema operacional ser Fedora e pela falta de suporte da aplicação por meio do site oficial para versões mais recentes do SO, iremos instalar por meio do SnapCraft.
        
   Assumiremos que o SnapCraft já está instalado (caso não esteja, o link para instalação é este: 
                  https://snapcraft.io/docs/installing-snap-on-fedora).
   Abrimos um terminal e usamos o comando "sudo snap install arangodb3", digitamos a senha do sudo e esperamos o download e posterior instalação a serem executadas pelo comando. A versão instalada, neste caso, é a mais recente, a 3.5.
        
   Ao terminar a instalação, o servidor ArangoDB já estará rodando em sua máquina pronto para ser acessado.
   

## Configuração
  Para acessar o servidor entramos no browser e acessamos a seguinte url: http://127.0.0.1:8529, a qual é aponta para a definição padrão de url e porta para o servidor ArangoDB. Nessa página, temos que fazer login, o qual faremos com o usuário "root" que é o já criado como padrão para o servidor.
  
## Comandos Básicos
  ### Nesta seção é exemplificado o uso da linguagem AQL, que é a usada no ArangoDB, para alguns comando básicos e vamos comparar suas estruturas com a SQL.
  
  #### Terminologia
  
  ![](https://raw.githubusercontent.com/rabbit11/Massive-Data-Processing/master/Project/img/terminologia.png)
  
  #### Inserção de documento:
  
                 AQL                                                                SQL
  
   ```                                                                  
   INSERT { name: "John Doe", gender: "m" }                             INSERT INTO users (name,gender)
        INTO users                                                      VALUES ('John Doe','m');
   ```
   
  #### Atualização de documento:
  ```
   UPDATE { _key: "1" }
       WITH { name: "John Smith" }
       IN users 
  ```
  
  #### Remoção de documento:
  ```
   REMOVE \{ \_key:"value" \} 
              IN collections
  ```
  
  #### Seleção de documentos:
  ```
   FOR user IN users
              RETURN user
  ```
  
  #### Filtro de documentos em seleção:
  ```
  FOR user IN users
              FILTER user.active == 1
              RETURN {
                name: CONCAT(user.firstName, " ",
                             user.lastName),
                gender: user.gender
              }
  ```
  
  #### Ordenação de documentos numa seleção:
 ```
  FOR user IN users
              FILTER user.active == 1
              SORT user.name, user.gender
              RETURN user
 ```
 
 #### Contagem de documentos de uma coleção:
 ```
 FOR user IN users
              FILTER user.active == 1
              COLLECT gender = user.gender 
                WITH COUNT INTO number
              RETURN { 
                gender: gender, 
                number: number 
              }
 ```
 
 #### Agrupamento de documentos de uma coleção:
 ```
  FOR user IN users
            FILTER user.active == 1
            COLLECT
              year = DATE_YEAR(user.dateRegistered), 
              month = DATE_MONTH(user.dateRegistered) 
              WITH COUNT INTO number
              FILTER number > 20
              RETURN {
                year: year, 
                month: month, 
                number: number 
              }
 ```
 
 ## Arquitetura de distribuição de dados e replicação
 
 #### Nesta seção serão exemplicados todos os tipos de arquitetura presentes no ArangoDB.
 
 ### Single Instance
   *Single Instance* é a arquitetura mais simples que existe neste banco de dados. Ao utilizar esta arquitetura sozinha, não existirá métodos de replicações, sem oportunidades de *failover* e nenhum *cluster* com outros nós.
   
   Mesmo sem todos os recursos citados anteriormente, é possível rodar múltiplos processos na mesma máquina com esta arquitetura, contudo que as portas e os dados sejam configurados diferentemente.
   
 ### Master/Slave
  O ArangoDB possui a arquitetura *Master/Slave* onde os *Slaves* recebem dados assíncronos de um *Master*. Nos *Slaves* apenas é possível realizar apenas a leitura dos dados, enquanto o *Master* realiza inserções e atualizações dos dados. Na figura 1, pode ser observado ,de modo simplificada, esta arquitetura.
  
   ![](https://raw.githubusercontent.com/rabbit11/Massive-Data-Processing/master/Project/img/master-slave1.png)
  
### Active Failover
  Uma arquitetura de *Active Failover* possui as seguintes características:
  
  * Uma instância chamada de Leader que clientes podem realizar operações de leitura e escrita.
  * Ao menos uma instância chamada de Follower, que clientes podem realizar apenas operações de leitura, essas instâncias replicam os dados do Leader de maneira assíncrona.
  *  Uma instância chamada de Agency, cuja função é ser uma espécie de testemunha, que decide qual servidor assume a posição de líder, caso alguma falha ocorra.
  
  ![](https://raw.githubusercontent.com/rabbit11/Massive-Data-Processing/master/Project/img/leader-follower(1).png)
  
   A vantagem do uso de Active Failover quando comparada a arquitetura Master/Slave tradicional seria a terceira entidade chamada Agency que observa e está envolvida em todos os processos dos servidores. Followers podem contar com o Agency para determinar se estão conectados ao server Leader correto. Isso faz com que este tipo de arquitetura tenha uma maior resiliência, já que todos os drivers oficiais do ArangoDB são capazes de determinar o servidor Leader corretamente e redirecionar aquela requisição apropriadamente.
   
### Cluster
  A arquitetura de clusters no ArangoDB é CP master/master, o que significa (em termos do teorema CAP) que durante uma falha de conexão entre nós no servidor, este SGBD prioriza consistência interna no lugar de disponibilidade. Além disso uma arquitetura master/master permite que clientes podem mandar requisições de maneira arbitrária para qualquer nó e obter a mesma "visão" dos dados, sem um único ponto de falha, já que o cluster pode ainda servir a requisições mesmo com falhas em algumas máquinas.
    
### Replicação
   A replicação pode ser definida como cópias dos dados de um nó do sistema para outro, de forma a permitir recuperação de falhas, no caso de quedas de conexões entre servidores por exemplo. Para isso o ArangoDB oferece dois tipos de replicação: síncrona e assíncrona.
   
   #### Replicação síncrona
   A replicação síncrona funciona apenas entre servidores dentro de um mesmo cluster, e é normalmente utilizada para operações críticas, onde os dados devem estar disponíveis a todo o momento. Normalmente este tipo de replicação guarda uma cópia de um fragmento dos dados em um ou mais outros servidores. Desta forma as operações de escrita só são devidamente validadas quando todas as réplicas efetuarem aquela operação de escrita, o que aumenta a latência desta operação, porém caso uma falha ocorra logo após a escrita, temos a garantia de que os dados presentes em qualquer uma das réplicas é o dado mais recente.
   
   #### Replicação assíncrona
   A replicação assíncrona é usada principalmente na arquitetura de master/slave, de forma que os slaves conectam-se aos seus respectivos masters e aplicam localmente os eventos já aplicados ao master em mesma ordem, como resultado os slaves terão os dados no mesmo estado que os seus respectivos masters.

## Implementação de Propriedades

  ### Consistência
   Como ArangoDB é um banco de dados de múltiplos modelos, a consistência dos dados se torna um empecilho, devido a cada um dos modelos não terem sido desenvolvidos para conversarem entre si, o que faz com que se tenha que desenvolver alguma funcionalidade de transação a fim de manter seus dados consistentes através dos diversos modelos suportados.
   
   Assim no ArangoDB, há uma única aplicação back-end gerenciando cada modelo de dados com suporte de transações ACID para desse modo manter uma forte consistência tanto em instâncias únicas como em operações atômicas no modo cluster.
   
 ### Transações
  As transações no ArangoDB  são atômicas, consistentes, isoladas e duráveis (ACID). Tais propriedades do ACID fornecem as seguintes garantias para as transações:
  
  * A atomicidade faz com que as transações sejam executadas completamente ou não sejam executadas.
  * A consistência garante que as transações ou alteram o estado para um estado novo e válido em caso de sucesso ou voltam ao estado anterior sem perda dos dados.
  * O isolamento mantém a transação atual não finalizada sem sofrer alterações de outras transações correntes.
  * A durabilidade garante que as operações das transações confirmadas sejam mantidas persistentes. Tal durabilidade da transação é configurável no ArangoDB, assim como a durabilidade no nível da coleção.
  
### Disponibilidade
  O grau de disponibilidade do ArangoDB pode ser descrito como de certa forma "flexível", já que depende da forma como ele é configurado. A arquitetura de cluster CP utilizada por essa base de dados em geral prioriza consistência interna em detrimento da disponibilidade. Para amenizar este efeito, o sistema pode ser configurado de forma a utilizar replicação síncrona apenas, o que geraria grandes ganhos em termos de disponibilidade em troca de grande parcela da performance, devida a alta latência gerada por operações de escrita.
 
### Escalabilidade
   Como o ArangoDB é uma base de dados distribuída em 3 tipos de modelos de dados diferentes, ele pode escalar de maneira horizontal, ou seja, pode se utilizar de diversos servidores. Essa técnica além de resultar em uma maior performance, também gera maior resiliência e melhora em capacidade.
   
   Base de dados que podem escalar de maneira horizontal também permitem que sistemas adéquem sua capacidade de processamento e armazenamento de acordo com a demanda, ou seja, durante altas demandas podemos aumentar o número de servidores (aumentando a escala horizontal) e durante momentos de baixa demanda, podemos desalocar alguns nós do sistema de forma a cortar custos.
   
   Apesar de todas as vantagens citadas acima sobre escalar o sistema horizontalmente, o ArangoDB também permite que o sistema o escale de maneira vertical, apesar de ser menos eficiente na maior parte dos casos.
   
## Quando utilizar ArangoDB

  O ArangoDB por possuir vários tipos de armazenamento de dados, seja em formato de chave/valor, documentos ou grafos, isso traz uma maior performance nas aplicações e uma maior escabilidade horizontal dos dados quando combinados os três formatos. Além disso, é possível em uma única query relacionar os três formatos.
  
  Existem três principais situações onde o este banco de dados é extremamente recomendável, que são:
  
  * Ao realizar um projeto, pode não se ter todos os requisitos do sistema e de como organizar os dados, assim futuras modificações podem ocorrer na estruturação dos dados. Ao utilizar o ArangoDB não tem essa preocupação, já que possui um modelo de vários formatos de estruturar os dados, é possível alterar os dados de forma fácil para novos casos e novas características de um sistema.
  * Quando se está desenvolvendo uma aplicação em uma equipe, este banco permite que os times se cooperem através de casos de usos. Por exemplo, um time trabalha na formatação dos dados em chave/valor, enquanto outro pode trabalhar na parte dos grafos e depois trocar experiências e otimizando a usabilidade do sistema. Além disso, permite um time ter uma curva de aprendizado mais rápido.
  * Neste banco de dados, por ter um incrível sistema de vários formatos de estruturas de dados, é possível também ter uma única query para obtenção dos dados a fim de modular diferentes aplicações. 
  
## Quando não utilizar

## Parte Prática
  Agora que já cobrimos a parte teórica, nesta seção iremos dar a você o passo a passo de como utilizar o ArangoDB para criar um cluster com 3 nós (sendo cada computador um nó diferente) na arquitetura Master/Slave.
  
## Configuração
  ### Criação do cluster
  Primeiro devemos configurar aquele nó que inicialmente será o Master, podemos escolher qualquer nó arbitrariamente. O seguinte comando inicia uma instância do arangodb, define um diretório no qual os arquivos armazenados dentro da base de dados deverão ser salvos, e a porta a qual os outros nós devem se conectar. E então no terminal do futuro nó que escolhemos, devemos executar o seguinte comando:
    ```
      arangodb --starter.data-dir=./db1 --starter.port="8530"
    ```
  E devemos receber uma mensagem de sucesso como: ArangoDB Starter listening on 0.0.0.0:8530.
  
  Após esta mensagem de sucesso, devemos executar o seguinte comando no terminal do segundo computador:
   ```
      arangodb --starter.data-dir=./db2 --starter.port="8530" --starter.join 192.168.15.6:8530
   ```
   
   Devemos então receber uma mensagem de sucesso semelhante a primeira. Este comando executado define o diretório onde os dados daquele nó deverão ser armazenados, a porta a qual outros nós devem se conectar a ele, e o IP e porta do nó o qual ele deve se conectar (no caso seria o IP do computador o qual executamos o primeiro comando, substitua "192.168.15.6" pelo IP do primeiro computador).
   
   Após esta mensagem de sucesso, devemos executar um comando similar no terminal do terceiro computador:
   ```
      arangodb --starter.data-dir=./db2 --starter.port="8530" --starter.join 192.168.15.6:8530
   ```
  Devemos então receber uma mensagem de sucesso e o endereço de IP e porta que deveremos acessar em nosso browser para acessar a interface do ArangoDB. Normalmente deve ser algo semelhante à: 0.0.0.0:8529
  
  ### Configuração do Banco de Dados
   Após acessar a interface web do ArangoDB, devemos selecionar o DB System e e clickar em *Select DB:_system*, como nesta imagem:
   
   E então veremos uma tela semelhante à essa:
   
   Iremos então para a aba *Database* e clickar em *Add Database*, que então gerará esta tela:
   
   Neste caso daremos o nome de Restaurants para o nosso database.
   
   Após isso devemos clickar nas setas brancas no topo da página, ao lado de *DB:_System*. Que então nos redirecionará a tela inicial, onde desta vez devemos escolher o database Restaurants e clickar no botão *Select DB: Restaurants*.
   
   Então deveremos acessar a aba Collections e adicionar uma nova coleção chamada *restaurants*, alterando as seguintes opções:
    * Number of shards: 1
    * Replication Factor: 3
    
   Isso irá permitir que a replicação seja realizada em nosso banco.
   
  ### Inserção de dados na coleção Restaurants

## Referências
