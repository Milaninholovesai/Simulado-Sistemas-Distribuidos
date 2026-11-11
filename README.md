Simulado Prático – Sistemas Distribuídos

**Anhanguera Educacional** | Prof.ª Dr.ª Simone Tatiane do Canto

## Problema

Central de pedidos com trabalhadores concorrentes, um servidor principal e uma réplica. O código inicial tinha 3 erros (A, B e C) que impediam o comportamento exigido:

- Processar exatamente 3 pedidos (101, 102 e 103).
- No máximo 2 trabalhadores simultâneos.
- Estoque principal: 10 → 7.
- Réplica recebe o estoque atualizado (7).
- Se o principal cair, a consulta é atendida pela réplica.

## Como executar

```bash
python central_pedidos.py   # Windows: py central_pedidos.py
```

## Correções realizadas

| Linha | Antes | Depois | Motivo |
|---|---|---|---|
| A | `MAXIMO_SIMULTANEO = 3` | `MAXIMO_SIMULTANEO = 2` | A regra exige no máximo 2 trabalhadores simultâneos. |
| B | `replica["estoque"] = replica["estoque"]` | `replica["estoque"] = principal["estoque"]` | A réplica copiava o próprio valor e nunca recebia o do principal. |
| C | `if principal["online"]:` | `if replica["online"]:` | A condição repetida impedia o failover; agora a réplica é consultada quando o principal cai. |

## Saída ANTES da correção

```text
=== CENTRAL DE PEDIDOS ===
Limite simultaneo: 3
Estoque inicial: 10
Pedidos: [101, 102, 103]
Estoque principal: 7
Replica antes: 10
Replica depois: 10
Consulta apos falha: INDISPONIVEL None
```

Problemas: limite de 3 (deveria ser 2), réplica continuou com 10 e a consulta após a falha retornou `INDISPONIVEL`.

## Saída DEPOIS da correção

```text
=== CENTRAL DE PEDIDOS ===
Limite simultaneo: 2
Estoque inicial: 10
Pedidos: [101, 102, 103]
Estoque principal: 7
Replica antes: 10
Replica depois: 7
Consulta apos falha: REPLICA 7
```

## Respostas teóricas

**1. Qual o papel do semáforo?**
O semáforo controla quantas threads podem executar a seção crítica ao mesmo tempo, funcionando como um limitador de concorrência. Aqui, com valor 2, apenas dois trabalhadores processam pedidos simultaneamente e o terceiro espera a liberação de uma vaga, evitando sobrecarga do sistema.

**2. Por que a réplica precisava ser sincronizada?**
Porque a réplica é uma cópia do servidor principal e, sem sincronização, ficava com dados desatualizados (10 unidades em vez de 7). Se o principal falhasse, a réplica responderia com informação incorreta, comprometendo a consistência dos dados.

**3. O que é failover?**
Failover é a troca automática para um servidor reserva (a réplica) quando o servidor principal falha ou deixa de responder. Ele garante a disponibilidade do serviço, pois as consultas continuam sendo atendidas mesmo com a falha.

**4. Por que o uso do Lock ajuda a evitar problemas de concorrência?**
O Lock garante exclusão mútua: apenas uma thread por vez altera o estoque e a lista de pedidos confirmados. Sem ele, duas threads poderiam ler o mesmo valor e sobrescrever a atualização uma da outra (condição de corrida), gerando estoque e pedidos inconsistentes.
