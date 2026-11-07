# 🏦 Squad de Corebanking — Guia de Arquitetura, Estudo & Sprints

Este documento funciona como o nosso **Confluence + Board de Sprints** do dia a dia da Squad de Corebanking.

---

## 1. O que faz uma Squad de Corebanking na prática?

Em entrevista técnica para Corebanking, os avaliadores avaliam os **4 Pilares de um Core Bancário**:

1. **Double-Entry Bookkeeping (Partidas Dobradas / Ledger Imutável)**
   - Em banco, você **nunca** faz apenas `UPDATE Accounts SET Balance = 90` sem rastro.
   - Toda movimentação gera lançamentos contábeis imutáveis (`LedgerEntry`: **Débito** na conta origem, **Crédito** na conta destino). O saldo é o reflexo exato desses lançamentos.
2. **Controle de Concorrência (Race Conditions)**
   - O que acontece se o usuário disparar 2 Pix de R$ 100 ao mesmo tempo no milissegundo exato tendo apenas R$ 100 de saldo?
   - Usamos **Optimistic Concurrency** (`RowVersion` / `xmin` no EF Core) ou **Pessimistic Locking** (`SELECT ... FOR UPDATE` no SQL) dentro de transações ACID.
3. **Idempotência (`Idempotency-Key`)**
   - Se o app React ou um worker do RabbitMQ sofrer timeout de rede e reenviar a requisição de transferência, o banco **jamais** pode debitar duas vezes.
4. **Processamento Assíncrono Confiável (RabbitMQ + Outbox Pattern + DLQ)**
   - Usamos **MassTransit + RabbitMQ** com retentativas exponenciais, **Dead Letter Queue (DLQ)** para mensagens com falha e armazenamento de comprovantes no **AWS S3 (LocalStack)**.

---

## 2. Mapa Mental: De TypeScript/React para C# / .NET 8

| Conceito no TypeScript / Node / React | Equivalente no C# / .NET 8 | Onde usaremos no Corebanking |
| :--- | :--- | :--- |
| `interface AccountDto { ... }` | `public record AccountDto(...)` | DTOs imutáveis de Request/Response na API |
| `package.json` + `npm install` | `.csproj` + `dotnet add package` (NuGet) | Gerenciamento de pacotes por camada |
| Context API / Custom Hooks injetados | **Injeção de Dependência Nativa** (`AddScoped`, `AddSingleton`) | No `Program.cs` para injetar Repositórios e Serviços |
| `Promise<T>` + `async / await` | `Task<T>` + `async / await` (+ `CancellationToken`) | Todas as chamadas de banco, RabbitMQ e AWS |
| Array `.map()`, `.filter()`, `.reduce()` | **LINQ**: `.Select()`, `.Where()`, `.Sum()` | Consultas no domínio e no Entity Framework |
| Prisma / TypeORM | **Entity Framework Core (EF Core)** + **Dapper** | EF Core para transações complexas; Dapper para leitura ultra-rápida de extrato |
| Jest (`describe`, `it`, `expect`, `jest.fn()`) | **xUnit** (`[Fact]`, `[Theory]`) + **FluentAssertions** + **Moq/NSubstitute** | Testes unitários de domínio e casos de uso |

