

## Diagnóstico: Contatos Perdidos na Geração de Lista

### Problema Encontrado

A busca de leads encontrou **45 resultados** (pessoas + empresas fantasma), mas apenas **13 contatos** foram salvos na lista. A investigação do banco revelou três causas raízes:

### Causa 1 — Campo `company_phone` ignorado
Muitos contatos têm apenas `company_phone` preenchido (30 de 45), mas o `GenerateListDialog` só verifica `whatsapp_site` e `phone`. Contatos como "Camila Andrade" (Segura Logistics) que têm `company_phone: (11) 4199-2399` mas `phone: ""` são filtrados como "sem telefone" e nunca chegam à lista.

### Causa 2 — Número salvo errado
Na linha 139, o upsert usa `c.phone` diretamente, mas o filtro `contactsWithPhone` usa `getBestPhone(c)` que prefere `whatsapp_site`. Resultado: contatos que passam no filtro por terem `whatsapp_site` são salvos com o `phone` vazio.

### Causa 3 — Deduplicação por telefone colapsa contatos
Contatos "fantasma" de empresa (ex: 3 pessoas da Brix Cargo) compartilham o mesmo `company_phone`/`whatsapp_site`. O `upsert_lead_contact` deduplica por telefone, então 3 viram 1.

### Causa 4 — URLs de WhatsApp não parseadas
Alguns `whatsapp_site` são URLs (`https://wa.me/5511971765049?text=...`) em vez de números limpos. O sistema não extrai o número deles.

---

### Plano de Correção

#### 1. Melhorar `getBestPhone` no `GenerateListDialog.tsx`
- Adicionar fallback para `company_phone`
- Parsear URLs `wa.me` para extrair o número
- Hierarquia: `whatsapp_site` (se número limpo) → `phone` → `company_phone`

#### 2. Corrigir o upsert para usar o melhor telefone
- Na linha 139, substituir `c.phone` por `getBestPhone(c)` normalizado

#### 3. Evitar colapso de contatos distintos
- Quando múltiplas pessoas compartilham o mesmo número (empresa), criar cada contato com nome distinto para preservar a identidade individual
- Alternativa: usar o nome+empresa como critério de dedup ao invés de só telefone

---

### Arquivos Afetados
- `src/components/scraping/GenerateListDialog.tsx` — lógica de seleção de telefone e upsert

### Resultado Esperado
Com a correção, os 30 contatos que possuem algum telefone (pessoal ou corporativo) seriam elegíveis para a lista, em vez dos 13 atuais.

