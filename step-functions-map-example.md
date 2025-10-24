## Explicando o Mapa no AWS Step Functions para processar uma lista de itens.

O estado de mapa permite executar ações repetidamente para cada item de uma lista, sendo útil para tarefas como processamento em lote.

**Exemplo de Definição do Fluxo de Trabalho (JSON)**

[Veja o exemplo de Definição do Fluxo de Trabalho (JSON)](meu_workflow.json)

```json
{
  "Comment": "Exemplo de uso do estado Map no Step Functions",
  "StartAt": "ProcessarItens",
  "States": {
    "ProcessarItens": {
      "Type": "Map",
      "InputPath": "$.itens",
      "ItemProcessor": {
        "ProcessorConfig": {
          "Mode": "INLINE"
        },
        "StartAt": "ProcessarItem",
        "States": {
          "ProcessarItem": {
            "Type": "Task",
            "Resource": "arn:aws:lambda:REGIAO:ID_DA_CONTA:function:NomeDaFuncao",
            "End": true
          }
        }
      },
      "MaxConcurrency": 2,
      "End": true
    }
  }
}
```

  * **ProcessarItens:** É o estado do tipo Map, que processa cada item da lista fornecida em `$.itens`.
  * **InputPath:** Define o caminho no JSON de entrada onde está a lista de itens.
  * **ItemProcessor:** Define o fluxo de trabalho que será executado para cada item da lista.
  * **ProcessarItem:** Um estado do tipo Task que chama uma função Lambda para processar cada item.
  * **MaxConcurrency:** Limita o número de execuções simultâneas (neste caso, 2).

### Exemplo de Entrada:

```json
{
  "itens": [
    { "id": 1, "nome": "Item A" },
    { "id": 2, "nome": "Item B" },
    { "id": 3, "nome": "Item C" }
  ]
}
```

### Funcionamento

O estado Map itera sobre cada item da lista `itens`.
Para cada item, ele executa o estado `ProcessarItem`, que chama uma função Lambda para processar o item.

-----
## Exemplo de Diagrama
![Texto alternativo aqui](caminho/para/sua/imagem.png)

### Explicação do Diagrama


  * **Fluxo Principal:** O fluxo principal é muito simples: Início -\> ProcessarItens -\> Fim.
  * **A Mágica (O Bloco Map):** Toda a complexidade está dentro do estado `ProcessarItens`. Ele pega a entrada (`{ "itens": [...] }`), encontra o array "itens" e, para cada objeto desse array (Item A, Item B, Item C), ele executa uma nova instância do sub-workflow `ItemProcessor`.
  * **Sub-workflow (ItemProcessor):** Este fluxo interno simplesmente chama a função Lambda "ProcessarItem" e termina.
  * **Execução (Baseado em uma entrada):**
    1.  O workflow começa.
    2.  O estado Map vê 3 itens.
    3.  Como `MaxConcurrency` é 2, ele inicia 2 execuções do `ItemProcessor` em paralelo (provavelmente para o "Item A" e "Item B").
    4.  Assim que um deles termina (ex: "Item A" termina), o Map inicia o próximo item da fila ("Item C").
    5.  Quando todos os 3 itens tiverem sido processados, o estado Map "ProcessarItens" é considerado concluído.
    6.  O workflow principal segue para o estado Fim.
