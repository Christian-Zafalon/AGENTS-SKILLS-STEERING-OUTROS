---
name: analisar-arquitetura
description: Analisa, somente leitura e com base em evidências no código, a arquitetura de uma solução C#/.NET (projetos, camadas, dependências, fluxo principal, integrações, padrões, convenções). Use sempre que o usuário pedir para entender, mapear, explorar ou fazer engenharia reversa de uma solução .NET, ou antes de documentar ou replicar uma arquitetura.
---

# Analisar Arquitetura (C#/.NET)

Engenharia reversa objetiva. O resultado alimenta `documentar-arquitetura` e `validar-conformidade-arquitetural`.

## Regras inegociáveis

- **Somente leitura.** Nunca altere código. O único arquivo que pode ser gravado é `docs/arquitetura/analise.md`, e só depois de o usuário confirmar.
- **Evidência vale mais que nome.** Uma pasta `Repository` ou uma classe `Facade` não prova nada. Todo padrão precisa de evidência `arquivo:linha`.
- **Status obrigatório em cada achado:**
  - `[CONFIRMADO]` evidência direta no código (arquivo:linha).
  - `[INFERIDO]` dedução razoável a partir de evidências indiretas; diga qual.
  - `[NÃO CONFIRMADO]` sem evidência suficiente; diga o que faltaria para confirmar.
- Nunca invente o *motivo* de uma decisão. Se não houver ADR, comentário ou doc, escreva "motivo desconhecido".

## Economia de contexto

- Prefira glob e grep a ler arquivos. Leia com `view_range` quando o arquivo passar de ~200 linhas.
- Ignore sempre: `bin/`, `obj/`, `.git/`, `node_modules/`, `Migrations/`, `*.Designer.cs`, `*.g.cs`.
- Pare quando a evidência for suficiente. Não faça inventário exaustivo de classes.
- Para cada padrão candidato, leia no máximo 1 a 2 exemplos.

## Workflow

**Passo 0. Escopo.** Se o usuário não delimitou, analise a solução inteira em profundidade "mapa" (passos 1 a 3) e pergunte qual fluxo aprofundar. Se já existir `docs/arquitetura/analise.md`, pergunte se deve atualizar em vez de refazer.

**Passo 1. Inventário sem ler código-fonte.** Glob de `*.sln`, `*.csproj`, `Program.cs`, `appsettings*.json`, `Directory.Build.props`, `Directory.Packages.props`. Dos `.csproj`, extraia: `TargetFramework`, `ProjectReference`, `PackageReference`.

**Passo 2. Grafo de dependências.** Monte a direção das referências entre projetos a partir dos `ProjectReference`. Infira as camadas pela direção das referências, não pelo nome dos projetos.

**Passo 3. Composição e entrada.** Leia `Program.cs` (e extensões `AddXxx` chamadas nele). É onde se concentra a evidência: DI, middlewares, `HttpClient`, Options, autenticação, health checks.

**Passo 4. Fluxo principal.** Escolha 1 endpoint representativo e siga do ponto de entrada até a saída (integração externa, persistência ou resposta). Leia só os arquivos desse caminho.

**Passo 5. Padrões.** Para cada candidato: formule a hipótese, busque evidência por grep e classifique. Consulte `references/sinais-de-padroes.md` para saber a evidência mínima e os falsos positivos comuns de cada padrão.

**Passo 6. Convenções.** Amostre 2 a 3 arquivos para: nomenclatura, tratamento de erros, logging, configuração, testes.

**Passo 7. Saída.** Produza o relatório no formato de `references/modelo-analise.md`. Apresente no chat e pergunte se deve salvar em `docs/arquitetura/analise.md`.

## Fechamento

Termine sempre com a seção "Não confirmado / a investigar" e a seção "Escopo analisado" (o que foi lido e o que ficou de fora), para o usuário saber o limite da análise.
