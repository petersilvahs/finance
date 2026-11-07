# 🏦 Finance CoreBanking — Squad Study & Architecture

Projeto prático simulando o dia a dia de uma **Squad de Corebanking** utilizando **.NET 8 (C#)**, **React + TypeScript**, **PostgreSQL (SQL)**, **RabbitMQ**, **AWS (LocalStack S3/SQS)** e **Testes Unitários (xUnit + FluentAssertions)**.

---

## 🎯 Pilares Técnicos Implementados

1. **Double-Entry Bookkeeping (Partidas Dobradas / Ledger Imutável)**
   - Nenhuma alteração de saldo acontece "solta". Toda transação gera lançamentos contábeis imutáveis (`LedgerEntry` de Débito e Crédito).
2. **Controle de Concorrência & Transações ACID (SQL)**
   - Proteção contra *Race Conditions* (ex: dois Pix simultâneos) usando **Optimistic Concurrency (`RowVersion` / `xmin`)** e transações atômicas no PostgreSQL.
   - **CQRS na prática**: Escrita transacional via **Entity Framework Core 8** e leitura rápida de extrato via **Dapper**.
3. **Idempotência (`Idempotency-Key`)**
   - Garantia de que retentativas de rede (do Front-end ou de Workers) nunca dupliquem uma transação financeira.
4. **Mensageria Assíncrona (RabbitMQ)**
   - Publicação de eventos de domínio (`TransferCompletedEvent`), retentativas exponenciais e **Dead Letter Queue (DLQ)**.
5. **Cloud AWS (S3 via LocalStack)**
   - Armazenamento de comprovantes de auditoria de transações no Amazon S3.
6. **Front-end (React + TypeScript + Axios)**
   - Painel de Internet Banking & Backoffice de Operações Financeiras.

---

## 🚀 Como Subir a Infraestrutura Local (Docker)

```bash
docker compose up -d
```

Serviços disponíveis:
- **PostgreSQL 16**: `localhost:5432` (`corebanking_ledger`)
- **RabbitMQ Management UI**: [http://localhost:15672](http://localhost:15672) (`guest` / `guest`)
- **LocalStack (AWS S3/SQS)**: `http://localhost:4566`
