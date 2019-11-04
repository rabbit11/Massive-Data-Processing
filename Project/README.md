# ArangoDB Tutorial

#### Neste tutorial iremos cobrir alguns conceitos básicos sobre ArangoDB e algumas de suas arquiteturas, além de mostrar o passo a passo necessário para a criação de um sistema com um cluster e 3 nós na arquitetura Master/Slave

## Visão Geral
  ArangoDB é um NoSQL, native multi-model, SGBD (Sistema de Gerenciamento de Banco de Dados), o que significa que ele une 3 tipos de modelos de dados dentro de seu núcleo, sendo eles: chave-valor, documentos e grafos. Isso permite que usuários possam armazenar seus dados em qualquer um destes modelos e utilizar apenas uma linguagem declarativa para consultas, inserções, remoções e alterações na base de dados.
        
  Como ArangoDB é native multi-model é possível realizar consultas combinando padrões de acesso a diferentes tipos de dados em uma única consulta, assim possibilitando consultas que relacionem os dados armazenados nos diferentes modelos.
  
  ## Instalação
  Devido a escolha do sistema operacional ser Fedora e pela falta de suporte da aplicação por meio do site oficial para versões mais recentes do SO, iremos instalar por meio do SnapCraft.
        
   Assumiremos que o SnapCraft já está instalado (caso não esteja, o link para instalação é este: 
                  https://snapcraft.io/docs/installing-snap-on-fedora).
   Abrimos um terminal e usamos o comando "sudo snap install arangodb3", digitamos a senha do sudo e esperamos o download e posterior instalação a serem executadas pelo comando.
        
   Ao terminar a instalação, o servidor ArangoDB já estará rodando em sua máquina pronto para ser acessado.
   

## Configuração
  Para acessar o servidor entramos no browser e acessamos a seguinte url: http://127.0.0.1:8529, a qual é aponta para a definição padrão de url e porta para o servidor ArangoDB. Nessa página, temos que fazer login, o qual faremos com o usuário "root" que é o já criado como padrão para o servidor.
  
## Comandos Básicos
  #### Inserção de documento:
   ``` 
   INSERT { name: "John Doe", gender: "m" }
        INTO users
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
