# CLAUDE.md — Cérebro de Entrega (deliverybrain)

## O que é esse projeto
App de gestão operacional para açaiteria com duas lojas:
- **Raízes de Açaí** — loja principal
- **Vem do Norte** — segunda loja

Plataformas: iFood e 99Food (cada loja tem as duas plataformas = 4 combinações)

App publicado em: https://cerebro-entrega.vercel.app
Repositório: GitHub (sobe index.html direto na main)
Banco de dados: Supabase (URL: https://rmonkrnwtaabfnkmaack.supabase.co)
Chave pública: sb_publishable_Hx9SOdoMSh4h8mBjCPup2A_wFStx7_q

---

## Estrutura do banco (Supabase)

### Tabela: produtos
- id, loja_id, nome, tamanho_ml, ativo, created_at, updated_at, cmv_medio

### Tabela: pedidos
- id, loja_id, produto_id, plataforma_id, preco_venda, custo_producao
- custo_motoboy, taxa_plataforma, lucro, margem, distancia_km
- quantidade, status, created_at, pedido_ref

### Tabela: caixa
- Entradas e saídas manuais — repasse iFood/99Food cai toda quarta-feira

### Tabela: contas_pagar
- Contas fixas e variáveis da operação

### RLS (Row Level Security):
- produtos: TRUE (habilitado — política allow_all criada)
- pedidos: FALSE (sem RLS — acesso livre)
- demais tabelas: FALSE

---

## Taxas das plataformas
| Plataforma | Taxa | Entrega | Observação |
|---|---|---|---|
| iFood Raízes | 15.2% | Própria | Motoboy pago pelo dono (quarta) |
| 99Food Raízes | 3.2% | Própria | Motoboy pago pelo dono |
| iFood Vem do Norte | 30.2% | Plataforma | Entrega pelo iFood |
| 99Food Vem do Norte | 3.2% | Própria | Motoboy pago pelo dono |

Custo fixo por pedido: R$5,34
Motoboy por distância: até 1km=R$6 / até 2km=R$7 / acima=R$7,80
Repasse sempre cai na quarta-feira da semana seguinte

---

## CMVs reais pesados (cmv_medio no banco)
| Sabor | 330ml | 550ml | 770ml |
|---|---|---|---|
| Mix de Frutas | R$6,02 | — | — |
| Sonho de Ninho | R$6,44 | — | — |
| Brisa do Norte | R$6,98 | — | — |
| Três Amores | R$8,35 | — | R$15,31 |
| Avelinho | — | R$10,46 | R$14,49 |
| Dois Amores | — | — | R$14,96 |
| Avelã Crocante | — | — | R$14,00 |
| Cookies Cream | — | — | R$13,37 |

CMV médio por faixa de preço (usado quando produto não identificado):
- Até R$28 (330ml) → R$6,94
- Até R$38 (550ml) → R$10,46
- Acima (770ml) → R$14,43

---

## Estrutura do app (abas)
1. **Início** — visão geral rápida
2. **Caixa** — lançamentos manuais de entrada/saída
3. **Contas** — contas a pagar fixas e variáveis
4. **Análise** — importação de relatórios CSV/XLSX + gráficos
5. **Admin** — cadastro de produtos e insumos

---

## Lógica de importação de relatórios
- iFood: filtra STATUS FINAL DO PEDIDO === 'CONCLUIDO'
- 99Food: filtra Receita real da loja > 0 (cancela pedidos cancelados)
- Anti-duplicata: usa pedido_ref (ID completo do pedido)
- Faturamento bruto = preço que cliente pagou (não o repasse)
- Repasse real = lançado manualmente no Caixa (sempre quarta-feira)
- Após importar: ajusta período automaticamente pra cobrir datas do arquivo

---

## Precificação (simulador no Admin → Produtos)
Fórmula DNA do Lucro (Magno Fernandes):
`Preço = (CMV + custo fixo + frete) ÷ (1 - taxa - margem)`

Margem saudável: 25% por pedido
Frete varia por tamanho: 330ml=R$6 / 550ml=R$6,50 / 770ml=R$7

---

## Regras importantes
- NUNCA usar showToast() — função correta é mostrarToast()
- Campos corretos da tabela produtos: tamanho_ml e cmv_medio
- Campos corretos da tabela pedidos: custo_producao, taxa_plataforma, pedido_ref
- O app é um único arquivo index.html — tudo em um só arquivo
- Não criar abas novas sem necessidade — manter app limpo
- Sempre revisar nomes de colunas no Supabase antes de fazer update/insert
- Repasse financeiro real não bate com valor líquido do arquivo de pedidos (iFood desconta extras)

---

## Pendências abertas
- [ ] Preencher CMVs faltantes no banco (maioria sem tamanho_ml e cmv_medio)
- [ ] Ajustar preços Vem do Norte — margem atual 3.1% no iFood (urgente)
- [ ] Ajustar preço 330ml iFood Raízes — vários pedidos no prejuízo
- [ ] Importar relatórios de semanas anteriores
- [ ] Conciliação financeira: comparar repasse esperado vs recebido
- [ ] Lista de insumos completa (usuário vai passar em texto)
- [ ] CMV Cookies Cream 330ml e 550ml ainda não pesados
