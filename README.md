# Tutorial RabbitMQ – Work Queues com Python

Este projeto segue o tutorial “Work Queues” do RabbitMQ, usando a biblioteca **pika** para criar uma fila de tarefas distribuídas entre vários “workers”.

## Visão geral

Neste tutorial, em vez de apenas enviar/receber mensagens simples, vamos simular tarefas “pesadas” (com duração baseada no número de “.” na mensagem) e distribuí-las entre múltiplos consumidores (workers). 
Alguns conceitos chave abordados:

- Dispatch round-robin entre consumidores :contentReference[oaicite:3]{index=3}  
- Acknowledgments manuais, para evitar perda de mensagens se um worker morrer durante o processamento  
- Durabilidade de filas e mensagens (queue / message persistence)
- Pré-fetch / QoS para distribuição “justa” (fair dispatch)

## Ferramentas utilizadas

- **VS Code** — editor de código usado durante o desenvolvimento  
- **PowerShell** (ou terminal) — para rodar scripts Python  
- **RabbitMQ** — servidor de mensageria (local ou remoto)  
- **Python 3** — linguagem para os scripts  
- **Biblioteca pika** — cliente Python para comunicação com RabbitMQ

## Estrutura dos scripts

- **new_task.py** — envia uma mensagem para a fila `task_queue`.  
  - Declara a fila como durável (`queue_declare(... durable=True)`)  
  - Publica a mensagem com `delivery_mode = pika.DeliveryMode.Persistent` para persistência

- **worker.py** — consome mensagens da fila `task_queue`.  
  - Declara a fila durável  
  - Usa `basic_qos(prefetch_count=1)` para não atribuir mais de uma tarefa não confirmada por vez a cada consumidor (fair dispatch) 
  - Ao processar a mensagem (simulado com `time.sleep(...)`), envia `ch.basic_ack(...)` para confirmar que terminou. Se o worker morrer antes do `ack`, a mensagem volta para a fila.

## Como executar

1. Certifique-se que o RabbitMQ está instalado e rodando (por padrão em `localhost:5672`). 
2. Instale a biblioteca `pika` via pip:

   ```bash
   pip install pika
3. Execute os códigos:

Abra dois terminais e em cada um execute:
  ```bash
    python worker.py
````
Execute em um terminal o código do produtor:
  ```bash
    python new_task.py Mensagem.
    python new_task.py Outra mensagem...
````
4. Se desejar executar mais de um consumidor receba mensagens:
   Exemplo: 5 consumidores
   Execute 5 worker.py em 5 terminais diferentes para os 5 consumidores:
   ```bash
   #shell1
   python worker.py
   #shell2
   python worker.py
   #shell3
   python worker.py
   #shell4
   python worker.py
   #shell5
   python worker.py
   ````
   Agora abra 1 terminal novo para executar o código do produtor:
   digite as mensagens que você desejar

   ````bash
   #shell6
   python new_task Primeira Mensagem
   python new_task Segunda Mensagem
   python new_task Terceira Mensagem
   python new_task Quarta Mensagem
   python new_task Quinta Mensagem
   ````
