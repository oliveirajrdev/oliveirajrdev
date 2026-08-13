# Missão Mesada — Arquitetura Conceitual de Produto

> Documento de concepção: estrutura conceitual, fluxo de telas, regras de interação e diferenciais de produto.
> Escopo: sem layout, paleta ou código — apenas arquitetura da informação e UX.

---

## 1. Conceito central: uma economia familiar de mérito

O Missão Mesada não é um "app de tarefas com pontos". Ele é uma **economia em miniatura** dentro da família, com três pilares:

1. **Trabalho gera valor** — missões concluídas viram **Estrelas** (moeda do app).
2. **Valor tem lastro real** — os pais definem a taxa de conversão (ex.: 1 ⭐ = R$ 0,50) e "sacam" as estrelas em mesada de verdade, ou em recompensas não-monetárias.
3. **Escolha ensina** — a criança decide entre **gastar** (loja de recompensas), **poupar** (metas de consumo com preço definido pelos pais) ou **sacar** (mesada em dinheiro).

Essa tríade **ganhar → decidir → conquistar** é o que transforma rotina doméstica em educação financeira, e não apenas em obediência recompensada.

### 1.1 Vocabulário do produto

| Termo | O que é |
|---|---|
| **Missão** | Uma tarefa/rotina atribuída à criança, com valor em Estrelas |
| **Estrela (⭐)** | Moeda interna, ganha por missões aprovadas |
| **Cofre** | Saldo da criança, dividido em "Livre" e "Reservado para metas" |
| **Meta** | Objeto de desejo (bicicleta, jogo) com preço em Estrelas definido pelos pais |
| **Loja da Família** | Catálogo de recompensas resgatáveis (tempo de tela, passeio, dinheiro) |
| **Saque** | Conversão de Estrelas em mesada real (aprovada/paga pelos pais) |
| **Cartão Amarelo/Vermelho** | Multa rápida por mau comportamento (desconto de Estrelas) |
| **Sequência (Streak)** | Dias consecutivos com todas as missões do dia cumpridas |
| **Nível / Patente** | Progressão de longo prazo da criança (não gastável, nunca diminui) |

### 1.2 Decisão de design fundamental: duas moedas, dois propósitos

- **Estrelas** são gastáveis e **podem ser perdidas** (multas, resgates). Representam dinheiro.
- **XP / Nível** é só progressão: **nunca diminui**. Representa reputação e esforço acumulado.

Por quê: se a punição zerasse "todo o progresso", a criança desanimaria e abandonaria o app. Multa dói no bolso (Estrelas), mas o histórico de esforço (Nível) permanece — exatamente como na vida real, onde uma multa não apaga sua carreira.

---

## 2. Modelo conceitual (entidades e relações)

```
Família
├── Responsáveis (1..n)  — pai, mãe, avó... com papéis (admin / colaborador)
├── Crianças (1..n)      — perfil com idade, avatar, nível, cofre
│
├── Missões
│   ├── Modelo de missão (template reutilizável, com valor sugerido por idade)
│   ├── Instância diária/semanal (gerada por recorrência)
│   └── Estados: Disponível → Em andamento → Entregue → Aprovada | Devolvida | Expirada
│
├── Economia
│   ├── Taxa de conversão (⭐ → R$), por criança
│   ├── Livro-razão (ledger): todo crédito/débito de Estrela com origem e autor
│   ├── Metas de consumo (item, preço em ⭐, foto, progresso, "entrada" opcional dos pais)
│   ├── Loja da Família (recompensas: monetárias, privilégios, experiências)
│   └── Saques (solicitação → aprovação → marcação de "pago")
│
└── Disciplina
    ├── Cartões (amarelo = advertência sem custo; vermelho = multa em ⭐)
    └── Motivos pré-definidos + campo livre
```

**Regra de ouro do modelo:** tudo que altera saldo passa pelo **ledger imutável**. Nada de "editar saldo na mão" — até ajustes manuais dos pais entram como lançamento com motivo. Isso dá transparência (a criança vê *por que* ganhou/perdeu) e é a base do extrato educativo.

---

## 3. As regras da economia (o coração do produto)

### 3.1 Ganho — missões e avaliação por qualidade

- Cada missão vale **N Estrelas base**, definidas na criação (com sugestões por idade/esforço).
- Na aprovação, o responsável dá uma **avaliação de 1 a 3 estrelas de qualidade**, que aplica multiplicador:
  - ⭐ "Feito nas coxas" → 50% do valor
  - ⭐⭐ "Bem feito" → 100%
  - ⭐⭐⭐ "Caprichou!" → 120% (bônus de excelência)
- **Por quê:** binário feito/não-feito ensina a cumprir; avaliação de qualidade ensina a *caprichar*. E dá aos pais uma alavanca pedagógica sem precisar rejeitar a entrega.

### 3.2 Multiplicadores de constância (o motor do hábito)

- **Sequência diária:** completar 100% das missões do dia mantém a chama acesa. A cada 7 dias de sequência, bônus crescente (ex.: +10%, +15%, +20%, teto em +30%).
- **Proteção de sequência:** 1 "Passe Livre" por mês (dia de folga sem quebrar a sequência) — evita o efeito "quebrei, desisto", conhecido de apps de hábito.
- **Missão relâmpago:** os pais podem marcar uma missão como "Relâmpago ⚡" (vale 2x se feita nas próximas X horas). Perfeito para "o lixo precisa descer AGORA".

### 3.3 Perda — multas com processo justo

- **Cartão amarelo:** advertência registrada, sem custo. 3 amarelos no mesmo motivo em 30 dias sugerem virar vermelho.
- **Cartão vermelho:** multa em Estrelas, com motivo obrigatório (lista rápida: "desrespeito", "não cumpriu combinado", "briga com irmão" + livre).
- **Trava anti-injustiça:** multa nunca deixa saldo negativo e **não confisca Estrelas já reservadas em meta** (o que já foi poupado é intocável — reforça a poupança como comportamento seguro).
- **Direito de resposta:** a criança pode "contestar" um cartão uma vez, com texto/áudio; o responsável confirma ou cancela. Não é burocracia — é a criança aprendendo a argumentar em vez de fazer birra.

### 3.4 Conversão e saque

- Taxa configurada por criança (permite pagar mais ao adolescente que ao caçula).
- **Dia da Mesada:** data configurável em que o app sugere/automatiza o saque do saldo livre. O pagamento em si é fora do app (dinheiro, Pix); o pai apenas marca "Pago ✓" e o ledger registra.
- **Modo Juros do Cofre (opcional, 10+ anos):** Estrelas paradas no cofre rendem X% ao mês, pago pelos pais. É a introdução mais concreta possível a juros compostos.

### 3.5 Metas de consumo (o "preço das coisas")

- A criança pede: "quero um skate". O pai cria a meta e **estipula o preço em Estrelas** (com calculadora auxiliar: valor em R$ ÷ taxa de conversão, ajustável).
- A criança **reserva** Estrelas na meta (transferência Cofre Livre → Meta). Pode resgatar de volta, mas com fricção proposital (confirmação + aviso "isso atrasa seu skate em ~2 semanas").
- **Entrada dos pais (matching):** opcionalmente, os pais configuram "a cada 2 ⭐ que você guardar, colocamos 1" — espelha o conceito de contrapartida/investimento e acelera metas grandes sem dá-las de graça.
- Meta atingida → celebração para os dois lados + item vai para a "Estante de Conquistas" da criança com foto do dia da compra.

---

## 4. App da Criança / Adolescente

### 4.1 Princípios

- **Zero fricção para agir:** abrir o app → ver o que fazer hoje → marcar como feito, em no máximo 2 toques.
- **Autonomia visível:** a criança nunca depende do pai para *saber* algo (quanto tem, quanto falta, por quê perdeu).
- **Dois modos de idade:** *Modo Explorador* (6–9: ícones grandes, áudio nas instruções, mapa lúdico) e *Modo Pro* (10+: visual de "banco/game", gráficos, sem infantilização — adolescente odeia app "de criancinha").

### 4.2 Navegação (4 abas + ação central)

```
┌──────────────────────────────────────────────┐
│  [Hoje]  [Missões]  (✓ Entregar)  [Cofre]  [Eu] │
└──────────────────────────────────────────────┘
```

#### Tela 1 — **Hoje** (home)
A pergunta que responde: *"o que eu faço agora e como estou indo?"*

- **Cabeçalho vivo:** saldo de Estrelas + chama da sequência (com contagem de dias) + barra de nível.
- **Trilha do dia:** missões de hoje como uma trilha/checklist ordenada por período (manhã / tarde / noite). Missões Relâmpago ⚡ aparecem no topo com cronômetro.
- **Card da meta principal:** "Faltam 120 ⭐ para o skate" com barra de progresso — o desejo sempre à vista é o que faz a missão chata valer a pena.
- **Feed curto de novidades:** "Papai aprovou 'Arrumar a cama' ⭐⭐⭐ +12", "Nova missão disponível".

#### Tela 2 — **Missões**
- Abas internas: *Hoje* / *Semana* / *Extras*.
- **Extras = Mural de Bicos:** missões avulsas publicadas pelos pais que **qualquer filho pode pegar** (primeiro que aceitar, leva). Introduz iniciativa e, entre irmãos, uma competição saudável por oportunidades — como no mercado de trabalho.
- Detalhe da missão: descrição, valor, prazo, se exige foto, e instruções (com áudio no Modo Explorador).

#### Ação central — **Entregar** (botão de destaque)
- Fluxo de 10 segundos: escolher missão em andamento → foto opcional/obrigatória (conforme configuração) → enviar → animação de "missão entregue, aguardando aprovação".
- Estado "aguardando" é visível mas **não bloqueia**: a criança segue para a próxima missão.

#### Tela 3 — **Cofre** (o banco da criança)
- Saldo dividido visualmente: **Livre** | **Reservado em metas**.
- **Minhas Metas:** cards com foto do objeto, progresso, previsão ("no seu ritmo, faltam ~18 dias") e botão "Guardar Estrelas".
- **Loja da Família:** recompensas resgatáveis agora (30 min de videogame extra, escolher o filme da sexta, R$ 10 no Pix...). Resgate gera pedido para aprovação dos pais.
- **Extrato:** cada entrada e saída com motivo — a criança aprende a ler um extrato bancário sem perceber.
- **Botão Sacar:** solicita conversão em dinheiro (aparece a partir da idade/configuração definida pelos pais).

#### Tela 4 — **Eu** (identidade e progresso)
- Avatar customizável — itens cosméticos são **comprados com XP/nível, nunca com Estrelas** (cosmético não compete com a poupança real).
- Patentes ("Recruta" → "Caçador de Missões" → "Lenda da Casa"), álbum de medalhas (1ª meta atingida, 30 dias de sequência, 100 missões).
- **Estante de Conquistas:** fotos das metas realizadas — o "hall da fama" pessoal que prova que poupar funciona.

### 4.3 O ciclo diário da criança (por que ela abre o app)

| Momento | Gatilho | Ação no app | Recompensa emocional |
|---|---|---|---|
| Manhã | Notificação "Suas missões de hoje chegaram 🌅" | Ver a trilha do dia | Clareza + chama da sequência em jogo |
| Durante o dia | Missão Relâmpago ⚡ / lembrete de período | Executar e entregar com foto | Animação de entrega, expectativa da avaliação |
| Fim da tarde | "Papai aprovou! ⭐⭐⭐ +12" | Abrir para ver avaliação e saldo | Reconhecimento (a avaliação de qualidade é elogio dos pais em forma de jogo) |
| Noite | "Feche o dia: falta 1 missão para manter sua chama 🔥" | Completar a última | Fechamento do dia, sequência mantida, barra da meta subindo |

Os três ganchos de retenção, em ordem de força: **a meta de consumo visível** (desejo concreto), **a sequência com proteção** (aversão à perda), e **a avaliação dos pais** (reconhecimento afetivo disfarçado de mecânica).

---

## 5. App dos Pais (Painel de Gestão)

### 5.1 Princípios

- **O pai é um gestor com 90 segundos por dia.** Tudo que é rotineiro tem que ser resolvido da tela inicial, sem navegar.
- **Criar deve ser mais fácil que cobrar:** publicar uma missão precisa ser mais rápido que gritar "arruma teu quarto!" da cozinha.
- **Transparência bidirecional:** toda ação do pai vira registro que a criança vê — o app é o "contrato" da família.

### 5.2 Navegação (4 abas + ação central)

```
┌─────────────────────────────────────────────────┐
│ [Painel] [Missões] (+ Criar) [Financeiro] [Família] │
└─────────────────────────────────────────────────┘
```

#### Tela 1 — **Painel** (home = caixa de entrada de decisões)
Ordenada por "o que precisa de mim agora":

1. **Fila de Aprovações** — o recurso mais importante do app. Cards com foto da entrega, deslizáveis:
   - **Deslizar para a direita** = aprovar (e escolher ⭐/⭐⭐/⭐⭐⭐ de qualidade em um toque);
   - **Deslizar para a esquerda** = devolver com motivo rápido ("refazer", "faltou X") — devolver não é punição, é segunda chance com prazo;
   - Aprovação em lote para dias corridos ("aprovar tudo com ⭐⭐").
2. **Resgates e saques pendentes** — aprovar recompensa da loja / marcar mesada como paga.
3. **Pulso do dia por filho:** avatar + anel de progresso ("Ana 3/4", "Léo 1/5 ⚠️") + chama da sequência.
4. **Botões de ação rápida:** `⚡ Missão Relâmpago` · `🟨🟥 Cartão` · `+ Bônus surpresa`.

#### Ação central — **Criar** (o "menos de 20 segundos")
- **Biblioteca de modelos por idade** ("Arrumar a cama, 6+, 5 ⭐ sugeridas") — dois toques: escolher filho(s), confirmar.
- **Criação por voz/texto livre:** "Léo, tirar o lixo todo dia às 19h, 5 estrelas" → o app monta a missão estruturada para confirmação.
- Recorrência simples (dias da semana), exigência de foto, prazo, e opção "publicar no Mural de Bicos".

#### Tela 2 — **Missões**
- Visão semanal por filho (grade dia × missão) — enxergar sobrecarga ou folga de rotina num relance.
- Gestão dos modelos da família (editar valores, aposentar missões que viraram hábito — ver §7).

#### Tela 3 — **Financeiro** (o banco central da família)
- **Por filho:** saldo, reservado em metas, projeção da próxima mesada em R$.
- **Taxa de conversão** e simulador ("nesse ritmo, Léo ganha ~R$ 42/mês").
- **Metas:** criar/precificar metas, configurar matching ("a cada 2 ⭐ dele, 1 nossa"), acompanhar progresso.
- **Loja da Família:** gerenciar catálogo de recompensas (sugestões prontas + personalizadas).
- **Dia da Mesada:** configurar data, ver histórico de pagamentos, marcar "pago".
- **Teto de gasto mensal:** o app avisa se o total prometido em Estrelas ultrapassa o orçamento que o pai definiu — protege o pai de "quebrar" e minar a confiança no sistema.

#### Tela 4 — **Família**
- Perfis dos filhos (idade → modo de interface, permissões, taxa individual).
- Co-gestores (outro responsável, avós) com papéis: admin ou colaborador (colaborador aprova missões, não mexe em dinheiro).
- **Relatório mensal ("Boletim"):** taxa de conclusão, pontualidade, qualidade média, evolução da poupança — e **insights acionáveis**: "Ana falha nas missões de manhã; que tal movê-las para a tarde?", "Léo gasta tudo no dia que recebe; sugerimos ativar o Juros do Cofre".

### 5.3 O ciclo diário dos pais (< 2 minutos)

- **Manhã (passivo):** missões recorrentes se publicam sozinhas. O pai não faz nada.
- **Ao longo do dia (10 s por evento):** notificação de entrega → deslizar para aprovar direto da notificação (ação rápida de push, sem abrir o app).
- **Noite (60 s):** resumo das 20h30 — "3 entregas para avaliar, 1 resgate pendente" → fila de aprovações → pronto.
- **Domingo (5 min, opcional):** revisão da semana + boletim + ajustes de rotina sugeridos pelo app.

---

## 6. Regras de interação entre as duas pontas

| Situação | Regra | Racional |
|---|---|---|
| Entrega sem avaliação | Aprovação automática com ⭐⭐ após 24 h (configurável) | O esquecimento do pai nunca pode punir a criança — mata a confiança no sistema |
| Missão não feita no prazo | Expira sem multa automática; apenas quebra a sequência e fica no histórico | Punir por omissão automaticamente gera injustiça (a criança pode ter tido um imprevisto); o pai decide se cabe cartão |
| Devolução ("refazer") | Sempre com motivo + novo prazo curto; 2ª entrega não sofre redução | Devolver ensina padrão de qualidade; reduzir valor na segunda tentativa ensinaria a desistir |
| Multa (cartão vermelho) | Motivo obrigatório, notificação à criança com direito a 1 contestação | Disciplina sem processo vira arbitrariedade — e a contestação canaliza a revolta para o diálogo |
| Resgate na Loja | Pedido → aprovação → "voucher" na tela da criança para mostrar na hora de usar | O voucher físico-digital ("30 min de videogame ✓ liberado por Papai às 19h") elimina o "mas você deixou!" |
| Irmãos | Bicos são disputáveis; comparação de saldo entre irmãos **não existe** no app da criança | Competição por oportunidade é saudável; ranking de riqueza entre irmãos é briga garantida |
| Pais separados / dois lares | Dois admins com carteiras opcionalmente separadas ("mesada da casa da mãe / do pai") no mesmo perfil da criança | Realidade de grande parte do mercado; o app vira terreno neutro de combinados |

### Anti-gaming (dos dois lados)

- **Criança:** foto obrigatória configurável por missão; entrega em rajada (5 missões em 2 min) é sinalizada ao pai; Estrelas só entram após aprovação.
- **Pais:** o ledger é imutável e visível — remoções de Estrela sempre aparecem com motivo. O app protege a criança de "confisco silencioso", porque a confiança da criança no sistema **é** o produto.

---

## 7. Diferenciais de produto (o que eu proporia para este mercado)

1. **Avaliação de qualidade com multiplicador (⭐→⭐⭐⭐)** — concorrentes tratam tarefa como checkbox. Avaliar qualidade transforma aprovação em feedback pedagógico e dá aos pais nuance sem conflito.
2. **Duas moedas (Estrela gastável vs. XP permanente)** — permite punir financeiramente sem destruir a motivação de longo prazo. É a decisão de design que sustenta a mecânica de multas.
3. **Poupança blindada + matching dos pais** — Estrelas reservadas em meta são imunes a multa, e os pais podem dar contrapartida. Ensina que poupar é seguro e recompensado: educação financeira embutida na mecânica, não em "conteúdo educativo" que ninguém lê.
4. **Mural de Bicos entre irmãos** — missões abertas disputáveis criam iniciativa ("vou olhar se apareceu bico novo") — um gatilho de abertura do app que não depende de notificação.
5. **Cartões com direito de resposta** — nenhum concorrente modela o *processo justo* da disciplina. Vira argumento de venda para os pais ("o app briga com seu filho por você, com regras claras").
6. **Aprovação pela notificação (10 segundos)** — o pai gerencia sem abrir o app. A maior causa de abandono nesses apps é o *pai* desistir, não a criança; o design otimiza para o elo mais fraco.
7. **Boletim com insights acionáveis** — o app não só registra, aconselha: horários com mais falha, padrão gastador/poupador, sugestão de aposentar missão dominada.
8. **Aposentadoria de missão (graduação de hábito)** — quando uma missão tem ~30 dias de conclusão consistente, o app sugere: "Arrumar a cama virou hábito da Ana 🎓. Aposentar a recompensa e liberar espaço para uma missão nova?" — com cerimônia de "graduação" para a criança (medalha permanente). **Este é o diferencial filosófico:** o objetivo do app é a criança *deixar de precisar* de recompensa para aquilo — combate a crítica clássica ("recompensa mata motivação intrínseca") e dá aos pais um argumento pedagógico que nenhum concorrente oferece.
9. **Voucher de recompensa resgatada** — a recompensa aprovada vira um "ticket" visível (quem liberou, quando, validade). Resolve a fonte nº 1 de conflito pós-resgate.
10. **Modo dois lares** — carteiras separadas por responsável para famílias de pais separados: mercado enorme, dor real, ignorada pelos apps atuais.

---

## 8. Onboarding (os primeiros 10 minutos decidem tudo)

1. **Pai baixa e cria a família** → assistente pergunta idades → **sugere um "pacote inicial" de 4–5 missões apropriadas por filho** (evita a tela em branco).
2. Define taxa de conversão com ajuda ("famílias com filhos de 8 anos costumam praticar ~R$ X/semana").
3. **Convida a criança por QR code / código curto** — o app da criança nasce já com missões do dia e uma **missão-tutorial** ("Configure seu avatar: +5 ⭐") para o primeiro ganho acontecer em 2 minutos.
4. Primeira semana em **"modo aquecimento"**: valores levemente inflados e aprovação incentivada com ⭐⭐⭐ — o objetivo é a criança chegar ao primeiro resgate/meta parcial em 7 dias (momento "aha!" das duas pontas: *isso funciona*).

---

## 9. Resumo dos fluxos (mapa de navegação)

```
CRIANÇA                                   PAIS
━━━━━━━                                   ━━━━
Hoje ──────────────┐                      Painel ─────────────┐
 ├─ trilha do dia  │                       ├─ fila de aprovações (swipe + nota ⭐..⭐⭐⭐)
 ├─ chama/nível    │                       ├─ resgates & saques pendentes
 └─ card da meta   │                       ├─ pulso por filho
Missões            │                       └─ ações rápidas (⚡ relâmpago, 🟨🟥 cartão, 🎁 bônus)
 ├─ hoje/semana    │                      Missões
 └─ mural de bicos │                       ├─ grade semanal por filho
[Entregar] ← ação central                  └─ modelos da família
 └─ foto → enviar → aguardando            [+ Criar] ← ação central
Cofre                                      └─ modelo | voz/texto livre | recorrência
 ├─ livre / reservado                     Financeiro
 ├─ metas (guardar ⭐)                     ├─ saldos, taxa, simulador, teto de gasto
 ├─ loja da família                        ├─ metas & matching │ loja │ dia da mesada
 ├─ extrato                                └─ histórico de pagamentos
 └─ sacar                                 Família
Eu                                         ├─ perfis, modos por idade, co-gestores
 ├─ avatar (XP), patentes, medalhas        └─ boletim mensal + insights
 └─ estante de conquistas
```

---

*Documento vivo — próxima etapa sugerida: priorização de MVP (fila de aprovações + missões recorrentes + cofre com uma meta) e prototipação dos dois fluxos de 10 segundos (entregar / aprovar).*
