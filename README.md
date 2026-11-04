# Sistema de Gestão de Eventos e Espaços

Banco de dados relacional (PostgreSQL) para organizar eventos: quem organiza, quais atividades e sessões existem, onde acontecem, quais recursos usam e quem está inscrito. Inclui um protótipo em Python onde você faz perguntas em português e uma LLM local converte para SQL.


## O que o projeto faz

- **Organizadores variados:** um evento pode ser promovido por pessoa, empresa ou organização, sozinhos ou em conjunto.
- **Vários papéis para a mesma pessoa:** participante, organizador, palestrante, instrutor ou mediador.
- **Atividade x Sessão:** a atividade é a definição (título, tipo, descrição). A sessão é a realização concreta (horário, local, estado, vagas).
- **Séries de eventos:** edições recorrentes são agrupadas, sem repetir número de edição na mesma série.
- **Vagas e fila de espera:** sessões com vagas limitadas têm fila por ordem de inscrição. Se alguém cancela, o primeiro da fila é promovido automaticamente (trigger).
- **Integridade:** regras validadas no próprio banco (triggers em PL/pgSQL), com rollback em caso de violação e bloqueio de escritas concorrentes via `pg_advisory_xact_lock`.

## Tecnologias

| Item | Uso |
|---|---|
| PostgreSQL 16+ | Banco de dados |
| Python 3.11+ e `psycopg` 3 | Protótipo CLI |
| Ollama + `qwen2.5-coder:3b` | Conversão de linguagem natural em SQL |
| Draw.io / Mermaid | Diagramas ER e relacional |

## Estrutura

```text
.
├── Projeto.txt        # Decisões de projeto, DDL e consultas
├── schema.sql         # Criação das tabelas, views, funções e triggers
├── DML.sql            # Carga de dados de teste (1.686 registros)
├── Instrucoes.txt     # Guia de instalação e execução
├── ER.pdf / .xml      # Diagrama Entidade-Relacionamento
├── Relacional.pdf     # Modelo relacional (Crow's Foot)
└── Prototipo/
    ├── main.py        # Aplicação CLI
    ├── config.json    # Configuração do banco e do Ollama
    └── requirements.txt
```

## Como rodar

**1. Banco de dados**

```sql
CREATE DATABASE eventos;
```

Conecte-se ao banco `eventos` e execute, nesta ordem:

1. `schema.sql` (cria 21 tabelas, 2 views, funções e triggers)
2. `DML.sql` (**uma única vez**, para carregar os dados de teste)

**2. Modelo de linguagem**

```bash
ollama pull qwen2.5-coder:3b
```

**3. Protótipo**

Ajuste o `config.json` com os dados do seu PostgreSQL e do Ollama, depois:

```bash
cd Prototipo
python -m pip install -r requirements.txt
python main.py
```

## Consultas implementadas

1. **Programação completa de um evento:** atividades, horários, estado e espaços.
2. **Pendências de recursos:** recursos (projetor, computador, som) ainda não alocados a sessões planejadas.
3. **Divisão relacional (`NOT EXISTS` duplo):** participantes inscritos em *todas* as sessões restritas confirmadas de um evento.
4. **Fila de espera FIFO (`ROW_NUMBER()`):** primeiros da fila em sessões lotadas.
5. **Espaços livres:** locais autorizados, com a capacidade mínima, sem conflito em uma janela de data e horário.

## Segurança da consulta em linguagem natural

O SQL gerado pela LLM só é executado se for de leitura (`SELECT` ou `WITH`). Comandos como `INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER` e `TRUNCATE` são bloqueados antes de chegar ao banco.
