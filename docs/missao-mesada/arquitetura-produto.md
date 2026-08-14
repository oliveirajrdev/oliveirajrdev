# Missão Mesada — Arquitetura Conceitual de Produto

> Documento de concepção: estrutura conceitual, fluxo de telas, regras de interação e diferenciais de produto.
> Escopo: sem layout, paleta ou código — apenas arquitetura da informação e UX.
>
> **v2 — sincronizado com o protótipo navegável** (`prototipo.html`). Este documento evoluiu junto com várias rodadas de prototipagem: algumas mecânicas da v1 foram substituídas por decisões tomadas durante esse processo (ver §10 para o histórico). O que está descrito aqui é o que o protótipo implementa hoje.

---

## 1. Conceito central: uma economia familiar de mérito

O Missão Mesada não é um "app de tarefas com pontos". Ele é uma **economia em miniatura** dentro da família, com três pilares:

1. **Trabalho gera valor** — missões concluídas viram **Estrelas** (moeda do app).
2. **Valor tem lastro real** — os pais definem a cotação (ex.: 1 ⭐ = R$ 0,50) e as Estrelas viram mesada de verdade ou recompensas não-monetárias.
3. **Escolha ensina** — a criança decide entre **trocar** (catálogo de recompensas, incluindo a meta em destaque) ou **converter em dinheiro** (troca por Pix).

Essa tríade **ganhar → decidir → conquistar** é o que transforma rotina doméstica em educação financeira, e não apenas em obediência recompensada.

### 1.1 Vocabulário do produto

| Termo | O que é |
|---|---|
| **Missão / Tarefa** | Uma rotina atribuída à criança, com valor em Estrelas |
| **Estrela (⭐)** | Moeda interna, ganha por missões aprovadas |
| **Saldo** | Total de Estrelas da criança — um único saldo, sem divisão entre "livre" e "reservado" |
| **Meta em destaque** | O item mais caro/desejado do catálogo de recompensas, com card e barra de progresso próprios |
| **Catálogo de recompensas** | Todos os itens trocáveis por Estrelas (a meta em destaque + privilégios, experiências, dinheiro) |
| **Troca** | Ato de trocar Estrelas por um item do catálogo (substituiu o antigo "resgate") |
| **Troca por Pix** | Conversão do saldo total em dinheiro real, paga no Dia da Mesada |
| **Sugestão** | Pedido de recompensa feito pela criança, ainda sem preço — aguarda o pai precificar e aprovar |
| **Cartão Amarelo/Vermelho** | Multa rápida por mau comportamento (desconto de Estrelas) |
| **Sequência (Streak)** | Dias consecutivos com todas as missões do dia cumpridas |
| **Nível / Patente** | Progressão de longo prazo da criança (não gastável, nunca diminui) |
| **Vínculo** | Código/QR que conecta o dispositivo de um filho ou responsável à família |
| **Controle de acesso** | Papéis dos responsáveis (Admin / Colaborador) e o que cada um pode fazer |

### 1.2 Decisão de design fundamental: duas moedas, dois propósitos

- **Estrelas** são gastáveis e **podem ser perdidas** (multas, trocas). Representam dinheiro.
- **XP / Nível** é só progressão: **nunca diminui**. Representa reputação e esforço acumulado.

Por quê: se a punição zerasse "todo o progresso", a criança desanimaria e abandonaria o app. Multa dói no bolso (Estrelas), mas o histórico de esforço (Nível) permanece — exatamente como na vida real, onde uma multa não apaga sua carreira.

---

## 2. Modelo conceitual (entidades e relações)

```
Família
├── Responsáveis (1..n)  — papéis: admin / colaborador, cada um com um Vínculo próprio
├── Crianças (1..n)      — perfil com idade, avatar, nível, saldo, Vínculo próprio
│
├── Missões
│   ├── Missão cadastrada (definição reutilizável: nome, ícone, valor, recorrência, horário, foto)
│   ├── Instância do dia (gerada pela recorrência, é o que a criança vê em "Tarefas")
│   └── Estados: A fazer → Entregue → Aprovada | Devolvida | Expirada
│
├── Economia
│   ├── Cotação (⭐ → R$), por criança — configurada só pelos pais
│   ├── Livro-razão (ledger): todo crédito/débito de Estrela com origem e motivo
│   ├── Catálogo de recompensas (itens com preço em ⭐; um deles pode ser a "meta em destaque")
│   ├── Sugestões (pedidos da criança, sem preço, aguardando o pai)
│   └── Trocas (solicitação → aprovação → voucher | pagamento via Pix)
│
└── Disciplina
    ├── Cartões (amarelo = advertência sem custo; vermelho = multa em ⭐)
    └── Motivos pré-definidos + campo livre
```

**Regra de ouro do modelo:** tudo que altera saldo passa pelo **ledger imutável**. Nada de "editar saldo na mão" — até ajustes manuais dos pais entram como lançamento com motivo. Isso dá transparência (a criança vê *por que* ganhou/perdeu) e é a base do extrato educativo.

---

## 3. As regras da economia (o coração do produto)

### 3.1 Ganho — missões e avaliação por qualidade

- Cada missão vale **N Estrelas base**, definidas no cadastro.
- Na aprovação, o responsável dá uma **avaliação de 1 a 3 estrelas de qualidade**, que aplica multiplicador:
  - ⭐ "Feito nas coxas" → 50% do valor
  - ⭐⭐ "Bem feito" → 100%
  - ⭐⭐⭐ "Caprichou!" → 120% (bônus de excelência)
- **Por quê:** binário feito/não-feito ensina a cumprir; avaliação de qualidade ensina a *caprichar*. E dá aos pais uma alavanca pedagógica sem precisar rejeitar a entrega.
- No protótipo, essa nota é escolhida em um **modal dedicado**, aberto ao tocar "Aprovar" no card compacto da fila (ver §5).

### 3.2 Multiplicadores de constância (o motor do hábito)

- **Sequência diária:** completar 100% das missões do dia mantém a chama 🔥 acesa, exibida junto da lista de tarefas do dia.
- **Missão relâmpago:** os pais podem marcar uma missão como "Relâmpago ⚡" (vale 2× se feita nas próximas X horas). Perfeito para "o lixo precisa descer AGORA". *(No protótipo atual, essa ação ainda não tem um botão de acesso — ver §10.)*

### 3.3 Perda — multas com processo justo

- **Cartão amarelo:** advertência registrada, sem custo.
- **Cartão vermelho:** multa em Estrelas, com motivo obrigatório (lista rápida: "desrespeito", "briga com irmão" + livre).
- **Trava anti-injustiça:** a multa nunca deixa o saldo negativo. Diferente da v1 do conceito, **não existe mais um saldo "reservado" imune** — como o saldo agora é único, uma multa pode, sim, atrasar uma troca em andamento. É uma dor real (intencional: multa deve doer), mas sempre transparente — some do saldo com o motivo visível no extrato, nunca "some sem explicação".
- **Direito de resposta:** a criança pode "contestar" um cartão uma vez; o responsável confirma ou cancela.
- *(No protótipo atual, o fluxo de aplicar cartão ainda não tem um botão de acesso na interface — ver §10.)*

### 3.4 Conversão e troca por Pix

- Cotação configurada por criança, **sempre pelos pais** (a criança só visualiza o valor, nunca edita).
- **Troca por Pix:** a criança pode pedir a troca do saldo total por dinheiro a qualquer momento; o pai aprova como qualquer outra troca e paga fora do app (Pix/dinheiro), marcando "Pago ✓" no Dia da Mesada.
- **Dia da Mesada:** data configurável em que o app sugere a próxima troca por Pix e mantém o histórico de pagamentos.
- **Juros do saldo (opcional):** Estrelas que ficam paradas rendem um % ao mês, pago pelos pais — a introdução mais concreta possível a juros compostos.

### 3.5 Catálogo de recompensas e a meta em destaque

A v1 deste documento descrevia um sistema de **poupança com reserva e matching dos pais** ("a cada 2 ⭐ guardadas, os pais colocam 1"). Esse mecanismo foi **substituído por um sistema de troca direta**, mais simples de entender e sem uma segunda "conta" para gerenciar:

- Todo item — da meta grande (um skate) ao privilégio pequeno (30 min de tela extra) — vive no **mesmo catálogo**, cada um com um preço em Estrelas.
- Um item pode ser marcado como **meta em destaque**: ganha um card especial com barra de progresso, exibido tanto na tela de Tarefas quanto no topo de Recompensas — mantendo o desejo sempre visível, mesmo sem existir mais uma "conta separada" para ele.
- **Não existe mais reserva.** A criança troca assim que o saldo total cobre o preço — o botão do item mostra "Trocar" quando dá, ou "Faltam N ⭐" (desabilitado) quando ainda não dá.
- **Quem cadastra o quê:**
  - **A criança sugere** um item que quer ("Ir ao cinema no fim de semana") — sem definir preço.
  - **O pai precifica e aprova** a sugestão (ou recusa), e o item entra no catálogo — ou **cadastra um item direto**, sem esperar sugestão.
  - Essa decisão (criança sugere / pai decide o valor) mantém o pai no controle da economia, mas dá à criança voz sobre o que ela quer — o mesmo espírito que a v1 já usava para a meta ("a criança pede, o pai precifica"), agora generalizado para qualquer recompensa.
- Meta atingida (ou qualquer troca de item de destaque) → item vai para a "Estante de Conquistas" da criança.

---

## 4. App da Criança / Adolescente

### 4.1 Princípios

- **Zero fricção para agir:** abrir o app → ver o que fazer hoje → entregar, em 1 toque por tarefa.
- **Autonomia visível:** a criança nunca depende do pai para *saber* algo (quanto tem, quanto falta, por quê perdeu).
- **Dois modos de idade:** *Modo Explorador* (6–9: ícones grandes, áudio nas instruções) e *Modo Pro* (10+: visual de "banco/game", sem infantilização).

### 4.2 Navegação (3 abas)

```
┌──────────────────────────────────┐
│   [Tarefas]  [Recompensas]  [Conquistas]   │
└──────────────────────────────────┘
```
Sem botão de ação central — cada tela resolve sua própria ação principal no lugar onde ela acontece (ver abaixo). Um ícone de notificações (🔔) fica sempre visível no topo, ao lado do saldo.

#### Tela 1 — **Tarefas** (home)
A pergunta que responde: *"o que eu faço agora e como estou indo?"*

- **Cabeçalho:** avatar, nível, saldo de Estrelas e o sino de notificações.
- **Card da meta em destaque:** sempre no topo, com barra de progresso e (quando o saldo já cobre o preço) um botão "🎉 Trocar agora".
- **Lista de tarefas do dia**, com a chama 🔥 da sequência ao lado do título da seção — cada tarefa mostra seu período e, se estiver pendente, um **botão com o próprio valor** (ex.: "+5 ⭐") que já entrega ao tocar. Missões entregues mostram "⏳ avaliando"; aprovadas mostram a nota (⭐⭐⭐); devolvidas mostram "↩️ refazer".

#### Tela 2 — **Recompensas** (o "banco" da criança)
- **Saldo único** (sem divisão livre/reservado) + equivalente em R$ na cotação de hoje.
- O **mesmo card da meta em destaque** da tela Tarefas, repetido aqui no topo.
- Botão **"💡 Sugerir"** para pedir um item novo — a sugestão aparece logo abaixo com o status "Aguardando o papai decidir o preço".
- **Catálogo de trocas:** cada item com preço e botão "Trocar" (ou "Faltam N ⭐" desabilitado).
- **Virar dinheiro (Pix):** card com o saldo atual convertido em R$ e botão para pedir a troca — pago no Dia da Mesada.
- **Extrato:** cada entrada e saída com motivo.

#### Tela 3 — **Conquistas** (identidade e progresso)
- Avatar customizável — itens cosméticos comprados com XP/nível, nunca com Estrelas.
- Patentes, álbum de medalhas (1ª missão, sequência de dias, 1ª meta trocada, marcos de trocas).
- **Estante de Conquistas:** itens já trocados, com data — prova visual de que trocar funciona.

### 4.3 O ciclo diário da criança (por que ela abre o app)

| Momento | Gatilho | Ação no app | Recompensa emocional |
|---|---|---|---|
| Manhã | Notificação "Suas tarefas de hoje chegaram 🌅" | Ver a lista do dia | Clareza + chama da sequência em jogo |
| Durante o dia | Lembrete de horário de uma tarefa | Entregar tocando no valor (+N ⭐) | Feedback imediato, expectativa da avaliação |
| Fim da tarde | Sino 🔔 acende: "Papai aprovou! ⭐⭐⭐" | Abrir para ver a nota e o saldo subir | Reconhecimento — a nota de qualidade é elogio dos pais em forma de jogo |
| Noite | "Falta 1 tarefa para manter sua chama 🔥" | Completar a última | Sequência mantida, barra da meta subindo |

Os ganchos de retenção mais fortes: **a meta em destaque sempre visível** (desejo concreto), **a sequência** (aversão à perda), e **a avaliação dos pais** (reconhecimento afetivo disfarçado de mecânica).

---

## 5. App dos Pais (Painel de Gestão)

### 5.1 Princípios

- **O pai é um gestor com 90 segundos por dia.** Tudo que é rotineiro tem que ser resolvido da tela inicial, sem navegar.
- **Criar deve ser mais fácil que cobrar:** publicar uma missão precisa ser mais rápido que gritar "arruma teu quarto!" da cozinha.
- **Transparência bidirecional:** toda ação do pai vira registro que a criança vê — o app é o "contrato" da família.

### 5.2 Navegação (4 abas + FAB discreto)

```
┌─────────────────────────────────────────────────┐
│  [Painel]   [Missões]   [Recompensas]   [Configurações]  │
└─────────────────────────────────────────────────┘
                                                (＋) ← FAB flutuante,
                                            canto inferior direito
```
Diferente da v1 (que usava um botão de ação central grande), a criação de missão agora é um **FAB pequeno**, sem competir visualmente com a navegação. Um sino 🔔 com contador no cabeçalho acende quando há aprovações, trocas ou sugestões pendentes.

#### Tela 1 — **Painel** (caixa de entrada de decisões)
Enxuto por design — só o que exige decisão do pai agora:

1. **Fila de Aprovações** — cards compactos (ícone, nome da tarefa, valor) com dois botões: **↩️** (devolver) e **✓ Aprovar**. Tocar em qualquer um abre um **modal**:
   - Aprovar → escolher a nota de qualidade (⭐ / ⭐⭐ / ⭐⭐⭐), cada uma já mostrando o valor calculado.
   - Devolver → escolher um motivo rápido ("Faltou capricho", "Não terminou", "Foto não confere") — a tarefa volta com novo prazo, sem redução na 2ª entrega.
2. **Trocas pendentes** — aprovar itens do catálogo ou pedidos de troca por Pix.

*(As antigas seções "Pulso do dia" e "Ações rápidas" — Missão Relâmpago, Cartão, Bônus — foram retiradas do Painel para deixá-lo mais direto. Ver §10 sobre onde essas ações deveriam reaparecer.)*

#### Ação — **Cadastrar missão** (FAB)
Diferente da v1 (biblioteca de modelos em poucos toques), o cadastro agora é um **formulário completo em tela cheia**, no padrão de apps nativos de tarefas:
- Ícone (tocável, cicla entre opções) + nome + atalho **"Usar predefinição"** (aplica um modelo pronto, editável depois) + anotações.
- **Estrelas por conclusão** via stepper (−/+).
- **Data de início** (Hoje/Amanhã) e **Repetir** (Todos os dias / Dias úteis / Fins de semana / Uma vez).
- **Período do dia** e alternância de **horário específico**, com lembrete para a criança.
- **Comprovante fotográfico** (toggle).
- Botão "Adicionar" fica desabilitado até o nome ser preenchido.

Esse mesmo padrão visual (ícone + campos + stepper) é reaproveitado no cadastro/edição de itens do catálogo de recompensas.

#### Tela 2 — **Missões**
- **Missões cadastradas:** lista de todas as missões da família (não mais uma grade semanal), alimentada tanto pelas missões de exemplo quanto por tudo que é criado pelo FAB.
- *(A v1 tinha uma mecânica de "aposentar" missão que virou hábito — foi removida; ver §10.)*

#### Tela 3 — **Recompensas** (era "Financeiro")
- **Saldo do filho** + equivalente em R$.
- **Sugestões da criança:** cada uma com **✕ Recusar** ou **💲 Precificar** (abre a mesma tela cheia de cadastro, com o nome já preenchido — o pai só define ícone e preço).
- **Catálogo de recompensas:** lista com botão **"+ Nova"** para cadastro direto, e **✏️** em cada item para editar ou remover.
- **Regras da economia:** cotação da estrela, Dia da Mesada, teto de gasto mensal, juros do saldo.
- **Meta em destaque:** progresso + **✏️** para reprecificar (é só mais um item do catálogo, com card especial).

#### Tela 4 — **Configurações** (era "Família")
Reestruturada por completo — a v1 misturava filhos e responsáveis numa lista única e incluía um "Boletim mensal" com insights. Isso foi removido; a estrutura atual:

- **Filhos:** um card por filho (avatar, saldo) com **"Vincular"** (gera um código/QR para parear o dispositivo) e **"Excluir"** (remove com confirmação). **"+ Adicionar filho"** abre um cadastro leve: avatar, nome, idade — o app deriva o Modo Explorador/Pro automaticamente pela idade.
- **Conta:** o perfil do próprio responsável, **só exibido** (sem tela de edição — decisão explícita de não construir isso agora, ver §10), mais **Assinatura** (Grátis) e **"Mudar para Premium"**.
- **Controle de acesso:** a lista de co-gestores (ex.: outro responsável, avó) com o papel de cada um — **Admin** (acesso total) ou **Colaborador** (aprova missões, não mexe em dinheiro) — com opção de alternar o papel ou remover o acesso. **"+ Convidar responsável"** reaproveita o mesmo fluxo de código/QR do "Vincular".
- **Informações:** Termos e condições, Política de privacidade, Fale conosco.
- **Sair** / **Excluir conta** (ambos com confirmação) e número da versão.

### 5.3 O ciclo diário dos pais (< 2 minutos)

- **Manhã (passivo):** missões recorrentes se publicam sozinhas. O pai não faz nada.
- **Ao longo do dia (2 toques por evento):** entrega chega → tocar "Aprovar" → escolher a nota no modal → pronto.
- **Noite (60 s):** abrir o Painel, resolver o que restou na fila de aprovações e trocas pendentes.

---

## 6. Regras de interação entre as duas pontas

| Situação | Regra | Racional |
|---|---|---|
| Entrega sem avaliação | Aprovação automática com ⭐⭐ após 24 h (configurável) | O esquecimento do pai nunca pode punir a criança — mata a confiança no sistema |
| Missão não feita no prazo | Expira sem multa automática; apenas quebra a sequência | Punir por omissão automaticamente gera injustiça; o pai decide se cabe cartão |
| Devolução ("refazer") | Sempre com motivo + novo prazo curto; 2ª entrega não sofre redução | Devolver ensina padrão de qualidade; reduzir valor na 2ª tentativa ensinaria a desistir |
| Multa (cartão vermelho) | Motivo obrigatório, direito a 1 contestação; desconta do saldo total (não existe mais saldo "protegido") | Disciplina sem processo vira arbitrariedade; a transparência no extrato evita a sensação de "confisco silencioso" |
| Sugestão de recompensa | A criança propõe, mas nunca define preço; o pai precifica, aprova ou recusa | Mantém o pai no controle da economia sem tirar a voz da criança sobre o que ela quer |
| Troca no catálogo | Pedido → aprovação → "voucher" na tela da criança para mostrar na hora de usar | O voucher digital ("30 min de videogame ✓ liberado por Papai às 19h") elimina o "mas você deixou!" |
| Pais separados / dois lares | Dois admins com controle de acesso próprio | Realidade de grande parte do mercado — ainda documentado como visão, não modelado como um toggle específico no protótipo (ver §10) |

### Anti-gaming (dos dois lados)

- **Criança:** foto obrigatória configurável por missão; Estrelas só entram após aprovação.
- **Pais:** o ledger é imutável e visível — toda remoção de Estrela aparece com motivo. A confiança da criança no sistema **é** o produto.

---

## 7. Diferenciais de produto

1. **Avaliação de qualidade com multiplicador (⭐→⭐⭐⭐)** — concorrentes tratam tarefa como checkbox. Avaliar qualidade transforma aprovação em feedback pedagógico.
2. **Duas moedas (Estrela gastável vs. XP permanente)** — permite punir financeiramente sem destruir a motivação de longo prazo.
3. **Criança sugere, pai precifica** — a economia inteira (não só a meta) é co-criada: a criança tem voz sobre o que quer trocar, mas o pai mantém o controle do valor. Menos fricção que o antigo modelo de poupança com matching, e mais fácil de entender.
4. **Cadastro no padrão de apps nativos de tarefas** — formulário detalhado (ícone, stepper de estrelas, recorrência, horário com lembrete, comprovante fotográfico) em vez de um sheet simplista — reduz a sensação de "protótipo" e aumenta a confiança do pai no produto.
5. **Aprovação em dois toques com nota de qualidade em modal** — o pai nunca precisa escrever nada para aprovar; a fila fica compacta, e o julgamento de qualidade vira uma decisão rápida e visual.
6. **Voucher de troca** — o item aprovado vira um "ticket" visível (quem liberou, quando). Resolve a fonte nº 1 de conflito pós-troca.
7. **Vínculo por código/QR + Controle de acesso por papéis** — parear o dispositivo do filho ou convidar um co-gestor é o mesmo fluxo simples, e cada responsável entra com um papel claro (Admin/Colaborador).
8. **Cartões com direito de resposta** *(mecânica documentada, ainda sem ponto de entrada na UI — ver §10)* — nenhum concorrente modela o *processo justo* da disciplina.

---

## 8. Onboarding (os primeiros 10 minutos decidem tudo)

1. **Pai baixa e cria a família** → assistente pergunta idades → sugere um "pacote inicial" de missões apropriadas por filho (evita a tela em branco).
2. Define a cotação com ajuda ("famílias com filhos de 8 anos costumam praticar ~R$ X/semana").
3. **Convida a criança pelo Vínculo** (código/QR, em Configurações) — o app da criança nasce já com tarefas do dia.
4. Primeira semana em "modo aquecimento": aprovação incentivada com ⭐⭐⭐ — o objetivo é a criança chegar à primeira troca em 7 dias (o momento "aha!" das duas pontas: *isso funciona*).

---

## 9. Resumo dos fluxos (mapa de navegação)

```
CRIANÇA                                    PAIS
━━━━━━━                                    ━━━━
Tarefas ─────────────┐                     Painel ──────────────────┐
 ├─ card da meta      │                     ├─ fila de aprovações (compacta → modal de nota)
 ├─ chama da sequência│                     └─ trocas pendentes
 └─ tarefa → +N ⭐     │                    Missões
Recompensas           │                     └─ missões cadastradas (lista única)
 ├─ saldo único       │                    (＋) FAB ← cadastro de missão (tela cheia)
 ├─ card da meta       │                    Recompensas
 ├─ 💡 sugerir         │                     ├─ sugestões da criança (precificar/recusar)
 ├─ catálogo de trocas │                     ├─ catálogo (+ nova / editar)
 ├─ virar Pix          │                     ├─ regras da economia
 └─ extrato            │                     └─ meta em destaque (editar preço)
Conquistas                                  Configurações
 ├─ avatar (XP), patentes, medalhas          ├─ filhos (vincular / excluir / adicionar)
 └─ estante de conquistas                    ├─ conta (perfil, assinatura)
                                              ├─ controle de acesso (papéis)
                                              ├─ informações
                                              └─ sair / excluir conta
```

---

## 10. Pendências e ideias estacionadas

Registro do que mudou de rumo durante a prototipagem, para não perder o histórico de decisões:

- **Poupança com reserva + matching dos pais** (v1 §3.5) — substituída pelo modelo de troca direta (§3.5 atual). Simplifica o mental model, mas perde o gancho pedagógico específico de "contrapartida de investimento"; pode valer revisitar se o feedback de usuários pedir mais incentivo a guardar em vez de trocar rápido.
- **Mural de Bicos entre irmãos** (missões avulsas disputáveis) — removido a pedido explícito. Era um gatilho de abertura do app interessante; considerar reintroduzir em uma fase futura se houver mais de um filho ativo.
- **Aposentadoria de missão / graduação de hábito** — removido a pedido explícito. Era o diferencial filosófico mais forte da v1 ("a recompensa deveria deixar de ser necessária"); vale reavaliar como feature de fase 2, sem reintroduzir a complexidade de UI que tinha.
- **Boletim mensal com insights** — cortado do escopo atual. Ideia ainda válida para uma camada de analytics/insights, só não faz parte do MVP prototipado.
- **Ações rápidas (⚡ Relâmpago, 🟨🟥 Cartão, 🎁 Bônus)** — o código e as telas existem, mas perderam o ponto de entrada quando "Pulso do dia" e "Ações rápidas" saíram do Painel. Decisão em aberto: para onde realocar (aba Missões? um menu dentro do FAB? uma nova ação rápida por tarefa?).
- **Perfil do responsável (edição)** — decisão explícita de não construir uma tela de edição de perfil por enquanto; o Controle de acesso resolve a necessidade real (quem pode fazer o quê), que era o problema por trás do pedido original.
- **Modo dois lares (carteiras separadas por responsável)** — segue como visão de produto documentada, mas ainda não modelado como um toggle real em Controle de acesso.

---

*Documento vivo — sincronizado com `prototipo.html`. Próxima decisão em aberto: destino das Ações Rápidas (item acima) e priorização de MVP para desenvolvimento real.*
