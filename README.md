# Accounting-entry-generator-with-standard-Dom-nio-layout
# 📒 Importação Contábil Domínio ERP — Sistema Prompt v1.0

> **System prompt de produção** para geração automatizada de arquivos TXT no layout Domínio Web ERP, cobrindo contabilização de documentos brutos e conversão de planilhas pré-contabilizadas com conformidade aos Pronunciamentos Técnicos (CPCs).

[![Status](https://img.shields.io/badge/status-produção-green)]()
[![ERP](https://img.shields.io/badge/ERP-Domínio%20Web-blue)]()
[![CPCs](https://img.shields.io/badge/CPCs-18%20mapeados-orange)]()
[![Tolerância](https://img.shields.io/badge/alucinação-tolerância%20zero-red)]()

---

## 🎯 O Problema

O Domínio Web é o ERP contábil mais usado em escritórios de contabilidade no Brasil. Sua importação em lote exige um layout TXT proprietário extremamente rigoroso — um pipe fora do lugar, um ponto no lugar de vírgula ou um `6000|V||||` duplicado já invalida o arquivo inteiro.

Contadores passam horas por mês montando esses arquivos manualmente a partir de extratos bancários, guias de tributos, folhas de pagamento e planilhas internas. O processo é repetitivo, propenso a erro humano e não escala.

Um LLM genérico resolve parte do problema mas falha nos detalhes críticos: inventa códigos de conta que não existem no plano, usa ponto como separador decimal, duplica registros `6000` para lançamentos do mesmo dia, e não sabe qual CPC aplicar para cada operação.

---

## 💡 A Solução

Um system prompt que atua como um **contador sênior + especialista em Domínio**, capaz de:

- **Cenário A:** Receber documentos brutos (extratos, guias, folhas) → classificar pelo CPC correto → gerar o TXT pronto para importar
- **Cenário B:** Receber planilha pré-contabilizada → validar partidas dobradas → converter para o layout Domínio sem alterar nenhum valor

Com tolerância zero a alucinação: se não encontra a conta no plano fornecido, bloqueia e alerta — nunca inventa.

---

## 🏗️ Arquitetura do Prompt

```
system_prompt/
├── MISSÃO                         # Dois cenários de atuação
├── REGRAS ANTI-ALUCINAÇÃO         # 5 regras de tolerância zero
├── TOKEN_OPTIMIZATION_AGGRESSIVE  # Output compacto por padrão
│   ├── Cenário A: mapeamento compacto interno
│   ├── Cenário B: validação silenciosa + TXT direto
│   └── Formato padrão: TXT + alertas apenas
├── PROCESSAMENTO DE ENTRADAS      # Classificação por tipo de documento
├── FLUXO OBRIGATÓRIO              # Cenário A (6 passos) e B (3 passos)
├── MAPA CPC × OPERAÇÃO            # 18 CPCs mapeados por tipo de operação
└── LAYOUT TXT DOMÍNIO             # Especificação completa do formato
    ├── Regra 6000|V||||           # UMA vez por nova data
    ├── 10 pipes, 10 campos        # Estrutura obrigatória
    ├── Formatação de valores      # Vírgula decimal, sem milhar
    ├── COD_FILIAL                 # Bloqueio se não informado
    ├── COD_SCP                    # Coluna MAT/MATRICULA
    └── Exemplo errado vs correto  # Contraste explícito
```

---

## 📐 Layout TXT Domínio — Especificação Completa

### Estrutura de campos

```
0000|{CNPJ_14_DIGITOS}|
6000|V||||                                     ← UMA vez por nova data
6100|{DD/MM/AAAA}|{CONTA_DEB}||{VALOR}||{HISTORICO}||{COD_FILIAL}|{COD_SCP}|
6100|{DD/MM/AAAA}||{CONTA_CRED}|{VALOR}||{HISTORICO}||{COD_FILIAL}|{COD_SCP}|
```

### Regras críticas de formatação

| Regra | ✅ Correto | ❌ Incorreto |
|-------|-----------|------------|
| Decimal | `5000,00` | `5000.00` ou `5.000,00` |
| 6000 por data | 1 registro antes da primeira linha | 1 por lançamento |
| Mesma data | Sem 6000 entre lançamentos | Com 6000 entre eles |
| COD_FILIAL ausente | Bloqueia e solicita | Assume ou deixa vazio |
| Conta não encontrada | Alerta e separa | Inventa código |

### Exemplo — mesma data, múltiplos lançamentos

```
✅ CORRETO
0000|00000000000191|
6000|V||||
6100|31/01/2025|331||3894,07||Provisão folha 01/2025||001||
6100|31/01/2025||187|3894,07||Provisão folha 01/2025||001||
6100|31/01/2025|336||778,81||INSS patronal 01/2025||001||
6100|31/01/2025||191|778,81||INSS patronal 01/2025||001||
6100|31/01/2025|337||311,52||FGTS 01/2025||001||
6100|31/01/2025||192|311,52||FGTS 01/2025||001||
6000|V||||                    ← Só aqui porque mudou a data
6100|01/02/2025|934||7000,00||Provisão folha FEV/2025||001||
6100|01/02/2025||920|7000,00||Provisão folha FEV/2025||001||

❌ ERRADO — 6000 duplicado para mesmo dia
6000|V||||
6100|31/01/2025|331||3894,07||Provisão folha 01/2025||001||
6100|31/01/2025||187|3894,07||Provisão folha 01/2025||001||
6000|V||||   ← ERRO: mesma data, não precisa de novo 6000
6100|31/01/2025|336||778,81||INSS patronal 01/2025||001||
```

---

## 🗺️ Mapa CPC × Operação

| Operação | CPC |
|----------|-----|
| Definir ativo/passivo/receita/despesa; dúvida de classificação | **00** |
| Receita de serviços, vendas, contratos, construção civil | **47** |
| Provisão de folha, férias, 13º, tributos, contingências | **25** |
| Tributos diferidos (IRPJ/CSLL) | **32** |
| Regime de competência, circulante/não circulante | **26** |
| Fluxo de caixa | **03** |
| Imobilizado (compra, depreciação, baixa) | **27** |
| Intangível (software, marcas, patentes) | **04** |
| Estoques | **16** |
| Impairment | **01** |
| Arrendamento mercantil / leasing | **06** |
| Benefícios a empregados (Folha de Pagamento) | **33** |
| Custos de empréstimos e financiamentos | **20** |
| Ajuste a Valor Presente (dívidas longo prazo) | **12** |
| Aplicações financeiras, empréstimos, derivativos | **48** |
| Demonstrações consolidadas | **36** |
| Valor justo | **46** |
| PMEs | **PME** |

> CPCs revogados — **nunca usar:** 13, 14, 17, 30, 34, 38

---

## 🛑 Regras Anti-Alucinação

O sistema opera com **5 regras de tolerância zero**, aplicadas antes de qualquer geração de output:

```
1. CONTAS CONTÁBEIS
   Nunca inventa, presume ou "adivinha" códigos reduzidos.
   Se não encontrar no Plano de Contas → separa e alerta.

2. VALORES
   Cópia exata do documento de origem.
   Nunca altera, arredonda ou cria valores.

3. HISTÓRICOS
   Construídos apenas com elementos factuais do extrato/planilha.
   Nunca narra o que não está nos dados.

4. AUSÊNCIA DE DADOS
   Partida dobrada incompleta → lançamento pendente + alerta.
   Nunca deduz a origem de um recurso desconhecido.

5. ESTRUTURA OFICIAL
   Acompanha rigorosamente o layout de LANÇAMENTO EM LOTE.
   Qualquer desvio estrutural é bloqueado antes do output.
```

---

## ⚡ Token Optimization

O sistema retorna respostas compactas por padrão, expandindo apenas quando necessário.

```
MODO COMPACTO (padrão)
✅ TXT gerado
⚠️ Alertas: 2 problema(s) → conta 334 não localizada no plano | D≠C em 15/03
📎 arquivo.txt

MODO COMPLETO (digitar "completo" ou lote ≤ 10 lançamentos)
→ Raciocínio CPC linha a linha
→ Conferência de todas as partidas
→ Alertas detalhados com contexto
```

**Economia em sessão multi-documento:** Plano de Contas mapeado uma vez na sessão → reutilizado em todos os períodos seguintes sem reprocessamento.

---

## 🔧 Como Usar

### Pré-requisitos
- Acesso à API da Anthropic (Claude Sonnet 4.6 ou superior)
- Plano de Contas do cliente em texto ou PDF
- Documentos para contabilizar (extrato, guia, folha ou planilha)

### Setup Básico

```python
import anthropic

client = anthropic.Anthropic()

with open("system_prompt.txt", "r") as f:
    system_prompt = f.read()

# Fluxo recomendado:
# 1. Primeira mensagem: envie o Plano de Contas
# 2. Mensagens seguintes: envie cada documento a contabilizar

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    system=system_prompt,
    messages=[
        {
            "role": "user",
            "content": """
            CNPJ da empresa: 00.000.000/0001-91
            Filial: 001
            
            Plano de Contas:
            [cole ou anexe o plano de contas aqui]
            
            Documento para contabilizar:
            [cole ou anexe o extrato/guia/folha]
            """
        }
    ]
)

print(response.content[0].text)
```

### Fluxo Recomendado para Múltiplos Documentos

```
Sessão de trabalho típica:

Turn 1: [Plano de Contas] → sistema mapeia e armazena
Turn 2: [Folha de pagamento Jan] → gera TXT folha
Turn 3: [Guia PGDAS Jan] → gera TXT tributos  
Turn 4: [Extrato bancário Jan] → gera TXT banco
Turn 5: "completo" + [Extrato com dúvida] → análise detalhada com CPC
```

### Comandos Disponíveis

```
"completo"         → Resposta detalhada com raciocínio CPC
[sem comando]      → Modo compacto (padrão)
```

---

## 📁 Estrutura do Repositório

```
importacao-contabil-dominio/
├── README.md
├── system_prompt.txt              ← Prompt completo pronto para uso
├── examples/
│   ├── folha_pagamento.txt        ← TXT gerado de folha CLT
│   ├── pgdas_tributos.txt         ← TXT gerado de guia PGDAS
│   ├── extrato_bancario.txt       ← TXT gerado de extrato
│   └── planilha_precontab.txt     ← TXT gerado de planilha pronta
└── docs/
    ├── layout_dominio.md          ← Especificação completa do layout
    └── mapa_cpc.md                ← CPC por tipo de operação com exemplos
```

---

## 🔗 Integração com o Auditor de Folha

Este repositório e o [auditor-folha-pagamento](https://github.com/seu-usuario/auditor-folha-pagamento) foram projetados para operar em sequência:

```
Fluxo completo de folha de pagamento:

1. [auditor-folha-pagamento]
   → Recebe holerite
   → Detecta divergências e erros
   → Gera relatório de auditoria com valores corrigidos

2. [importacao-contabil-dominio]
   → Recebe resumo de folha auditado
   → Classifica por CPC 33 (Benefícios a Empregados)
   → Gera TXT pronto para importar no Domínio Web
```

---

## 👤 Autor

**Carlos Torres** — AI Solutions Architect  
[LinkedIn](https://linkedin.com/in/carlostorressjr) · [GitHub](https://github.com/CTorressjr)

> Desenvolvido a partir da integração real com o Domínio Web ERP, processando lançamentos contábeis de centenas de empresas clientes mensalmente.

---

## ⚠️ Aviso Legal

Este sistema é uma ferramenta de apoio à escrituração contábil. Os lançamentos gerados devem ser revisados por contador habilitado (CRC) antes da importação definitiva. O autor não se responsabiliza por lançamentos incorretos que resultem em apuração fiscal inadequada.
