# Desafio-Aws-Step-Function

---

# Este repositório documenta anotações e insights técnicos adquiridos durante a aula sobre AWS Step Functions. 

## Objetivos: 🎯

* Documentar processos técnicos de forma clara e estruturada.
* Aplicar os conceitos aprendidos em um ambiente prático (hands-on).
* Servir como material de apoio para estudos e futuras implementações.

---

## Conceitos: 💡

### AWS Step Functions -

O AWS Step Functions nada mais é que do que um construtor visual serverless que nos auxilia a criar fluxos de trabalho (Workflows) flexíveis. Ele utiliza o conceito de máquinas de estado visuais para definir a lógica do fluxo, gerenciando automaticamente sequenciamento, execução paralela, tratamento de erros, novas tentativas (retries) e a passagem de dados (estado) entre os passos.Isso simplifica desenvolvimento de aplicações distribuídas, a orquestação de microsserviços e a automação de processos de negócios e pipelines de dados.

## Tipos de WorkFlow no AWS Step Functions - 📊

O Step Functions oferece dois tipos de workflows ("Máquinas de Estado"), cada um otimizado para um caso de uso diferente.

### 1. Workflows Padrão (Standard Workflows)

Este é o tipo original e o mais robusto, ideais para processos de longa duração, que precisam ser auditáveis e extremamente confiáveis.
**Duração:** Podem executar por até 1 ano.
**Modelo de Execução:** "Exatamente uma vez" (Exactly-once). Isso garante que cada passo do fluxo não será executado mais de uma vez, o que é crítico para operações como processamento de pagamentos ou pedidos.
**Visualização e Auditoria:** Oferecem um histórico visual detalhado de cada execução, passo a passo, o que é perfeito para depuração e auditoria.
**Callbacks:** Suportam "padrões de callback", permitindo que o workflow pause e espere por uma ação externa (como uma aprovação humana ou a finalização de um processo em outro sistema) antes de continuar.
**Preço:** O custo é baseado no número de transições de estado. Você paga por cada passo que seu workflow executa.
**Casos de Uso Ideais:** Processamento de pedidos de e-commerce, fluxos de aprovação de documentos ou despesas (com intervenção humana), Pipelines de ETL (Extração, Transformação e Carga) que podem levar horas e orquestração de microsserviços onde cada passo é crítico e não pode ser duplicado.

### 2. Workflows Expressos (Express Workflows)

Foram introduzidos para casos de uso de altíssimo volume e curta duração, onde a latência é mais importante do que a garantia de execução única ou a auditoria visual detalhada.
**Duração:** Podem executar por até 5 minutos.
**Modelo de Execução:** "Pelo menos uma vez" (At-least-once). Em raras situações, um passo pode ser executado mais de uma vez. Isso os torna inadequados para operações que não podem ser repetidas sem efeitos colaterais.
**Visualização e Auditoria:** Não oferecem o histórico visual em tempo real. Os logs são enviados para o Amazon CloudWatch Logs.
**Callbacks:** Não suportam pausas para callbacks externos como os Workflows Padrão.
**Preço:** O custo é baseado no número de execuções, na duração e na memória consumida, o que os torna muito mais baratos para cargas de trabalho de alto volume.
**Casos de Uso Ideais:** Processamento de dados de streaming, orquestração de microsserviços de alta frequência (Ex: um serviço de autenticação ou validação de dados), Ingestão de dados de IoT (Internet das Coisas), onde milhares de dispositivos enviam dados por segundo e Aplicações web onde você precisa de uma resposta rápida de uma orquestração de back-end.

---

## Estados Notáveis: 🧩

Os "estados" são os blocos de construção fundamentais de um workflow no AWS Step Functions, pois são eles que definem a lógica da máquina de estado (o workflow).
Eles são definidos usando a Amazon States Language (ASL), que é uma estrutura JSON.

Temos os seguintes estados:

* **`Task` (Tarefa):** Ele executa um trabalho, que é quase sempre uma integração com outro serviço da AWS, como invocar uma Função Lambda, enviar uma mensagem para uma fila SQS (Simple Queue Service), gravar ou ler um item no DynamoDB, entre outros. Seu padrão notável (CallBack) é que ele pode ser configurado para "pausar e esperar por um token", ou seja, o workflow pausa até que outro serviço (ou humano) devolva esse token para ele continuar.

* **`Choice` (Escolha):** É o estado de lógica condicional, o if/else dentro do workflow. Ele permite que o workflow tome caminhos diferentes com base nos dados (Json) que estão fluindo por ele. Por exemplo, verificar o campo status de pagamento no Json de entrada, se o pagamento está com status aprovado ele segue para o próximo estado, senão ele notifica uma falha.

* **`Parallel` (Paralelo):** É o estado de concorrência, o fork/join. Ele permite executar múltiplas sequências de estado de forma simultânea, onde o workflow só continua após todos as sequências terminarem. Um bom exemplo é um pipeline de imagens onde a primeira sequência cria um thumbnail (chama uma Lambda), a segunda adiciona a marca d'água (chama outra Lambda), e por último analisa o conteúdo (chama o Rekognition). O estado Parallel espera todas essas sequência terminarem antes de ir para o próximo passo.

* **`Map` (Mapa):** É o estado de iteração, o for-each loop. Ele pega um array (lista) do JSON de entrada e executa um conjunto de passos (ou até um sub-workflow) para cada item nesse array. Por exemplo em um pedido de e-commerce com 5 itens diferentes, o estado Map pode iterar sobre a lista de itens e chamar a Lambda "Verificar Inventário" 5 vezes, uma para cada item. Ele pode executar essas iterações em paralelo (modo "Distribuído"), permitindo o processamento em massa de milhares de itens de forma muito rápida.

* **`Wait` (Espera):** É o estado de pausa ou atraso, o sleep que pausa o workflow por um tempo determinado. Ele é útil para checagens periódicas (esperar 5 minutos e verificar o status de um job de novo) ou para respeitar limites de taxa (rate limits) de APIs externas.

* **Estados terminais:** São o `Succeed` e `Fail`, eles finalizam o workflow. O `Succeed` (Sucesso), marca a execução do workflow como bem-sucedida e o, `Fail` (Falha) marca a execução do workflow como falha. Se pode, inclusive, definir um tipo de erro e uma mensagem, o que é ótimo para o tratamento de erros (o "catch" try/catch).

* **`Pass` (Passagem):** É um estado utilitário, muitas vezes usado para depuração ou para manipular os dados. Ele simplesmente passa da entrada para a saída, não fazendo nenhum trabalho. Seus casos de uso mais comuns são adicionar dados fixos ao JSON, filtrar ou transformar o JSON de estado (usando InputPath, ResultPath, OutputPath) e atuar como um "placeholder" durante o desenvolvimento.

**Com esses estados podemos praticamente construir qualquer lógica dentro de um workflow.**



## 📚 Referências

* **Documentação:** [Documentação oficial da AWS Step Functions](https://aws.amazon.com/pt/step-functions/) - Usado como fonte de pesquisa principal.
